# Orphans, wiring, reading a block back, and repairing one that reads back wrong

The mechanism behind three `SKILL.md` rules: orphans, the two halves of the diff, and repairing a
block.

- [Orphans and wiring](#orphans-and-wiring) — read at step 1(c)
- [Building the operations](#building-the-operations) — read before your first batch of writes (step 4)
- [Reading a block back](#reading-a-block-back) — read before your first read-back diff (step 6)
- [Repairing a block that reads back wrong](#repairing-a-block-that-reads-back-wrong) — read when one does

## Orphans and wiring

**An orphan is a block with no incoming edge** — which is *not* the same as a block you cannot
reach from the start block. It is not a harmless leftover: it **fails validation**.

**The write path does tell you — but only that it happened.** Every write answer carries a
validation list naming each unwired and each unfilled block, **by block id**. It is not a refusal: a
batch that comes back carrying entries was applied, and an empty list is how you see the last of them
cleared — so it is a progress meter across a run of batches, and worth reading after every one.

It says a block is unwired, never where the edge belongs, and it clears on **any** edge
(`writing.overview` → `validationErrors`) — so decide the wiring **before** the batch, which is also
what keeps a deletion in the same batch as the rest.

Three things about that list, each of which has produced a wrong answer:

- **It counts incoming edges only.** A detached *pair* of blocks passes it, because the second
  one's edge comes from the first. So a clean list is not evidence that every block runs, and
  resolving an orphan means wiring it **into the graph the start block feeds** — not merely giving
  it an incoming edge.
- **Only a whole-flow read carries wiring and orphans**; a scoped read prints them empty either way
  (`writing.verification`).
- **Deleting the edge is not deleting the block.** And some block types cannot legally sit at the
  end of a flow, so "re-wire it somewhere" is not always available.

**A block you were never asked about is still yours to resolve** — wire it into the graph the start
block feeds if the design wants it, delete it if not. Leaving it is not neutral.

## Building the operations

Fill a **simple block at creation**, in the same create operation; defer a **block whose settings are
many interdependent fields to step 5**. **Wire the edges from the reference's output vocabulary**,
not from a read-back — a block you just created has nothing to read there. **Check every dead end is
intended:** executions on a branch with no outgoing edge leave the flow. **A flow-version setting has
its own operation** and is never smuggled into a block body, where it will not be written and a
read-back can appear to confirm it.

**Every opaque fragment in a body goes in complete**, built from the block's pre-write read plus your
change: a fragment is replaced whole, so a member you leave out lands at its default and a missing
required member gets the whole call refused (`SKILL.md` CRITICAL 3). A top-level field you simply do
not name is left alone.

**Carry the row version forward** — step 1(c)'s for the first write, each write's answer for the
next. A **refused** call applied nothing, so fix the cause and re-send rather than re-reading. Two
exceptions have rows in `references/troubleshooting.md` → *Writes*: a **version conflict**, and a
call that **never answered**. The retry budget itself is `SKILL.md` step 5's, and only its.

## Reading a block back

This is how half (b) of CRITICAL 4's diff is read.

**A read can print the as-sent shape or the platform's own projection** — resolved entity objects,
server-assigned ids. Neither means a loss, and the platform's members are not writable
(`writing.read_vs_write` → *Two read-back shapes*). Members you *can* write — a filter, a wait, a set
of conditions, a name — are still yours to check, and their loss is still a failed write.

**A member the write contract says the read path overrides** — the send step's mode on a draft
mailing is the documented case (`steps.send_mailing`) — comes back changed by design: it is not a
CRITICAL 4(a) mismatch, so do not repair it, and report it as that document says.

**Which members are writable is the write contract's to say.** An absence of one that is not writable
is never a failed write, and repairing it spends a write and a budget slot on a correct block.

**A `flows_validate` between the two reads does not spoil them.** The call leaves every block's members
as they were — only the row version moves — so a read taken before a validation is still a valid
baseline for half (b) of a write made after it (`SKILL.md` step 7).

**The *create* write has no half (b)** — the block held nothing. **Every later write to that block
owes it**, whether or not you created the block: the start block whose filter went in at creation and
whose timing you set afterwards is exactly that case. With no pre-write read, half (b) has not been
run and the write is not verified. Both failure branches are in `references/troubleshooting.md` →
*Reading back*.

## Repairing a block that reads back wrong

**Match the symptom first.** Several ways a read-back can disagree are documented in the `flows`
domain's known-issues sub-folder, each with a recovery of its own that ends somewhere the two
routes below do not: a block that came back **unchanged**, a value that came back **different**
from the one you sent, a value that will not change however you send it, something you never sent
that has **gone missing**. Match what you observed there and take that recovery — it replaces the
routes below and decides when to stop (`SKILL.md` → `## Where to look`, step 5). Guess here and the
likely guess is a deletion, which costs you a block that was fine.

Only if nothing matches, take the two routes in order (they are `writing.verification`'s), and stop
only after both — except for a stored filter, which has its own single re-send (`SKILL.md` step 6).

**Route 1: repair in place.** Re-send the block alone with a **complete** body and complete fragments
(`writing.verification`) — that is what makes it a corrected re-send rather than the same body twice.
Then read it back again. If it still disagrees, **match the symptom again** before going on — a
repair that leaves the block **unchanged** is itself a documented symptom.

**Route 2: delete and recreate**, reading back once more. Deleting does not depend on the block's
contents being readable, so this route stays open when the first one fails. A start block's delete and
its replacement's create go in **one** batch (`writing.overview`). The pre-delete read is `SKILL.md`
step 5's obligation and it is what the recreate is built from.

**This route is available once per block.** A second delete-and-recreate of the same block is not a
further step in the ladder; it is the loop `SKILL.md` CRITICAL 6 stops, and each of these calls counts
against that session budget.

**The session budget can also stop you before both routes are finished** — report the block as
unverified either way.

**Only if the block still reads back wrong after that: stop.** Report exactly which block, which
field, and what you tried. Do not move on, and do not describe that block as built. **If the
disagreement is an objection naming something no write of yours can send, neither route applies at
all** — the body is not the cause, and CRITICAL 6 says what to do instead.

# References

- `SKILL.md` → step 1(c) — the pre-write read this page's orphan check belongs to
- `SKILL.md` → CRITICAL 4 — the two halves of the diff this page's *Reading a block back* serves
- `SKILL.md` → step 6 — the verification step
- `SKILL.md` → CRITICAL 6 — the session budget, and objections a rewrite cannot close
- `references/troubleshooting.md` — one row per situation, including the read-back rows
