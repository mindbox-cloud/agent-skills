---
name: filter-build
description: >-
  Build or edit a CDP filter from a natural-language request. Use for audience or object
  selections and changes to existing filter JSON.
  Russian triggers: "собери фильтр", "построй фильтр", "сделай выборку", "нужен сегмент",
  "выбери клиентов, которые...", "фильтр по товарам", "добавь условие в фильтр".
  English triggers: "build a filter", "make a segment", "select customers who...",
  "filter products", "edit this filter", "add a filter condition".
  Returns a platform-confirmed filter and a link when available; does not save a segment.
  For explaining an existing filter use filter-explain.
argument-hint: "who to select, or the existing filter JSON and the requested change"
author: AI tribe
---

# Filter build

Use the project and environment already settled in the conversation. Find that server's
`filter_*` tools and `feedback`. One connection may serve several projects. When its tools
require `tenant`, use the exact project system name; if it is unknown, discover accessible
projects with `tenants_list`. Choose the sole applicable project or ask when several fit.
Explain a missing connection. Keep all calls on the chosen server and project: catalogue
IDs belong to that project.

Examples below are independent and use synthetic requests. Replace `<tenant>` with the
confirmed project system name before calling; omit it only if the tool's schema has no
`tenant` argument. Use the actual request, editor and returned records. Tool names may have
a server prefix.

## 1. Read the current entry point

At the start of every build or edit, freshly read the connected server's root README, even
if you read it in an earlier task:

```text
filter_wiki_read({"tenant": "<tenant>", "paths": ["README.md"]})
```

This is the maintained, up-to-date entry point. Use its current navigation and tool guidance
alongside the workflow below. Reuse pages within this build; follow returned paths instead
of reconstructing filenames. Do not preload the full syntax and build guides.

For an edit, retain the complete set of existing conditions, plus the requested changes.
Reuse a confirmed `finalSql` and selected records from this conversation. Otherwise call
`filter_json_to_sql` with the complete original JSON; keep its selected records and inspect
`unlabelled` and `unsupported`. A list URL cannot be imported by these tools: ask for JSON
if the filter is not already in context. Do not silently drop an unreadable condition.

## 2. Choose the root and editor

If either the root or property set is unknown, read the root catalogue:

```text
filter_wiki_read({"tenant": "<tenant>", "paths": ["root/README.md"]})
```

Choose the root by what one result row represents: customers, products, orders or another
listed object. Objects mentioned in conditions may be related to that root. Preserve any
root and property set supplied by the caller. For a standalone list, use the root's `Default`
editor; for an embedded mechanic, use its required editor. Ask if the destination is unclear
and the choice changes what can be expressed.

Open the chosen editor page to learn its available fields, relations and restrictions. If
both values were supplied, go directly to that page after step 1. API identifiers and wiki
paths have different casing:

| `root` | `filterablePropertySet` | Wiki path |
|---|---|---|
| `User` | `Default` | `root/user.default.md` |
| `User` | `ScenarioInboundEvent` | `root/user.scenario_inbound_event.md` |
| `RetailProduct` | `Default` | `root/retail_product.default.md` |

```text
filter_wiki_read({"tenant": "<tenant>", "paths": ["root/user.scenario_inbound_event.md"], "root": "User", "filterablePropertySet": "ScenarioInboundEvent"})
```

Use `filterablePropertySet` as the argument name, not `filter_property_set`. Copy document
paths including underscores and `.md`; `README.md` is uppercase. If a path is unknown, discover
it with `filter_wiki_ls({"tenant": "<tenant>", "path": "root"})`. Carry the chosen root and property set into subsequent
wiki, validation and compile calls that accept them. Never widen the editor to bypass a refusal.

## 3. Find the meaning and documented constructs

For business labels or a request that may have a recipe, search recipes and the glossary:

```text
filter_wiki_grep({"tenant": "<tenant>", "pattern": "реактив|спящ|dorman(?:t|cy)|reactivat", "paths": ["recipe", "glossary"], "root": "User", "filterablePropertySet": "Default"})
filter_wiki_read({"tenant": "<tenant>", "paths": ["glossary/reactivation.md", "recipe/user.reactivation.md"], "root": "User", "filterablePropertySet": "Default"})
```

Read relevant hits, especially `Use for`, `Not for`, `Not offered` and `Ask`. For straightforward
conditions, open the editor's linked field or relation pages directly. Learn the full construct,
including which conditions must hold on the same related object. Read syntax guidance only
when needed. Reuse documents already read for this build.

`pattern` is a case-insensitive regular expression; it does not translate or stem words.
Search useful stems and alternatives explicitly. For example, `реактив` matches forms such
as `реактивация` and `реактивировать`; `спящ` finds `спящие` and `спящих`; `dorman(?:t|cy)`
covers `dormant` and `dormancy`; `reactivat` covers `reactivate` and `reactivation`.

For subscriptions and mailings, respectively:

```text
filter_wiki_grep({"tenant": "<tenant>", "pattern": "подпис(?:к|ок|ан)|subscri(?:b|pt)", "paths": ["pattern", "field"], "root": "User", "filterablePropertySet": "Default"})
filter_wiki_grep({"tenant": "<tenant>", "pattern": "рассыл(?:к|ок)|mailings?", "paths": ["lookup", "glossary"], "root": "User", "filterablePropertySet": "Default"})
```

The first covers `подписка`, `подписок`, `подписан`, `subscribe` and `subscription`; the
second covers `рассылка`, `рассылок`, `mailing` and `mailings`. These cover the shown forms,
not every synonym. Use only patterns relevant to the request, read useful hits and follow
the `skip`/`limit` footer when results continue. `paths` prioritizes sections rather than
excluding all others. Search excerpts alone are not the full rule.

## 4. Resolve remaining business questions through help

If the local pages leave a product or business meaning unclear, search `help` explicitly.
Its absence from the README map does not establish that help is unavailable:

```text
filter_wiki_grep({"tenant": "<tenant>", "pattern": "реактив|спящ|dorman(?:t|cy)|reactivat", "paths": ["help"], "root": "User", "filterablePropertySet": "Default"})
```

Read relevant returned `help/` articles. Use business wording here, not internal field names.
Then check the proposed condition against the editor and filter reference. A help article
can explain a feature without making it available in this editor. If help is unavailable or
the meaning remains unclear, ask about the point that changes the audience.

## 5. Resolve catalogue records before writing SQL

When conditions name project records, discover the catalogue types and read the relevant pages:

```text
filter_wiki_ls({"tenant": "<tenant>", "path": "lookup", "root": "User", "filterablePropertySet": "Default"})
filter_wiki_read({"tenant": "<tenant>", "paths": ["lookup/segment.md"], "root": "User", "filterablePropertySet": "Default"})
filter_search_entities({"tenant": "<tenant>", "context": "Customers in the Example segment", "types": ["segment"], "query": "Example segment", "mode": "match"})
```

`query` is the name to find; `context` supplies the surrounding request, not another search
query or a semantic selection guarantee. Batch names in `queries` when they share types and
parent; do not send both `query` and `queries`. For custom-field values, `of` is the field
name. Choose records by meaning, type and parent. A score ranks wording similarity; it is
not confidence that the record expresses the request.

Browse one catalogue type, or search literal substrings with `mode=list`:

```text
filter_search_entities({"tenant": "<tenant>", "context": "Find the requested customer segment", "types": ["segment"], "mode": "list", "pageSize": 20})
filter_search_entities({"tenant": "<tenant>", "context": "Find the requested customer segment", "types": ["segment"], "query": "Example", "mode": "list", "pageSize": 20})
```

These are two separate searches. For another page of either, repeat the same arguments and
add `"cursor": "<exact nextCursor from that response>"`. If the cursor is rejected, restart
the same listing once without `cursor`; report a repeated failure through feedback. Follow
`nextCursor` as needed; an incomplete or truncated response does not establish absence. For segments, `pageSize` counts
segmentations. A matching segmentation can include child segments whose names do not
match the query. A segmentation and one segment within it are different selections: keep
the chosen level and parent.

Keep every chosen `ref` object, including all `ids`, `type`, `name`, `parent` and `tool` when
present. Use its canonical name in SQL. Resolve every named catalogue value, including any
parent the condition needs; do not invent an ID or use an unrelated record to fill a slot.

## 6. Clarify disputed choices

Ask together about remaining ambiguous records or interpretations that would change the
selection. Show the candidate names and parents and explain the difference. A single clear
match needs no confirmation. Reuse answers already given; if the user delegates a choice,
make it and name the assumption. Establish any omission or alternative with the user before
building a narrower filter. A failed lookup alone is not permission to drop a condition.

## 7. Draft and validate

Write the complete FilterSQL draft using the documented constructs and chosen records.
Validate with the same root and property set. In `coverage`, quote the relevant user wording
and record your chosen representation and why you chose it, including assumptions and agreed omissions.
It records intent; it is not a proof that the SQL matches the request.

A standalone example with no catalogue references:

```text
filter_sql_validate({
  "tenant": "<tenant>",
  "sql": "FROM User WHERE user.age >= 30",
  "root": "User",
  "filterablePropertySet": "Default",
  "coverage": [{
    "request": "Customers aged at least 30",
    "decision": "user.age >= 30",
    "reason": "At least includes the boundary; age is measured in years."
  }]
})
```

Proceed on `status: valid`. When `reference` lines are returned, match each to the already
chosen record by type, value, parent and occurrence. Copy its `slot` to that record's
`selected` entry; keep every returned ID. Slots belong to this draft: refresh them after
changing SQL. Follow reported `repair` guidance and validate the corrected draft before
compilation. If validation exposes a missing lookup, resolve it before proceeding.

## 8. Compile the validated draft once

For the no-reference example above:

```text
filter_compile_preview({
  "tenant": "<tenant>",
  "sql": "FROM User WHERE user.age >= 30",
  "root": "User",
  "filterablePropertySet": "Default",
  "selected": [],
  "includeFilterJson": false
})
```

For a draft with references, fill `selected` with the chosen `ref` objects and their current
slots. Set `includeFilterJson=true` on this first compile if the user requests JSON or another
AI agent, tool or skill needs a filter to configure a mechanic. Otherwise return the link.
A successful result without a list link includes JSON automatically.

A successful compile already includes the platform check: do not compile again to confirm it.
Require `status: ready` and `platform: accepted`, then compare `finalSql` with the full request.
Validation and platform acceptance do not establish semantic correctness. Fix a missing,
extra or misinterpreted condition before handing over the result.

On failure, follow the specific `repair` and pass returned `feedbackState` unchanged into the
next compile. Change the draft or selections only when the problem calls for it. A platform
processing failure or access denial is not a SQL defect: preserve the conditions, report the
blocker, and retry only after it can be resolved. Never present an unsuccessful preview or a
stale result after a failed edit as complete.

## 9. Hand over the result

Copy the returned link and explain what the user will see: the selected objects, effective
conditions, time windows, exclusions, material assumptions and agreed omissions. Use the
conversation's language and identify the project and environment. Keep SQL out of the answer
unless requested. When JSON is needed, pass the complete returned payload unchanged to its
caller; a link or SQL cannot replace it. If no link is available, provide the returned JSON
and the tool's explanation of the limitation. Building does not save a segment or change data.

## 10. Handle an incorrect result with feedback

If the user says the result is wrong, clarify the mismatch and ask for a link to a filter
that represents the intended selection. Use that URL as a reference in feedback; request
its JSON as well if you need to inspect or edit it. Send the report through the same server:

```text
feedback({"tenant": "<tenant>", "feedback": "<report filled from the template below>"})
```

Fill this template from the current session. Keep both URL fields explicit; write
`not provided` or `unavailable` when the source does not exist. Never invent a trace ID.

```text
Request: <the user's filter request>
Project and editor: <project system name, environment, root, filterablePropertySet>
Expected: <the intended selection>
Actual: <the observed mismatch>
Correct filter URL: <URL of the correct filter supplied by the user, or not provided>
Trace URL / ID: <technical session trace URL or trace ID if available, or unavailable>
Technical trace:
1. Tool: <exact tool name>
   Arguments: <actual JSON arguments>
   Result: <actual response text or error>
2. Tool: <next tool name>
   Arguments: <actual JSON arguments>
   Result: <actual response text or error>
<continue in chronological order, including failed attempts and repairs>
```

The technical trace is the sequence of calls and responses, not a prose summary of what
you tried. Include the relevant wiki and catalogue lookups, validation and compile calls,
with SQL, selected refs, coverage, finalSql, platform verdicts and feedbackState when present.
Remove credentials and personal data. Report only observed behavior. A missing reference
filter or trace URL does not prevent reporting: include the visible technical trace anyway.

`feedback` accepts one text report, not a trace attachment. Say it was submitted only after
the tool succeeds. If unavailable, provide a copyable report. Continue a requested correction
through this workflow; do not promise a reply from the feedback channel.
