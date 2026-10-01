# Creating the flow

Read before your first create. The **live tool schema** owns the argument shapes; this file owns the
order you do things in and the decisions you must not take for the user.

- [What a create needs, and where each part comes from](#what-a-create-needs-and-where-each-part-comes-from)
- [Why the questions come first](#why-the-questions-come-first)
- [The folder is the decision you cannot take back](#the-folder-is-the-decision-you-cannot-take-back)
- [Asking about the folder](#asking-about-the-folder)
- [The brand: asked, never derived](#the-brand-asked-never-derived)
- [Before you create: does the folder permit the brand?](#before-you-create-does-the-folder-permit-the-brand)
- [The name, and why a repeat is safe](#the-name-and-why-a-repeat-is-safe)
- [Starting from a preset](#starting-from-a-preset)
- [Verify the create like any other write](#verify-the-create-like-any-other-write)
- [When a create is refused](#when-a-create-is-refused)
- [A draft for a flow that has none](#a-draft-for-a-flow-that-has-none)

## What a create needs, and where each part comes from

Four things, and none of them is yours to invent:

| What | Where it comes from |
|---|---|
| The **folder** the flow lives in | the user names it; the entity listing turns that name into the id (step 2) |
| The **brand** the flow belongs to | the user picks, in words they recognise, **in the same questions section as the folder** (step 3) — **never derived** — see below |
| The flow's **name** | the user, always — see below |
| A **preset**, optionally | the live tool schema — see below |

Which of an entity's ids each argument takes is keyed on the argument's **type**, not its name — read
`writing.presets` and the `entities` documents for the folder and the brand, and take the column they
name.

## Why the questions come first

A flow's folder and its brand are settled **when it is created** and are not what the block
operations write. So the order for a new flow is: read the reference (step 1a) → resolve what the
request already names (step 2) → **ask your batch** (step 3) → **check the folder permits the brand**
→ create → read the new flow back → then design and write blocks (step 4 onward). Creating first and
asking afterwards is not a shortcut, it is the mistake: you would be choosing the one thing that
cannot be changed later before asking the one person who knows it.

## The folder is the decision you cannot take back

**The folder fixes, for the flow's whole life, which mailings it can ever send.** The reference
states the boundary exactly; what matters here is the shape of the failure. A folder that does not
suit the flow is **accepted at creation without complaint**. Nothing is wrong until, much later, a
send step is written — and then the refusal reads like an unrelated scope error about a mailing, at
the point furthest from its cause, on a flow that cannot be repaired by editing that step.

**Resolve the folder from what the user named, or stop:** never copy one from another flow, from a
loose match because it was the only row that came back, or from a previous session — including when a
refusal message suggests it.

## Asking about the folder

In the step-3 batch, in the user's own words:

1. **Ask whether a folder with the mailings this flow will send already exists — with the candidates
   your survey found and what each one holds**, or, **where it found none, with no list and what you
   looked through** (`references/questions.md`: a folder's name often says nothing about what it
   holds, so a survey that turns up nothing relevant is a normal outcome). Either way it is a question about
   what the folder is *called* — never "which folder id", which is not a thing they hold.
2. **They name one:** resolve it in the listing, by name. Several matches, or none, is a second short
   question naming what came back — not a guess.
3. **They say there is none, or they want a new one:** say **inside the question** that a brand-new
   top-level folder can only ever send mailings you create in it, while a **subfolder of the folder
   their mailings live in** keeps those reachable. **Read the reachability boundary in the reference
   before you word this** — which neighbouring folders a flow can send from is the reference's to
   state, and it is exactly what makes the subfolder answer work. Offer both, in that order. If they
   still choose a bare new folder, carry that into the hand-over as a named shortfall.
4. **Creating the folder is `folder_create`'s job** — take its exact name from your tool listing. It
   takes the **name** the user gave and a **required short description** of what the folder is for:
   write that description yourself, in one sentence, from the request — it is not another question
   for the user. Only where no folder-creating tool is in the session does that branch end: say
   plainly that you cannot create one and stop, and never pick a folder nobody named (CRITICAL 6).

With a prepared folder there is nothing to say at hand-over; with a new one, a named shortfall.

## The brand: asked, never derived

The brand is settled at creation and **cannot be changed afterwards** — a brand you got wrong is a
flow that has to be created again. So **it is asked, every time, and it is asked of them.**

**Never derive it** — not from the folder, not from a mailing. A folder's `brands` column can be
`all`, a list or `none`, so it does not narrow the brand at all (`entities` → folder), and a derived
brand is a guess at a permanent, unrepairable decision.

**Ask it in the same questions section as the folder** (step 3), with the brand catalogue already
read in the research pass, so the question can offer names instead of the word *brand*:

- **Ask about the thing, not the term** — *"which of your shops or sites is this for?"*, with the
  names from the brand catalogue (short enough to read whole), never a system name and never an id.
- **Say it is permanent in the same breath.** "This one can't be changed afterwards — a different
  brand would mean building the flow again." Said afterwards it is no longer a choice they could
  have made, it is a repair.
- **Never break the tie on how a name looks** — a default-looking, test-looking, first or
  only-active brand is not evidence.

**If the answer does not come** — they skip it, or say "whatever the usual one is" — **ask again**,
once, with the shortest recognisable list. Do not fill it from the folder, and do not pick one to keep
moving. If they still cannot choose, **stop and report it** as the one thing blocking the create: a
decision you cannot make for them and they cannot undo.

## Before you create: does the folder permit the brand?

Their answers are in, nothing is created yet: **this is the one moment the two can be checked against
each other.** After the create the pairing is fixed for the flow's life, and the failure surfaces
somewhere else entirely.

Read the chosen folder's `brands` column and judge from what the reference says it means — the values
are not all lists, and the one that permits everything does not look like a brand name.

- **It permits the brand** → create.
- **It does not** → put the conflict to them and let them change one of the two — never switch either
  yourself; both are permanent.
- **You could not check** — the row does not resolve, or the reference does not print what its value
  means → say so and stop before the create. An unverified pairing reported honestly costs a
  question; created, it costs the flow.

## The name, and why a repeat is safe

**The name is required and it is the user's.** Do not default it, do not compose one from the
request, do not append a date to make it unique.

A name already used is **refused** rather than duplicated, which is what makes a create safe to repeat
(the tool description says so) — and that only holds for a name the user chose.

## Starting from a preset

A preset builds the graph and **fills no project entity**, so a created flow cannot pass validation as
created (`writing.presets`); everything from step 1(c) onward applies to it exactly as to a flow you
built block by block. `writing.presets` also says why you cannot choose one by its contents or its
name. Where the mechanic itself is documented, it is documented in the wiki's mechanics sub-folder and
no preset substitutes for it (`references/mechanics.md`).

Take the preset set from the live schema. Unless the user described something a preset is plainly
*for*, create the empty one — you have to read back and fill whatever a preset gives you anyway.

## Verify the create like any other write

A create is a write, so CRITICAL 4 applies to it: **read the new flow back and check it landed
where you meant.** Confirm the folder and the brand are the ones you resolved, and the name is the
one the user gave. This is the only moment where the folder is cheap to check — the next chance is
the send step, and by then it is not a check but a diagnosis.

Then read its structure (step 1c): from a preset it is **not empty**.

## When a create is refused

**A refusal applied nothing** — no flow exists, so there is nothing to clean up and nothing to
report as half-built.

| The refusal | What to do |
|---|---|
| The **name is already in use** | Not a failure to work around. Ask whether they meant the flow that already has that name — and if not, ask for another name. Never make the name unique yourself. |
| It names the **folder** | The id you passed did not resolve. Go back to the listing and re-resolve from the name the user gave. Never substitute one from another flow, whatever the message suggests. |
| It names the **brand** | Same, and check first that you passed the id the reference asks for rather than the entity's other one. |
| **Permission** to create is denied | Report it plainly as a permission you do not have, naming the project. Do not retry, and do not build into some other flow instead. |
| Anything opaque, or no answer at all | `references/troubleshooting.md` → *Getting started: session, wiki, draft*. |

## A draft for a flow that has none

A flow launched from the interface has only running or paused versions until someone opens it for
editing; `flows_create_draft` does what that opening does — it copies one of them into a new draft.

- **Copy from the version the user means to change**, a running or paused one from `flows_get`; where
  more than one qualifies, ask (step 3) and offer the one currently executing as the default.
- **It is not writable at once**: a transitional creating status while the blocks are copied, then
  editable in `flows_get`. Re-read the record after other work — never a tight loop; then report.
- **Take a fresh row version from `flows_lookup` before the first write** — the copy changes the flow
  once more as it finishes, so the token the create returned is already stale.
- **A flow has at most one draft**, and the refusal names it: use that version instead of retrying; a
  source in a status that cannot be copied is a different refusal — pick a running or paused version.
- **It is a write**: `metadata.iteration` where the schema takes one, and the copied blocks read back
  are the baseline for CRITICAL 4(b).
