# Units and non-obvious fields

The meaning of a record comes from the catalog wording; the shape of the fields is in [MCP_CONTRACT.md](MCP_CONTRACT.md).
This file covers only the fields whose name does not speak for itself: their unit, their direction, or what to
check them against. A mixed-up unit flips the answer.

Some fields sit directly on the record and the rest in `payload`, as the calculation wrote them; which is which
depends on the record, so look in both places. A record or a field that is not described here
or in [RECORDS.md](RECORDS.md) belongs to an older trace format: describe it by its fields, do not guess its meaning.

## Record address

| Field | What it is |
|---|---|
| `scope` | the level of the record: `Calculation`, `Promotion`, `Block`, `Line` |
| `promotionIds[]` | the promotions the record is about: one for a single record, several for a folded one; empty for shared records. For `MaxBenefitWinnerChosen`, the losers |
| `benefitId` | the promotion's benefit setting; a promotion may have several. Empty for records that are not about a benefit |
| `benefitKind` | the class name of the benefit setting, not a category: `DiscountBenefitSettingDetail`, `OrderDiscountBenefitSettingDetail`, `BalanceDiscountBenefitSettingDetail` (spending points), `AcquireBalanceBenefitSettingDetail` (earning points), `FreeDeliveryBenefitSettingDetail`, `PriceRoundingBenefitSettingDetails`, `PlaceHolderBenefitSettingDetail` and others |
| `blockOrdinal` | the index of the **product** block within the version, zero-based, counting dropped blocks; records about order-level blocks do not carry it. `0` is either the first block or an absent field; the two cannot be told apart: [PITFALLS.md § Zero and false](PITFALLS.md#zero-and-false-often-mean-the-field-was-absent) |
| `lineKey` | the key of the order line, numbered from one; check against `inputs.orderLines[].lineKey`; `0` means not set. For `ProductNotInBlock`, `LinesExcludedByMatrix`, `CancelledLinesExcluded` and `BenefitApplied` the lines come as a `lineKeys` list in `payload` |
| `seq` | the order of the record in the trace: "what came later" is read from this; "what is final" is read from `isTerminal` |
| `failureKind` | the kind of reason; the sets are closed, and the wording of each kind is in the catalog |
| `excluderPromotionId` | the first excluder; may be empty |
| `offerId`, `filterId` | the product that conflicts between blocks; the filter that dropped lines |
| `promoCodePoolId` | the pool of the coupon itself (`CouponApplied`, `CouponNotApplied`); the pool the promotion expects is in `history.accessibility.promoCodePoolRestriction` |
| `couponInternalId`, `couponStatus`, `couponSource` | the coupon's internal number, its status in the CDP and where it came from (`Input`: loaded; `RequestedPromotion`: passed in the input but not loaded). Do not show the number to the user |
| `balanceSystemName` | the bonus points balance used for earning, spending and the spend cap; check against `inputs.balances[].systemName` |

## Money, bonus points and quantities

Money fields are **in the project's currency (rubles for Russian projects)**, in whole units of the
corresponding balance, not in minor units. Do not divide by 100.

| Field | Unit | What it is |
|---|---|---|
| `discountAmount` (`BenefitApplied`) | money | the benefit's final discount after arbitration and caps, including the delivery discount and the money covered by spent bonus points |
| `earnedBonusPoints` / `spentBonusPoints` | bonus points | how many points the benefit earned and spent |
| `requestedAmount` | depends on the record | for `BenefitApplied`: how much the benefit asked for before limits; money for a discount, bonus points for earning and spending (for spending, no more than the POS's spend request). If the key is absent, the request is unknown and is **not** equal to what was granted. For `SpreadShortfall` and `LineDiscountCapped`: how much the benefit asked for at this step. For `BalanceSpendRequested`: how many points the operation asked to spend |
| `grantedAmount` | the same as `requestedAmount` on the same record | what remained after the cap (`SpreadShortfall`, `LineDiscountCapped`); for `LinePriceRaised`, how much the line price went up by |
| `lineDiscountAmount` / `appliedDiscountAmount` | money | for `LineDiscountComputed`: how much the formula wanted to give the line; this is an expectation, the result is in `BenefitApplied` |
| `groupCapAmount` | money | the discount cap of the nearest promotion tree group: a group setting, not a promotion setting |
| `thresholdAmount` / `achievedAmount` | money | the condition threshold (or the step size of "for every X") and how much had been accumulated at the time of the check |
| `priceRangeFrom` / `priceRangeTo` | money | the bounds of the price range of an earning condition |
| `possibleGiftsCount` | times | how many times the "for every X" step fired; `0` means never |
| `matchedTierCount` | tiers | how many tiers of a cascading discount fired |
| `applicableQuantity` | product units | the quantity of the line to which the benefit was applied |
| `deliveryCost` | money | the delivery cost (for `BenefitZeroEffect` with `NoDeliveryCost`) |
| `requestedCount` / `noAmountCount` | items | the total number of external coupon requests and the number without an amount |
| `localDateTime` | local time | the local time of the calculation against which days and hours were checked; empty means the time zone could not be determined |

For `LineDiscountCapped` with kind `QuantityTruncated`, `requestedAmount` and `grantedAmount` are in **product units**:
how many units the benefit assigned and how many the line has. For earning caps the amounts are in bonus points.

## Limits

| Field | Unit | What it is |
|---|---|---|
| `discountResidue` | money | the remaining discount amount on the promotion's counter (`LimitExhausted`); `0` means exhausted |
| `applicationsResidue` | **applications, not money** | the remaining applications on the same counter |
| `inputs.limits[].limitAmount` / `usedAmount` | per `attribute` | the configured limit and what has been used |

`2147483647` in a residue means there is no limit on this dimension. Unavailable counters, and how to match
`inputs.limits[]` rows to a promotion: [PITFALLS.md § Unavailable counters](PITFALLS.md#unavailable-counters-are-not-an-exhausted-limit).

## Lines and selection

| Field | Unit | What it is |
|---|---|---|
| `keptLineCount` / `totalLineCount` | lines | how many lines were kept, out of how many in total |
| `droppedLineKeys[]` | line keys | which lines were dropped |
| `lineKeys[]` | line keys | for `BenefitApplied`: the lines the benefit ended up applied to (empty for delivery, promo code issuance and an order-level message); for `ProductNotInBlock`: the lines that did not fit the block; for `LinesExcludedByMatrix`: the lines taken by the excluder; for `CancelledLinesExcluded`: the cancelled lines |
| `restrictionKind` | type name | the kind of the benefit's product restriction (`LinesFilteredBySkuRestriction`) |
| `groupEntryPrice` / `effectiveBasePrice` | money | the price at which the line entered the group, and the calculation base after the previous discount reduced it (`SerialBaseReduced`) |

## Promotion competition

| Field | Where | What it is |
|---|---|---|
| `displacedByPromotionIds` | `LostMaxBenefit` | the winner's promotions that are absent from the loser's variant, in the winner's order; empty means a variant with no promotions won |
| `winnerKeyValue` / `variantKeyValue` | `LostMaxBenefit` | the value of the deciding measure for the winner and for the loser's variant, as a string: money, bonus points, items, `true` / `false` or a date. With `tieBreakOrder` both are absent |
| `winnerPromotionIds` | `MaxBenefitWinnerChosen` | the full composition of the winning variant, including promotions applied before the group; empty means a variant with no promotions won |
| `treeVersion` | `NotInArbitrationTree` | the version of the tree in which the promotion was looked up; the same field exists in `inputs` |

The deciding measure is named by the `failureKind` of the loss record (in the verdict, by `comparison.measure`, in upper case:
`TOTAL_DISCOUNT`, `HAS_BALANCE_DISCOUNTS` and so on). The measures by which the calculation picks a variant are listed below, top
to bottom. The first measure on which the variants differ decides; on a full tie the variant that comes earlier in
the traversal wins (`tieBreakOrder`):

| # | Measure | Better |
|---|---|---|
| 1 | `totalDiscount` — the variant's total discount on the order | larger |
| 2 | `externalDiscount` — the part of the discount from external systems | larger |
| 3 | `discountWithoutCoupons` — discount without coupons | larger |
| 4 | `hasBalanceDiscounts` — whether there are balance spends | **smaller**: no spends is better |
| 5 | `discountCouponsCount` — number of discount coupons | **smaller** |
| 6 | `balanceDiscountsWithoutCoupons` — balance spend without coupons | larger |
| 7 | `balanceDiscountCouponsCount` — number of balance-spend coupons | **smaller** |
| 8 | `totalEarnedBonusPoints` — bonus points earned in total | larger |
| 9 | `earnedBonusPointsWithoutCoupons` — earned without coupons | larger |
| 10 | `earnedBonusPointsCouponsCount` — number of earning coupons | **smaller** |
| 11 | `earnedBonusPointsExpirationIsNull` — bonus points expiration not set | larger: no expiration beats any date |
| 12 | `earnedBonusPointsExpirationDateTimeUtc` — the earliest expiration date | larger: later is better |

So "whoever has more wins" is a wrong reading: for four measures the smaller value is better, and the expiration
date is compared only after the "no expiration" case has been ruled out. The winner is the one the record names.
