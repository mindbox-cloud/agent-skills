# mindbox

Build marketing scenarios and audience filters in a Mindbox project from plain-language
requests. Connect the MCP server for the project you want to work with.

| Skill | Input | Result |
|---|---|---|
| `/mindbox:flow-create` | A description of the scenario in plain language; optionally an existing draft to build into. | A verified draft flow in the project with a link to it. Questions are asked once, in a batch, before the first write; one mailing is created per send step, named but not filled. |
| `/mindbox:filter-build` | An audience request, or existing filter JSON and the requested change. | A platform-confirmed filter, a link when available, and an explanation of its conditions. |
| `/mindbox:filter-explain` | Complete filter JSON, or a confirmed build already in the conversation. | A business explanation, including limitations and anything that could not be interpreted. |

The scenario skill delegates audience filters to the filter-building skill, which ships
here alongside it.

## Requirements

An MCP connection to the project. Scenarios need the flow tools, the wiki tool and the
entity listing; filters need the filter tools. The plugin ships no `.mcp.json`: the server
and the access depend on the project and authorize the user.

## Boundaries

- Nothing is launched, paused, stopped or deleted, and no mailing is activated. A flow is
  handed over as a draft, with a checklist of what a human still has to do.
- Building a filter does not save a segment or change project data. A link or segment name
  cannot be imported as a filter.
- Every write is checked by reading the result back, block by block, before it is reported
  as done.
- Reports are written in the language you asked in.

The skills contain the workflow and tool-call examples. Each task starts by reading the
connected server's current `README.md` for navigation and reference updates.
