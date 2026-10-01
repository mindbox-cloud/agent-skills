# If something goes wrong

One row per situation, each pointing at the rule that covers it.

- [When a documented recovery outranks this file](#when-a-documented-recovery-outranks-this-file)
- [Getting started: session, wiki, draft](#getting-started-session-wiki-draft)
- [Writes](#writes)
- [Entities](#entities)
- [Reading back](#reading-back)
- [Validation](#validation)
- [Filters and delegation](#filters-and-delegation)
- [Out of scope requests](#out-of-scope-requests)

## When a documented recovery outranks this file

`SKILL.md` step 5 states the precedence: where the symptom matches one in the `flows` domain's
known-issues sub-folder, the recovery written there **replaces the per-body retry budget and every
generic ladder** — here and in `SKILL.md` — for as long as it runs, and it decides the order of what
you try next. **It does not decide when the session stops.** Four limits keep it from swallowing
everything else. Read them before you follow a recovery.

- **A recovery is spent once you have run its steps.** Matching the same one again is not running it,
  it is improvising, so the budget governs from there — in step 6 that means going on to route 2
  rather than round the same recovery again.
- **The precedence is over the per-body budget only, never over the session budget.** CRITICAL 6
  counts every write call that did not move you forward, a recovery's calls included, and nothing
  suspends it. A recovery cannot license an unbounded run of calls; where its own steps outlast the
  session budget, the budget stops you and the rest is reported, not attempted.
- **A recovery whose step is an unbounded search is not run at all.** Where a documented recovery
  says to harvest a value by paging a catalogue, or by reading flow after flow until one carries it,
  **that step is not taken**: the value is declared unfillable and ranked in the hand-over instead
  (`SKILL.md` step 2). The rest of that recovery still applies where it does not depend on the sweep.
- **A recovery never overrides a `## CRITICAL` rule:** satisfy its intent the rule's way — build the
  body rather than echoing a read (CRITICAL 3), with its preserve-unchanged exception intact, since
  retyping such a value is exactly what would lose it.

## Getting started: session, wiki, draft

| Situation | What to do |
|---|---|
| A capability you need is not in this session — the project's flow tools, a flow-creating tool, a folder-creating tool | Stop and report a **missing capability** in plain words, naming the tools you do have. Do not substitute another tool, do not build into an unrelated flow, and **do not hand the user a step to perform** (CRITICAL 6). |
| The `wiki` tool answers nothing for a domain, or you cannot see one in the session | **A schema is not the check — make the call, naming the domain.** With genuinely no `wiki` tool you have no reference, which is the stop in `SKILL.md` → `## Where to look`: report the missing capability, and never proceed from memory of field names. |
| `flows_create` errors, or the draft it should return never arrives | Stop and report it as a tool failure, naming the call and the error. Before any retry, check whether the flow now exists — a repeat under the same name is refused, which is what makes that check cheap. Do not build into a flow you were not given. |
| The user wants a flow that **does not exist yet** | Create it (step 1b) — ask the batch first: the folder, the brand and the name are settled at creation. |
| The flow exists, but `flows_get` lists **no editable version** — every version is running or paused | Not a missing capability: create the draft from one of them with `flows_create_draft` (step 1b (iii)) — `references/creation.md` → *A draft for a flow that has none*, for the transitional status, the fresh row version and the one-draft rule. |
| **A draft already exists**, and `flows_create_draft` refuses on that | Nothing was created, and the refusal names the existing draft. Use that version — a second draft cannot exist beside it, so retrying cannot produce one. Wait for it the same way if it is still being copied. |
| `flows_create_draft` refuses because **the source version is not in a copyable status** | Only a running or paused version can be copied; one that is already editable is written into directly. Pick the right one from `flows_get`, and where it is unclear which the user means, ask (step 3). |
| The user names **no folder** with the mailings the flow will send | `references/creation.md` → *Asking about the folder* — including creating one with `folder_create`, and the branch where no folder-creating tool is in the session. |
| **A create is refused** — the name is in use, or it names the folder or the brand, or permission is denied, or the message is opaque | A refusal applied nothing. One row per refusal in `references/creation.md` → *When a create is refused*. |

## Writes

| Situation | What to do |
|---|---|
| A write is refused as not editable for this version | Only some statuses accept writes and the tool description names which: get the version list from `flows_get`, re-read for a current row version, and **if none is editable, create a draft** (step 1b (iii)) — `references/creation.md` → *A draft for a flow that has none*. |
| A write is refused as a version conflict | Nothing was applied, **and the token you hold is stale**. Re-read the flow for a current row version, then re-send the same batch. |
| A write is refused for any other reason | **First: is it a refusal at all?** A batch carrying validation entries applied. **Then match the message in the known-issues sub-folder before acting** — a refusal naming nothing is a documented symptom whose recovery replaces this row. Otherwise fix the named cause and re-send, on step 5's budget. |
| A validation entry or a refusal names **something no write of yours can send** | **The body is not the cause, and no re-send closes it** — `SKILL.md` CRITICAL 6, second bullet. **Do not re-send the body in any shape, do not delete and recreate the block, and do not re-delegate to hear the same answer again.** |
| You are about to **delete a block** | Read it at full detail first and keep that read — `SKILL.md` step 5. |
| **A write never answered** — a timeout, a dropped connection, a cancelled call | **This is not a refusal, and you do not know that nothing was applied.** **Read the flow back before doing anything else**: the batch may have applied in full. A blind retry duplicates what it created and applies what it changed twice. Decide from the read whether to retry, and on the budget in `SKILL.md` step 5. |

## Entities

| Situation | What to do |
|---|---|
| `entities_list` isn't available on this connection | Say so plainly: with no listing there is nothing that turns a name into an id, so those entities cannot be resolved and the fields that need them will be left unset — say what the flow will not do as a result. **Do not ask the user for ids**, never invent one, and never pass a name where an id is wanted. |
| An entity resolves to nothing, or to several candidates | Ask for the **name** again, or for which candidate they meant (step 3). Never pick for them, never guess an id, and never ask them to supply one. |
| The reference says no listing covers the kind of entity you need | **First check the listing's own schema** — its coverage moves and the reference can lag it; if the kind is there, call it (`SKILL.md` → `## Where to look`). If it genuinely is not, name the entity, what is left unset and what the flow will not do, and **stop** (`SKILL.md` step 2). Do not ask for an id, do not borrow a value from another flow, do not write a placeholder, **and do not sweep** — paging a catalogue or reading flow after flow is not a resolution route, whatever a recovery suggests. Rank it in the hand-over. |
| You need a field or a value the wiki doesn't document | **Report a documentation gap and stop that one write**, then build the rest — `SKILL.md` CRITICAL 2 has what the report names and what it rules out. Rank it in the hand-over. |

## Reading back

| Situation | What to do |
|---|---|
| A read-back carries keys you never sent | **Normal, not a failure** — CRITICAL 4(a). |
| A read-back disagrees with what you sent | `references/graph-and-verify.md` → *Repairing a block that reads back wrong*: match the symptom first. Do not describe the block as built until a read-back agrees. |
| **Something the block held before is gone, and your body never mentioned it** | **First: is it a member of a fragment you did send?** Then your fragment was incomplete and reset it, and the fix is to re-send that fragment complete (CRITICAL 3). **Otherwise: could a write of yours have sent that member at all?** `references/graph-and-verify.md` → *Reading a block back* settles that. Where it **is** writable this is a **failed write**: restore it from your pre-write read **unchanged** (CRITICAL 3's exception), inside a body that also carries everything else the block needs, then verify again from step 6.1. **Do not** re-send the destroying body and hope. |
| The value is gone and you have **no pre-write read** of the block | You cannot verify that write and you cannot restore from the flow. Say so plainly: name what is missing, name what the flow does **instead** right now (a lost audience filter means *everyone*), and rank it in the hand-over. Then rebuild it only the way it was built the first time — ask the user what the audience should be and delegate again (`references/filter-delegation.md`). **Never reconstruct it from what you infer it must have been**, and never report the block as verified. |
| A read-back doesn't contain a block you wrote | It is **unverified**, not fine. Re-read it scoped to that block alone (step 6); if you still cannot read it, report it as unverified. |

## Validation

| Situation | What to do |
|---|---|
| **The user asks for another change *after* a validation that passed** | `references/validation.md` → *A change asked for after a validation that passed*. |
| Validation reports something you already knew you could not author | `references/validation.md` → *Sorting what it reports*, kind 1. |
| A validation problem doesn't say which block it is about | `references/validation.md` → *Sorting what it reports*. |
| Any validation row here, in more detail | `references/validation.md`. |
| The validate call itself is refused | Do not loop on it. Read the flow back once and check your writes are as you left them; then report the message verbatim as a validation you could not complete, naming the blocks it could concern. Do not start rewriting blocks that verified. |
| Validation says your row version is stale and hands back **the token you already hold** | The token has not moved, so re-reading changes nothing. Treat it as a failed validation, not as a conflict: read the rest of the message for the real problems, and do not loop. |
| Validation fails with a message naming nothing at all | Validation did not run, so asking again changes nothing: go to the known-issues sub-folder, match the symptom and follow its recovery. **Do not delete a block on suspicion.** If its causes are ruled out, report the flow as unvalidated. |

## Filters and delegation

| Situation | What to do |
|---|---|
| **The filter-building skill is not in your skill listing**, or the server has no filter-building tools | The skill and the tools are gated independently, so either can be missing while the flow tools work. Stop the conditions that need a filter and report a **missing capability**. Never compose a body yourself, never leave a condition unfiltered and call it built — no filter means *everyone* — and **do not build one in a sub-agent that has the tools but not the skill**, which is the silent failure this rule exists for. |
| More than one project's server is connected | Ask the user which project **before the research pass**, not in the step-3 batch (`references/questions.md` → *Before the pass: which project, in their words*). A filter or a survey run against the wrong project does not error; it describes a different audience. |
| The sub-agent reports a different project or environment, answers with a link and no payload, or does not evidence which skill built it | `references/filter-delegation.md` → *What to check in the answer, before you write anything* owns the moves and the budget. |
| **The sub-agent never answers** | `references/filter-delegation.md` → *The answer is a return value, not a conversation*. **Do not report the branch as unbuildable.** |
| The reference says the flow's scope set cannot express what was asked | **Stop that condition and report it** (`SKILL.md` → `## Filters and entity scope`). Do not substitute a scope, do not ask for a filter on another root, do not approximate the audience. |
| The rendered read-back of a filter reads differently from what the sub-agent reported, or a value the clause **names** does not read like the name | `references/filter-delegation.md` → *Verifying it after the write*: the rendering normalises, and the value that names a thing is out of the comparison altogether. |

## Out of scope requests

| Situation | What to do |
|---|---|
| The request names a marketing mechanic instead of describing a flow | `references/mechanics.md`. |
| The user asks you to launch / stop / delete the flow | Say what you can't do: launching, stopping and deleting are outside this skill and no tool of yours may be used for them. Hand over the draft instead. |
| The user asks you to put the flow in testing mode | Allowed, but only after a clean validation and only once they have seen the result and accepted it — `SKILL.md` step 8. Earlier than that, it is refused like a launch. |
