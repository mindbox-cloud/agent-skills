# Feedback message examples

**When to read:** you are composing the text for MCP `feedback` following the protocol in
`SKILL.md` (section "Skill feedback via MCP feedback") and want to
see what well-filled `problem`/`context` look like on real
runs. Read it if you are unsure where the line between "the gist of the problem" and "context" lies,
or when the user asked to pass on their words and you need to understand what to add
to `context` without replacing their wording.

**Return:** to the "Skill feedback via MCP feedback" section in `SKILL.md`.

The format is the same everywhere:

```text
[skill-feedback:v1]
skill=<email|email-ops>
skillVersion=<metadata.version of that skill's SKILL.md>
source=user-report|agent-observation
problem=<the gist: what blocks the work, what went wrong>
context=<inputs and requests, how the situation came about, verbatim backend errors>
area=<layout|save|preview|gallery|personalization|other>
```

`area` is optional — add it only if the area is obvious. There is no separate field
for "reproduced/not reproduced": the fact of reproduction is part of
`context` (run date, one-off campaign, inputs).

## Example 1. user-report: the layout fell apart during migration

Initiated by: the user. Their words became `problem` verbatim, the agent
added `context`.

```text
[skill-feedback:v1]
skill=email
skillVersion=1.2.0
source=user-report
problem=Migration from another email platform silently builds the wrong layout: two adjacent images from one row of the source fall apart into a column.
context=Source: a 1200px container, two sibling images of 600px each. The source analysis marked them as two full-width ones, so the generator built two 12+12 rows instead of 6+6. There were no backend errors — the migration was formally "successful", but the rows in the email do not match the source. It repeated on several emails in a row.
area=layout
```

Why it is done this way:

- `problem` is the user's words, not a retelling and not "packaging" into one sentence.
- `context` explains **why** the error went unnoticed: the backend did not refuse,
  the source analysis looked correct, the result was simply semantically wrong.
- `area=layout` — the area is obvious from the description, so the field is included.

## Example 2. agent-observation: partial persist on save

Initiated by: the agent. The agent writes both fields — it saw the problem itself and
can describe it with technical precision.

```text
[skill-feedback:v1]
skill=email-ops
skillVersion=1.2.0
source=agent-observation
problem=visual_template_save can store the visual template but not attach it to the campaign; from the response it is easy to take this for success.
context=One-off campaign in a test project, run 2026-09-02. A save with a stale mailingRowVersion returned ChangeConflict with the verbatim text «The template itself WAS stored — only the campaign was not». A repeated campaign_get showed: the template grew to rowVersion=3, while the body attached to the email stayed at rowVersion=2. The subagent meanwhile reported "saved".
area=save
```

Why it is done this way:

- `problem` is one or two lines: what exactly blocks the work and why it is dangerous.
- `context` is a verifiable chain: date, campaign, verbatim error,
  version numbers, what happened in the end.
- The fact of reproduction ("run 2026-09-02, rowVersion=3 with body=2") goes
  in `context`, not in a separate field.

## Example 3. agent-observation: the tool description does not match the actual DTO

Initiated by: the agent. The problem is in the tool documentation, not in the
backend behavior; the context makes this clear.

```text
[skill-feedback:v1]
skill=email-ops
skillVersion=1.2.0
source=agent-observation
problem=The gallery_images_list description promises fields that are not in the actual response — the skill documentation has nothing to rely on.
context=Run 2026-09-02, a test project. The tool description lists size/date; the actual response for the project contained only name, fileExtension, isSystem, url. With a .webp filter, some URLs led to a converted PNG copy, some — directly to the original file. Both kinds of URL are valid for insertion into an email.
area=gallery
```

Why it is done this way:

- `problem` separates: the tool works, but its description is misleading.
- `context` lists the actual set of fields and the observation about `.webp`
  — this is what a developer will check first.
- No personal data, signed URLs or email content — only
  the actual structure of the response.

## Antipatterns

### Antipattern 1. Too abstract a `problem`

```text
problem=The gallery works incorrectly.
```

Why it is bad: it is unclear what exactly blocks the work — search, upload, URL, the tool
description. A developer will not be able to tell this report from a dozen similar ones.

Fix: add specifics — "the tool description promises size/date that are not
in the response" (see Example 3).

### Antipattern 2. `context` without inputs

```text
problem=Two images fell apart into a column.
context=The error was reproduced in a test run.
```

Why it is bad: `context` answers none of the questions — what the inputs
were, how wide the container was, what the backend returned, on which campaign
it was reproduced.

Fix: add the container, the image widths, the absence of a backend error
(see Example 1).

### Antipattern 3. Replacing the user's words

The user said:

> The images from the old email platform fell apart into a column when I was migrating the email.

Wrong:

```text
problem=The layout does not adapt to mobile devices.
```

Why it is bad: the agent made up the problem for the user — the user did not
say anything about mobile devices. With `user-report`, `problem` is their words
verbatim; the agent puts its own interpretation in `context` as a separate phrase.

Right:

```text
problem=The images from the old email platform fell apart into a column when I was migrating the email.
context=Migration from another email platform, a 1200px container with two sibling images of 600px each. The backend returned no errors, but the result is two full-width rows instead of 6+6.
```
