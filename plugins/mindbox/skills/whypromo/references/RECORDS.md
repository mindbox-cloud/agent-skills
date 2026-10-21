# Trace records: which exist and what to read in them

A map of the records the calculation engine writes today. The meaning of a specific kind
(`failureKind`) comes from the wording in the reason-code catalog; this file covers only what cannot be understood from
the response: what the record is about, where it lives, which fields to carry into the explanation and which
verdict it leads to. A record not listed here is from an older trace
format: describe it by its fields, do not guess its meaning. Field units: [DECISION_FIELDS.md](DECISION_FIELDS.md).

## How to read the tables

"Outcome, where" is the record's `outcome` and its scope (`scope`); "shared" means a record with no promotion,
which lives in `sharedDecisions[]`. Kinds are listed where the reading depends on them; the full set of record kinds
is the set of keys in the catalog (`loyalty_reason_codes`).

## Selection

Stage `Selection`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `SelectionCriterionFailed` | Fail, promotion | the promotion failed selection: operation mode, channel and touchpoint, zone, customer segment, brand, store timezone. Which way the segment check failed is in the kind name (`CustomerNotInSegment`: a segment is required and the customer is not in it; `CustomerInExcludedSegment`: the customer is in an excluded one). Check against `inputs.selection`, `customerSegmentIds[]` and the restrictions in `history` |
| `PromoCodeNotSuitable` | Fail, promotion | the promotion requires a promo code and no suitable one exists: `NoPromoCodePassed`, `CouponDataUnavailable`, `PromoCodeOfOtherPool`, `PoolDraft`, or the fate of the first rejected coupon of its pool (`CouponUsed`, `CouponBlocked`, `NotIssuedToCustomer`…). The promotion's pool is in `history.accessibility.promoCodePoolRestriction` |
| `DayTimeWindowNotMatched`, `DayOfWeekNotMatched` | Fail, promotion | the time or day of week is outside the promotion's window (`OutsideWindow`), or the store timezone could not be determined (`StoreTimezoneMissing`): check `localDateTime` against the window or the days |
| `ExternalPromotionShadowed` | Fail, promotion | a discount passed by the POS matched a Mindbox promotion by number: the Mindbox calculation gives the discount and the passed value is discarded |
| `CouponNotApplied` | Fail, shared | the coupon passed into the calculation did not apply: status in the CDP (`CouponUsed`, `CouponBlocked`, `CouponYetNotActive`, `CouponAlreadyNotActive`, `CouponNotIssued`, `CouponNotFound`, `CouponNotEnoughData`, `CouponGotByFriendInvitation`), `ReferralOwnCoupon`, `NotIssuedToCustomer`, `NotRequested`, `NotLoaded`, `ApplicationsLimitReached`, or `CouponChecksPassed` (the coupon is fine; the reason lies in the promotion itself). Fields: `couponInternalId`, `promoCodePoolId`, `couponStatus`, `couponSource` |
| `UnmappedCandidate` | Fail, promotion | technical: the candidate reached the end of the calculation without a verdict; verdict `UndeterminedVerdict` |

## Blocks and lines

Stages `Blocks` and `LineFit`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `BlockHasNoSuitableProducts` | Fail, promotion | the product block `blockOrdinal` was discarded: no products match the condition (`NoProductInSegment`, `NoProductInList`), line items cancelled (`AllLinesCancelled`, `SuitableLinesCancelled`), no product line items (`OrderHasNoProducts`). The block's benefits land in the table as `BenefitConditionsNotMet` |
| `PromotionHasNoCalculators` | Fail, promotion | all product blocks were discarded and the promotion has no order-level blocks: nothing to calculate |
| `SkuBlocksOverlap`, `BlockConflict` | Fail, block | product `offerId` matched two product conditions of the same promotion, a setup error; as long as the product is in the order, the promotion does not apply |
| `ProductNotInBlock` | Fail, block | the lines listed in `lineKeys` in `payload` did not fit the block. `ProductUnknown`: the product is not in the project's catalog; the cause is the catalog integration, not the promotion setup |
| `LinesFilteredBySkuRestriction`, `LinesFilteredByFilter` | Fail or ContextInfo, promotion | the benefit's product restriction or filter dropped lines: `keptLineCount` out of `totalLineCount` were kept, `droppedLineKeys` lists the dropped ones |
| `CustomerNotAuthorized` | Fail, promotion | the benefit is only for authorized customers, but the calculation ran anonymously; `benefitId` is the **first** inapplicable benefit |
| `OrderRestrictionNotMatched` | Fail, promotion | the order did not match the block's order segmentation (`OrderFilterNotMatched`) or matched an excluded one (`OrderFilterExcluded`) |
| `CancelledLinesExcluded` | ContextInfo, shared | cancelled line items (`lineKeys`) are not part of the promotion calculation; returned ones are |

## Benefit

Stage `Benefit`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `LineDiscountComputed` | Pass, line | how much the per-line discount formula wanted to give the line (`lineDiscountAmount`): an **expectation**, not the result. Written only by per-line product discounts |
| `BenefitThresholdNotReached` | Fail, promotion | the threshold was not reached: `PreconditionSumBelowStep` (compare `achievedAmount` with `thresholdAmount`), `OrderPriceOutOfRange` (compare with `priceRangeFrom` / `priceRangeTo`), `PreconditionBelowStep` (the record carries no amounts) |
| `BenefitStepNotReached`, `PriceRangeNotMatched`, `PurchaseSetNotMatched`, `FavoriteListNotMatched`, `CustomFieldValueNotMatched` | Fail | a condition of the benefit itself was not met: the "for every X" step, price tiers, a joint-purchase set, favorites, a price from a product field |
| `BenefitZeroEffect` | ZeroEffect, promotion | the benefit gave zero as a whole. The kind names the reason: a cap (`ApplicationsLimit`, `PromotionDiscountLimit`, `PromotionAccrualLimit`, `PromotionSpendLimit`, `CustomerBalance`, `GroupDiscountCap`, `LinesAtMinPrice`, `RequestedAmount`, `BalanceReserved`), line items (`AllLinesCancelled`, `AllLinesReturned`, `SuitableLines…`, `NoInputLines`, `NoSuitableLines`, `NoLinesMatchedProductRestriction`, `NoDiscountableLines`), the spend request (`NoBalanceSpendRequested`, `ZeroBalanceSpendRequested`, `BalanceSpendRequestedForOtherBalance`, `BalanceAvailabilityRequestedOnly`, `BalanceAlreadySpent`), the accrual field (`Accrual…`), delivery, rounding, placeholder, spreading |
| `LineDiscountZero` | **Applied**, line | the line was part of the benefit and got zero: it is free or already zeroed (`ZeroPriceInOrder`, `ZeroPriceAfterDiscounts`), it is at the minimum price (`LineMinPrice`, `GroupMinPrice`), the group cap ran out (`GroupDiscountCap`), the limit was used up by previous lines or was exhausted before the order (`…LimitSpent`, `…LimitExhausted`), the bonus points were used up by previous lines, the line was discounted by earlier promotions, starting units of a cascade, spreading, rounding, `UnexplainedZeroLineDiscount` |

A benefit condition not met leads to `BenefitConditionsNotMetVerdict`; a benefit zeroed as a whole leads to
`NoEffectVerdict` or, if another benefit alongside it worked, `AppliedPartiallyVerdict`.

## Limits

Stage `Limit`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `LimitExhausted` | Fail, promotion | the limit was already exhausted when the benefit was entered: `ApplicationsLimitExhausted`, `DiscountLimitExhausted`, `AccrualLimitExhausted`; the residues are `applicationsResidue` and `discountResidue` (`2147483647` means no limit on this dimension). Verdict: `NotSuitableVerdict` |
| `LimitDataUnavailable` | ContextInfo, promotion | the limit counters could not be read: the limit was **not checked**; it is not exhausted |

## Exclusions

Stage `Exclusion`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `ExcludedByMatrix` | Fail, promotion | incompatibility. With `OrderLevelExclusion` or `ExcluderOnOrder` the whole promotion is knocked out; with `ExcluderOnOrderLine` or `AllLinesExcluded` one benefit is knocked out (`benefitId` is set) and the rest work. `excluderPromotionId` is the **first** excluder |
| `LinesExcludedByMatrix` | Fail, line | the excluder took the lines in `lineKeys` away from the promotion; the promotion is calculated on the rest, and this record does not decide the verdict |

## Arbitration

Stage `Arbitration`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `NotInArbitrationTree` | Fail, promotion | the promotion is not in the project's promotion tree (`treeVersion`): verdict `NotSuitableVerdict` |
| `SerialBaseReduced` | ContextInfo, promotion | a previous discount reduced the base: `groupEntryPrice` → `effectiveBasePrice`; background, not a reason |
| `MaxBenefitWinnerChosen` | ContextInfo, promotion | one per variant comparison: `promotionIds` holds the **losers**, `winnerPromotionIds` the full composition of the winner. That is why it lives in the losing promotion's `decisions[]`, not the winner's |
| `LostMaxBenefit` | Fail, promotion | the promotion lost the variant comparison: `failureKind` is the deciding measure, or `tieBreakOrder` for a tie; `winnerKeyValue` and `variantKeyValue` (absent with `tieBreakOrder`); `displacedByPromotionIds` |

The winner of the comparison has no arbitration record: the fact that the promotion applied is visible from `PromotionApplied`.

## Application and caps

Stage `Application`.

| Record | Outcome, where | What it means and what to read |
|---|---|---|
| `BenefitApplied` | Applied, promotion | the benefit's result after arbitration and caps: `discountAmount`, `earnedBonusPoints`, `spentBonusPoints`, `lineKeys`, `balanceSystemName`, and `requestedAmount`: how much the benefit asked for before limits (for a spend, already no more than the POS's spend request). Written only if the benefit gave something |
| `PromotionApplied` | Applied, promotion | the promotion applied |
| `CouponApplied` | ContextInfo, promotion | the coupon applied to this promotion: `couponInternalId`, `promoCodePoolId`, `couponSource` |
| `SpreadShortfall` | ContextInfo, promotion | the benefit amount was cut **before spreading** across lines: `requestedAmount` → `grantedAmount` in the benefit's units. The kind is a cap (the same nine as in `BenefitZeroEffect`; for the group cap the number is also given in `groupCapAmount`), a spreading residue (`SpreadNoBase`, `SpreadLineLimitReached`, `SpreadDivisibility`, `UnexplainedSpreadResidue`) or a quantity budget (`QuantityBudgetSpent`, `QuantityCapByApplicationsLimit`). These are the steps listed in `caps[]` |
| `LineDiscountCapped` | ContextInfo, line | the line accepted less than it was assigned: `requestedAmount` → `grantedAmount`. The kind is a minimum price (`LineMinPrice`: passed in the order; `GroupMinPrice`: from the promotion tree group; `BenefitMinPrice`: from the promotion setup; `MinimalItemPrice`: not below 1 per unit), the group cap (`GroupDiscountCap`, `groupCapAmount`), limits and balance, a discount by earlier promotions (`DiscountedByEarlierPromotions`), the per-line-item accrual ceiling, a benefit larger than the product price, or `QuantityTruncated`, for which the amounts are in **product units**, not money |
| `LinePriceRaised` | ContextInfo, line | the line price went up: order price rounding (`OrderPriceRounding`), a negative discount of an external promotion (`ExternalPromotion`) or `UnexplainedPriceMarkup`; `grantedAmount` is how much was added |
| `BalanceSpendRequested` | ContextInfo, shared | how many bonus points the operation asked to spend (`requestedAmount`, `balanceSystemName`): input, not result |
| `ExternalPromotionNotApplied` | Fail, promotion | an external promotion did not apply; the reason is in `failureKind` |
| `ExternalCouponRequests` | Fail, shared | external coupon requests with no promotion: how many were requested (`requestedCount`) and how many had no amount (`noAmountCount`) |

Cutting the amount makes the benefit partial, except for the caps of spreading itself and for free lines:
[VERDICTS.md § Caps](VERDICTS.md#caps-caps).

## Shared records

`sharedDecisions[]` holds records that name no promotion: `CouponNotApplied`,
`CancelledLinesExcluded`, `BalanceSpendRequested`, `ExternalCouponRequests`, as well as the per-line
`LineDiscountZero`, `LineDiscountCapped`, `LinePriceRaised` and `SpreadShortfall` from the results of external
promotions. Do not attribute them to the promotion being checked.

There are no longer separate records for group ceilings and minimum-price floors: they come as the kinds
`GroupDiscountCap` / `GroupMinPrice` on the promotion's own records, with the group cap also given as the number `groupCapAmount`.

## Records without wording

The catalog has no text for `LostMaxBenefit`, `UnmappedCandidate`, `MaxBenefitWinnerChosen`,
`BalanceSpendRequested`, `ExternalPromotionNotApplied` and `ExternalCouponRequests`. This is a known gap, not
lost data: describe them by their fields. The first two need no wording: the verdict carries the meaning. The kinds of
`BenefitThresholdNotReached` have no text of their own; the record's general text is used instead.

## Records not to expect

- An operation rejected before promotion selection: the calculation simply did not run, so there is no record.
- "No suitable lines in the order" as a separate record: it is the `NoSuitableLines` kind of `BenefitZeroEffect`.
- A record for the arbitration winner: there is none; the winner's composition lives in the losers' `MaxBenefitWinnerChosen`.
- Separate records for the group ceiling and the minimum-price floor: see [§ Shared records](#shared-records).
