# Feedback and the self-check

Read this at the first docs mismatch, at step 7c, and when the user accepts the result (step 8).

## The feedback message

`feedback` is on the same connection as the flow tools; take its name from your tool listing. Its one
free-text field carries all the structure (any other argument its schema asks for is this build's
project). **No such tool — skip silently:** it is telemetry, never a shortfall, never told to the user.
First line the literal marker, then one `key=value` per line; `skillVersion` is `SKILL.md`'s
`metadata.version`. Never a secret, personal data, a filter body or a customer's value.

## A documentation mismatch

**The moment you establish** that live behaviour contradicts the wiki or `SKILL.md` — a recovery that does
not reproduce, a field said to be refused that was stored, a misdescribed schema — send, unasked, unnarrated:

```
[skill-feedback:v1]
skill=flow-create
skillVersion=<metadata.version>
source=agent-observation
kind=docs-mismatch
doc=<the wiki id, or the SKILL.md or reference file § section, that is contradicted>
expected=<what the doc says, one line>
observed=<what actually happened, one line>
calls=<in order: tool name + a one-line argument summary, per call that establishes it>
```

At most **three per session**, one per distinct mismatch: feedback spends the client's tokens, so it
is capped, but one slot can be too few. It replaces no handling — the live contract still wins.

## Rating the build

**Once per conversation thread, after the user accepts** (`references/hand-over.md` → *Actions*, last
item; several flows: once all are accepted), one unobtrusive line in their language asks for a 1-to-10
rating, flows and mailings together. Never earlier; asked once — an unanswered ask is spent, not
repeatable, even when a later build starts in the same thread. **A number** → send:

```
[skill-feedback:v1]
skill=flow-create
skillVersion=<metadata.version>
source=user-rating
score=<1-10>
comment=<their words verbatim; if long, a faithful one-line retelling; omit the line if none>
flows=<the flow ids built this session>
iterations=<the final metadata.iteration value>
```

**No number** (silence, refusal, words alone) sends nothing, is never pressed for, blocks nothing.

## The self-check

After step 7b's final clean validation, launch **one sub-agent per flow built this session** on the
read-only structural audit skill (it checks a flow for known anti-patterns; name from your skill
listing, "use that skill" as the first sentence, per-flow mode, the flow id and version). No such skill,
no check. It reads no wiki and lacks this session's context: **its findings are leads, not verdicts**.

- **Contradicts a decision the user made explicitly, or a documented platform behaviour** → drop it.
- **Names a real risk the user has not decided on** → into the hand-over, plainly, in their language:
  *Actions* if it needs their decision, *Suggestions* if optional. Never a separate report, a check
  number or the audit's vocabulary.

**Nothing survives → say nothing about the check.** No re-runs; a failed or unusable sub-agent is skipped
silently (a mismatch it exposed still goes to feedback). Read-only: no write budget, no reopened validation.

# References

- `SKILL.md` → step 7c, step 8, `## Where to look`; `references/hand-over.md` → *The three parts*
