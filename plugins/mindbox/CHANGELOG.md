# Changelog

## 1.0.0

First public release: marketing scenarios and audience filters in a Mindbox project, built
from plain-language requests.

- **`/mindbox:flow-create`** — from a description such as "welcome series for new
  subscribers, three emails over a week", it designs the scenario, resolves the entities it
  needs to real ids, fills and wires the blocks, and verifies each one by reading it back.
  Questions are asked once, in a batch, before anything is written. One mailing is created
  per send step, named but not filled. The flow is handed over as a draft with a link and a
  checklist of what is left to a human.
- **`/mindbox:filter-build`** — from a request such as "customers who bought last month and
  are subscribed to email", it builds the filter, checks the entries it picked from the
  project's catalogues against what you meant, validates the result on the platform, and
  returns a link to the list when one is available. Filter JSON is available on request —
  you need it when another tool or agent configures a mechanic from it. It also edits: give
  it an existing filter together with the change you want.
- **`/mindbox:filter-explain`** — from filter JSON, it explains in plain words which
  audience the filter selects, where its limits are, and what could not be interpreted.

Nothing is launched, paused or deleted, and no filter is saved as a segment: starting a flow
stays your decision. Reports are written in the language you asked in.

Requires the MCP server of the project you want to work with. Scenarios need the flow tools,
the wiki tool and the entity listing; filters need the filter tools. The plugin ships no
`.mcp.json`: the server and the access depend on the project and authorize the user.
