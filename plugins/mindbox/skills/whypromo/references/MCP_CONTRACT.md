# Tool contract

The Mindbox MCP server exposes three tools, all read-only: `loyalty_find_order_calculations`,
`loyalty_explain_promotions`, `loyalty_reason_codes`. This file covers inputs, statuses and response fields
as they appear in the tool response. How to read a promotion's verdict — [VERDICTS.md](VERDICTS.md); which
records exist — [RECORDS.md](RECORDS.md); the server and its refusals — [GATEWAY.md](GATEWAY.md).

If a live response diverges from this description, trust the response and state the divergence
explicitly: the reference is out of date.

## Boundary

If the answer is not in these three tools, name what is unknown and stop. Promotion settings beyond the
version used by the calculation (`promotion.history`), the promotion tree, filters and the change history
are not in the response. If the connected server offers `tenants_list`, it is needed only to confirm the
project's system name ([GATEWAY.md § tenants_list](GATEWAY.md#tenants_list)).

## `loyalty_find_order_calculations` — the order's calculations

Input: `tenant` and **exactly one** of the two order ids — `orderExternalId` (string, surrounding whitespace
is trimmed) or `orderInternalId` (**integer**, not a quoted string: a value of the wrong type is rejected by
schema validation before it reaches the service).

| Status | Meaning | Fields |
|---|---|---|
| `FOUND` | orders were found | `orders[]` |
| `ORDER_NOT_FOUND` | no order with this id in the project | — |
| `TENANT_NOT_FOUND` | the project is not available on the connected server; rarely reaches you, since the server rejects an unknown name earlier | — |
| `BAD_REQUEST` | both ids at once, neither, or an empty external id | `reason` |
| `UNAVAILABLE` | either the project has no admin panel configured, so the service cannot work with it, or the order service did not respond | `reason` says which of the two |

`orders[]` — one element per match **across all external systems of the project**:

| Field | Meaning |
|---|---|
| `orderInternalId` | the order's internal number; shown in the admin panel as "Mindbox Order ID" on the save action |
| `historyItems[]` | all saved states of the order: `savedAtUtc` (UTC), `isCurrent`, `calculationId` |
| `matchedByExternalIdSystemNames[]` | the kinds of external id the value matched; empty when searching by internal id and for projects whose order storage predates the field |

A history item without `calculationId` is a save for which loyalty ran no calculation (an import, or a save
without promotion calculation). That is a fact about the order, not a data gap; why it was not calculated
is not visible from the response.

## `loyalty_explain_promotions` — what happened to the promotions

Input: `tenant`, `calculationId` (a string holding a UUID) and `promotionIds[]` — an array of **strings**,
at most 25: a promotion number (`"40434911"`) or the raw id of an external discount that the calculation
did not match to a promotion (`"someshop-528"`). An unquoted number is rejected by schema validation. An
empty array, or an empty string in it, yields `BAD_REQUEST`; duplicates are collapsed. The tool never returns
the full list of candidates: the promotions must be named up front.

| Status | Meaning | Fields |
|---|---|---|
| `EXPLAINED` | the explanation exists | `explanation` |
| `BAD_REQUEST` | not a UUID, empty list, more than 25, an empty string in the list | `reason` |
| `TENANT_NOT_FOUND` | the project is not available on the connected server | — |
| `TENANT_UNAVAILABLE` | the project exists, but the service cannot work with it (no admin panel configured) | `reason` |
| `TRACE_NOT_STORED` | no calculation record | `calculationAge` |
| `TRACE_UNAVAILABLE` | the trace could not be read | `reason` |

## `TRACE_NOT_STORED` — four answers

`calculationAge` is a string. The retention window is 60 days, the freshness threshold is one minute.

| `calculationAge` | What happened | What to do |
|---|---|---|
| `TOO_FRESH` | the calculation is less than a minute old; the record is still on its way | retry in a minute |
| `WITHIN_RETENTION` | the record should exist, but it does not | check that this is the calculation the user means; if it is, the record was not saved — say so, do not guess the reason |
| `EXPIRED` | older than 60 days | the data is gone and will not come back |
| `NO_TIMESTAMP` | the id carries no creation time (not a UUID v7) | nothing can be said about the age |

## `TRACE_UNAVAILABLE` — read `reason`

One status covers several cases, told apart by the text of `reason`:

| `reason` | What it is | What to do |
|---|---|---|
| `The calculation is recorded in a form the diagnostics no longer read (<code>); asking again will not help.` | the trace is in an old format: it contains `BalanceInsufficient`, `LineDiscountCappedByLineAvailability`, `SpreadAmountCapped` or `ZeroLineDiscount` (calculations were written this way before 2026-09-18) | this calculation cannot be explained and retrying will not help; take a newer calculation |
| `Clickhouse timed out.`, `Clickhouse is unreachable.`, `Clickhouse rejected the query.` | the trace storage did not respond | retry in a few minutes; if that does not help, contact Mindbox support |
| `Stored trace does not match the schema.`, `Stored trace is not valid JSON.`, `Stored trace has a duplicate candidate promotion.` | the trace is corrupted | retrying will not help; contact Mindbox support |

## `explanation` — the response header

| Field | Meaning |
|---|---|
| `calculationId`, `calculatedAtUtc`, `operation` | the calculation header |
| `candidateCount` | how many distinct promotions the calculation considered |
| `decisionCount` | how many records the trace holds. The response contains only the records of the requested promotions plus up to 50 shared ones, so this number is almost always larger than the number of visible records. The explanation reads only the first 5000 records; the tail of a very large calculation is not read |
| `inputs` | the calculation input, one per response |
| `promotions[]` | one element per requested promotion |
| `sharedDecisions[]` | records that name no promotion: up to 50, oldest first ([RECORDS.md § Shared records](RECORDS.md#shared-records)) |

## `promotions[]`

| Field | Meaning |
|---|---|
| `promotionId` | as requested |
| `verdict` | the promotion's verdict, **computed by the server** — [VERDICTS.md](VERDICTS.md) |
| `details` | how the promotion lookup ended; see below |
| `promotion` | the promotion itself; only when `details = AVAILABLE` |
| `lookupFailureReason` | why the promotion could not be found; only when `details = LOOKUP_FAILED` |
| `decisions[]` | all trace records naming the promotion, oldest first |

## `details` and `promotion`

| `details` | Meaning |
|---|---|
| `AVAILABLE` | the promotion was found and is in `promotion` |
| `NOT_FOUND` | the project has no promotion with this number, or the promotion has not been launched yet |
| `NOT_A_PROMOTION` | the id is not a promotion number (not a positive integer), so the promotion service was not queried: an unmatched external discount or a typo |
| `LOOKUP_FAILED` | the promotion service did not respond; **the verdict stands**, but benefit settings will be missing |

`promotion` comes in two kinds by `kind`:

- `MindboxPromotion` — a platform promotion: `id`, `name`, `state`, `createdDateTimeUtc`, `timeZoneMode`,
  `startDateTime`, `endDateTime`, `currentHistoryId` and `history`;
- `ExternalPromotion` — an external promotion created for a partner's discount: only `id`, `name`, `state`,
  `createdDateTimeUtc`. It has no versions; its benefits arrive in the operation request.

`state`, the dates and `currentHistoryId` describe the promotion **today**, not at calculation time. The
dates are local, without an offset, and are interpreted according to `timeZoneMode` (`store` — the store's
clock, `project` — the project's time zone); the project's time zone is not in the response, so do not
compare them with `calculatedAtUtc`.

`history` is the settings version **the calculation ran on**: `historyId`, `activeFromDateTimeUtc` (when it
was saved), its own dates and its own `timeZoneMode`, and also:

- `productBlocks[]` and `orderBlocks[]` in settings order, each with `benefits[]` (`benefitId` + `settings`
  as saved, without units or decoding; quote as is); a product block also has `productRestriction` (by
  segment in `restriction`, or as a list in `productInternalIds`), and both kinds of block have
  `orderRestriction` (order segmentation);
- `customerRestriction` — the customer segment;
- `regionRestriction` — `pointOfContactRestriction` and `areaRestriction`: as a list (`channelInternalIds`,
  `pointOfContactInternalIds`, `areaInternalIds`) or by segment;
- `accessibility` — `promoCodePoolRestriction` (`promoCodePoolInternalId`, `maxOrderApplicationCount`),
  `brandRestriction`, `dayOfWeekRestriction`, `dayTimeRestriction`.

Segment restrictions have `mode`: `ALL` — no restriction, `IN_SEGMENT` — only within the segment,
`NOT_IN_SEGMENT` — everyone except the segment. A restriction absent from the response means that the
response did not describe it, or that it is a kind the explanation does not read; it does **not** mean the
version has no restrictions. An empty object (`regionRestriction: {}`) means the version sets no restriction
of that kind.

`history` is absent when the calculation did not rely on a version of this promotion (the promotion was not
a candidate, or it is an external promotion) or the service did not find the version. If `history.historyId`
≠ `currentHistoryId`, the promotion was re-saved after the calculation: the admin panel no longer shows the
settings the order was calculated on.

## `inputs`

| Field | Meaning |
|---|---|
| `isCustomerAuthorized` | whether the customer was authorized |
| `treeUsedFallback` | a fallback promotion tree was used instead of the regular one |
| `limitsDataUnavailable` | the limit counters could not be read |
| `treeVersion` | the promotion tree version |
| `selection` | `pointOfContactId`, `areaId`, `brandId` |
| `customerSegmentIds[]` | the customer's segments, up to 200 |
| `orderLines[]` | order lines, up to 100: `lineKey`, `productInternalId`, `basePricePerItem`, `quantity`, `productSegmentIds[]` |
| `limits[]` | `limitId`, `promotionHistoryId`, `attribute` (`Discount`, `ApplicationNumber`…), `limitAmount` — configured, `usedAmount` — spent, `dataUnavailable` |
| `balances[]` | `systemName`, `available`, `total` |

Limits are listed per promotion **version** the calculation used: those from the promotion tree and those
from the previous save of this order. A row carries no promotion number; it belongs to a promotion only when
`promotionHistoryId` equals `promotion.history.historyId`. Coupons are not in the input; customer
identifiers are not in the response at all.

## `decisions[]` and `sharedDecisions[]`

Each trace record: `seq`, `code`, `stage`, `outcome`, `scope`, `isTerminal`, `promotionIds[]`,
`benefitId`, `benefitKind`, `blockOrdinal`, `lineKey`, `criterionName`, `failureKind`,
`excluderPromotionId`, `lineDiscountAmount`, `appliedDiscountAmount`, `payload`. An empty string means the
record left the field empty; a zero in a numeric field often means the same ([PITFALLS.md § Zero and false](PITFALLS.md#zero-and-false-often-mean-the-field-was-absent)).

| Field | Values |
|---|---|
| `code` | the record's **name** (`SpreadShortfall`); there is no numeric code in the response. Which exist — [RECORDS.md](RECORDS.md) |
| `stage` | `Selection`, `Blocks`, `LineFit`, `Benefit`, `Limit`, `Exclusion`, `Arbitration`, `Application` — the calculation runs in this order |
| `outcome` | `Applied` — applied; for a record about a line, the line was part of the benefit and received a share, possibly zero (`LineDiscountZero`). `Pass` — a per-line discount was computed. `Fail` — the promotion or benefit was discarded. `ZeroEffect` — ran with no effect. `ContextInfo` — a note, not an outcome |
| `scope` | `Calculation`, `Promotion`, `Block`, `Line`; a record about one line decides the line, not the promotion |
| `isTerminal` | the promotion's terminal record, chosen by the diagnostics server — [VERDICTS.md § Terminal record](VERDICTS.md#terminal-record) |
| `failureKind` | the kind of reason from the record's closed set |
| `criterionName` | always empty in fresh traces; a non-empty value marks a trace older than 2026-09-03 |
| `payload` | the record's remaining data as the calculation wrote it: numbers as numbers, arrays as arrays, names as strings; its composition depends on the record and is not constrained |

`promotionIds[]` is always an array: one element for an ordinary record, several for a collapsed one, in
which case the record applies to each listed promotion individually. A promotion's `decisions[]` holds every record where
it is in `promotionIds[]`; `sharedDecisions[]` holds records with no promotions at all.

## The wording of the reason — the catalog

The meaning of a record is set by `code` and `failureKind`; the human-readable wording lives in the
reason-code catalog of the promotion diagnostics service and reaches you by two routes:

| Where | What to do |
|---|---|
| the verdict's `reason`, the `reason` of `benefits[]` rows, the `reason` of `caps[]`, the `reason` of `lineVerdicts[]` groups | the text is already in `explanation` — take it from there |
| `decisions[]` and `sharedDecisions[]` records | they carry no text by design — read the `loyalty_reason_codes` catalog |

`loyalty_reason_codes` returns `locale` and `entries[]` of `key` + `text`: the templates as stored, with
`${ Macro }` placeholders unfilled. Call it **once per conversation**: the server has a single catalog,
`tenant` does not affect the response, and a second call returns the same thing. `entries[]` has no service
key for "no data".

**The key** is `code` and `failureKind` joined with `_`: `SelectionCriterionFailed_CustomerNotInSegment`.
Keys are matched from specific to general, and the first hit supplies the text: `code_failureKind`, then
`code`. The bare record name, without a kind, is the fallback: for records with a closed set of kinds it
covers the kinds that have no text yet. The record about lines taken by an excluder (`ExcludedByMatrix` with
lines) is looked up under the key `LinesExcludedByMatrix`. Three-part keys with `criterionName` exist only
for traces older than 2026-09-03: `code_criterionName_failureKind` → `code_criterionName` → `code_failureKind` → `code`.

**Language.** The catalog text comes in English (`locale: en-US`). Answer the user in their language,
staying close to the catalog text and adding no facts.

**Macros.** `${ CustomerSegment }`, `${ PointOfContactSegment }`, `${ AreaSegment }`, `${ ProductSegment }`,
`${ PromoCodePool }` and `${ ExcluderPromotion }` are rendered in `explanation` as a Markdown link
`[name](admin panel address)`; leave it as is. `${ GroupCapAmount }` becomes a number: the discount cap of
the promotion tree group, in the order's currency. When the entity has no value, `[no data]` appears in its
place: the record has no source for it, and this is not a read error.

If there is no text anywhere, neither in `explanation` nor under a key, the reason is nevertheless
**established** by the code and fields. Say plainly that it has no wording yet, name it by code and fields,
and escalate the gap ([ESCALATION.md](ESCALATION.md)). Say "the reason is unknown" only where the verdict
says so (`UndeterminedVerdict`). Records known to have no wording — [RECORDS.md § Records without wording](RECORDS.md#records-without-wording).
