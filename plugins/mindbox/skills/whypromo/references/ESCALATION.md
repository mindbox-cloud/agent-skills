# Who to send the user to

An answer without an addressee is useless: the user learns the reason and does not know what to do next. The
last sentence of the explanation always names who fixes it. The rows below go by the meaning of the reason, not
by the record name: the meaning arrives as the wording in `explanation`.

## Reasons from the calculation

| What the calculation showed | Who to send to | What to ask them to check |
|---|---|---|
| Products, categories, segments, amounts, thresholds, dates, days and hours of the promotion | the marketer who set up the promotion | the specific setting that did not match |
| The promotion limit on discount amount, accrual or number of applications is used up | the marketer who set up the promotion | the promotion limit and its period |
| Group discount cap or minimum price of a promotion tree group | the marketer who configures the promotion tree | the group setting: it is not a setting of the promotion itself |
| The promotion is not in the project's promotion tree | the marketer who set up the promotion | whether the promotion is added to the tree |
| One product matched several product conditions of the same promotion | the marketer who set up the promotion | the conditions must be separated: while this product is in the order, the promotion does not work |
| The promotion's promo code pool is still a draft | the marketer who set up the promotion | whether the promo code pool is published |
| A product from the order is not in the project's catalog | the catalog integration team | whether the product is exported to Mindbox: the promotion settings have nothing to do with it |
| The product field used for price or accrual is empty, has several values or is not greater than zero | the catalog integration team | how the field selected in the promotion is filled |
| The product's minimum price came in the order itself | the POS or integration team | what minimum price the POS sends |
| Order lines are cancelled | the POS or integration team | whether the line status is sent correctly: cancelled lines do not enter the calculation |
| The customer is not attached to the calculation, the calculation ran as anonymous, an odd operation type | the POS or integration team | how the order is sent to Mindbox |
| The operation calculated only bonus points, and the promotion is not about bonus points | the POS or integration team | which calculation operation the POS sends |
| The POS did not request a spend, requested zero, requested another account or only the available amount | the POS or integration team | which spend request the POS sends |
| Saving the order did not run the loyalty calculation | the integration team | whether this scenario triggers the promotion calculation; whether final prices come from an external system |
| A discount from the POS has the same number as a Mindbox promotion | the integration team | which discount numbers the external system sends: in this case it is the Mindbox promotion, not the POS, that grants the discount |
| An external promotion raised the line price | the integration team | which discount the external system sends |
| The promo code was not issued to the customer, is inactive, already used or blocked | the team that issues promo codes | whether the coupon was issued to this customer, whether the pool is active |
| No bonus points on the balance, or the balance is taken by another promotion of the same calculation | nobody | this is a fact about the customer's balance, nothing to fix |
| The store has no time zone set | Mindbox support | the time zone of the point of sale |
| Limit counters could not be read | Mindbox support | that the limit could not be checked, not that it is used up |
| The promotion number is not found in the project (`details: NOT_FOUND`) | whoever gave the promotion number | the number: in the admin panel it is visible in the address of the promotion card; a promotion that has never been launched answers the same way |
| The value passed is an external discount, not a promotion (`details: NOT_A_PROMOTION`) | the integration team | how the external system sends discounts in the order; such a discount has no promotion settings in Mindbox |
| The promotion did not enter the calculation (`NotCandidateVerdict` with `details: AVAILABLE`) | the marketer who set up the promotion | why the promotion did not enter the calculation of this order: the response does not show it, do not name a reason |
| The calculation did not close the promotion (`UndeterminedVerdict`) | Mindbox support | the calculation id and the promotion number |
| `failureKind` starts with `Unexplained` | Mindbox support | the calculation id and the promotion number: this is an instrumentation gap in the calculation, not a promotion setting |
| No wording for the reason: `explanation` did not arrive and the catalog has no such key | Mindbox support | the calculation id, the record name and its `failureKind`; do not guess the meaning from the name |
| The decision record is missing where it was expected | Mindbox support | the calculation id: looks like a lost record |

## Tool and access

| What happened | Who to send to | What to ask them to check |
|---|---|---|
| Trace in the old format: `TRACE_UNAVAILABLE` with "no longer read" | nobody | this calculation cannot be explained; take a calculation newer than 18 Sep 2026 |
| The explanation fails entirely on one promotion while the catalog responds (the service response carries a kind the Mindbox MCP server does not know yet) | Mindbox support | the project name, the calculation id and the promotion number |
| `TRACE_UNAVAILABLE` with a storage failure | Mindbox support | if a retry after a few minutes did not help |
| "An error occurred. Please try again or ask support.", and the catalog does not respond either | Mindbox support | the project name and that the retry did not help: a failure calling the promotion diagnostics service, or the connection has no access to the diagnostics service |
| `loyalty_*` tools are not visible, or the Mindbox MCP server does not respond even after reconnecting (`/mcp` in Claude Code) | Mindbox support | the project name and the error text |
| The project is not available or access is denied | whoever gave the project name; if the name is right, Mindbox support | the spelling of the project's system name against the admin-panel link; if it is right, this MCP connection has no access to the project |
| The project's data must not be passed to AI agents (`FORBIDDEN`) | nobody | the ban cannot be lifted: this project cannot be explained |

**Mindbox support: the user's usual support channel; do not invent a channel name.**

## Common addressing mistakes

- **Counters unavailable → the marketer.** Wrong: the marketer will see the configured limit and shrug. This is
  Mindbox support.
- **Anonymous calculation → the marketer.** Wrong: all audience promotions were cut off at once because there was
  no customer in the calculation. This is the POS or integration team.
- **Product not in the catalog → the marketer.** Wrong: a product can be neither in a segment nor in a list while
  it is not in the project's catalog. This is the catalog integration team.
- **"The promotion did not enter the calculation" → "it is not in the promotion tree".** Wrong: absence from the
  tree has its own record and its own verdict. Here the reason is unknown, and the addressee must be told the
  fact itself, not an invented explanation.
