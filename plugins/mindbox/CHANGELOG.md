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

## 1.1.0

Emails: two new skills that build an email in the Mindbox visual editor from a plain-language
description and put it into a campaign.

- **`/mindbox:email`** — from a description such as "a sale announcement with a hero banner,
  three products and a button to the catalogue", it lays out the email for the visual editor:
  structure, text, images, buttons, styles, personalization and product rows, with an
  unsubscribe link. It also edits an existing email: give it the email and the change you want.
  It invents no links, ids or fonts; a button whose address is not known yet is left without
  one and listed for you to fill in.
- **`/mindbox:email-ops`** — does everything that touches the project, together with `email`:
  creates and opens campaigns, changes the name, subject, sender, preheader, UTM tags, schedule
  and recipients (a segment, or a filter built by `filter-build`), finds images in the project
  gallery or uploads them from your machine, shows a preview, saves the email into the campaign
  and sends a test to the project's test recipients.
  When several gallery images could fit, you pick one in the gallery panel that opens next to
  the answer in clients that support MCP Apps; elsewhere the candidates are listed in text.

No campaign is sent to customers, activated or deleted. Saving an email, editing a campaign and
a test send each wait for your explicit confirmation, and a save is checked by reading the
campaign back before it is reported as done.

Emails need the campaign, visual template and gallery tools of the project's MCP server.
