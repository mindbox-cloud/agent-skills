---
name: filter-explain
description: >-
  Explain existing CDP filter JSON in plain language, including conditions and values that
  could not be read. Use when asked what a filter or segment selects. Does not change it.
  Russian triggers: "объясни фильтр", "что делает этот фильтр", "кого выбирает эта выборка",
  "разбери фильтр", "объясни условия сегмента".
  English triggers: "explain this filter", "what does this segment select",
  "who matches this filter", "explain these filter conditions".
  For building or editing use filter-build.
argument-hint: "the complete filter JSON from the project"
author: AI tribe
---

# Filter explain

## Choose the project and input

Use the project and environment settled in the conversation. Find its `filter_*` tools.
One connection may serve several projects. When the tools require `tenant`, use the exact
project system name; discover it with `tenants_list` if unknown. Choose the sole applicable
project or ask when several fit. Explain a missing connection. Keep calls on the chosen
server and project: names and IDs identify records only within that project.

Use the complete original filter JSON. A filtered-list URL cannot be imported, and these
tools do not fetch saved filters by name. If only a link or an unseen filter is supplied,
ask for its JSON. For a successful build already in this conversation, reuse its confirmed
`finalSql` and selected records instead of converting it again.

Freshly read the connected server's maintained, up-to-date entry point:

```text
filter_wiki_read({"tenant": "<tenant>", "paths": ["README.md"]})
```

Use its navigation and current tool guidance. Reuse pages within this explanation. Examples
below are synthetic. Replace `<tenant>` with the confirmed project system name; omit the
argument only if the tool's schema has no `tenant`. Use the actual input and returned paths.
Tool names may have a server prefix.

## 1. Convert the original JSON to SQL

Pass the complete JSON unchanged as the `filterJson` object. For example:

```text
filter_json_to_sql({"tenant": "<tenant>", "filterJson": {
  "entityType": "User",
  "filterFactory": "and",
  "innerConditions": [{
    "entityType": "User",
    "filterFactory": "age",
    "value": {"mode": "concrete", "value": {"range": {"from": "30", "to": ""}, "unit": "Years"}}
  }]
}})
```

Read the returned `sql`, selected records and warnings. The tool already attempts to resolve
catalogue IDs to names; do not repeat successful lookups. `status: read` means the JSON was
read, not that the platform has accepted the filter. `unlabelled` means some values remain
unnamed; `unsupported` or `UNSUPPORTED_FILTER(...)` marks conditions the conversion cannot
explain. Keep the original JSON and these gaps visible in your reasoning. Do not reconstruct
a simpler filter to make them disappear.

## 2. Read the meaning and resolve remaining labels

Identify the root, fields, relations, operators and business concepts in the SQL. Search the
wiki for each distinct concept whose meaning or limitations you need to establish, combining
related terms in a regex and reusing relevant pages already read:

```text
filter_wiki_grep({"tenant": "<tenant>", "pattern": "возраст|\\bage(?:d|s)?\\b", "paths": ["field/user"], "root": "User"})
filter_wiki_read({"tenant": "<tenant>", "paths": ["field/user.age.md"], "root": "User"})
```

`pattern` is a case-insensitive regular expression. Here `возраст` matches forms such as
`возраста` and `возрастной`; `\\bage(?:d|s)?\\b` matches the English words `age`, `aged` and
`ages` without matching `message`. JSON strings need `\\b` to send the regex boundary `\b`.
For subscription conditions, `подпис(?:к|ок|ан)|subscri(?:b|pt)` covers forms such as `подписка`,
`подписок`, `подписан`, `subscribe` and `subscription`. Choose the terms needed for the filter.

`paths` prioritizes sections; it does not exclude other sections. Follow `skip`/`limit` when
results continue. Copy returned paths exactly, including underscores, casing and `.md`.
Read field and relation pages for behavior, and recipes or glossary pages for business meaning.
If that still leaves a product question, search with business terms and `paths: ["help"]`;
a missing README entry alone does not establish that help is unavailable. Documentation
explains a condition; it does not add conditions to the filter.

For an unresolved catalogue value, discover the relevant lookup type and supported search:

```text
filter_wiki_ls({"tenant": "<tenant>", "path": "lookup", "root": "User"})
filter_wiki_read({"tenant": "<tenant>", "paths": ["lookup/segment.md"], "root": "User"})
filter_search_entities({"tenant": "<tenant>", "context": "Identify an unnamed segment referenced by this filter", "types": ["segment"], "mode": "list", "pageSize": 20})
```

Browse when practical, or search a known name with `query`. Continue with the returned
`nextCursor` as `cursor`, preserving the other arguments. If rejected, restart the same
listing once without `cursor`; disclose a repeated failure as an unresolved lookup.
Match the returned `ref.ids`,
type and parent against the original reference before assigning a label. A similar name or
ranking score cannot identify an ID, and `query` is not a general get-by-ID operation.
If the identity remains unresolved, say so. An incomplete search or a missing label alone
does not prove the record was deleted or that the audience is empty.

## 3. Explain the actual selection

Use the conversation's language and business terms. Structure the explanation around:

- **Result:** what one returned row is and which audience or objects the filter selects.
- **Conditions:** grouped inclusions, alternatives and exclusions, with values and periods.
- **Limits and gaps:** material behavior from the reference and anything that could not be read.

Preserve AND/OR grouping, negation, inclusive or exclusive boundaries, relative and absolute
time windows, counts, and conditions that must hold on the same related object. Distinguish
one segment from its whole segmentation, current membership from historical membership,
and any matching event from the latest event when those constructs occur. Do not replace
these distinctions with a broad label such as "active customers".

State documented caveats that affect this filter and explain why each matters. Identify
unnamed values by their role and ID when needed; never present an ID as a catalogue name.
Describe an unsupported condition as a gap rather than assuming the remaining SQL is the
whole filter. Do not infer an empty or unrestricted audience from an unreadable part.

Identify the project and environment. Keep SQL and internal field names out of the answer
unless requested or needed to identify a gap. Explanation needs no validation or compilation
and does not change project data. If the user requests a correction, switch to filter-build
with the original JSON, the recovered SQL and selected records; preserve untouched conditions.
