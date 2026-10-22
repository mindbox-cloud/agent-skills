---
name: analytics
description: >-
  Answer questions about a project's analytics with figures from the analytics datamarts.
  Use when asked for a figure, why a figure changed, how campaigns or flows performed, or for a
  summary of a period.
  Russian triggers: "сколько выручки", "почему упала выручка", "итоги рассылки", "как сработал сценарий",
  "сводка за месяц".
  English triggers: "how much revenue", "why did revenue drop", "how did the campaign perform",
  "summary for last month".
  Don't use when: the request is to build or change a flow, a mailing or a filter — those have their
  own skills; the user needs a list of customers rather than a figure.
metadata:
  version: 1.0.1
---

# Analytics

Answer a question about one project with figures from its analytics datamarts. What a table holds and how to read it is in its description; the platform rules behind the figures are in the wiki. This file sets the shape of the answer and its limits.

## Rules

1. Answer the question that was asked — no more. Its size sets the size of the answer; if the user asks for more detail or a particular format, follow the user.
2. Every figure comes from a tool call in this conversation. If the data has no answer, say so.
3. Before you present a figure, check it: parts add up to the whole, periods are of equal length and complete, values are plausible.
4. A cause is a finding only when the data shows it. Look for it in the data first; what you could not check, call possible.
5. Say what the figure covers — brand, target action, period — and what it leaves out.
6. Use the user's words: metric names as in the interface, report names as in the admin menu in the user's language (the localized wiki index gives the pairs), names of objects instead of keys, and an object's name as a link to its page when a tool gives you one; no SQL, table or column names unless the user asks how it was counted.

## Format

Answer in the user's language: conclusion with the figure → what it is made of, as a table when comparing periods or parts → causes, if the user asked why → one next step, if there is an obvious one you can carry out → the report from the admin menu with the same filters, as the last line, if the answer has figures — as a link: the admin address comes from the `admin_link` tool and the report path from the wiki, with the same filters; without `admin_link`, by name and filters; never guess the address.

Example:

> Mailing revenue for September was 4.2 million, 12% less than in August (brand Main, target action "Orders").
>
> | | August | September | Change |
> |---|---|---|---|
> | Bulk email | 2.9 million | 2.2 million | −24% |
> | Automated | 1.9 million | 2.0 million | +5% |
>
> A third fewer bulk emails were sent than in August: 1.2 million against 1.8 million. The largest of them, [Autumn sale](https://<project-domain>/newmailings/email/…/view), brought 0.6 million.
>
> Break it down by week?
>
> [Campaigns](https://<project-domain>/dashboards/mailings/?…)

## Limits

A benchmark or a "good / bad" threshold is either the product's own — from a tool or a table description, such as the email health report — or an outside one with its source named and marked as not the product's data. Without a source you can name, give no number.

## Every call

In `reason`, the source is `[analytics]` and the version is `v1.0.1`. Fill it in every call, in this order: `[analytics] v1.0.1 | <user's question> | <what this call fetches>`.

- User's question: what the user asked in their latest message, close to their wording, in the user's language, up to 300 characters. Leave out names, emails, phone numbers, addresses and customer ids.
- What this call fetches: 5–15 words.

## Feedback

When you conclude the user's task cannot be resolved — the data is missing or the tools refused, or the user disputed your result and the cause is in your calculation or was not found — call `feedback` with `type=task_not_resolved` once, as soon as you conclude it and before you reply to the user. In the text: what the user asked, what you calculated and why the task stayed unresolved; if the user disputed the result, what they compared it with — the report, the metric label, the period and the number shown there.

If the user's own comparison explains the difference and the user accepts your explanation, or you corrected your error and the user accepted the corrected answer, the task is resolved: do not send it.

An obstacle with a tool — an unclear description, a missing capability, a workaround you had to invent — goes to `feedback` with `type=tool_problem`, as the server instructions ask.

If `feedback` has no `type` parameter on this connection, send it without one. Do not include secrets or personal data.
