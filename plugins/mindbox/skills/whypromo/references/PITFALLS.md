# What the response does not show

Rules the tool response does not state, and without which the explanation is wrong. Each one, if missed, costs a wrong
answer to a person who will then go and fix the wrong thing. Field shapes: [MCP_CONTRACT.md](MCP_CONTRACT.md),
the verdict per promotion: [VERDICTS.md](VERDICTS.md), records: [RECORDS.md](RECORDS.md).

## A missing record proves nothing

Decision recording is built so that **a collection failure is swallowed silently**: the calculation does not
fail because of it, the record simply never appears. No record means either the event did not happen or
collection failed at that step, and the response cannot tell the two apart. Say "the calculation has no record
of X", not "X did not happen".

## The verdict comes ready-made

`verdict` and `isTerminal` were computed by the promotion diagnostics service. Do not derive the verdict
yourself from the list of records and do not invent outcome priorities: the service has its own rule order
([VERDICTS.md](VERDICTS.md#terminal-record)), and the terminal record is not necessarily the last one by `seq`.

A zero effect is not a rejection: a promotion whose benefits ran but produced nothing comes as `NoEffectVerdict`,
not as "did not qualify". Partial application is neither a rejection nor "applied in full": `AppliedPartiallyVerdict`
means the promotion gave the order less than it computed, and `reason` says what cut it.

## Four different "the promotion is not here"

| What | Where | Meaning |
|---|---|---|
| `verdict.kind: NotCandidateVerdict` | verdict | there is not a single record about the promotion: it did not take part in the calculation. For an external discount this means only "the trace says nothing about it": external discounts are not registered as candidates |
| `verdict.kind: UndeterminedVerdict` | verdict | there are records, but no verdict follows from them: the calculation did not close the promotion |
| `details: NOT_A_PROMOTION` | details | the value passed is not a promotion number: an unmatched external discount or a typo |
| `details: NOT_FOUND` | details | the number was not found in the project, or the promotion has never been launched |

Do not merge them: each has its own addressee ([ESCALATION.md](ESCALATION.md)). `details: LOOKUP_FAILED` does not
cancel the verdict: the promotion service did not respond, but the promotion's fate in the calculation is known.
Why the promotion did not become a candidate is **not visible** in the response: absence from the promotion tree
has its own record (`NotInArbitrationTree`) and its own verdict. With `NotCandidateVerdict`, do not name a reason.

## `promotion`: today, not at calculation time

`state`, `startDateTime`, `endDateTime` and `currentHistoryId` describe the promotion **today**: a pause now does
not explain a zero in a past calculation. Dates are local, without an offset, according to `timeZoneMode`: the promotion
has its own mode and the version has its own; the project's time zone is not in the response, so the dates cannot be compared
with `calculatedAtUtc`.

`MindboxPromotion` has two version numbers: `history.historyId` is what the order was calculated on,
`currentHistoryId` is what the promotion carries now. If they differ, the promotion was re-saved after the
calculation, and checking `history` against the admin panel is pointless: the person is looking at a different
version. If they are equal, there should be no discrepancy, and "check the settings" makes sense.

`ExternalPromotion` is an external promotion for a partner discount. It has no `history` **by definition**: the
benefits arrive in the operation request; it never has versions. A missing `history` here is no reason to escalate.

## `TOO_FRESH` and the old format are not "no data"

Reacting to `TRACE_NOT_STORED` without reading `calculationAge` is the most expensive mistake: most often it is `TOO_FRESH`,
and the person must be told "try again in a minute", not "no data"
([MCP_CONTRACT.md § `TRACE_NOT_STORED`](MCP_CONTRACT.md#trace_not_stored--four-answers)).

`TRACE_UNAVAILABLE` is not always a failure either. With the text "…no longer read (<code>)…" it is a trace in
the old format: it cannot be explained, and neither retrying in a minute nor Mindbox support will help
([MCP_CONTRACT.md § `TRACE_UNAVAILABLE`](MCP_CONTRACT.md#trace_unavailable--read-reason)).

## Amounts are in currency units, not minor units

Money fields come in the project's currency (rubles for Russian projects), in the units of the corresponding
balance. Dividing by 100 flips the conclusion: "47 rubles short" turns into "47 kopecks short".

## How much the benefit gave and how much it wanted

- **The result**: the amounts in the benefit row, in the promotion's `amounts` and on the `BenefitApplied` record:
  what the customer actually received after arbitration and caps.
- **The expectation**: `requested*` in `amounts` (what the promotion would have given without caps),
  `requestedAmount` on `BenefitAppliedPartially` and on the `BenefitApplied` record (what the benefit asked for
  before limits) and `LineDiscountComputed` (what the formula wanted to give the line). Calling the expectation
  the result overstates the amount.
- **What cut it**: for a partial benefit, `reason` and `caps[]`; per product, the `lineVerdicts` groups
  `AppliedCappedBenefitLineVerdict` (took less) and `NotAppliedBenefitLineVerdict` (got nothing), each with its
  own `reason`; in records, `SpreadShortfall` (the amount before spreading the discount across lines),
  `LineDiscountCapped` (the line took less), `LineDiscountZero` (the line got zero).
- **An empty `caps` proves nothing**: on `BenefitApplied` it is always empty, and per-line caps are never
  included in `caps` at all.
- `amounts.discount` does not include money covered by redeemed bonus points: redemption goes into `amounts`
  only as points, although the benefit row has a `discount` for it too.

## No amount ≠ zero

A benefit row lacks an amount field when the benefit by its nature does not give that kind of amount: a discount has no points,
an accrual has no discount, a message and a marker have nothing. Writing "0 points accrued" about a discount
is wrong.

`amounts` is absent entirely only when the trace has no `BenefitApplied` record at all. If there are records but
no amounts in them, `amounts` comes without fields, and that is not "the promotion gave zero".

## Zero and `false` often mean "the field was absent"

Numeric fields of a record always arrive: the calculation does not write them when there is no value, and a
zero is substituted when the record is read from storage. Therefore:

- `lineKey` is numbered **from one**, and `0` means "not set". There is no "line 0";
- `blockOrdinal` is numbered **from zero**, and `0` is either the first product block or a field that did not
  arrive. Discarding the zero as "not set" is as wrong as confidently naming the first block: check whether the
  record is about a block at all and whether the version has product blocks.

`treeUsedFallback` and `limitsDataUnavailable` in `inputs` always come as `true` or `false`. `false` means either
"no" or "the calculation recorded nothing about it"; the conclusion "limits are fine" does not follow from it.

## Order lines come in two ways

For most records the line is the `lineKey` field. But `ProductNotInBlock`, `LinesExcludedByMatrix`,
`CancelledLinesExcluded` and `BenefitApplied` carry lines **as a `lineKeys` list in `payload`**, and their
`lineKey` is not set: look for the line in both places, and do not conclude "no lines" from an empty `lineKey`.
What the benefit did with each line is quickest to see in its `lineVerdicts`
([VERDICTS.md](VERDICTS.md#benefit-products-lineverdicts)): there, every line already sits in exactly one group.

## Cancelled and returned line items

Cancelled line items do not take part in the calculation at all (`CancelledLinesExcluded` in the shared records),
while returned ones do. When all line items, or all suitable line items, are cancelled or returned, the reason is
named by the record kind (`AllLinesCancelled`, `SuitableLinesCancelled` and their neighbours). This is not a
promotion setting; it is how the POS passed the line item statuses.

## Unavailable counters are not an exhausted limit

`limitsDataUnavailable`, `limits[].dataUnavailable` and `LimitDataUnavailable` mean the counters **could not be
read**: the values are unreliable, and the promotion may have applied beyond the limit. Exhaustion is signalled by `LimitExhausted`,
by the `…LimitSpent` / `…LimitExhausted` kinds on `LineDiscountZero`, and by limit caps on `SpreadShortfall` and
`LineDiscountCapped`.

A row of `inputs.limits[]` carries no promotion number, only a version (`promotionHistoryId`). Limits are loaded
for versions from the promotion tree **and** for versions from a previous save of this order:

- the row's version equals `promotion.history.historyId`: this is the limit of the promotion in question; the
  amounts can be named;
- the version differs: it is the limit of another promotion or of an earlier version of this one; do not quote
  the numbers;
- exhaustion is proven by the promotion's own record anyway, so "the promotion limit is spent" can be said
  even without the numbers from this row.

`2147483647` in a residue of `LimitExhausted` means there is no limit on this dimension. The discount
residue and the applications residue are different fields with different units: money and applications.

## The operation's request and another promotion's reserve are not a loss

The `RequestedAmount` kind on `SpreadShortfall`, `LineDiscountCapped` and `BenefitZeroEffect` means the cap was
**the operation's request** (the POS's spend request): the POS asked to redeem less than the promotion could give. The difference from what was
possible is not a loss for the customer: it was never asked for, and "cut" is the wrong word here. The `BalanceReserved`
kind means the balance was already taken by another promotion in the same calculation: there is nothing to fix;
it is one balance being distributed among promotions.

## Group records: many promotions, one record

Some rejections of the same type are folded into one record: `promotionIds[]` then holds several promotions, and
the record applies to each of them individually, not to the group as a whole. Which records get folded is not visible
in the response, so a single promotion in the array proves nothing. `LostMaxBenefit` is not folded; on
`MaxBenefitWinnerChosen`, `promotionIds[]` holds all the losers of one comparison. Details of the restrictions
(pool, segments, brands) are not in group records; they are in the version settings.

## Records without a promotion are not about the promotion in question

Records in `sharedDecisions[]` name no promotion at all ([RECORDS.md § Shared records](RECORDS.md#shared-records)):
a coupon rejection, cancelled line items, the spend request, per-line records of external promotions. Do not
attribute them to the promotion in question. The group discount cap and the group minimum price now arrive as
kinds on the promotion's own records: there is no need to cross-check them; they name the promotion.

## The first is not the only one

- An exclusion record names the **first** excluder that fired: removing it does not necessarily revive the
  promotion. Different excluders on different lines produce different records.
- `CustomerNotAuthorized` names the **first** inapplicable benefit of the promotion.
- `PromoCodeNotSuitable` names the fate of the **first** rejected coupon of the pool.

## Lists in the response are truncated

Shared records (up to 50), order lines (up to 100) and customer segments (up to 200) arrive truncated, and the
explanation itself reads only the first 5000 records of the trace. "Not in the list" does not mean "was not in the
calculation". The tool does not return the full list of candidates at all: the promotions to explain are named in advance.

## The product number is an internal Mindbox id

`productInternalId` is the product's internal number in Mindbox, not the SKU and not the external id. Write
"product no. 452 in Mindbox", otherwise the person will go looking for that number in their own catalog and will
not find it. The product name and external id come in `lineVerdicts[].lines[]` (`productName`, `externalId`):
when they are present, name the product by them.

## The catalog gives wording, not facts

Where to take the text from: [MCP_CONTRACT.md § The wording of the reason](MCP_CONTRACT.md#the-wording-of-the-reason--the-catalog);
how to translate it for the person: [HUMAN_STYLE.md § Reason wording](HUMAN_STYLE.md#wording-of-the-reason).

- **The text carries almost no facts.** It contains the entity name as a link (the segment, the pool, the promotion that
  excluded it) and the group cap as a number (`${ GroupCapAmount }`), and nothing more: promotion names, amounts and
  numbers come from `params` and the verdict fields. `[no data]` in place of a link means the record has no
  source for the entity.
- **Do not read more into the text than it says.** "The promotion's discount amount limit cut the amount" is about
  the promotion's own limit, not about another promotion having already lowered the price; "no suitable products"
  is about this benefit's condition, not about an empty order. Do not name neighbouring records in the chain as
  the reason.
- **Do not look for `explanation` in the decision chain.** `decisions[]` rows do not have it by design; the
  wording for such a record is in `loyalty_reason_codes`. If there is no text there either, the reason is still
  established by the record's fields.
- **The set of record fields is open-ended.** The response returns `payload` as is, and the reason's
  `params` is filled the same way: a new field added by the producer appears in the response on its own. Hence the ban on
  outputting the promo code and internal numbers ([HUMAN_STYLE.md § What not to output](HUMAN_STYLE.md#what-not-to-output)):
  it does not depend on which fields arrive today.
