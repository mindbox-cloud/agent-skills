# mindbox

Build marketing scenarios, audience filters and emails in a Mindbox project from
plain-language requests. Connect the MCP server for the project you want to work with.

| Skill | Input | Result |
|---|---|---|
| `/mindbox:flow-create` | A description of the scenario in plain language; optionally an existing draft to build into. | A verified draft flow in the project with a link to it. Questions are asked once, in a batch, before the first write; one mailing is created per send step, named but not filled. |
| `/mindbox:filter-build` | An audience request, or existing filter JSON and the requested change. | A platform-confirmed filter, a link when available, and an explanation of its conditions. |
| `/mindbox:filter-explain` | Complete filter JSON, or a confirmed build already in the conversation. | A business explanation, including limitations and anything that could not be interpreted. |
| `/mindbox:email` | A description of the email, or an existing email and the change you want. | An email layout for the Mindbox visual editor: structure, text, images, buttons, styles and personalization, with an unsubscribe link. It invents no links, ids or fonts. |
| `/mindbox:email-ops` | A campaign to create or open, and what to do with it; works together with `email`. | Campaigns created and edited: name, subject, sender, preheader, UTM, schedule and recipients; images found in or uploaded to the project gallery; a preview of the email; the email saved into the campaign; a test send to the project's test recipients. |

The scenario skill delegates audience filters to the filter-building skill, which ships
here alongside it; the email skills use it too when campaign recipients are set by
conditions. `email` writes the layout and `email-ops` does everything that touches the
project, so the two are used together.

## Requirements

An MCP connection to the project. Scenarios need the flow tools, the wiki tool and the
entity listing; filters need the filter tools; emails need the campaign, visual template
and gallery tools. The plugin ships no `.mcp.json`: the server
and the access depend on the project and authorize the user.

## Boundaries

- Nothing is launched, paused, stopped or deleted, and no mailing is activated. A flow is
  handed over as a draft, with a checklist of what a human still has to do.
- No campaign is sent, activated or deleted: only a test send to the project's test
  recipients is available. Saving an email, editing a campaign and a test send each wait
  for your explicit confirmation, and a version conflict is handed back to you rather than
  retried.
- Building a filter does not save a segment or change project data. A link or segment name
  cannot be imported as a filter.
- Every write is checked by reading the result back, block by block, before it is reported
  as done.
- Reports are written in the language you asked in.

The skills contain the workflow and tool-call examples. Each task starts by reading the
connected server's current `README.md` for navigation and reference updates.
