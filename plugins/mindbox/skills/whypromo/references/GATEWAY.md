# The Mindbox MCP server: the project and refusals

The tools are exposed by the connected Mindbox MCP server; the data in the response is produced by the
promotion diagnostics service. This file is about everything that happens before the service and on the way
back: how to pass the project, how the server refuses, and where the server itself loses the response. What
the response fields mean — [MCP_CONTRACT.md](MCP_CONTRACT.md).

## The server and the project

There is one connected Mindbox MCP server: the server of the client's project. The plugin declares no MCP
servers of its own. If the `loyalty_*` tools are not visible, the MCP connection is not set up or has
dropped: say that the promotion diagnostics service could not be reached and suggest reconnecting the server
(`/mcp` in Claude Code).

The project is the `tenant` argument of every call: the project's system name in Latin letters, taken from
the subdomain of the admin-panel link. The server normalizes case itself; any other deviation in spelling is
rejected. The optional `reason` argument is free text — "why this call"; it does not affect the response.

## Response shape

The tool response is JSON that the server assembled from the service response:

- the kind of a variant object is in the `kind` field, as the type name (`AppliedVerdict`, `BenefitApplied`,
  `MindboxPromotion`);
- enumerations are UPPERCASE with underscores (`EXPLAINED`, `TOO_FRESH`, `AVAILABLE`); string fields of
  records (`stage`, `outcome`, `scope`, `code`) are as the calculation engine wrote them (`Benefit`, `Applied`,
  `Line`);
- fields without a value are not written at all: a missing key is "none".

A refusal arrives in one of two ways: as a call error, where the client adds the prefix
`An error occurred invoking '<tool>': …` (match on the text after the prefix), or as the plain text of the response.

## Server refusals

| Text | What it is | What to do |
|---|---|---|
| `Tenant '<name>' is not available: it either does not exist or has not granted consent…` | the project name is wrong **or** this MCP connection has no access to the project; the text does not distinguish them | check the spelling against the admin-panel link; if it is right — Mindbox support |
| `Unknown tenant '<name>'…` | the name failed the server's check | as the row above |
| `Access denied.` | this MCP connection has no access to the project | check the spelling against the admin-panel link; if it is right — Mindbox support |
| `This environment has no promotion diagnostics service…` | the promotion diagnostics service is not available on the connected server | Mindbox support; retrying will not help |
| `FORBIDDEN. This project's data must not be passed to AI agents…` | a platform ban; no access setting lifts it | say it plainly and stop; do not look for workarounds |
| `Authentication failed.` | the server did not accept the token | reconnect the MCP connection (`/mcp` in Claude Code) |
| `Service temporarily unavailable. Please retry.` | the server could not issue a token because of a network failure | retry |
| `An error occurred. Please try again or ask support.` — **as the response text** | a failure on the way to the promotion diagnostics service | [next section](#an-error-occurred) |

A connection or authorization error instead of any response points to the MCP connection (not set up, the
session dropped), not to the data. An argument validation error (wrong type, an extra or missing parameter) — fix
the call.

## "An error occurred…"

The server writes this text on any failure of the call to the promotion diagnostics service and does not say
which one:

- network, timeout, an error of the service itself;
- the connection has no access to the diagnostics service;
- the service response carries a kind the server does not know yet —
  [next section](#the-explanation-fails-as-a-whole-a-new-kind-in-the-service-response); such a failure
  repeats on the same arguments any number of times.

Do not read it as "no data": the project and the calculation may be fine.

## The explanation fails as a whole: a new kind in the service response

The server reads the response of the promotion diagnostics service by its own snapshot of the service schema.
When the service starts returning a new kind of verdict (`verdict.kind`) or benefit status (`status.kind`)
that is not in the snapshot, the server cannot read the response and returns "An error occurred…" **for the
whole call**, for all promotions at once, until the snapshot is updated. This happened with the partial
verdict (`AppliedPartiallyVerdict`, `BenefitAppliedPartially`, `BenefitExcluded`). Today the server reads
every kind from [VERDICTS.md](VERDICTS.md); this section is for the next one.

What to do when `loyalty_explain_promotions` answered "An error occurred…":

1. Repeat the same call once: the failure may have been a one-off.
2. Error again — call `loyalty_reason_codes` on the same server and project. If it answered — access and the
   service are fine (all three tools share one access check), the problem is in the response about these
   promotions. If it also errors — it is access or the service: Mindbox support, go no further.
3. Several promotions in the call — explain each one with a separate call. Those that answered are explained
   as usual.
4. About a promotion on which the call fails consistently, say: the tool cannot show the verdict for it right
   now. Do not guess the reason. Escalate to Mindbox support with the project name, the `calculationId` and the
   promotion number ([ESCALATION.md](ESCALATION.md)).

The catalog read at step 2 is useful later — do not call it a second time.

## `tenants_list`

If the connected server offers `tenants_list`, use it once to confirm the project's system name; otherwise
ask the user for it. No arguments. Needed when the project name is ambiguous or the server answered that the
project is not available; in all other cases the name is taken from the admin-panel link.
