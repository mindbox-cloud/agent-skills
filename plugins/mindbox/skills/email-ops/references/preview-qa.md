# Checks, preview and QA

**When to read:** you need an HTML download, PNG, desktop/mobile, QA/debug, diagnostics of
a specific rendering problem, or a fallback. For a simple "show/refresh the preview",
the invariants in `SKILL.md` are enough; do not open this file.
**Return:** to step 3 "Editing, freshness and optional preview/QA" of the canonical workflow
in `SKILL.md`, or to the user with the QA result.

```text
visual_template_preview(jsx, formatInternalId) → temporary htmlUrl + MCP App widget in a supporting host
```

This is the editor canvas: personalization is filled in with sample values, a link with
personalization is rendered as `#`, an empty image as a placeholder. The response also contains
`containerWidth` — the preview panel needs it; do not relay it to the user.

Always pass `formatInternalId` when the format is known: with it, the width, background,
“Global (CSS) styles” («Глобальные (CSS) стили») and the email's own shared styles are laid under
the document, and the snapshot shows the email. Otherwise the snapshot shows only the document —
and it looks just as convincing, which is exactly why QA based on it is deceptive: the divergence
from the real email is not visible in the picture at all. In this case the tool itself states what
was rendered, and you must judge by its response, not by the argument you passed: a format with a
typo and a format with no saved email produce the same snapshot as one not named at all.

- **Preview is no longer an automatic step after every edit.** Each
  `visual_template_preview` call opens a new MCP App widget in a supporting host;
  old widgets are not updated. So do not call preview "just in case" and do not
  run it after every JSX modification.
- **A preview has a freshness status.** After any JSX change, the previous
  link/panel no longer proves the current state of the email. Tell the user:
  "The previous preview no longer matches the current version. If you like,
  I will show a new preview." If there has been no preview yet, say that it has not been
  opened yet. Do not create a new preview until the user asks or until
  QA/diagnostics have started under the rules below.
- **When to call `visual_template_preview`:**
  1. the user explicitly asks to show/refresh the preview;
  2. the user asks to check the email, the HTML, PNG, desktop/mobile or the rendering;
  3. the user comments on specific content or a visual
     problem of a campaign already shown/generated;
  4. preview is needed as a read-only action for a campaign that cannot be saved.
- **Return preview/editor errors to the generator verbatim; silent repair is forbidden.**
  If preview was not called and the error was returned by `visual_template_save`, also pass it
  to the generator verbatim — it is the backend check of the current JSX in the write path.
- **A bare `Internal server error` without a line number** — suspect the value format of an
  attribute (a scalar instead of an object), not the block structure.
- **The tool returns a temporary `htmlUrl`, not inline HTML.** When preview is called,
  show the link to the user as a fallback to the MCP App widget. In a host without MCP App, this is
  the only user-facing preview. If preview was not called after
  the last edit, say honestly that there is no fresh backend preview.
- **HTML is not downloaded automatically.** Download HTML only for HTML QA or
  diagnostics where the HTML has to be read. PNG QA no longer needs HTML: MCP provides the PNG.
  First fix the absolute `<projectRoot>`; do not rely on the runner's cwd. For a
  campaign, use the name `<mailingInternalId>.html`:

  ```bash
  # macOS/Linux
  curl --fail --location --silent --show-error "<htmlUrl>" \
    --output "<projectRoot>/<mailingInternalId>.html"

  # Windows PowerShell
  & curl.exe --fail --location --silent --show-error "<htmlUrl>" `
    --output "<projectRoot>/<mailingInternalId>.html"
  ```

  A campaign GUID needs no name sanitizing and prevents conflicts between different
  campaigns. A new QA/debug download of the same campaign replaces the previous local
  file: this is expected; the file represents the last downloaded backend render, not
  necessarily the last JSX edit. Do not use the campaign name alone and do not invent
  your own name without `mailingInternalId`. If the download failed,
  keep `htmlUrl`, report the `curl` error and do not substitute the HTML.
  Count a download as successful only with exit code `0` and a non-empty output file.
  A download error does not cancel a successful backend preview and does not by itself block
  save. HTML QA is unavailable until the download is repeated successfully; PNG QA via MCP
  does not depend on the HTML download.
  For a standalone preview without a target campaign and `mailingInternalId`, use
  `<projectRoot>/standalone-preview.html`; it represents the last standalone
  render and must not be used as an artifact of a specific campaign.
- **The preview for the user is the MCP App widget + fallback link before save, and the editor
  canvas after save.** If preview was not called after the last edit, do not
  present an old link as current: mark it stale and offer to refresh.
- **Never build the email's HTML/preview by hand.** Do not make approximate HTML,
  even if the backend is temporarily unavailable: fix the cause or report the status.
- **QA/debug is a separate capability, not part of an ordinary edit.** If the user
  asks to check the email, asks for PNG/mobile/desktop, talks about specific text,
  a link, an image, clipping, responsiveness or other rendering — that is already a request for
  diagnostics: call the HTML preview of the current JSX, show the link/widget, if
  needed download the HTML, and take the PNG from the response of the same preview. Do not ask
  a separate question whether you may open the HTML/get the PNG.
- **An ordinary edit without QA:** after a JSX change, only write that the previous
  preview is stale and offer to show a new one. If the user does not ask for
  preview/QA, do not call `visual_template_preview`, do not download HTML and do not request PNG.
- **Before save**, show the exact target, the change and the preview freshness status.
  If there is no fresh preview, say explicitly: "The preview has not been refreshed since the last
  edit; I can save without a new preview or show it first."
  Save confirmation is acceptable only after such a warning.

  Example of an ordinary message after an edit:

  ```text
  The change is ready. The previous preview no longer matches the current version.
  If you like, I will show a new preview in the panel. I can also run a full check
  with HTML and desktop/mobile PNG links.
  ```

  A write confirmation without a fresh preview is assembled from the canonical set of fields in the
  "Confirmation" section of the main `SKILL.md` — campaign, A/B variant, formats with Draft spelled out,
  change, freshness and the mandatory line about the shared styles. Only the choice is added here:

  ```text
  Choose: 1) show a new preview; 2) run a full QA with HTML and PNG;
  3) save without a new preview.
  ```

  After option 1 or 2, report the result and request the save confirmation separately;
  preview/QA never means consent to write. HTML QA reads the downloaded file
  selectively and checks the expected texts, images, layout and obvious
  escaping/structure problems against the user's request. Do not read the whole HTML
  into the model's context without need. **What the canvas HTML does not prove:**

  - personalization is filled in with sample values: `${Customer.FirstName}` and other
    `${...}` will no longer appear in the canvas HTML;
  - **all** links inside `<Text>` in the canvas HTML have `href="#"` — both ordinary ones and the
    canonical unsubscribe link `${Message.UnsubscribeLink}`. The same goes for `<Button>`/`<Image>`/`<Icon>`
    if their `url` contains personalization;
  - an empty image is rendered as a placeholder.

  None of this is a defect. Do not present the substituted values as customer data and do not
  check link addresses on the canvas: `href="#"` in the canvas HTML **does not mean** that the link
  is lost or that the JSX has a fake URL — addresses and the presence of `${Message.UnsubscribeLink}`
  must be checked against the JSX (`visual_template_get` / the one passed by Generator), not against the HTML.

  PNG QA uses the desktop/mobile links from the preview response and visually checks
  content, clipping, the responsive branch and image loading. This is best-effort QA, not
  a full Outlook/Gmail compatibility test.
- **PNGs come in the same response as the preview link.** There is no separate PNG tool:
  `visual_template_preview` returns links to the desktop and mobile snapshots together with
  `htmlUrl`, in one call. When PNG/mobile/desktop is requested, do not look for a second tool and do not call
  preview a second time — take the links from the response you already have, and if there was no preview
  after the last edit, call it once. Do not use a local screenshot as
  the standard path. The exception is the Cowork fallback below, if the response has no links.
  If the user asks only for mobile or only for desktop, you may show only
  the corresponding link; for full responsive diagnostics, show both.
  Take the field names in the response from the actual MCP tool output; when talking to
  the user, call them "the desktop snapshot" and "the mobile snapshot".
- **Fallback if there are no PNG links.** If preview returned an error or came without
  PNG links, and the user still needs a visual check, in Cowork you may use
  Claude in Chrome as an emergency fallback to view/snapshot the HTML preview. Tell
  the user explicitly that this is a fallback, not the standard PNGs from the tool's response. The browser here only
  reads: using it to go into the Mindbox interface and click anything on the user's behalf is forbidden.
  Do not bring back the removed legacy path via local PNG scripts.
- **How to use the PNG links.** Show the user the links to the desktop/mobile PNG and
  use them for the visual check if the current host can open/view
  images. If the host cannot visualize a PNG from a link, say honestly that the PNGs
  were received but a visual check by the agent is unavailable in this environment; the user
  can open the links themselves.
- **Diagnosing follow-up complaints.** If the user reports a problem with the
  content/rendering of a generated campaign, get the current JSX, call a fresh
  HTML preview, download the HTML only if HTML fragments need checking; for a
  visual/responsive problem, check the desktop/mobile PNG links from the preview
  response. Use an existing local HTML only if it is provably fresh
  for the same JSX.

If the additional QA found a defect and Generator changed the JSX, the previous `htmlUrl`,
MCP App widget, local HTML and PNG links no longer prove the state of the new JSX.
Mark the preview as stale. If the user continues QA/debug, repeat the HTML
preview and the needed HTML download; if not — only offer to refresh the preview.

Do not edit the JSX yourself — run preview/QA only under this policy, keep the artifacts and
perform the save.
