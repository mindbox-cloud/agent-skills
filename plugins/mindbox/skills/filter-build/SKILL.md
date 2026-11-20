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
  Don't use when: the filter only needs explaining and not changing — that is filter-explain;
  the question is how many or how much, which is a counting task, not a filter.
argument-hint: "who to select, or the existing filter JSON and the requested change"
metadata:
  version: 1.1.0
---

# Filter build

Find the connected server's `filter_*` tools, `entities_search` and `feedback`. Explain a missing
connection.
Keep all calls on that server: catalogue IDs belong to the project it serves.

Examples below are independent and use synthetic requests. Each names a tool and the
arguments that carry the request; fill in the rest from the schema the server exposes. Use
the actual request, editor and returned records. Tool names may have a server prefix.

## 1. Read the current entry point

At the start of every build or edit, freshly read the connected server's root README, even
if you read it in an earlier task:

Read it with `filter_wiki_read`, `paths = ["README.md"]`.

This is the maintained, up-to-date entry point. Use its current navigation and tool guidance
alongside the workflow below. Reuse pages within this build; follow returned paths instead
of reconstructing filenames. Do not preload the full syntax and build guides.

For an edit, retain the complete set of existing conditions, plus the requested changes.
Reuse a confirmed `finalSql` and its chosen hits from this conversation. Otherwise call
`filter_json_to_sql_core` with the complete original JSON as `filterJson` (and `root` when the
editor is known). Keep the records it confirms for `searchItems`, inspect `Unlabelled`,
`Unsupported` and its problems, and find each of its "Records to find before compiling" in
step 5. `NOT_READ` gives no SQL to edit: name the conditions it lists to the user. A list URL
cannot be imported by these tools: ask for JSON if the filter is not already in context. Do
not silently drop an unreadable condition.

## 2. Choose the root and editor

If either the root or property set is unknown, read the root catalogue:

`filter_wiki_read` with `paths = ["root/README.md"]`.

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

`filter_wiki_read` with `paths = ["root/user.scenario_inbound_event.md"]`, `root = "User"`
and `filterablePropertySet = "ScenarioInboundEvent"`.

Use `filterablePropertySet` as the argument name, not `filter_property_set`. Copy document
paths including underscores and `.md`; `README.md` is uppercase. If a path is unknown, discover
it with `filter_wiki_ls` at `path = "root"`. Carry the chosen root and property set into subsequent
wiki and compile calls that accept them. Never widen the editor to bypass a refusal.

## 3. Find the meaning and documented constructs

For business labels or a request that may have a recipe, search recipes and the glossary:

`filter_wiki_grep` with `pattern = "реактив|спящ|dorman(?:t|cy)|reactivat"` over
`paths = ["recipe", "glossary"]`, then `filter_wiki_read` on the hits —
`paths = ["glossary/reactivation.md", "recipe/user.reactivation.md"]`. Both carry
`root = "User"` and `filterablePropertySet = "Default"`.

Read relevant hits, especially `Use for`, `Not for`, `Not offered` and `Ask`. For straightforward
conditions, open the editor's linked field or relation pages directly. Learn the full construct,
including which conditions must hold on the same related object. Read syntax guidance only
when needed. Reuse documents already read for this build.

`pattern` is a case-insensitive regular expression; it does not translate or stem words.
Search useful stems and alternatives explicitly. For example, `реактив` matches forms such
as `реактивация` and `реактивировать`; `спящ` finds `спящие` and `спящих`; `dorman(?:t|cy)`
covers `dormant` and `dormancy`; `reactivat` covers `reactivate` and `reactivation`.

For subscriptions and mailings, respectively:

Two searches with `filter_wiki_grep`: `pattern = "подпис(?:к|ок|ан)|subscri(?:b|pt)"` over
`paths = ["pattern", "field"]`, and `pattern = "рассыл(?:к|ок)|mailings?"` over
`paths = ["lookup", "glossary"]`.

The first covers `подписка`, `подписок`, `подписан`, `subscribe` and `subscription`; the
second covers `рассылка`, `рассылок`, `mailing` and `mailings`. These cover the shown forms,
not every synonym. Use only patterns relevant to the request, read useful hits and follow
the `skip`/`limit` footer when results continue. `paths` prioritizes sections rather than
excluding all others. Search excerpts alone are not the full rule.

## 4. Resolve remaining business questions through help

If the local pages leave a product or business meaning unclear, search `help` explicitly.
Its absence from the README map does not establish that help is unavailable:

The same `filter_wiki_grep` pattern over `paths = ["help"]`.

Read relevant returned `help/` articles. Use business wording here, not internal field names.
Then check the proposed condition against the editor and filter reference. A help article
can explain a feature without making it available in this editor. If help is unavailable or
the meaning remains unclear, ask about the point that changes the audience.

## 5. Resolve catalogue records before writing SQL

When conditions name project records, find every one of them before the draft. One grep over
`lookup` names the catalogue of each record; then search them all in one turn:

`filter_wiki_grep` with `pattern = "сегмент|segment|бренд|brand"` over `paths = ["lookup"]`,
then `entities_search` with `query = "Example segment"` and
`sources = ["segmentation", "segment"]`.

`query` is the name as the user put it; `sources` is a JSON array of catalogue names — name
every plausible catalogue for an ambiguous name. An empty `query` is only for a closed list.
Read `meta` per catalogue: only a catalogue that answered with nothing found means the record
is absent; a timed-out, unavailable or forbidden catalogue rules nothing out. When `shown` is
less than `found`, make `query` more precise. Choose records by meaning, entity, owner and
parent; hit order ranks wording similarity, not confidence that the record expresses the
request. A request that names a field takes the value of that field among the hits; a segment
only when the user speaks of a segment. A segmentation and one segment within it are
different selections: keep the chosen level and parent.

Keep every chosen hit of `items[]` whole — `ref`, `entity`, `name`, `ids`, `type`, `owner`,
`status`, `parent`, `children` — exactly as returned; do not trim, merge or rebuild one.
Resolve every named catalogue value, including any parent the condition needs; do not invent
an ID or use an unrelated record to fill a slot.

## 6. Clarify disputed choices

Ask together about remaining ambiguous records or interpretations that would change the
selection. Show the candidate names and parents and explain the difference. A single clear
match needs no confirmation. Reuse answers already given; if the user delegates a choice,
make it and name the assumption. Establish any omission or alternative with the user before
building a narrower filter. A failed lookup alone is not permission to drop a condition.

## 7. Draft from the chosen records

Write the complete FilterSQL draft using the documented constructs and chosen records. Every
record is the call of its catalogue with the id from its hit's `ids`, as the wiki's
`references/calls.md` writes it — for example `BRAND(17)`, `SEGMENT(336, 330)` (segmentation
id, then segment id), `CUSTOM_FIELD_VALUE(43, 8267)`. Never put a record's name or a bare
literal in a record's slot. FilterSQL is not SQL: a query starts with `FROM <Root>`, with no
`SELECT` and no alias for the root. There is no separate validation call: the compile in
step 8 is the validation.

## 8. Compile the draft once

Call `filter_compile_core` with the draft, the editor and every chosen hit:

```json
{
  "sql": "FROM User WHERE user.age >= 30 AND EXISTS segment AS s:SegmentMembership (s.segmentation = SEGMENTATION(336) AND s.segment = SEGMENT(336, 330))",
  "root": "User",
  "filterablePropertySet": "Default",
  "searchItems": [{ "...": "every chosen hit from entities_search items[], unchanged" }],
  "includeFilterJson": false
}
```

The example only shows where things go: the construct comes from the editor's pages, and the
server matches the calls to `searchItems` strictly by id. Pass arrays and objects as JSON
values and `includeFilterJson` as `true`/`false`, never as strings. Set `includeFilterJson=true`
on this call if the user requests JSON or another AI agent, tool or skill needs a filter to
configure a mechanic. Otherwise return the link. A successful result without a list link
includes JSON automatically. Send one compile at a time, each after the previous answer.

`FILTER_OK` already includes the platform check: do not compile again to confirm it. Compare
`finalSql` with the full request. Platform acceptance does not establish semantic
correctness. Fix a missing, extra or misinterpreted condition before handing over the result.

On `FILTER_PROBLEMS`, follow each problem's `repair`. A problem with `reference` names the
record its slot needs: find it with `entities_search` over `reference.catalogues`, rewrite the
slot with a call from `reference.calls` and the hit's id, add that hit to `searchItems`, and
compile again. Do not repeat an unchanged failure. A repair changes how a condition is
written, never which condition or record it is; a different record is a question to the user.
A platform processing failure or access denial is not a SQL defect: preserve the conditions,
report the blocker, and retry only after it can be resolved. Never present an unsuccessful
compile or a stale result after a failed edit as complete.

## 9. Hand over the result

Copy the returned link and explain what the user will see: the selected objects, effective
conditions, time windows, exclusions, material assumptions and agreed omissions. Use the
conversation's language. Keep SQL out of the answer unless requested. When JSON is needed, pass
the complete returned payload unchanged to its caller; a link or SQL cannot replace it. If no
link is available, provide the returned JSON
and the tool's explanation of the limitation. Building does not save a segment or change data.

## 10. Handle an incorrect result with feedback

If the user says the result is wrong, clarify the mismatch and ask for a link to a filter
that represents the intended selection. Use that URL as a reference in feedback; request
its JSON as well if you need to inspect or edit it. Send the report through the same server:

Call `feedback`, passing the report below as its `feedback` argument.

Fill this template from the current session. Keep both URL fields explicit; write
`not provided` or `unavailable` when the source does not exist. Never invent a trace ID.

```text
Request: <the user's filter request>
Editor: <root, filterablePropertySet>
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
you tried. Include the relevant wiki and catalogue lookups and compile calls, with SQL,
`searchItems`, finalSql, the platform verdict and problems with their repairs when present.
Remove credentials and personal data. Report only observed behavior. A missing reference
filter or trace URL does not prevent reporting: include the visible technical trace anyway.

`feedback` accepts one text report, not a trace attachment. Say it was submitted only after
the tool succeeds. If unavailable, provide a copyable report. Continue a requested correction
through this workflow; do not promise a reply from the feedback channel.
