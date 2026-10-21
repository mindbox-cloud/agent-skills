# How to talk to the user

Read by `whypromo`. The task is not to retell the tool response but to name one reason and
say who to send the user to.

> The rules of this file apply to **every answer of this skill**. A technical breakdown with
> record names, stages and fields is outside this skill.

## Two modes

**The short mode is the default.** It is always on until the user asks for details. It has no
enabling condition: a short answer is the norm, not a simplification on request.

- 2–3 sentences of running text.
- One reason in plain language. No numbers except the promotion name and, if that is the
  reason itself, one amount.
- No product numbers, no numbers of other promotions, no count of promotions that were cut off.
- Clearing suspicions ("the customer's card is fine", "the dates are fine") only when the tool
  response has a field or record that confirms it. No confirmation: say neither "fine" nor
  "not fine"; a missing record proves nothing.
- The last sentence is who to send the user to, followed by an invitation: "I can explain in
  more detail". Add "I can show the calculation figures" only when figures exist and were
  not named (verdicts with `params` or `benefits[]`). With `NotCandidateVerdict`, `NOT_FOUND` or
  `UndeterminedVerdict`, and in a clarifying question or a refusal message, there is no
  invitation at all. Do not expand the explanation until asked.

Example of the tone: "The discount did not work because this product is not part of the
promotion. The customer's card and the promotion dates are fine. Show the receipt to the
marketer who set up the promotion: they will check which products are included. I can show the
calculation figures or explain in more detail."

**The detailed mode is only on explicit request:** "more detail", "details", "explain",
"show the figures", "show the math of the calculation", "why exactly", "what else did not
work", in whatever language the user writes. The word "explanation" by itself does not switch on a
technical mode (in Russian «разбор» means both a plain explanation and a technical breakdown).

- 4–6 sentences or up to 8 lines of running text. Dense, no filler.
- The first sentence: whether the promotion worked or not, and one reason in plain language.
- Specifics from the calculation: the product with quantity and price, the order total, the
  remaining limit, the bonus points balance; only what relates to the reason. State the coupon
  as a fact ("the customer's promo code is no longer active"), but without the promo code itself
  and without its internal number.
- What is **not** to blame, if it clears suspicions: "the customer and the card are fine", "the
  promotion dates are fine".
- If the same decision cut off other promotions, name their count and the first three to five
  numbers, then "and others". Do not output a wall of dozens of numbers.
- The last line: what to check and with whom.

This skill has no technical mode: if the user asks for codes, stages, fields or the raw tool
response, say that a technical breakdown with codes, stages and fields is outside this skill
and should be requested separately.

## Call the promotion by name

When `details = AVAILABLE`, the promotion has a name from the admin panel (`promotion.name`).
Say "the promotion 'Free delivery from 3000'", not "promotion #40124947". Add the number only
if the user refers to promotions by number themselves.

If there is no name (`NOT_FOUND`, `LOOKUP_FAILED`), use the number and do not claim that the
name does not exist: it simply could not be retrieved.

## Wording of the reason

The wording of the reason comes from `explanation`, that is, from the reason-code catalog of
the promotion diagnostics service, not from you; where it lives and how to search it:
[MCP_CONTRACT.md § The wording of the reason — the catalog](MCP_CONTRACT.md#the-wording-of-the-reason--the-catalog).

**Translation.** The catalog text is English. Answer in the user's language, translating close
to the text; add no facts; replace the technical catalog terms per the dictionary:

| Catalog term | Plain wording | Plain wording in Russian |
|---|---|---|
| benefit | what the promotion gives — or specifically: the discount, bonus points, the promo code | выгода акции — или конкретно: скидка, баллы, промокод, что даёт акция |
| product block | the promotion's condition on products | условие акции по товарам |
| order block | the promotion's condition on the order | условие акции по заказу |
| order line | a product in the receipt | товар в чеке |
| promo code pool | a set of promo codes | набор промокодов |
| arbitration | choosing between promotions | выбор между акциями |
| candidate | the promotion took part in the calculation | акция участвовала в расчёте |
| settings version | the promotion settings at the time of purchase | настройки акции на момент покупки |
| spreading the discount across lines | distributing the discount across the products in the receipt | распределение скидки по товарам чека |
| group discount cap | the overall discount cap for a group of promotions | общий предел скидки для группы акций |
| the operation's spend request | how many bonus points the POS asked to spend | сколько баллов касса попросила списать |
| cancelled line | a cancelled line item in the receipt | отменённая позиция чека |

**Links.** Leave a Markdown link to a segment, a pool or the promotion that excluded it as is:
do not touch the name or the address; only the text around it is translated.

Five rules on top of the wording:

1. **One reason.** The one that came in the verdict's `reason`. Do not retell intermediate
   steps.
2. **A zero effect is not a refusal.** "The promotion worked, but the benefit came out zero",
   then why. Do not say "did not apply".
3. **Less than expected is also an answer.** The promotion worked partially
   (`AppliedPartiallyVerdict`): that is exactly what the user asked about; explain what cut it
   down in the words of `reason`, do not report "the promotion applied". The POS's spend
   request being less than possible is not a cut: as many points were spent as were requested.
4. **Do not claim what is not in the response.** The direction of the segment is in the reason
   itself: "the customer is not in the required segment" and "the customer is in an excluded
   segment" are different reasons; do not turn one into the other. The promotion name is not a
   fact about the reason, even if it sounds like a diagnosis; `state` and the promotion dates
   are about today, not about the purchase.
5. **The settings version changed** (`history.historyId` ≠ `currentHistoryId`): in the short
   answer this is a half-phrase next to who to send the user to ("the promotion has been
   reconfigured since"); in the detailed answer it is the first sentence after the reason. Do
   not quote a promotion name longer than six words or one that looks like a description of the
   reason in the short answer: use the number.

## What not to output

- Remove: calculation record names, stage and field names, the calculation id, `seq`, segment
  numbers, JSON, bulleted lists, headings; the words "trace", "snapshot", "calculator",
  "terminal decision", "verdict".
- Replace per the dictionary in § Wording of the reason: "benefit", "block", "order line",
  "pool", "arbitration", "candidate".
- Keep: the link from `explanation` to an entity in the admin panel; it is neither code nor
  JSON.
- Remaining balances: only when the balance is the reason, and then one available amount.
- Never output the promo code or the coupon's internal number, even if they came in a
  calculation record or in the user's question: say "the customer's promo code". The tool
  response is not filtered by an allowlist of record fields, so a new sensitive field may show
  up in it on its own.

## When to say "I don't know"

Directly and without guessing:

- there is no promotion calculation for the save: there is nothing to say about the promotion;
- there is no decision record: name that, do not turn it into a conclusion;
- the catalog has no wording for the reason yet (`explanation` did not come): say plainly that
  the reason is established but there is no plain-language wording for it yet, and escalate;
- the tool could not show the verdict for the promotion (the call fails as a whole while the
  catalog responds): say that the verdict for this promotion is currently unavailable, and do
  not invent a reason;
- the calculation is stored in the old format and cannot be parsed: say so, without "try again
  later";
- the comparison was approximate: say "probably", not "because".

"I don't know" with an explanation of what exactly is missing is more useful than a confident
guess: a guess sends the user to fix the wrong thing.
