# Filters

Build CDP filters from a description, edit an existing filter, or explain what it selects.
Connect the MCP server for the project you want to work with.

| Skill | Input | Result |
|---|---|---|
| `/filters:filter-build` | An audience request, or existing filter JSON and the requested change. | A platform-confirmed filter, a link when available, and an explanation of its conditions. |
| `/filters:filter-explain` | Complete filter JSON, or a confirmed build already in the conversation. | A business explanation, including limitations and anything that could not be interpreted. |

Building does not save a segment or change project data. Request JSON when another AI agent,
tool or skill needs the filter to configure a mechanic. Otherwise a link is enough; a result
without a list link includes JSON. A link or segment name cannot be imported as a filter.

The skills contain the workflow and tool-call examples. Each task starts by reading the
connected server's current `README.md` for navigation and reference updates.
