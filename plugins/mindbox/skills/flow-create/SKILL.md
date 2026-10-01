---
name: flow-create
description: >-
  Build a correct draft marketing scenario (flow) from a plain-language request: read the
  flows reference, resolve the entities the design needs to real ids, create and fill the
  blocks, wire the outputs, verify every block by reading it back, and hand over a link.
  Use when asked to create, build, assemble or set up a new flow, or to add blocks to an
  existing draft; when asked to move, transfer or migrate a mechanic or a flow from one project
  to another, whether one or a batch of them; when asked for several flows in one go; and when
  asked to rebuild here the logic a client is moving off an external marketing-automation
  platform — in whatever language the request comes in, and whatever the reader calls
  the thing: a flow is also a scenario or a journey, a send step is a mailing or a campaign,
  an entry filter is an audience or a segment.
  Don't use when: the flow already exists and only needs describing or checking — this skill
  builds, it does not report; the request is to launch, stop or delete something, which stays
  a human decision.
metadata:
  version: 1.0.0
---

# Flow create

Turn a plain-language request into a correct draft flow. The hard part is getting every block's
properties, field names and entity ids right — and *knowing* you did; this file carries none of
them (`## Where to look`).

You fetch and write everything yourself with the flow tools. Two things are **not** yours to invent:

- **Filter conditions** — the predicate inside a condition block, and the entry filter on a start
  block. They are authored by the **filter-building skill, which you run in a sub-agent**; take its
  name from your skill listing, never from memory. Never compose, edit or retype a filter body, and never
  resolve the entities inside one. A body is only usable if the answer shows that skill produced it
  (`## Filters and entity scope`).
- **Marketing mechanics.** Before you design, read the `flows` domain's mechanics sub-folder. Where
  one matches, it is the design unless the user says otherwise — ask about any departure in the
  step-3 batch; where none matches, say so and build only what the user describes (step 3). Never
  improvise one (`references/mechanics.md`).

**Never launch, pause, stop or delete a flow, and never call a tool that does.** Creating a flow, reading,
applying operations and validating are yours, and you must not claim to have started one. **Testing mode —
the platform's test run — is the one exception, and only at the end:** you recommend it, and call
`flows_set_testing_mode` only after a clean validation, after the user has seen and accepted the result, and
on their explicit yes (step 8) — short of that it is as forbidden as a launch.

**Done** = a verified draft, plus a ranked account of what is missing — and stopping short is also
done (CRITICAL 6). Expect part of a routine request to be out of your reach, and find out while you
are still planning writes; validation is not your checklist.

**Rank what is missing; do not just list it.** Two kinds, not comparable: **a small human follow-up**,
and **a shortfall in the user's headline requirement**, where the flow will not do what they asked or
will not run at all. **Write each one down with its kind the moment you hit it**, from step 2 onward:
step 8 needs it and cannot reconstruct it (`references/hand-over.md`).

## CRITICAL

**1. Read the reference before you write. Every time.**
Not from memory, not from a previous session, not by analogy with another block type. The write-path
sub-folder before your first write; the block document for every type you touch. A field name or a value
you recall is a guess wearing a fact's clothes.

**2. Never guess a value.**
If the reference does not give it, you do not have it. Distinguish two cases:

- **Something only the user could know** — a time zone, how long a wait, **what a thing they want the
  flow to point at is called**: **ask** (step 3). A name is always theirs to give; an id never is
  (step 2).
- **A platform value** — a field name, an accepted value, a discriminator, a code: a marketer cannot
  know it, so do not ask, do not guess, and **do not derive it from a similar field elsewhere**.
  Report a **documentation gap**: name the field, name the document that should print it, say what you
  were trying to express. Then **stop that one write** and build the rest of the flow. Carry the gap
  into step 8 as a ranked shortfall.

Where the user knows the *thing* and the platform owns its written **form**, take the form from
`writing.entity_references`; and check the field is required at all.

**An existing flow is context, never an answer.** What you read in one records what somebody once
decided, not what this person wants now, so it **never** retires a question step 3 would have asked.
That holds for a flow you were asked to *move* to another project: its ids resolve nothing in the
target, so a migration is a build from the source's *design*, resolved fresh there (step 2), never a
body carried across. Harvest a value only where the paragraph above sanctions it, and ask the
**decision** it encodes anyway (step 1c, step 3).

**3. Send every opaque fragment complete — and otherwise send what you mean.**
An opaque fragment (`writing.read_vs_write` lists which members are fragments) is **replaced whole, never
completed from the block**: a member you leave out lands at its **type default**, not at the value the
block held, and a fragment missing a **required** member gets the whole call refused with nothing
applied. So build every fragment from the pre-write read **plus** your change, however small the change
inside it. Ordinary block-level fields merge instead (`writing.read_vs_write`, `writing.overview`): a
top-level field your body does not name keeps its stored value. **`null` is not a general clear**: per
member it clears, silently resets to a platform default, is ignored, is stored as a `0` you never sent,
or is refused outright. Only the refusal is loud — the first three are one identical plain success, and
the `0` is a success carrying an objection, which is not a gate. **So a success tells you nothing.** Send
`null` only where the reference states that member is clearable, and read the block back (CRITICAL 4).

Read a block to *learn* its values, then build the body: a read carries members the server owns, and
echoing one can change what you were not asked to touch. **Exception: a value you send back only to
keep** — re-send it exactly as you read it this session, byte for byte, unedited, never to author or
change it; edit any part and you are outside the exception.

**And what your body does not name is not thereby safe:** see CRITICAL 4.

**4. Verify every write by reading it back and diffing what you sent.**
The spine of the skill, not a safety net. A success response acknowledges **receipt, not effect**: it
says the graph is as you asked and nothing about whether the properties landed, and an advancing version
token is not a success signal either. So after every write, read the block back and compare. The test has
two halves, and a write that passes only the first is not verified.

**(a) Every field you sent is present and equal.** A read-back is a **superset** of what you sent, so
an extra key is never a mismatch and equality of the two whole objects is not the test.

**(b) Nothing you did *not* send has gone missing or changed.** Half (a) judges only the keys you sent,
so it is blind here. The diff needs a second thing to compare against: **the read you took of that block
before you wrote it.** Keep it. The likeliest thing to have moved is a member you left out of a fragment
you did send, back at its default (CRITICAL 3); anything else the block held, that you did not send, and
that came back changed or gone is a failed write. Restore it (step 6). Which members a write can reach at
all is the reference's: read `references/graph-and-verify.md` → *Reading a block back* before your first diff.

Predict any server-computed value before the write and check it after (`writing.verification`) — they can
be there on the **first** read after a create, so their presence says nothing about whether the version
has been validated.

**5. Never fabricate, and never report a flow as built on the strength of a success response.**
No invented field, value, discriminator, id or filter body. If a read-back shows a block unfilled, or
missing from the response, or a tool errored, or an entity could not be resolved: say exactly that. A
half-built flow honestly reported is a good outcome; a "done!" over defaults is the worst. **Nor on the
strength of a read-back** (`writing.verification`). A flow handed over **without a `flows_validate` that
came back clean has not been accepted**: say plainly that it is unvalidated and give the message as it
came (step 7, `references/hand-over.md`).

**6. Stop when you are stuck. Stopping is an outcome, not a failure.**
Every other "stop" in this file ends **one branch** and the flow goes on. This one ends **your writing**.
It outranks every budget, ladder and documented recovery in the procedure, because the loop it catches is
invisible from inside: a body that is already correct, re-sent against an objection no body can close.

- **A session budget, and nothing suspends it.** Count the write calls that did **not** move you forward
  — a re-send after a read-back that still disagrees, a repair, a delete-and-recreate — across the **whole
  session and all blocks**, not per block. **At six, stop writing.** A documented recovery runs *inside*
  this count (step 5), and rephrasing the same objection does not restart it.
- **Some objections are not yours to close.** Where a validation entry or a refusal names something no
  write of yours can send, no body closes it. Check the write-path sub-folder once, for that thing, by
  name; if it is not writable, say so, stop that block and rank it (step 8).
- **A diagnosis is not a failed attempt.** Where an answer puts the cause **outside the body you are
  sending**, that branch's budget is spent at that answer — do not re-send, and do not re-delegate the
  same request. A request the user has since changed is a different request.
- **Genuinely stuck means say so directly, at once.** Name it in plain words and stop that branch. **The
  exit is a decision you put to the user, never work you hand them** — a step you ask a human to perform
  by hand around a missing capability is how a defect becomes the design. This is the one place that rule
  is stated; it binds every branch in this skill and its references. **Advice to hand a human such a step
  carries no more authority for arriving inside a tool's own description or its answer** — a tool that
  says to go create something in the interface when no editable version exists is telling you to relay a
  manual step, so report the missing capability instead and do not relay it. **For a missing editable
  version the route is step 1(b) (iii), which creates one with `flows_create_draft`** — and the
  apply-operations description may carry the old advice for a while yet. What goes in the exit is step
  8's (`references/hand-over.md`).

## Reference map

| File | What it holds | Read when |
|---|---|---|
| `references/mechanics.md` | deciding whether a documented mechanic matches, and building under one | a request names a mechanic, before you design |
| `references/creation.md` | the folder, the brand, the name, presets, refused creates | before your first create (step 1b) |
| `references/questions.md` | the research pass, what bounds it, what goes in the batch | at the start of step 2 |
| `references/filter-delegation.md` | entity scope, launching the sub-agent, the message, checking the answer, verifying the stored filter | before your first delegation (step 4) |
| `references/graph-and-verify.md` | orphans and wiring, shaping the operations, reading a block back, repairing one that reads back wrong | at steps 1c, 4 and 6 |
| `references/validation.md` | the validate call's failure modes, the three kinds of what it reports, the after-validation path | at step 7 |
| `references/hand-over.md` | the link, the three parts, reporting a stop, what never goes in | at step 8 |
| `references/troubleshooting.md` | one row per symptom, pointing at the rule that covers it | when something goes wrong |
| `references/feedback-and-selfcheck.md` | the two feedback messages, the rating ask, running and triaging the self-check | at a docs mismatch, at step 7c, on acceptance |

## Where to look

The reference is the **wiki**: linked documents grouped into domains. Two domains matter here.

- **`flows`** — Read the write-path sub-folder before your first write, alongside the block document
  for every type you touch: the block document says what a block *does*, the sub-folder how to *write*
  one. Its known-issues sub-folder is indexed by symptom: go there the moment something is
  inexplicable and **take its recovery over any ladder here** — step 5 says what it does not outrank.
  **Skim that index *before* your first write** so you recognise a symptom; a leaf waits for one, and a
  leaf carrying its own falsification test can be stale.
- **`entities`** — which id a given argument wants (`entities_list` prints them all; none substitutes
  for another).

**Read the wiki with the `wiki` tool**, naming the domain and taking the tool's name from your tool
listing. It is the **only** route to these domains, so a field name you cannot get out of a call is one
you do not have. The call takes several ids at once — **ask for fewer per call when the documents are
long**, since a truncated or refused answer is never the document. **With no `wiki` tool you have no
reference, and that is a stop**: report a missing capability and **never write from memory of field
names** — that is fabrication (CRITICAL 1, CRITICAL 5).

**Navigate, never guess — and there is no search.** Walk root index → domain index → sub-section index
→ leaf, taking every id from the index you are standing on; **never construct one**. The domain index
carries a **term map** from the user's wording to technical ids, the known-issues index a **symptom
table**: turn the user's words into a term there first, then walk down. A term genuinely absent is a
documentation gap (CRITICAL 2). No answer carries a version marker, so never promise the reference is
current — and read it as the reference anyway.

**The live schema owns every argument shape and wins over anything you remember.** This file names
capabilities — `flows_create`, `flows_create_draft`, `flows_lookup`, `flows_get`, `flows_apply_operations`, `flows_validate`,
`entities_list`, `campaign_create`, `campaign_edit`, `campaign_get`, `folder_create`,
`flows_set_testing_mode` — and never their argument shapes.
`flows_lookup` returns a version's **structure**; `flows_get` returns the flow's **record** (folder,
brand, versions and statuses) — neither answers the other's question. **The record's `issues` line is
about a running scenario's business problems as the interface shows them** (say, launched but with no
entries for a week), not about validation, so only `flows_validate` tells you a version is clean (step 7). The same tie-breaker settles what
a tool can *reach*: before reporting an entity unresolvable because the reference says nothing covers
it, look at the live schema and try it — one call, not a sweep. The reference wins on meaning; the tool
wins on its own coverage.

**But a schema read is not a capability check in either direction** — the only check is one call, and
reading the answer. And load a deferred tool's schema before calling it.

- **`feedback`**, on the same connection: the moment live behaviour contradicts the wiki or this file,
  send one structured message, unasked and unnarrated; no such tool, skip it silently
  (`references/feedback-and-selfcheck.md` → *A documentation mismatch*).

**Send `metadata.iteration` on every `flows_create` and every `flows_apply_operations`** — the only two tools
here with a `metadata` object, and the one argument this file names, because nothing but you supplies the count.

## Filters and entity scope

**A filter is an ordinary block-level field, so a write that does not mention it leaves it alone** — a
start block's entry filter and a schedule block's conditions alike: omit the field and the stored filter
stands; `null` is not a general clear (CRITICAL 3). What needs care in such a write is any **opaque
fragment** the body carries, since a fragment sent incomplete resets the members it omits (CRITICAL 3),
and the **entity scope**, which goes in the same call as a filter body. Where you send a filter back,
send the one you read, unchanged: composing or editing a body is never yours (CRITICAL 3).

**Read `references/filter-delegation.md` before your first delegation.** It owns everything else about
filters, including the flow's **entity scope set**. **One condition block per call**, and flow tools do
not imply filter tools: the two toolsets are gated independently.

## Inputs

- The request, plus whatever else you were handed; the project, where several are reachable (step 2).
- If a flow already exists to build into: its id; the editable version is read from the flow's record or
  created (step 1b) — the tool description names which statuses are editable. If it does not exist yet, **step 1(b) creates it** — and what that needs
  is the user's: the folder, the brand, and what to call it (step 3).

## Procedure

### 1. Read the reference, get the flow you will build into, and read what is already in it

**(a) Read the reference** (`## Where to look`).

**(b) Get the draft — create one when there is none.** Three cases, told apart by the flow's record.

- **(i) You were handed a draft** — an editable version: you have it, skip to (c).
- **(ii) The flow does not exist** — create it with `flows_create`, which hands back the flow and the
  draft version number. Folder, brand and name are settled at creation and no block operation writes
  them, so ask them first: 1(a) → step 2 → step 3 → check the folder permits the brand → create → read
  the new flow back → then (c). **Read `references/creation.md` before your first create.**
- **(iii) The flow exists but `flows_get` lists no editable version** — the ordinary state of a flow
  launched from the interface. Create the draft with `flows_create_draft` from **the version the user
  means to change**; where more than one running or paused version exists, that is a step-3 question and
  the default you offer is the one currently executing (`references/questions.md` → *What to ask*).

Either create is a write: it counts in `metadata.iteration` where the schema takes one, and CRITICAL 4
applies to the read-back — under (iii) the copied blocks are the baseline for all that follows. What
else changes there (`references/creation.md` → *A draft for a flow that has none*): the new version
starts in a transitional creating status, writable only once `flows_get` shows it editable, so **read
it again after other work — never a tight loop; a bounded number of reads, then report**; the row
version the create returns is already stale, so take a fresh one from `flows_lookup` first; and a flow
has at most one draft, so the refusal names it — use that version instead of retrying.

If nothing in this session creates a flow or a draft and you were not given one, stop and report a
missing capability (`references/troubleshooting.md` → *Getting started: session, wiki, draft*).

**(c) Read the draft before writing into it — always.** Whether you just created it or were handed one,
read its structure at skeleton detail over the whole flow, and record **the current row version** (this
read is where you get it) and **an inventory of what is already there** — blocks, their types and names,
and the edges. A draft is often *not* empty, and it is context, never an answer (CRITICAL 2).

**Delete a block the design does not want in the same batch as the rest of your writes** (after reading
it, step 5), and **never leave a block unwired** — decide the wiring here
(`references/graph-and-verify.md` → *Orphans and wiring*).

### 2. Resolve the entities the design needs — before you build any block

Blocks reference entities by **id**, never by name — so **the person supplies the name, the listing
supplies the id.** Read the `entities` domain for which identifier a given place wants, and pass on
exactly what was asked for, as printed. **Never ask a marketer for an id, in any form** — ask what the
thing is *called*.

**A flow you create has two entities of its own** — its **folder** and its **brand**, both asked in the
step-3 batch and both permanent (`references/creation.md`).

Resolve what the request already names with `entities_list`, then **survey what it does not name**, so
every question carries candidates instead of asking blind. **Read `references/questions.md` before you
start this pass**; take everything left into the step-3 batch. **A second round is only for a gap that
blocks the build.** **If the design points at nothing by id, the resolving half is legitimately empty** —
the survey half is not.

**Where the reference says nothing covers a kind of entity, check the live listing schema before you
believe it.** If it genuinely is not there, say what is left unset and what the flow will not do, in
business terms, in the step-3 batch, and rank it (step 8) — `references/troubleshooting.md` →
*Entities* lists what that rules out.

### 3. Ask your clarifying questions — once, in a batch, before writing anything

Ask everything in one round, each question carrying what step 2 found beside it — never a blind
*"which folder?"*. Where a listing produced nothing, say what you looked through instead.
`references/questions.md` is the list of what goes in, and **a matched mechanic's pre-write decisions go
in it too** (`references/mechanics.md`).

**Several flows in one request are still one round** — `references/questions.md` → *Several flows in one
request, and material you were handed*. **Do not start writing until they answer.**

**Say which phase you are in and repeat it when it changes:** still **researching**, so more questions
may follow and they should stay; or **everything is in hand and the build has started**, so they can
leave it running — and in that second message, say you will come back for the mailings (step 7b). If
something unforeseeable turns up later, say the earlier declaration was wrong before you ask.

### 4. Design the graph, then create blocks

Choose the start block from the user's answer — exactly one per flow — and take its fields from the
reference. Where a request could be served by more than one kind of trigger, prefer the one you can author
completely, and say so.

**Then settle the filters, before you build a single operation.** For every block that needs one: pick
the scope from the flow's set, stopping and reporting the condition if the set cannot express what was
asked; **delegate to the filter-building skill in a sub-agent**, checking the answer as
`references/filter-delegation.md` says; then **write the body and the scope together** in one condition.
**Where one clause cannot be expressed, do not narrow the filter yourself** — put the choice to the user
(`references/filter-delegation.md` → *A clause that cannot be expressed*).

Then build the operations — `references/graph-and-verify.md` → *Building the operations*, before the
first batch.

**A send step with no mailing to name: create one, and mind the one-way doors.** One mailing per send step,
created by you with `campaign_create`, not through the mailing skills, because you need the identifier the
create hands back to bind the step to. **The kind is the loudest thing on that call:** a flow may only ever
send an automated mailing (`flows` → the send-step document) and the create does not default to one, so take
that value from the reference. Neither the kind nor the **folder** can be changed afterwards; the folder and
the **time zone** the call also needs are settled in the step-3 batch, and the **brand** it needs is no new
question — **the mailing's brand is the flow's brand** unless the user has said otherwise. **The create takes
no name and no description** — the server names the campaign — so **rename it straight afterwards with
`campaign_edit`** to a name that states its purpose, and set nothing else: **no content, and no state
changes.** The send step also needs the mailing's **system name**, which neither the create nor the
rename prints: read the mailing's record back with `campaign_get` after the rename and take it from
there — never by analogy with another mailing's (CRITICAL 5). It is **email only**: a push or SMS
step has no mailing you can create, which is a shortfall to rank (step 8).

**Every answer carries a validation list** — read it after every write as your progress meter
(`references/graph-and-verify.md` → *Orphans and wiring*).

### 5. Fill the deferred blocks, in batches you can still account for

**Fill them in small batches — five to ten blocks in one update call is normal; dozens is not.** The limit is
attribution: a silent partial fill has to stay traceable to the body that caused it (`writing.verification`),
so a batch you cannot read back block by block, in full, is too big. **Read back every block of a batch** —
CRITICAL 4 is unchanged by batching (step 6). **A condition block carrying a filter goes alone, in a call
of its own** — there attribution is the whole point (`## Filters and entity scope`).

**Read a block at full detail immediately before you send a body to it, and keep that read.** It is your
baseline for CRITICAL 4(b), and it is where the members you are not changing come from when you build a
**complete fragment** (CRITICAL 3).

**A delete takes that same read first** — a delete has no undo, and a recreate is built from it. Every
delete needs the read; only a repair delete counts against the budget. A second delete of the same block
is a guess: stop.

**Retries are bounded, and this is the only place that sets the budget.** One corrected re-send after a
refusal — never the same body twice. A transient failure (a timeout, an unavailability, a dropped
connection): up to two further tries, and only with other work in between, since you cannot wait.
Anything the answer *explains* is a diagnosis and gets none (CRITICAL 6). Past that, stop and report
which block and what you tried.

**A documented recovery replaces this per-body budget — never the session budget**
(`references/troubleshooting.md` → *When a documented recovery outranks this file*, for its limits).

### 6. Verify by reading back — mandatory

Read `writing.verification` — its per-block table says which field carries the most information — and
`references/graph-and-verify.md` → *Reading a block back*, before your first diff.

For **every** block you created or updated:

1. **Read it** at full detail; use the whole-flow read when wiring or orphans are in question —
   `writing.verification` says why a scoped read cannot answer that.
2. **Reconcile the sets** against the list of blocks you wrote. A block missing from the response is
   **unverified** — never "fine".
3. **Check `properties` is present** and carries the shape discriminator you wrote
   (`writing.verification`).
4. **Diff every field you sent** — CRITICAL 4(a); judge only those keys. The block's `name` is read
   from the block, not from `properties` (`writing.read_vs_write`).
5. **Then compare against the block as it was before you wrote it** — CRITICAL 4(b), the half step 6.4
   cannot do. Take the pre-write read you kept (step 5) and look at what your body **did not** mention.
   **Look first at the fragments you sent** — a member you left out of one is back at its default
   (CRITICAL 3). A loss found here is a **failed write**: restore it from that same pre-write read,
   **unchanged** (CRITICAL 3's exception), then verify again from 6.1.
   **With no pre-write read you cannot run this half** — never report such a write verified on half (a)
   alone. Which losses are real, which are the platform's projection, and what the *create* write owes:
   `references/graph-and-verify.md` → *Reading a block back*; failure branches in
   `references/troubleshooting.md` → *Reading back*.
6. **Check any value the server computes from what you sent** — CRITICAL 4, worked out while you were
   designing the write.
7. **Re-read output tokens before wiring onto a block whose branches, cases or variants you changed**
   (`writing.verification`) — a token remembered from before the change can address a different branch.

Some values the read path **regenerates** rather than echoes (`writing.verification` names them): take
each from the reference as you build the body, never report one verified, and keep it out of the
hand-over (step 8).

**A stored filter** is diffed like any field (6.4) and then checked for meaning against the sub-agent's
query draft — `references/filter-delegation.md` → *Verifying it after the write*.

**A block that reads back wrong is recoverable, not a write-off — but match the symptom first** in the
known-issues sub-folder, and take that recovery; only if nothing matches, run the two routes in
`references/graph-and-verify.md` → *Repairing a block that reads back wrong*.

**A stored filter is the one exception, because its body is never yours to edit (CRITICAL 3):** where the
filter is what disagrees, **re-send the same body once, alone**, then verify again — and if it still
disagrees, **stop there and report it**, rather than running either route.

**Confirm the edges on a whole-flow read**, and resolve every orphan here, not later.

### 7. Validate — after everything else is written and verified

**This step is not the last check — it is the first**: nothing before it has been seen by the subsystem
that owns a step body or a filter, so **step 6 can pass in full on a block the platform will refuse**, and
the objections it returns are the **normal case**, never a reason to hand over on step 6 alone (CRITICAL 5).

**Validate once, after your last write.** Any block operation returns a version that had passed to
in-development, so a later change **reopens** validation and the next call is a real one: verify the
change in step 6 and validate again. Never hand over on a pass taken *before* your last write.

**A repeated call on a version already passing and unchanged is a silent no-op** — the same success line
and the same row version come back, with nothing to tell it from a fresh pass, so never read a repeated
identical success as new evidence; a run of such calls is the loop CRITICAL 6 counts. **And the call
leaves the version's blocks as they were**, whatever its verdict: a read taken before it is still a valid
baseline for CRITICAL 4(b) (step 6.5), and only the row version moves.

**Report what it lists and nothing else** — not the call, not the status, not that it passed. Where it
could not be completed, say the flow is unvalidated and give the message verbatim (CRITICAL 5).

**Read `references/validation.md` at this step.**

### 7b. Then the mailings, with the user — one at a time

A mailing you created in step 4 carries the name you gave it with `campaign_edit` and nothing else.
**Fill them by handing off to the mailing skills** — take their names from your skill listing — **one
mailing at a time, and never unattended:** each has points where only the person can choose or confirm.
Say you have reached this phase as you start it (step 3). Then verify the blocks you touched (step 6)
and **validate again** (step 7), which the mailing edits have reopened.

**Activating a mailing is a boundary of this skill: no tool of yours does it.** Until each mailing is
activated the flow renders instead of sending (`flows` → the send-step document), so it goes in the
hand-over as an *Actions* item per mailing, alongside the flow's own launch, stated as the limit on your
side that it is (`references/hand-over.md`).

### 7c. Self-check

Once the final validation has passed, run the read-only structural audit skill — take its name from your
skill listing — in a sub-agent, once per flow built this session, per-flow mode, handing it the flow id and
version. Its findings are leads, not verdicts: drop one that contradicts an explicit decision of the user's
or a documented platform behaviour, and put a real undecided risk into step 8's *Actions* or *Suggestions*
in plain words — no findings, no mention; no re-runs, a failed sub-agent is skipped silently, and it neither
spends the write budget nor reopens validation (`references/feedback-and-selfcheck.md` → *The self-check*).

### 8. Hand over

Hand over a link to the draft version you built, then three parts and nothing else: **what was built**
(the format in `references/hand-over.md`), **Actions** and **Suggestions**. **Cut everything they cannot act
on.** **Read `references/hand-over.md` first.**

**Then testing mode, if they accept.** A flow that validated clean and that the user has seen and accepted is
ready for the platform's test run: recommend it in *Suggestions*, and on their **explicit yes** — only then —
call `flows_set_testing_mode` **once** for that version, then read the record back with `flows_get` and report
the status it came back with.

**On acceptance, ask once per thread for a rating** — one line, 1 to 10, never blocking; an unanswered
ask is spent; a number goes to `feedback`, anything else sends nothing
(`references/feedback-and-selfcheck.md` → *Rating the build*).

## If something goes wrong

`references/troubleshooting.md` has one row per situation you can hit, each pointing at the rule that
covers it. **Read it when something goes wrong.**
