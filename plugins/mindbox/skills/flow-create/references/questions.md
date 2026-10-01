# One research pass, then one question batch

Read this in step 2: the survey exists to feed the batch, so you need to know what the batch will
want before you go looking.

- [Before the pass: which project, in their words](#before-the-pass-which-project-in-their-words)
- [Before you ask: the one research pass](#before-you-ask-the-one-research-pass)
- [What the survey may not sweep](#what-the-survey-may-not-sweep)
- [Several flows in one request, and material you were handed](#several-flows-in-one-request-and-material-you-were-handed)
- [What to ask](#what-to-ask)

The list is not exhaustive and cannot be. **The reference is what says which fields a block cannot
be written without** (step 1a), so walk those required fields against what the request actually
said, and ask about every one that is a business decision rather than a platform value
(CRITICAL 2).

## Before the pass: which project, in their words

**Fix the project before the pass** — every listing, survey and delegation runs against one. Asking
it first does not break "ask once": the batch is the round the survey feeds, and the survey cannot
start until this one answer is in.

- **If only one project is reachable, do not ask at all.** Build there, and name that project at
  hand-over so a wrong assumption is visible while the flow is still a draft.
- **If more than one is, ask this one question on its own, immediately, and then run the pass.**
  Keep it to that single question — do not use the opening to ask half the batch blind.
- **Ask it in their world:** never print a deployment, environment, connection, server or account
  label. Name each project by its **human name**, or ask by what tells them apart in their terms —
  *"is this for the account you send real customer mailings from, or one you try things out in?"*
- **Their answer decides it; a mismatch is a stop, not a pick.** If what they name matches nothing
  you can reach, say so and ask again — never quietly build on whichever one you could reach.

## Before you ask: the one research pass

**Establish what you can without them, and put it inside the question:** not *"which folder?"* but
*"three folders match what you described — this one holds four automatic email mailings, the other
two hold none; which is it?"*. That is answerable from recognition, in one go.

What the pass establishes, in this order:

1. **What the design needs at all** — from the reference, and where the request named a mechanic from
   that mechanic's own documents, including its list of decisions to settle first
   (`references/mechanics.md`). Until you know which fields a block cannot be written without, you do
   not know what to survey or what to ask.
2. **Everything the request already names** — resolved in the project's entity listing (step 2).
3. **The containers and the brands the flow will need.** Two listings, and what you need from each
   is different — see below.
   - **Read the brand catalogue here** so the brand question can offer names they recognise. Where
     it is short (the `entities` domain says so on the brand entity), offer it whole; where it runs
     long or is mostly test junk, offer the candidates that plausibly match the request and say how
     many more there are — never a blind list of dozens.
   - **Find the folder yourself before you ask about it.** Look it up from the user's own words;
     where nothing matches, **read the catalogue and judge it by meaning** — search matches the
     display name only, and a folder's name often says nothing about what it holds. Then **open the
     plausible ones and see what is in them**. What goes into the question is what you read and
     judged, never rows that scrolled past. **Reading everything and finding nothing is a legitimate
     answer, with no list.**

   Do this **before** asking about the folder, about the brand, or about which mailings a step will
   send.
4. **What is inside the plausible containers** — bounded; see the next section.
5. **What is already in the draft**, if you were given one (step 1c), and anything in it that
   conflicts with the design.

**Then ask, once.** A second round is legitimate only for a **gap that blocks the build**: an answer
that resolves to nothing or to several candidates, a thing they named that no listing reaches, an
answer that skipped something a block cannot be written without. **A filter diagnosis is the other
expected trigger** — a clause the filter skill reports it cannot build can only arrive after the
batch has closed, so that question is the process working, not a breach of "ask once"
(`references/filter-delegation.md` → *A clause that cannot be expressed*). Never for something the
survey could have settled before you asked, and never for a detail the reference marks optional.

## What the survey may not sweep

**The mailing catalogue runs to many thousands of rows** (`entities` → mailing), so reading it whole
is a stall, not research.

So the mailing half of the survey is **bounded, and the boundary is the folder**:

- **Never list mailings unnarrowed.** Narrow by the container, and by whatever the reference says
  distinguishes a mailing a flow can actually be told to send from one it cannot — read that on the
  entity first, and take the narrowing keys from the live tool schema, never from here.
- **Survey only the folders you picked out as plausible**, plus the neighbours the reference says are
  reachable from them. A handful of folders, not the tree. Where the request narrows it to one, that
  is one call and you have your recommendation.
- **Reading the folder catalogue is cheap; opening every folder is the sweep this section forbids.**
  Open the few that could be it, ask with what you found, and survey the rest once they have named
  one. That is not a second round of questions; it is research finishing after the one question it
  needed answered.
- **Report what you read, and only what you read.** "No automatic mailing in this folder" is a finding
  worth putting in the batch; a count you did not read is not.

## Several flows in one request, and material you were handed

**A batch of flows is one question round, not one per flow.** Whether it is several mechanics, a set
of flows migrating from another project, or logic being moved off another platform: do the reference
pass and the survey for **all** of them first, then ask everything in **one** batch, grouped so the
user can see which flow each answer belongs to. Build them **one at a time** afterwards, each to a
verified draft. Asking per flow turns one round into N, and it invites a mechanic to be designed
before its own questions are answered.

**Read whatever you were handed, in whatever form it arrived**, take every decision it settles and
ask only about the rest: it **reduces** the batch, never replaces it, and where you cannot make sense
of a part, ask about that part rather than inferring a design from it. **Material that settles less
than it appears to is context of zero weight**: a flow you were pointed at that has no event chosen,
orphan blocks or no send step carries no design to harvest. Say in the batch what it does and does
not settle, and ask for the design as if you had been handed nothing.

## What to ask

- **The name or description of every thing the blocks point at** — never an id (step 2), with the
  candidates you found for anything ambiguous.
- **Which version to change**, where the flow you were handed has several running or paused versions and
  no draft — the draft is copied from one of them (step 1b (iii)), so offer them in the user's terms and
  make the one currently executing the default (`references/creation.md` → *A draft for a flow that has
  none*).
- **Which folder the flow belongs in**, if you are creating it — `references/creation.md` →
  *Asking about the folder*: the wording, the consequence that must go inside the question, and the
  branch where no folder-creating tool is in the session.
- **The brand, always, in the same section as the folder** — `references/creation.md` → *The brand:
  asked, never derived*.
- **What to call it**: required and theirs; the name is also what makes a create safe to repeat
  (step 1b).
- **Which folder the mailing you create lives in** — where a send step has no mailing to name. It
  must be a folder the flow's own folder permits, and it cannot be changed afterwards — **and which
  time zone its schedule is read in**, which `campaign_create` requires and no listing returns, so it
  is asked of the user here. Its **kind** is not asked (a flow may only send an automated mailing),
  and its **name** is not set at creation: the server names it and you rename it afterwards
  (`SKILL.md` step 4).
- **Who a send step's message goes to**, where its channel needs a recipient named: the reference
  makes that a business decision no listing settles (`flows` → the send-step document), and it is
  discoverable only at step 4, so ask it here rather than in a second round.
- **The project's admin URL** — the address they open the platform at. You need it to hand over a
  link (step 8) and **no tool of yours returns it**, so it belongs here, not at hand-over. Skip it
  only if they have already given you one in this session.
- **Every place the request is underdetermined** — which trigger, how long a wait, what a limit
  should be, which branch does what, where a branch should end.
- **Where more than one kind of message is in play — how the sends are *arranged*, not only which
  ones there are.** One message chasing another inside one mechanic is a path with a wait and a
  check between the sends; separate mechanics are separate branches. Ask it in what they send, never
  as a channel and never as a field — *"does one message go out and then another if nobody reacts,
  or does each part of this send its own?"* — and **leave the answer room to differ per mechanic**.
  Arriving late it costs a rebuild, so it belongs in this batch. Where a matched mechanic asks this
  question itself, its wording wins (`references/mechanics.md`). Note that only an email mailing can
  be created by you (`SKILL.md` step 4), so a push or SMS step needs one that already exists.
- **Every value a block *requires* that is the user's to choose** and the request never gave — an
  invented one is stored looking exactly like a chosen one, and neither the read-back in step 6 nor
  the user at hand-over can tell them apart.
- **Everything a matched mechanic says must be settled first.** Where the request matched a
  documented mechanic, that list is part of this batch — read it from the wiki now and ask it in the
  mechanic's own words, because no later write revises what those answers decide
  (`references/mechanics.md`).
- **Anything in the existing draft** (step 1c) that conflicts with the design.
- **Anything you have already established you cannot write** — an entity nothing resolves, a value
  the reference does not print, a setting no operation reaches. Say it now, in business terms, with
  what it blocks: it may change what they ask for, and a headline requirement they learn about at
  the end reads as a broken promise. Do **not** ask them for a way around it (step 2).
- **What every condition and every start-block entry filter should select, in business terms** — the
  audience in their own words is exactly what the filter agent needs, so ask for it here rather than
  guessing at it later. Say what a missing condition would mean (no filter means *everyone*); if the
  reference says this flow's scope set cannot express what they asked, say that now too.
- **Not the project**: settled before the pass.

# References

- `SKILL.md` → step 2 — the research pass this file is read during
- `SKILL.md` → step 3 — the rule this list serves
- `references/creation.md` — the folder and name questions in full, and why they cannot wait
- `references/filter-delegation.md` — what the filter agent needs from the audience answer
- `references/mechanics.md` — following a documented mechanic, and when its decisions are asked
