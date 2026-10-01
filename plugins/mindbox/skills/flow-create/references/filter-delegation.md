# Delegating a filter to the filter-building skill

Read before your first delegation: the wording and the checks.

One delegation per filter. Never two filters in one sub-agent.

- [Entity scope — where the set comes from](#entity-scope--where-the-set-comes-from)
- [Before you delegate](#before-you-delegate)
- [Launching the sub-agent](#launching-the-sub-agent--settle-this-before-you-send-anything)
- [The message](#the-message)
- [The answer is a return value, not a conversation](#the-answer-is-a-return-value-not-a-conversation)
- [What to check in the answer](#what-to-check-in-the-answer-before-you-write-anything)
- [Writing it](#writing-it)
- [Verifying it after the write](#verifying-it-after-the-write)

## Entity scope — where the set comes from

The flow's scope set is **decided by its start block and is the same at every block**; no read returns
it, so take it from `writing.entity_scope`, keyed on the start event. Check that what the user asked
is expressible **before** you ask for a filter; if it is not, stop and report that condition rather
than substituting a scope, asking for a filter on a different root, or improvising something adjacent.

A scope and a filter that disagree are accepted silently (`writing.filter_roots`), so send both in one
call.

## Before you delegate

You need three things in hand, and none of them is a filter's internals:

1. **The server** you are building the flow on — name it, not just the project. The filter tools and
   the flow tools live together per project, so naming the server is what makes the sub-agent work on
   the same project as the flow.
2. **The root entity** for the scope you already chose, in the **filters** vocabulary — take the
   translation from `writing.filter_roots`, never by echoing the scope name. That skill preserves a
   root the caller supplies, so supplying it is what keeps the filter on the surface your block
   expects. **Pass the root and the property set that reference names for your scope**; where it
   names none, say so in the message and the filter skill uses the root's default property set. A
   missing property set is **not** a documentation gap and never stops the delegation (CRITICAL 2).
3. **The request in business terms** — the audience as the marketer described it (step 3). Do not
   resolve any of the things it names; that is the sub-agent's job and it is better equipped for it.

## Launching the sub-agent — settle this before you send anything

**The message is only half of the delegation. The sub-agent has to be running the filter-building
skill.** A sub-agent is launched by agent *type*, and **no agent type corresponds to a skill** — so a
sub-agent handed only the message below runs as an ordinary agent. It has the project's filter tools
in its own list, so it will improvise a filter and hand back a payload of **exactly the shape you
asked for** — a plausible filter with the wrong provenance, and nothing about the answer's shape
gives it away. Every other guard in this skill fails loudly; this one fails silently. **The only
route is that the receiving sub-agent invokes the skill itself, because you told it to.** So:

1. **Take the skill's name from your own skill listing**, never from memory or from this file — it is
   the skill whose job is building a CDP filter from a request in words, and it may be renamed or
   re-homed. **If no such skill is listed, stop**: having the filter *tools* is not a substitute.
2. **Make "use that skill for this task" the first sentence of the message** — it is the only
   mechanism there is.
3. **Make the answer evidence it** — the provenance clause below — and check it before you write.
   You cannot verify provenance from the payload, only from what the answer says about how it was
   produced.

## The message

Clause zero is the "use the skill" sentence. After it, keep every clause: without "store it
programmatically" you get a link, without "verbatim" you get something retyped, and without
provenance you cannot tell a built filter from an improvised one.

> Build a filter on the project served by MCP server `<server>` that selects: **\<the request in
> plain words\>**. The filter must be rooted on **\<root entity\>** — a filter on any other root is
> unusable for me, so if that root cannot express the request, say so instead of switching roots.
> Then one property-set sentence, whichever applies: where your reference names a property set for
> this scope, *"Use the property set **\<property set\>**."*; where it names none, *"My reference
> names no property set for this scope — use the root's default property set."*
>
> The flow's brand is **\<brand\>**: build the filter for that brand's records, and never assume
> another brand or leave the brand to your own choice.
>
> **A skill needs this filter to configure a mechanic: return the complete filter JSON payload, not
> only the link** — verbatim, exactly as your compile returned it, in a fenced code block. I store it
> programmatically, so the payload is the deliverable. Include a link as well if you have one. Do not
> reformat, re-indent, abbreviate or retype it, and do not hand me the readable draft in its place.
>
> **Also give me the query draft** — the filter query text your build step compiled, in its own query
> language, in a second fenced code block. It is what I compare the stored filter's rendering
> against, and a prose sentence cannot be compared with a rendering. So: the prose sentence **and**
> the draft.
>
> Also tell me, in one line each: the root entity, the property set you built in, the project and
> environment you built on, and any assumption you made.
>
> And tell me how it was produced: **which skill you followed**, **which of its reference documents
> you read** to choose the root and the predicates (give their ids as the reference prints them),
> that the payload is your compile's confirmed output and **not** one you composed, patched or edited
> by hand, and that **the compile came back `status: ready` and `platform: accepted` with that root
> passed — and with the property set above passed too, where I named one**.
>
> If your build step returned no payload, say so and stop — do not substitute a draft, an earlier
> filter, or one written by hand.

## The answer is a return value, not a conversation

**The answer is the value the sub-agent returns when it finishes** — the reply path can fail one way,
and a finished sub-agent may be gone. So every *"ask once more"* below means a **fresh delegation**: a
new sub-agent, the whole message again including clause zero, plus one sentence naming what was
missing last time. It never means messaging a sub-agent that is already running, and never means
messaging one that has not answered.

**A delegation that produced no answer is not a delegation that produced a bad one.** They look
alike from where you sit and they need opposite moves:

| What you are holding | What it tells you | What to do |
|---|---|---|
| An answer that fails a check below | The delegation worked and gave you something you cannot use | That row's move, including discarding the body where the row says discard |
| **No answer at all** — unreachable, returns nothing, or nothing comes back | **The channel failed. This says nothing about the filter** — the work may be finished and undeliverable | **Launch a fresh sub-agent with the same message.** Never re-message the silent one |

**Budget: one fresh launch per condition on silence, then stop.** Report it as *a delegation you could
not get an answer out of* — **never** as an audience that could not be expressed, and never write the
condition unfiltered: no filter means *everyone*. Rank it at hand-over as a shortfall in your reach,
name the block, and say what the flow does not do while that condition is missing.

## What to check in the answer, before you write anything

| Check | If it fails |
|---|---|
| A filter payload is present, in a code block | Ask once more, restating why the payload is the deliverable. Still nothing: stop that condition and report it. Never reconstruct one. |
| The **query draft** is present too, in the language a stored filter renders back into | Ask once more, by name — see *The query draft* below. |
| The answer **evidences its provenance**: it names the skill it followed, cites reference documents it read, and states the payload is the confirmed output of its build step | **Ask once more, for the provenance specifically** — see *Provenance* below, which owns the budget. |
| The answer states the compile came back **`status: ready` and `platform: accepted`, with your root passed — and with your property set passed, where you named one** | Ask once more for that statement — see *The root statement* below. |
| The answer builds **every** clause of the request | If it diagnoses one clause as unbuildable, the delegation worked and what is open is a business decision. **Do not narrow the request yourself** — see *A clause that cannot be expressed* below. |
| The root is the one you asked for | Discard it. Do not re-root it yourself and do not adjust the scope to match — the scope came from the graph, not from the filter. Where you named no property set, the root's default is the expected answer and is not a mismatch; only a property set you did name and did not get is one. |
| The project and environment match the server the flow is on | Discard it and re-delegate naming the server. A wrong-project filter does not error; it silently describes a different audience. |
| It reports having built something, not having saved something | That skill saves nothing. If it claims to have saved or applied a filter, treat the answer as unreliable and report it. |
| Its stated assumptions are ones the user would accept | If an assumption is a business decision the marketer should make, take it back to the user rather than writing it. |

**The query draft.** Without it there is nothing to compare the stored filter with. If a second ask
produces none, proceed but put that condition in *Actions* as an instruction in their words, asking
them to check it does what they meant (`references/hand-over.md`).

**Provenance.** Ask again for the provenance specifically — that is itself a fresh launch, so **two
delegations on this condition in total**. If the second answer still does not evidence it, **discard
the body as unprovenanced** and report the condition: the payload can never settle this, since an
improvising agent returns the same shape. A vague "I built a filter" is a fail; so is a claim to have
followed the skill with nothing it read to show for it.

**The root statement.** Without it you do not know the root was checked. Then it is yours to
corroborate: have the payload rendered and see that it is about the entity you asked for. Corroborate
or discard; never write it on a bare "it validated". The property set half of the statement is the
only evidence you get that the set was passed — do not treat the payload as corroborating it. Where
you named a set and the statement does not cover it, ask once more; still missing, discard.

**A clause that cannot be expressed.** A diagnosed clause is a **finding**, not silence and not a bad
answer: what is open is **which filter to build instead**, and that is the marketer's decision.
**Never re-scope the filter yourself and delegate the narrower request on your own authority** — a
filter that quietly selects more people than the user asked for is the failure this skill exists to
prevent.

So: **stop that condition and put the choice to them** — drop the clause, replace it with one they
name, or drop that block — with what each would select, as the one blocking question the second round
is for (`SKILL.md` step 3). **With their decision in hand, delegate once more:** a changed request is
not the re-delegation CRITICAL 6 forbids. If they cannot be reached, leave that condition unbuilt,
never unfiltered.

With provenance and the root statement in hand the payload is already platform-checked — **do not
re-run that checking**, and do not treat their absence as something you can work around.

## Writing it

The payload goes into the block's condition **as it stands**, together with the entity scope, in the
same call — never the body alone, and never the scope alone. Which field takes which, and the shape
around them, is the write-path sub-folder's to say. One condition block per call.

Keep the query draft the sub-agent reported, and its prose sentence. You need the draft in the next
step, and the sentence for the hand-over.

## Verifying it after the write

Read the block back and have the stored filter **rendered** into readable query form, then compare its
**meaning** with the sub-agent's query draft:

- The rendering **normalises** — a condition can come back phrased differently, or inverted into an
  equivalent form, with nothing wrong. A textual diff of the two will raise false alarms; do not
  report one as a mismatch.
- A **genuine** mismatch is a difference in what the filter would select: a different attribute, a
  missing or extra condition, a negation that is not equivalent, a quantity or period other than the
  one the marketer stated, or anything the rendering says it could not read. Judge clause by clause —
  those are what this check is for.

**This check settles structure; it cannot settle a value that names a thing.** Where a clause names
something out of the project's catalogue, the rendering prints a stored value the draft printed as a
name, so a correct filter always looks mismatched at that spot.

**So take that spot out of the comparison**: do not discard, re-send, rebuild or report a mismatch on
it — and do not read a number as proof of resolution either. Do not go looking for another comparison
at this surface; there is none these tools support.

Record it as **verified by field, not by meaning** — never as unverified — and turn it into one
*Actions* item in the user's words (*"check the filter uses the right product lists — Cart and
Favourites"*). None of the reasoning above travels with it (`references/hand-over.md` → *What never
goes in*).

To get the rendering, find the read-only rendering capability in your tool list; if there is none,
have a sub-agent render it. Never read the stored payload by eye and call that verified, and never
skip the step.

On a genuine mismatch, the filter exception in `SKILL.md` step 6 applies: one re-send of the same
body, alone, then a stop.
