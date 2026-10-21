# Promotion verdict: kind, benefits, amounts

The promotion diagnostics service derives `verdict` from the trace records itself. This file lists the
kinds, how the service picks the terminal record, and what the benefits table holds. Do not recompute the
verdict: the rules are here so that you read the fields correctly, not so that you repeat the derivation. Response fields —
[MCP_CONTRACT.md](MCP_CONTRACT.md), records — [RECORDS.md](RECORDS.md).

## Nine kinds

| `kind` | What happened to the promotion | Fields |
|---|---|---|
| `AppliedVerdict` | applied, and every benefit gave what it computed | `benefits[]`, `amounts` |
| `AppliedPartiallyVerdict` | applied, but one of the benefits gave less than computed, gave zero, or was knocked out by an incompatible promotion | `reason`, `benefits[]`, `amounts` |
| `NoEffectVerdict` | matched and the benefits ran, but the resulting benefit is zero | `reason`, `benefits[]` |
| `NotSuitableVerdict` | did not match: a selection condition failed, a limit was exhausted, the promotion is not in the arbitration tree; also any refusal without a verdict of its own, including record kinds newer than the service | `reason` |
| `BenefitConditionsNotMetVerdict` | matched, but a condition of its benefit or the product condition of its block was not met | `reason`, `benefits[]` |
| `ExcludedByPromotionVerdict` | excluded by an incompatible promotion, via a tree rule or an exclusion block; nothing from the promotion applied | `excluder`, `reason` |
| `ExcludedByMaxBenefitVerdict` | arbitration chose a variant without this promotion | `winningVariant[]`, `displacedBy[]`, `comparison` |
| `NotCandidateVerdict` | no record about the promotion at all: it was not a candidate | `checked` |
| `UndeterminedVerdict` | records exist, but none decides the outcome, or the deciding one is the service-level "unmapped candidate" | `outcomes[]` |

## Terminal record

A promotion with records has exactly one terminal record — the one with `isTerminal: true`. The promotion
diagnostics service picks it, not the calculation engine (since 15 Sep 2026 the engine's own flag is
neither written nor read):

1. Only a record with outcome `Applied` (and then only with scope `Promotion`), `ZeroEffect` or `Fail` can
   decide the outcome. `LineDiscountZero` is written as `Applied` for a line but is read as a zero for that
   line. Records about lines taken by the excluder do not decide the outcome.
2. A record about a single line decides only when there is no record wider than a line.
3. Of the remaining records, the strongest wins: by outcome (`Applied` > `ZeroEffect` > `Fail`), then by
   the later stage, then by the wider scope (`Promotion` > `Block` > `Line`), then by the larger `seq`.

So "last by `seq`" and "terminal" are not the same thing, and `seq` order does not follow stage order.
`BenefitApplied` and `PromotionApplied` tie on everything except `seq`, so the terminal record is usually
`PromotionApplied`, which is written last.

## From the terminal record to the verdict

1. No records about the promotion — `NotCandidateVerdict`; records exist but none decides —
   `UndeterminedVerdict`.
2. Terminal record with outcome `Applied` — `AppliedVerdict`; but if at least one benefit in the table is
   `BenefitAppliedPartially`, `BenefitNoEffect` or `BenefitExcluded` — `AppliedPartiallyVerdict` instead.
   `BenefitConditionsNotMet` does not make it partial: such a benefit did not run.
3. Terminal record with outcome `ZeroEffect` — `NoEffectVerdict`.
4. `ExcludedByMatrix` — `ExcludedByPromotionVerdict`; `LostMaxBenefit` — `ExcludedByMaxBenefitVerdict`;
   `UnmappedCandidate` — `UndeterminedVerdict`.
5. Terminal record of stage `Benefit` with a non-empty `benefitId`, or of stage `Blocks` with a non-empty
   benefits table — `BenefitConditionsNotMetVerdict`.
6. Everything else — `NotSuitableVerdict`.

## `reason`

`code` and `params` are always present; `failureKind` and `criterionName` only when the record filled them;
`explanation` only when the catalog has a text. `params` is the source record's `payload`, as is.

Where `reason` comes from:

- for `AppliedPartiallyVerdict` — the **headline**: the knocked-out benefit if there is one, otherwise the
  benefit that lost the most (a benefit with no effect lost everything); that benefit's row in the table
  carries the same reason;
- for `ExcludedByPromotionVerdict` — the record about the first knocked-out benefit if there is one in
  the table, otherwise the terminal record;
- for the rest — the terminal record.

## Benefits table

`benefits[]` comes with `AppliedVerdict`, `AppliedPartiallyVerdict`, `NoEffectVerdict` and
`BenefitConditionsNotMetVerdict`. One row per benefit whose `benefitId` some record names, ordered by each
benefit's first record; plus the benefits of a product block dropped as a whole
(`BlockHasNoSuitableProducts`), when the calculation version is known.

Row fields: `benefitId`; `blockType` (`PRODUCT` / `ORDER`) and `settings`, taken from the calculation
version and therefore absent without `history`; `lineVerdicts`, what the benefit did with each product of
the order ([below](#benefit-products-lineverdicts)); `status`, one of five kinds:

| `status.kind` | What happened to the benefit | Fields |
|---|---|---|
| `BenefitApplied` | ran and gave what it computed | `lineKeys[]`, amounts, `balanceSystemName`, `caps[]` — always empty |
| `BenefitAppliedPartially` | ran but got less than it asked for | `reason` (the costliest cap), `requestedAmount` / `grantedAmount` (how much it asked for and received at that cap), `lineKeys[]`, amounts, `balanceSystemName`, `caps[]` |
| `BenefitNoEffect` | ran with no effect | `reason` |
| `BenefitConditionsNotMet` | did not run; a record the explanation does not interpret also lands here, and then `reason` simply repeats that record | `reason` |
| `BenefitExcluded` | knocked out by an incompatible promotion while the rest of the promotion worked | `excluder`, `reason` |

Amounts come only in the units the benefit gives by its nature: `discount` for a discount or delivery
benefit, `earnedBonusPoints` for earning, `discount` + `spentBonusPoints` for spending (the money covered by
bonus points, and the points themselves). A message, a marker and promo code issuance have no amounts. A
missing field means "this benefit does not give that", not zero. Amounts are final: what the order actually
received after arbitration and caps. `lineKeys[]` lists the products on which the benefit remained applied;
empty for delivery, promo code issuance and an order-level message.

How a row gets its status, in check order:

1. There is a `BenefitApplied` record: the benefit ran. If it got less than it asked for and a cap explains
   the shortfall — `BenefitAppliedPartially` (the headline is the cap with the largest loss, requested −
   granted); otherwise `BenefitApplied`.
2. The benefit is zeroed as a whole (a `ZeroEffect` record of promotion scope, usually `BenefitZeroEffect`)
   — `BenefitNoEffect`.
3. There are only computed per-line discounts (`LineDiscountComputed`) — `BenefitApplied` with `discount`
   equal to the computed sum. This is what the benefit wanted to give, not what the order received; this is
   how a benefit of a promotion the engine did not apply looks.
4. A refusal wider than a single line — `BenefitExcluded` if it is `ExcludedByMatrix`, otherwise
   `BenefitConditionsNotMet`.
5. No application record, but the caps granted something — the benefit is treated as applied, without
   amounts, usually partially.
6. Otherwise, by the line records: a zero line gives `BenefitNoEffect`, a refusal gives
   `BenefitConditionsNotMet`.

What the table does not contain:

- a benefit the trace is silent about: for a platform promotion there is a row for every benefit that ran,
  except one whose only effect was marking suitable products;
- the benefits of an external promotion: it has none at all;
- the rows of a dropped block when there is no calculation version; the verdict is then
  `NotSuitableVerdict`, not `BenefitConditionsNotMetVerdict`.

An empty table for an applied promotion means one of the above, not "the benefits did not work".

## Caps `caps[]`

They come only with `BenefitAppliedPartially`. For `BenefitApplied` the list is empty **by construction**,
and that does not mean "nothing was capped". `caps[]` holds the steps that cut the benefit amount before
the discount was spread across lines (`SpreadShortfall` records carrying both amounts), the last one of each
kind, in order. Each step has `reason` with the wording, `requestedAmount` (what the benefit asked for at
that step) and `allowedAmount` (what the step left), in the benefit's units. Each step starts from what the
previous one left.

Per-line caps (`LineDiscountCapped`, `LineDiscountZero`) are not part of `caps[]`: they are visible in the
partial benefit's headline, its `lineVerdicts` and the promotion's records.

`requestedAmount` and `grantedAmount` of `BenefitAppliedPartially` are the amounts of the cap named in
`reason`, in the benefit's units: money for a discount, bonus points for earning and spending. Both fields
are absent when the cap record does not carry them: a line that received nothing does not say how much it
was owed.

Caps arising from the spreading itself and from free lines do **not** make a benefit partial: kinds
`SpreadNoBase`, `SpreadLineLimitReached`, `SpreadDivisibility`, `UnexplainedSpreadResidue`,
`ZeroPriceInOrder`, `ZeroPriceAfterDiscounts`, `ZeroRoundingCorrection`, `UnexplainedZeroLineDiscount`. A
benefit with only such caps comes as `BenefitApplied`, even though the trace has records about them.

A bonus-point spend limited only by the POS's spend request also comes as `BenefitApplied`: its
`requestedAmount` is already no larger than the request, so the benefit "got what it asked for". The request
itself is visible in the shared `BalanceSpendRequested` record and in `SpreadShortfall` with kind
`RequestedAmount` among the promotion's records.

## Benefit products `lineVerdicts`

Every table row, partial or not, carries the order's products grouped by what the benefit did with them.
Each product lands in exactly one group; groups follow the order in which their reasons appear in the trace,
with the unexplained group last.

| `kind` | What happened to the products | Fields |
|---|---|---|
| `AppliedBenefitLineVerdict` | received the assigned amount in full | `lines[]` |
| `AppliedCappedBenefitLineVerdict` | received an amount, but less than computed: a cap on the line itself cut it | `reason`, `lines[]` |
| `NotAppliedBenefitLineVerdict` | the benefit did nothing with them | `reason`, `lines[]` |
| `UnexplainedBenefitLineVerdict` | the trace is silent about them: neither application nor refusal | `lines[]` |

`reason` of `NotAppliedBenefitLineVerdict` comes from the nearest record, in this order of preference: a
record about the line itself, then the filter that dropped it, then the product block that did not take it,
then the excluding promotion, then the refusal of the benefit as a whole. There is one reason per group,
stated once for all its products.

`lines[]` lists the products as they are in the order: `lineKey` (the same as in `inputs.orderLines[]` and
in the records), `productId` and `productName` from the catalog, and `externalId` and `externalSystemName`,
the product's identifier in the client's external system. If the product is not in the catalog, `productId`
and `productName` are absent and `externalId` remains. Name the product to the user by `productName`, or by
`externalId` when there is none; never show `lineKey`.

`UnexplainedBenefitLineVerdict` is not "the benefit did not work": the trace did not record what happened
to these products, and there is nothing to claim about them.

## Amounts `amounts`

For `AppliedVerdict` and `AppliedPartiallyVerdict`, `amounts` is what the promotion as a whole gave the order:

- `discount` — money from discount benefits, including delivery; money covered by spent bonus points is
  **not** included;
- `earnedBonusPoints`, `spentBonusPoints` — bonus points;
- `requestedDiscount`, `requestedEarnedBonusPoints`, `requestedSpentBonusPoints` — how much the promotion
  would have given had nothing cut it. A capped spend is counted in bonus points, not in money; the POS's
  spend request is a ceiling, not a cap, so `requestedSpentBonusPoints` does not exceed it.

Each field is present only if at least one benefit that ran gives that kind of amount. `amounts` is absent
entirely when the trace has no `BenefitApplied` record at all: an external promotion, a promotion whose only
effect is marking suitable products, an old trace. When records exist but carry no amounts, `amounts` comes
with no fields (a placeholder, a marker, an issued promo code). A zero in a field is a recorded zero, not an
absence.

The `requested*` fields come for a capped promotion; a promotion that ran in full usually has none. A
missing field means there is nothing to compare against, not "cut to zero". The difference between
`requested*` and the final amount is how much the promotion lost to caps; the cause is named by the
verdict's `reason` and the benefit rows.

## Lost on max benefit

For `ExcludedByMaxBenefitVerdict`:

- `winningVariant[]` — the **whole** composition of the winning variant, each promotion as `promotionId` +
  `name`. An empty array means applying nothing turned out better; an absent field means the trace did not
  record the composition.
- `displacedBy[]` — the winner's promotions that are absent from the variant containing the checked
  promotion: they took its place, while the rest of the winner's promotions would have applied alongside it
  anyway. An absent field means it is unknown who displaced it.
- `comparison`, by `kind`. `VariantsWereEqual`: the variants are equal on every measure and traversal order
  decided; this says nothing about the promotion itself, and "lost to a more beneficial one" would be false
  here. `WinnerWasBetter`: the winner is better on the measure `measure`, with both sides' values in
  `winnerValue` and `variantValue`. An absent field means the trace did not name the measure.

`name` in compositions and in `excluder` comes only for promotions asked about in the same call; the
number in `excluder` may be empty if the record did not fill it.

Three traps of `WinnerWasBetter`. The values are the totals of the **variants for the whole order**, not
the promotion's contribution: their difference is not what the checked promotion would have given. Do not
judge from the values which side is better: for four of the twelve measures smaller is better, and the
better side is whichever `winningVariant` names. The measure determines the value type: money, bonus points,
pieces, a flag (`true` / `false`) or a date. The ladder of measures —
[DECISION_FIELDS.md § Competition between promotions](DECISION_FIELDS.md#promotion-competition).
