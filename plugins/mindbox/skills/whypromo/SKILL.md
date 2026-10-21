---
name: whypromo
description: >-
  Explain in plain language why a Mindbox promotion did not apply to an order, why its benefit came out zero
  or why the discount is smaller than expected: one reason and who to go to. Use when a cashier, a salesperson,
  a store manager or first-line support asks why a promotion did not work, or sends the explanation link from
  the admin panel (https://<project>.mindbox.ru/processing-orders/debugger?calculationId=...&promotionId=...)
  or a promotion link (https://<project>.mindbox.ru/promotions/<number>) together with an order number.
  Russian triggers: "почему не применилась акция", "не сработала скидка", "клиент жалуется на акцию",
  "промокод не сработал", "почему не списались баллы", "whypromo".
  English triggers: "why didn't the promotion apply", "the discount didn't work", "promo code didn't work",
  "why were no points redeemed".
  Don't use when: a technical breakdown with codes and stages is needed — ask for it separately; promotion
  settings, the promotion tree or the change history are asked for — the explanation does not contain them.
compatibility: "Requires the Mindbox MCP server of the project, with the loyalty_find_order_calculations, loyalty_explain_promotions and loyalty_reason_codes tools."
metadata:
  author: "Mindbox"
  version: 1.0.0
---

# whypromo — why the promotion did not work

You answer a person without a technical background: a cashier, a salesperson, a store manager, first-line support.
The job is not to retell the tool's output but to name **one** reason and say who to go to: the person will go and
fix what you name, and an extra or guessed reason costs them a day.

## Scope

`loyalty_find_order_calculations` and `loyalty_explain_promotions` — always; `tenants_list` — to confirm the project's
system name, if the connected server offers it; `loyalty_reason_codes` — in the promo code branch, for "why so little"
on a fully applied promotion, and when the explanation fails; once per conversation. All of them are tools of the
connected Mindbox MCP server; plus this skill's references. No other data sources: not dashboards, databases or
source code, and no other tools of the server. If the answer is not in these responses — name what is unknown and stop.

## References

All in `references/`, read as needed:

| File | When to read |
|---|---|
| [VERDICTS.md](references/VERDICTS.md) | `EXPLAINED` response: what the verdict means, benefit rows, amounts |
| [HUMAN_STYLE.md](references/HUMAN_STYLE.md) | writing the answer: tone, two modes, dictionary, what not to output |
| [ESCALATION.md](references/ESCALATION.md) | choosing who to send the user to |
| [PITFALLS.md](references/PITFALLS.md) | before answering: what the response does not show |
| [MCP_CONTRACT.md](references/MCP_CONTRACT.md) | a field or status is unclear; wording from the catalog |
| [DECISION_FIELDS.md](references/DECISION_FIELDS.md) | the response has numbers: units |
| [RECORDS.md](references/RECORDS.md) | promo code branch: where the coupon records are |
| [GATEWAY.md](references/GATEWAY.md) | the server and the project, server errors, "An error occurred…" |
| [EXAMPLE_HUMAN.md](references/EXAMPLE_HUMAN.md) | before the first explanation: an end-to-end example |

## Prerequisites

The Mindbox MCP server of the project must be connected. If the `loyalty_*` tools are not visible, or a connection
error comes back instead of a status, the problem is the MCP connection, not the data: say that the promotion diagnostics
service could not be reached and suggest reconnecting the server (`/mcp` in Claude Code). Server errors that arrive as
text are listed in [GATEWAY.md § Server errors](references/GATEWAY.md#server-refusals): none of them is a reason to say "the service is down".

## Input

Three things are needed: the **project**, the **promotion number** and the **calculation** (`calculationId`, a GUID
like `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`) or an **order number** by which the calculation can be found.
**If something is missing — ask for exactly that and do not call the tools.**

| What came in | Project | Promotion | Calculation or order |
|---|---|---|---|
| **Explanation link**: `…mindbox.ru/processing-orders/debugger?calculationId=…&promotionId=…` | subdomain | `promotionId` | `calculationId` — no order needed |
| **Promotion link**: `…mindbox.ru/promotions/<number>` plus a value next to it | subdomain | the segment after `/promotions/` | a GUID is the calculation, otherwise the order number |
| **Values in order** | system name in Latin letters | a number like `40124947` | a GUID or an order number |

The project from the link is the `tenant`. The `https://` scheme, the order of query parameters and extra ones do not
matter; in a promotion link everything after the number (`/`, `?`, `#`) is dropped. A GUID in the input always means
`calculationId` — the order search is then skipped.

**Ambiguous input:**

- The explanation link has no `promotionId` — take the project and `calculationId` from it and ask only for the
  promotion number.
- The values are not in that order — parse them yourself: a Latin word without digits is almost certainly the project,
  a GUID is the `calculationId`. If two long numbers remain and it is unclear which is the promotion and which is the
  order — **do not guess**: ask one short question. With the wrong order number the calculation will be found for a
  different order and will give a confident but wrong answer.
- The tool does not search by promotion name: ask for the number — it is visible in the promotion link in the admin panel.
- A non-numeric promotion number (`someshop-528`) is an external discount that the calculation did not match to a
  promotion: take it as is.

## The server and the project

There is one connected Mindbox MCP server; every call takes `tenant` — the project's system name from the subdomain
of the admin-panel link. If the connected server offers `tenants_list`, use it once to confirm the project's system
name; otherwise ask the user for it. A project-not-available or access-denied message means the project name is wrong
or this MCP connection has no access to the project — check the spelling against the admin-panel link; if it is right,
Mindbox support. Details — [GATEWAY.md § The server and the project](references/GATEWAY.md#the-server-and-the-project).

## Steps

1. `calculationId` in the input — go straight to step 5.
2. `loyalty_find_order_calculations` with `orderExternalId`; on `ORDER_NOT_FOUND` with a numeric order number
   repeat with `orderInternalId` (exactly one of the two per call). Both empty — "In project <project>, order
   <number> was not found by your system's number or by the Mindbox number. Check the number and the
   project". Do not retry on other servers.
3. Several orders (the search covers all external systems): if the search was by internal number, take the order
   with that `orderInternalId`; if by external number, take the order whose system in
   `matchedByExternalIdSystemNames[]` is the one the user asked about. If the system is not named or there are
   several matches — ask which order is needed, in one sentence.
4. From `historyItems[]` take only the items with a `calculationId`. None — these are saves without a promotion
   calculation (see special cases). One — take it, no questions. Two or more — **do not choose for the user** and do not go on until
   they answer; the exception is when they already named the moment ("yesterday's receipt", "before the return", a
   date), then pick the item by `savedAtUtc`. The question is one sentence with a list of up to five of the latest saves in
   the project's time zone (Moscow time for Russian projects), the `isCurrent: true` one marked "latest", and the total
   if there are more. Sample: "The order was recalculated 3 times: 12 Sep 14:02, 12 Sep 18:40, 13 Sep 10:15 (latest).
   Which one do we look at?". No calculation ids in the question.
5. `loyalty_explain_promotions` with `tenant`, `calculationId` and the promotion number. Any status other than
   `EXPLAINED`, or "An error occurred…" — see [special cases](#special-cases).
6. On `EXPLAINED` read: `verdict` → `reason` with its `explanation` → `details` and `promotion` (promotion
   name) → `inputs` to confirm.

## Reading the verdict

The `verdict` **is already computed by the server** — do not derive it from the list of records. How the
server picks the kind — [VERDICTS.md](references/VERDICTS.md).

| `verdict.kind` | What to say |
|---|---|
| `AppliedVerdict` | the promotion applied and gave everything it computed. If asked "why so little" — there is no reason wording, the answer is looked up in the records: see "Why so little" below |
| `AppliedPartiallyVerdict` | the promotion applied, but not in full: what cut it — in the words of `reason.explanation` (promotion limit, the customer's balance, the POS's spend request, the product's minimum price, another promotion). This is the answer to "why so little"; which products — the `lineVerdicts` groups |
| `NoEffectVerdict` | "the promotion applied, but the benefit came out zero": the reason is in `reason`, each benefit has its own in its `benefits[]` row (no suitable products, no bonus points on the balance, the POS did not request a spend, line items cancelled). A benefit the trace is silent about has no row — say so |
| `NotSuitableVerdict` | the promotion did not suit the order; what exactly — in the words of `reason.explanation` |
| `BenefitConditionsNotMetVerdict` | the promotion suited, but a condition inside it was not met: threshold not reached, not enough for a step, no suitable products in the condition |
| `ExcludedByPromotionVerdict` | another promotion excluded it — `excluder`; only the first excluder is recorded; removing it does not mean the promotion will apply. The excluder's name comes only if that promotion was asked about in the same call |
| `ExcludedByMaxBenefitVerdict` | the calculation chose a variant without this promotion: the winner's composition is `winningVariant`, those that took its place are `displacedBy`. **`comparison` first**: `VariantsWereEqual` — the benefit was the same, the comparison order decided, and "lost to a more profitable one" is untrue here; the `WinnerWasBetter` numbers are variant totals for the whole order, not the promotion's contribution |
| `NotCandidateVerdict` | the promotion was not in the calculation. By `details`: `NOT_FOUND` — the project has no such number or the promotion has not been launched yet, `NOT_A_PROMOTION` — not a promotion number (external discount or a typo), `AVAILABLE` — the promotion exists but did not get into the calculation |
| `UndeterminedVerdict` | the calculation left no verdict on the promotion — the reason cannot be named, escalate |

**The wording of the reason comes only from `explanation`**: it arrives ready on the verdict's `reason`, on the
`benefits[]` rows, on the `caps[]` and on the `lineVerdicts` groups. Do not invent wording, do not call neighbouring
records the reason, do not read more into the text than it says: "the promotion's discount amount limit cut the amount" is
about the limit of this promotion, not another discount. The promotion name is not a reason, even if it sounds like a
diagnosis. If there is no `explanation` — say there is no wording for this reason yet, name the facts from `params` and
escalate. Translation and links — [HUMAN_STYLE.md § Wording of the reason](references/HUMAN_STYLE.md#wording-of-the-reason).

**"Why so little".** On `AppliedPartiallyVerdict` the reason is `reason.explanation`; the promotion's total is
`amounts` (`requested*` next to it — what it would have given without caps), the benefit's total is the amounts of its
row, what cut the amount before spreading across products — the row's `caps[]`, and which products got less or nothing
— the `lineVerdicts` groups with their own wording; name the product by `productName`. On `AppliedVerdict` there is no
reason wording: every benefit gave what it computed. Then, as in the promo code branch, read `loyalty_reason_codes`
once and look at the promotion's records `SpreadShortfall`, `LineDiscountCapped`, `LineDiscountZero` and the shared
`BalanceSpendRequested` ([RECORDS.md § Application and caps](references/RECORDS.md#application-and-caps)):

- the kind `RequestedAmount`, or the POS's request in `BalanceSpendRequested` smaller than possible — as
  many points were spent as the POS asked for: this is not a cut but a question for the POS;
- `ZeroPriceAfterDiscounts` — the product was already zeroed by promotions applied earlier;
- rounding and spreading across products — cents, do not name as a separate reason.

If there is nothing of the kind — the gap from what was expected comes from the promotion settings (how much and on
which products): send the user to the marketer who set up the promotion. An empty `caps` does not mean "not cut". Do not analyse the other `decisions[]`
records here; if the chain is needed, that is a technical breakdown, outside this skill.

**A question about a promo code.** The reason of a promo code verdict comes as ready wording: on `PromoCodeNotSuitable`
the kind already says what is wrong with the coupon (used, blocked, from another pool, the pool is still a draft). The
coupon itself, as passed into the calculation, has separate records ([RECORDS.md § Selection](references/RECORDS.md#selection)):

- the coupon applied to the promotion — `CouponApplied` in this promotion's `decisions[]`;
- the coupon did not apply — `CouponNotApplied` in `sharedDecisions`: the kind is the coupon status (used,
  blocked, period not started or expired, not issued, not found, state cannot be determined, received via a
  friend's invitation), own referral code, issued to another customer, not passed in this order, failed to
  load, usage limit exhausted, or "checks passed" (`CouponChecksPassed` — the coupon is fine, answer by the
  promotion's verdict).

These records have no wording: in this branch read `loyalty_reason_codes` once and take the text by the key "record
name _ kind". Never output the promo code or the coupon's internal id — "the customer's promo code". The absence of a
coupon record does not prove there was no coupon: calculation-level records are truncated.

**The promotion name is for reference**: from `promotion.name`, so that the person understands which promotion is
meant. Draw no conclusions from it: "20% off" promises neither the discount size nor the conditions.

## How to answer

A clarifying question and a refusal message (no order, no access, no project) — one or two sentences: what is needed
and why, without an offer to "show the numbers". On `EXPLAINED` the tone, the two modes and the prohibitions —
[HUMAN_STYLE.md](references/HUMAN_STYLE.md), who the last sentence sends the user to — [ESCALATION.md](references/ESCALATION.md),
units of numbers — [DECISION_FIELDS.md](references/DECISION_FIELDS.md).

**The default is the short answer**: 2–3 sentences of running text, one reason in everyday language and who to go to.
No numbers, except the promotion name and — if it is the reason itself — one amount. The detailed mode — **only on
explicit request**. No codes, stages, ids or JSON in either mode: a technical breakdown with codes and stages is
outside this skill — tell the user to ask for it separately, with the project, the calculation id and the promotion number.

## Special cases

**A save without `calculationId`.** Say that the loyalty calculation did not run for this save, so nothing can be said
about the promotion; this happens with imports and with saves that do not trigger a calculation. Go to the integration team, or take
another save or order that has a calculation.

**`TRACE_NOT_STORED`** — four different answers by `calculationAge`
([MCP_CONTRACT.md § `TRACE_NOT_STORED`](references/MCP_CONTRACT.md#trace_not_stored--four-answers)). Most
often `TOO_FRESH` — "try again in a minute", not "there is no data".

**`TRACE_UNAVAILABLE`** — read `reason`. With the words "no longer read" the calculation is stored in an old format:
"This order was calculated before the diagnostics update, and it cannot be explained — retrying will not help. Take a
more recent order with this promotion". Otherwise the storage did not respond: retry later; if that does not help —
Mindbox support.

**`BAD_REQUEST`** — both order ids at once, an empty promotion list, a calculation id that does not look like an
identifier: fix the request and retry, do not tell the user. **`TENANT_NOT_FOUND`** — handle as the project-name error.

**`UNAVAILABLE`** (order search) **or `TENANT_UNAVAILABLE`** (explanation) — the project has no admin panel configured
or the order service did not respond (`reason` tells which): retry before reporting unavailability.

**"An error occurred. Please try again or ask support."** instead of an explanation — retry once; on a repeat
error call `loyalty_reason_codes`
([GATEWAY.md § The explanation fails as a whole](references/GATEWAY.md#the-explanation-fails-as-a-whole-a-new-kind-in-the-service-response)).
If the catalog does not respond — it is access or the service: Mindbox support. If the catalog responds — the
service returned a verdict kind for this promotion that the MCP server does not know yet. To the user: "The
tool cannot show the verdict on this promotion right now. Pass the explanation link or the order number to
Mindbox support". Do not guess the reason beyond that, do not mention access, do not put the calculation id
in the text: the admin-panel link already carries it.

Mindbox support: the user's usual support channel; do not invent a channel name. Interpretation traps —
[PITFALLS.md](references/PITFALLS.md): unavailable counters do not mean an exhausted limit, a missing record does not
prove the event did not happen, amounts are in the project's currency (rubles for Russian projects).

## Check before answering

Before sending the answer, go through it.

Facts:

- the reason is taken from the verdict's `reason` (for a partial one, the top-level `reason`, not the per-row wording), not fished out of the
  list of records;
- "X did not happen", "the segments are fine" — only where a record or a field confirms it;
- "the limit is exhausted" — only where the reason itself says so, not unavailable counters;
- the POS's spend request is not called a cut, an empty `caps` is not called proof that nothing was cut;
- no amount is divided by 100; a zero in `lineKey` did not become "line 0", a zero in `blockOrdinal` was not
  thrown away;
- the excluding promotion is named the first excluder, not the only one; on `VariantsWereEqual` there is no
  "lost to a more profitable one";
- on `TRACE_NOT_STORED` the `calculationAge` is read before anything is said, on
  `TRACE_UNAVAILABLE` the `reason` is read;
- nothing is derived from `state` and the promotion dates (they describe now, not the purchase) or from an
  `inputs.limits[]` row with a different `promotionHistoryId`.

Wording:

- the wording is a translation of the catalog text per the `HUMAN_STYLE.md` dictionary, not the English original
  verbatim; any link keeps its original title and address;
- the text has no codes, stages, calculation ids or the words "trace", "snapshot", "verdict";
- the text has no promo code and no coupon internal id — a prohibition without exceptions.

Who to send the user to:

- the last sentence names who to go to, per [ESCALATION.md](references/ESCALATION.md);
- `NotCandidateVerdict`, `UndeterminedVerdict`, `NOT_FOUND` and `NOT_A_PROMOTION` are not merged — each has
  its own addressee.

Found a mismatch — fix the answer and go through the list again. If it still does not add up — do not give a confident
answer: name what remained unclear and escalate to Mindbox support.
