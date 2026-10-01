# Validating the flow version

Read at step 7, and again at 7b — the mailing edits reopen validation.

- [The call's own failure modes](#the-calls-own-failure-modes)
- [Sorting what it reports](#sorting-what-it-reports)
- [A change asked for after a validation that passed](#a-change-asked-for-after-a-validation-that-passed)

## The call's own failure modes

A refusal; an answer **naming nothing at all** (validation did not run); a stale-token claim handing
back the token you already hold. **None of them is about your flow, and none is a licence to loop.**
Never delete a block on suspicion — a deletion made on a guess costs you a block that was fine and
leaves the real cause in place. One row each in `references/troubleshooting.md` → *Validation*.

**A second call on a version that is *still* passing is a silent no-op**: it returns the same success
line and the same row version, with nothing to tell it from a fresh pass — so never offer a repeated
identical success as new evidence. After any further write, or on a version that came back with
problems, validating again is the step, and that call is a **real** validation: **any block operation
returns a passing version to in-development**, which is what reopens it.

**The call does not re-shape the version's blocks**, pass or fail — the reads you are holding stay
valid baselines, and only the row version moves (`SKILL.md` step 7).

## Sorting what it reports

Three kinds, and the difference decides whether you write anything at all.

1. **Expected gaps** — what you already knew you could not author: an id no tool reaches, a
   condition the flow's scope set cannot express. **Do not retry and do not invent them away.** Carry
   them to step 8 as what the flow still needs.
2. **Problems that are yours to fix** — a malformed block, a missing required member, a dead end you
   did not intend, an unconnected block. Fix it, verify in step 6 and validate again — the version
   did not pass, so this is not the no-op. **Take a pre-write read before the fixing write** (step 5).
   The loop runs inside CRITICAL 6: what means stop is the *same* objection returning unchanged.
3. **A problem that is yours and that you could not write** — a value you had to stop on under
   CRITICAL 2, or an objection naming a member no write reaches (`SKILL.md` → CRITICAL 6). Do
   not retry it and do not invent a value to silence it. Carry it to step 8 **ranked**: if it is the
   user's headline requirement, say plainly that the flow will not run until it is set, as a gap on
   your side.

**If you cannot tell which kind a message is, report it verbatim rather than guessing.** And **do not
assume a problem tells you where it is:** if it names no block, match it yourself from what you
wrote — but if **more than one** could be the subject, do not pick one. Report it verbatim, name the
candidates, and leave it to the user. Guessing is how you rewrite a block that was fine.

## A change asked for after a validation that passed

Re-read the flow for the current row version, and take step 5's full-detail read of every block you
will touch — the validation did not invalidate the reads you already hold. **Validating does
not by itself put the version out of reach**; which statuses accept writes is the write path's to
state. If a write *is* refused as not editable, say plainly that the change could not be applied to
this version, and stop.

If it goes through, **verify it in step 6 and validate again**: what stands behind the flow is the
validation after your last write.

# References

- `SKILL.md` → step 7 — the ordering rule and the ban on offering a pass as evidence
- `SKILL.md` → CRITICAL 6 — the session budget these failure modes run inside
- `references/troubleshooting.md` → *Validation* — one row per situation
- `references/hand-over.md` — where the three kinds end up
