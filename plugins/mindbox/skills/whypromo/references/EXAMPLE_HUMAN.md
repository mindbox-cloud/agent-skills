# Example explanation for a human

One end-to-end pass: what the user wrote, what came back from the tools, what they got as the answer.

## The user's input

> `https://demo-shop.mindbox.ru/promotions/40124947 8813322`

A promotion link and an order number. The project `demo-shop` and the promotion `40124947` come from the link; the order `8813322` is next to it. `tenant` is `demo-shop`.

`loyalty_find_order_calculations` with `orderExternalId: "8813322"` answers `FOUND`: one order, with one
history entry carrying a `calculationId`. Nothing to choose from, so no questions to the user; go straight to
`loyalty_explain_promotions` with that `calculationId` and the promotion `"40124947"`.

## Tool response (trimmed to what matters)

```json
{
  "status": "EXPLAINED",
  "explanation": {
    "operation": "Website.CreateOrder",
    "candidateCount": 14,
    "inputs": { "isCustomerAuthorized": true, "limitsDataUnavailable": false },
    "promotions": [{
      "promotionId": "40124947",
      "details": "AVAILABLE",
      "promotion": { "kind": "MindboxPromotion", "name": "500 ₽ for every 5000 ₽", "state": "Execution" },
      "verdict": { "kind": "BenefitConditionsNotMetVerdict",
        "reason": { "code": "BenefitThresholdNotReached", "failureKind": "PreconditionSumBelowStep",
          "params": { "thresholdAmount": 5000, "achievedAmount": 3200 },
          "explanation": "The purchase amount for these products is less than the amount set in the benefit" },
        "benefits": [{ "benefitId": "42400915", "blockType": "PRODUCT",
          "status": { "kind": "BenefitConditionsNotMet",
            "reason": { "code": "BenefitThresholdNotReached", "failureKind": "PreconditionSumBelowStep",
              "params": { "thresholdAmount": 5000, "achievedAmount": 3200 },
              "explanation": "The purchase amount for these products is less than the amount set in the benefit" } } }] },
      "decisions": [
        { "seq": 118, "code": "BenefitThresholdNotReached", "stage": "Benefit", "outcome": "Fail",
          "scope": "Promotion", "isTerminal": true, "promotionIds": ["40124947"],
          "benefitId": "42400915", "benefitKind": "SegmentDiscountForOrderSumBenefitSettingDetail",
          "blockOrdinal": 0, "lineKey": 0, "criterionName": "", "failureKind": "PreconditionSumBelowStep",
          "payload": { "thresholdAmount": 5000, "achievedAmount": 3200 } }
      ]
    }]
  }
}
```

## Short answer: the default mode

> The promotion "500 ₽ for every 5000 ₽" did not apply: the purchase of the promotion's products is below the
> amount set in its conditions. It needs 5000 rubles, and the order reached 3200. The customer's card is fine.
> Add to the order, or ask the marketer who set up the promotion whether the threshold can be lowered. I can
> explain in more detail.

Three sentences, one reason, one pair of numbers, who to send the user to. The wording is a translation of
`explanation`, the numbers come from `params`. No record name, no word "benefit", no promotion number. The
closing offer does not say "show the numbers": the amount is the reason, and it is already named.

## The same case when the user asked for more detail

> The promotion "500 ₽ for every 5000 ₽" did not apply: it kicks in from 5000 rubles of purchases of its own
> products, and the order reached 3200, so 1800 was missing. The customer was identified, and the promotion
> itself took part in the calculation: it was rejected only at the amount condition, so that is the only thing in
> the way. If the threshold should be different, that is for the marketer who set up the promotion.

Added: the missing amount and the suspicions ruled out, each backed by data: the customer by
`isCustomerAuthorized`, the promotion's participation by the threshold record sitting at the benefit stage, which
means the promotion passed selection. Still no codes and no stages.

## What would be a mistake

- "The promotion was rejected at the Benefit stage by the BenefitThresholdNotReached record": the user knows
  neither stages nor record names.
- "1800 kopecks were missing" or "18 rubles were missing": dividing by 100. Amounts come in the project's
  currency (rubles for Russian projects), not in minor units.
- "The limit is exhausted": the response has `limitsDataUnavailable: false` and no record about exhaustion;
  that is a different reason, and it was not here.
- Answering with the English `explanation` phrase as is when the user asked in Russian: answer in the user's
  language.
- "The promotion is active now, so the dates are not the problem": `state` is about today, not about the purchase.
- Retelling all fourteen candidates: the question was about one promotion.
