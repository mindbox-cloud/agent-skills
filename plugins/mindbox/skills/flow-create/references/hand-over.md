# Writing the hand-over

Read this when you reach step 8. The rules behind it — what "done" means and how to rank a
shortfall — are in `SKILL.md`.

- [The link](#the-link)
- [The three parts](#the-three-parts)
- [Reporting a stop under CRITICAL 6](#reporting-a-stop-under-critical-6)
- [What never goes in](#what-never-goes-in)
- [A filled example](#a-filled-example)

The reader is a marketer who has never seen the flow's internals. **Same meaning, fewer words** is
the whole editorial rule: every sentence that is neither a fact they need nor an action they can take
comes out.

## The link

Hand over a link to **the draft version you built** — the project's admin base URL, the flow's
scenario path, and the marker that names the version you built rather than whichever one the
platform would pick. **A link without an explicit version marker is not guaranteed to open the
version you built**, so it is not a hand-over on its own.

**The path shape is a platform value:** read it in the `flows` domain's flow-address document
(`urls`), navigating to it from the domain index, and **never assemble one** from a pattern you have
seen. If it is missing, say you cannot produce a link, name the flow and the version number so they
can open it, and rank the gap (CRITICAL 2).

If the link names the version **by identity**, say nothing about it; if it selects **by state**, one
clause — the link opens the draft *now*, since re-opened after launch the same address can show the
running version with nothing saying it switched.

Put the version number in the same line as the link, and stop there. Where the draft was copied from a
launched version (step 1b (iii)), that number is the **new draft version's**, never the source's.

The admin base URL is asked for in the step-3 batch (`references/questions.md`), never here.

## The three parts

Exactly three, in this order, and nothing else.

### 1. What was built

Three parts, in the language the user is working in: **what starts the flow**, **what it does step
by step**, **what it is for**. If a flow-summary skill is in your listing, reuse its output format
instead — it is the same shape, kept in one place. Keep it compact, and expand only if the user asks.

Refer to a block by its name in quotes or by its role ("the 3-day wait", "the email step"). No block
ids, no type tags, no row versions, no field or value names.

### 2. Actions

**A checklist, and every item something the user can do.** Two kinds belong here and nothing else:

- what they must **check or fill themselves**, and
- what they must **tell you** so that you can do it — the name of a mailing to attach, which of two
  candidates they meant, an audience you could not build without their words.

Rules for the items:

- **Each item is an instruction, not a statement.** *"Check that the filters use the right product
  lists — Cart and Favourites"* is an action. The same thing prefaced with an account of how the
  platform stores those lists is a statement with an action buried in it: delete the preface.
- **Where an item is a shortfall, its consequence goes in the same line**, in the user's terms —
  *"until a mailing is chosen the flow will not run"*, or *"this needs a small follow-up from you and
  the rest is built"*. `SKILL.md` says the two kinds are not comparable, so **put the headline ones
  first**: a severe item listed beside three routine ones reads as routine.
- **What did not land is stated as a limitation on your side, not as a chore you are handing over** —
  and say what the flow does *instead* right now, so they can judge the risk ("the condition passes
  everyone").
- **The last item always asks them to look at the result and say whether it is what they wanted** —
  the trigger, the steps and their order, the audience each condition selects, and each mailing. That
  look is what acceptance means, and nothing in *Suggestions* happens before it.
- **Beyond that one, if there is nothing to do, say so in one line.** Do not pad the section.

### 3. Suggestions

**Offers only** — *"if you want, I can add an A/B test between the condition and the send"*. A note
about what you did not do and why is not an offer: drop the note, keep the offer. Nothing to offer,
no section.

**Where the flow validated clean, the first offer is the test run**: say that once they are happy with
it you can switch the flow to testing mode, so the platform runs it as a test without launching it for
real. It is an offer, never a step you take on your own — you act on an explicit yes and on nothing
else, and you never launch, pause, stop or delete the flow (`SKILL.md`, top and step 8).

**When they accept, the reply also asks once for a 1-to-10 rating of the build**, in one unobtrusive
line — never before acceptance, and once per thread: an unanswered ask is spent
(`references/feedback-and-selfcheck.md` → *Rating the build*).

## Reporting a stop under CRITICAL 6

A run that stopped short is handed over in the same three parts, not as an apology. **Say plainly
what is not built and what the flow will therefore not do**, in business terms, in *what was built*;
put the choice in **Actions** as something they decide, never as work they perform.

Three things belong in it and nowhere else:

- **The objection verbatim**, where there was one — platform vocabulary included (the exception to
  *What never goes in*).
- **What you tried**, in one line — enough that nobody repeats it, not a log of calls.
- **The choices you can see**, as choices. "This needs a value no write of mine can set — do you
  want it set another way, or the flow handed over without it?" is an Action. "Please open the editor
  and fill it in" is not: that is handing them the job, which CRITICAL 6 does not license.

**Do not report the budget, the count, or the rule's name.** That it stopped at six calls belongs
to the run; that the branch is unfinished and why belongs to the reader.

## What never goes in

Cut it, whatever else is true of it, if the user cannot act on it:

- **System names, ids, type tags, row versions, field names, discriminators** — anything from the
  platform's own vocabulary.
- **Tool behaviour** — how a write answered, that a listing had to be re-queried. If it is worth
  reporting, it goes to the `feedback` tool, not to the reader.
  > **One exception: a message step 7 tells you to pass on verbatim (CRITICAL 5) outranks this.**
  > Never tidy such a message out of the hand-over, and never paraphrase it into something acceptable.
- **A platform quirk with no user-side move.** A value the read path regenerates and nothing can
  confirm (`SKILL.md` step 6) is the standing example: naming it buys them nothing.
- **Explanations of how the link is built**, of why a rule is what it is, or of a constraint that no
  longer bears on a decision. Folder reachability is the case to watch: it belongs in the **question**,
  before the choice is made (`references/questions.md`) — after the fact it is an essay nobody reads.
- **Your own reasoning about what you did not do**, unless it turns into an offer in *Suggestions*.

## A filled example

One shape that satisfies the three parts; `‹link›` stands in because this file must not carry a path
shape. Write yours in the language the user is working in.

> Draft of the "Abandoned cart reminder" scenario, version 3: ‹link›
>
> **Trigger** — the customer left a cart behind.
> **Flow logic** — wait 3 hours → check "still hasn't bought" → email "Come back to your cart";
> everyone else leaves the flow. At most once a week per customer.
> **Goal** — bring the customer back to an unfinished order.
>
> **What to do**
> - Fill in the "Come back to your cart" mailing with me — I created it and gave it a name and a
>   purpose, but its content still needs your choices.
> - Then activate it — until someone does, the flow prepares the message and sends nothing, and there
>   is no way for me to activate it.
> - Check that the filters use the right product lists — Cart and Favourites.
> - Tell me whether an order in any status counts as a purchase, or only a paid one: for now the
>   condition passes everyone.
> - Look the flow over and tell me whether this is what you wanted.
>
> **If you want, I can**
> - switch the flow to testing mode once you're happy with it, so it runs as a test without going live;
> - add an A/B test between the condition and the send;
> - add an SMS to anyone who hasn't opened the email within a day.

No ids, no field names, no account of the tools, no folder essay; the blocking item comes first and
every line under *What to do* starts with something to do.

# References

- `SKILL.md` → CRITICAL 6 — when a run stops short, and what the exit is
- `SKILL.md` → step 8 — the rule this file serves, and testing mode after acceptance
- `SKILL.md`, top — the two kinds of shortfall, and why the ranking carries the message
- `references/questions.md` — what belongs in the question rather than in the hand-over
- `references/feedback-and-selfcheck.md` — the rating ask, and the self-check findings that land here
- a flow-summary skill, when one is listed → `## Output format` — the same shape part 1 describes;
  the three parts above stand on their own when it is not
