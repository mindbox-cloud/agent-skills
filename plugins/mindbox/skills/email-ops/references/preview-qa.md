# Checks, preview and QA

**When to read:** you need an HTML download, diagnostics of a specific rendering problem, or the preview came
without PNG links. The preview of every iteration needs only `SKILL.md`.
**Return:** to §3 "Editing" of the visual workflow in `SKILL.md`, or to the user with the QA result.

- **HTML is not downloaded automatically.** Download HTML only for HTML QA or
  diagnostics where the HTML has to be read. PNG QA does not need HTML: MCP provides the PNG.
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
- **A request to check the email or its rendering is already consent to diagnostics:** do not ask separately
  whether you may download the HTML or open the PNG.
- **HTML QA** reads the downloaded file
  selectively and checks the expected texts, images, layout and obvious
  escaping/structure problems against the user's request. Do not read the whole HTML
  into the model's context without need. Where the canvas differs from the sent email — `#` links, sample
  values, placeholders — is in the wiki page `email.tags.template`, "In the preview": none of it is a defect.
  In the canvas HTML every link inside `<Text>` is `#`, ordinary ones too. Addresses and the unsubscribe link are checked against the JSX, not the canvas HTML; a link inside a
  `<SyncBlock>` — `email.topics.unsubscribe_link`.

  PNG QA uses the desktop/mobile links from the preview response and visually checks
  content, clipping, the responsive branch and image loading. This is best-effort QA, not
  a full Outlook/Gmail compatibility test.
- **PNG links are in the same preview response** — no second tool, no second call. No local screenshot, except the fallback below. Only mobile or only desktop asked — show that link; for
  responsive diagnostics — both. To the user they are "the desktop snapshot" and "the mobile snapshot".
- **Fallback if there are no PNG links.** If preview returned an error or came without
  PNG links, and the user still needs a visual check, in Cowork you may use
  Claude in Chrome as an emergency fallback to view/snapshot the HTML preview. Tell
  the user explicitly that this is a fallback, not the standard PNGs from the tool's response. The browser here only
  reads: using it to go into the Mindbox interface and click anything on the user's behalf is forbidden.
- **Diagnosing follow-up complaints.** If the user reports a problem with the
  content/rendering of a generated campaign, get the current JSX, call a fresh
  HTML preview, download the HTML only if HTML fragments need checking; for a
  visual/responsive problem, check the desktop/mobile PNG links from the preview
  response. Use an existing local HTML only if it is provably fresh
  for the same JSX.

QA found a defect and Generator changed the JSX — the local HTML is stale too. The user continues QA — repeat the
preview and the download; otherwise the new JSX gets its usual preview.
