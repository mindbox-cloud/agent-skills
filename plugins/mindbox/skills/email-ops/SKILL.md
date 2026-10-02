---
name: email-ops
description: Operational actions for Mindbox emails — preview, reading and saving the visual template, reading/editing campaign metadata and recipients (a segment or a filter), finding gallery assets via MCP, uploading images from the user's machine through the MCP App panel, PNG preview. Use together with the email skill when images, a preview, a check or a write to Mindbox are needed.
metadata:
  author: Personalization + AI tribe
  version: 1.0.0
---

# email-ops — Mindbox operations for emails

The project-aware executor of all integration actions. The JSX generator is
`email`; this skill does not invent email content and **does not change JSX
silently**. Return a preview error to the generator verbatim — it is the editor's text,
with line numbers; do not rephrase or classify it.

## Communicating with the user

JSX, internal IDs and rowVersion are internal execution details. In normal conversation with a
CSM say "email", "email layout", "content", "preview" and "saving",
not JSX, `mailingInternalId`, `variantInternalId`, `formatInternalId`, `rowVersion`
or `visualTemplateRowVersion`. Do not replace the terms Active/Draft: if they are needed to
pick a version precisely, use exactly Active/Draft. Do not show raw JSX or GUIDs
unless the user explicitly asked for code, export, markup or technical details.
Return backend errors verbatim, even if they contain technical terms, tag
names, line numbers or identifiers.

## Skill feedback via MCP feedback

The MCP `feedback` of the current Mindbox/email MCP server is an optional channel for passing
confirmed feedback about the skill's work to the developers. Use one of two entry points:

1. **Agent's initiative.** If you have confirmed that you see a critical problem
   that cannot be solved with the current skill — a mismatch between the instructions or
   references and the actual backend behavior, repeated errors of the same
   type, a blocked safe workflow — offer the user to send
   feedback to the developers. Do not send it without explicit consent.
2. **User's initiative.** If the user asks how to pass a
   message/feedback to the developers, answer: "This is available. What
   message would you like to send?" Then format their words using the format below.

The sending protocol is mandatory in both cases:

1. First compose the exact text of the message. The first line is the marker
   `[skill-feedback:v1]`, followed by the fields `skill=mailings`,
   `skillVersion=<from plugin.json>`, `source=user-report|agent-observation`,
   `problem` and `context`. `problem` is the essence: what gets in the way, what went
   wrong. For `user-report`, write the user's exact words into `problem`, and
   add `context` yourself. For `agent-observation`, you write both fields. `context` —
   briefly: input data and requests, how the current situation came about, verbatim
   backend error texts, reproduction facts (run date, a throwaway
   campaign). Add `area=<layout|save|preview|gallery|personalization|other>`
   only if the area is obvious.
2. **Always first show the user the entire text to be sent** and ask for
   explicit permission, for example: "Send this feedback to the developers?"
3. Call `feedback` specifically in the current Mindbox/email MCP server through which
   this skill's operations are performed, and only after unambiguous confirmation.
   If the user asks to change the text, show the corrected version again
   and get confirmation again.
4. Do not send feedback covertly, do not assume consent and do not silently attach it
   to another operation. Do not include email content, personal data,
   secrets, signed URLs or tokens.

## Capabilities

- on the user's request or for diagnostics, renders JSX through
  `visual_template_preview`: the user gets the fallback link `htmlUrl`, and in a
  supporting host an MCP App widget opens;
- reads campaign metadata (`campaign_get`);
- reads the existing visual JSX of the chosen format (`visual_template_get`) and the JSX of saved blocks chosen by the user in the panel (`visual_template_saved_block_list` → `visual_template_saved_block_get`);
- gives the generator the email's current shared styles as a `<Theme>` fragment
  (`visual_template_theme_get`);
- saves JSX into the chosen campaign format after explicit confirmation
  (`visual_template_save`);
- changes the campaign's basic settings and the recipients of a bulk campaign — a segment or a filter (`campaign_edit`);
- changes subject/sender/reply-to/preheader or raw HTML on explicit request (`campaign_edit_content`);
- creates an empty email campaign after an explicit choice of folder/brand/timezone (`campaign_create`);
- sends a test email to staff test recipients after the user has explicitly chosen them
  and confirmed (`campaign_test_recipients` → `campaign_send_test`);
- finds existing gallery assets through MCP `gallery_images_list` and returns ready `{url, fileName}` to the generator;
- puts images into the project gallery through `gallery_image_upload`: without arguments it opens the upload
  panel, when the image has to be taken from the user's machine, and takes the addresses from the panel's context;
  with `imageUrl` — when the user gave the image as an HTTPS link;
- builds a visual contact sheet for choosing images from the gallery: HTML with remote image
  links, opened in an external browser;
- optionally checks the content of the downloaded HTML and inspects the desktop/mobile PNG snapshots,
  which come as links in the same preview response; this is additional
  QA diagnostics, not a gate of the main workflow.

Live send/activate/delete and recipients from a one-off file are not available in MCP — they are done in the campaign UI. A test send to staff test recipients is available — see the "Test send" section.

## MCP tools

Gallery operations are performed only through MCP. Do not configure project-local env,
cookies or tokens for finding/uploading images: the MCP tools already run in the
context of the project. If a gallery MCP tool returned an error, pass it to the user
as a tool error and do not diagnose it through local authorization.

PNGs come as links in the preview response, without local rendering or screenshots.

## Quickstart: pick a route

| Task | Route |
|---|---|
| New JSX email | target discovery if a save is needed → gallery → Generator → stale-preview notice / optional preview+QA → the canonical visual save workflow |
| Editing an existing email by ID | `campaign_get` → the canonical visual workflow with `visual_template_get` |
| A saved editor block | `visual_template_saved_block_list` → the user chooses in the panel → `visual_template_saved_block_get` with the `internalId` from their message |
| Change the hero/content in the email body | the canonical visual workflow, not subject |
| Change the template type: "rebuild it in the visual editor", "convert it to HTML" | the "Changing the template type" section: the type does not change, the email is carried over into a new campaign |
| Change the subject/sender/preheader | `campaign_get` → `campaign_edit_content` |
| Change the name/UTM/schedule | `campaign_get` → `campaign_edit` |
| Set recipients: a segment or a filter | `references/recipients.md`: `campaign_get` → `segments_list` or the filter-building skill (`filter-build`) → confirmation → `campaign_edit` |
| A new campaign from scratch | the end-to-end workflow below: `campaign_create` → `campaign_get` → metadata/content edits → visual save; recipients — `references/recipients.md`; live send/activate — UI |
| Test send of an email | the "Test send" section: `campaign_get` → `campaign_test_recipients` → choice of recipients → confirmation → `campaign_send_test` |
| The image is already in the project gallery | MCP `gallery_images_list` |
| The image has to be uploaded from the user's machine | MCP `gallery_image_upload` without arguments → panel |
| An image by the user's HTTPS link | MCP `gallery_image_upload` with `imageUrl` |
| PNG | the desktop/mobile links in the `visual_template_preview` response |

If data is missing:

- there is no mailing internalId for reading/editing by ID — stop and ask;
- there is no exact target for the save (`variantInternalId` + `formatInternalId` of each target format) or no save confirmation with the preview status — stop and ask;
- do not invent `mailingInternalId`, `variantInternalId`, `formatInternalId`, a URL, a gallery asset or a `fileName`.

## Images: confirmation procedure

Generator uses only URLs that Ops returned or that the user explicitly
provided. Ops chooses nothing silently.

If the email has to be saved into a specific campaign, before finding or uploading
images first run `campaign_get`, choose the exact target and complete
`visual_template_get`/bootstrap discovery. Only after confirming that the target
is available for the visual workflow, search the gallery for images or open the upload panel and pass
the URLs to Generator.
For standalone generation without a target campaign this preliminary discovery is not needed.

### Finding an already uploaded image

1. Call MCP `gallery_images_list` with `nameSubstring` from the user's request,
   `includeSystemImages: true` and a sufficient `limit` (usually 100). Do not pass
   `fileExtensions`: the tool itself returns only the formats an email can display, and an image
   in WebP — as a link to its PNG copy.
2. If the user explicitly asks for project assets only, pass
   `includeSystemImages: false`. In all other cases system images
   are included by default.
3. The tool searches all folders of the project and the platform; there is no separate folder workflow.
   Do not narrow the search with local folder workarounds.
4. If there are no candidates, say that no suitable image was found. Then there are two paths:
   an HTTPS link to the image, if the user has one (you put it into the gallery too —
   the "External HTTPS URL" section), or `gallery_image_upload` without arguments — the
   panel opens, and the user uploads the file themselves. But first keep in mind that a miss
   more often means a different spelling of the name than a missing file: if the user expected this
   image to be in the gallery, ask again about the name rather than creating a second copy.
5. If there is one candidate, return `{url, fileName}` to Generator. Take `url` exactly from the
   tool output row: do not build, edit or shorten it. `fileName` is
   `name + fileExtension` if `name` does not already end with that extension;
   otherwise use `name` as is.
6. If there are several candidates, do not choose silently. Show the user safe
   metadata from the actual MCP row: `name`, `fileExtension`, `isSystem`,
   and size/date only if they are actually present in the response. `isSystem: true`
   means a platform shared system icon; otherwise it is a project asset. Show the URL
   only if the user cannot tell the options apart without it.
7. When offering an image to the user, say explicitly whether it is a platform system icon
   or a project asset.

### Visual image selection from the gallery

A text list remains the quick default answer. Make a visual contact sheet
only when the user asks to view the images visually or when there are
several similar candidates and a text list is not enough.

Selection invariants (they apply even before reading the file):

- the email always gets the original `url` from `gallery_images_list`, exactly as
  MCP printed it; for an image stored in WebP, this is an imgproxy link to its PNG copy,
  not to the original S3 object — that is normal; `fileName = name + fileExtension`
  if `name` does not already end with that extension;
- the user chooses the card number; do not choose among similar options silently;
- a `data:` URL is forbidden in email JSX;
- the contact sheet is HTML with remote `<img>` links, opened in an external browser —
  both in Codex/local and in Cowork, whose side panel blocks external image domains.

**This case applies — you MUST read `references/visual-gallery-selection.md`**:
it has the full Codex/Cowork procedure, the card template and the result format after the choice.
Do not build a contact sheet from memory without the file.

### Uploading images from the user's machine

Upload through `gallery_image_upload`: it puts the file straight into the project gallery, with a permanent
address — suitable both for the visual template and for an HTML email. How to use it (the panel, where
to take the addresses from, what to do without the panel) is in its description and responses; follow them.

The skill adds two rules here:

- **When to upload.** The trigger is the intent to put an image into the email, not the presence of an
  attachment. A bug screenshot, a mockup reference, "this is how it looks now, fix it" — just
  look at the image and upload nothing. If it is unclear what is wanted, ask.
- **What goes where is the user's decision.** A file name is a label for the conversation, not a description
  of the content: do not guess by name or order, ask. Generator gets the pair
  `{url, fileName}`: `url` is the address returned by `gallery_image_upload`, `fileName` is the name
  from the same row.

### External HTTPS URL

Put an image from the user's direct HTTPS link into the gallery — `gallery_image_upload` with
`imageUrl` — and from then on use the `url` from its response: that way the address is permanent and does not depend on
someone else's server. Leave as is only a personal link (with variables) and a link
that the user explicitly asks not to copy.

### Gallery unavailable/errors

If `gallery_images_list` or `gallery_image_upload` returned an error,
pass the meaning of the error to the user and stop the image-dependent part of the workflow.
Do not invent a cause, do not substitute a similar image and do not ask for local
credentials. Without a confirmed URL the email can be built only without this
image or with explicit placeholder text by agreement.

### Social icons and utility icons

Look for social network icons and utility icons with the same `gallery_images_list` with system
images on by default. System rows come with `isSystem: true`; these are
platform shared system icons, not project assets. Icons are ordinary HTTPS URLs and
are placed into `<Image image={{ mode: "static", ... }}>` like any other image.
One social network often has several design variants — show the metadata and
ask; do not choose yourself. If the needed social network is missing, ask for an external HTTPS URL or
a file; do not substitute a similar icon.

## Checks, preview and QA

Preview invariants (not moved out; the full procedure is behind the trigger below):

- **Preview is no longer an automatic step after each edit.** Each call to
  `visual_template_preview` opens a new MCP App widget in a supporting host;
  old widgets are not updated. After any JSX change the previous
  preview/HTML/PNG links are stale: say "The previous preview no longer
  matches the current version. If you like, I'll show a new preview."
- **`visual_template_preview` is called only** when the user asks for a
  preview, HTML/PNG QA, desktop/mobile/debug, comments on specific
  content/rendering, or as read-only for a non-editable campaign.
- **The preview is the editor canvas, not the email that is sent**: personalization in it
  is filled with sample values, and they are not customer data. What can and cannot be checked on the canvas
  is in `references/preview-qa.md`. `containerWidth` from the response is for the panel; do not relay
  it to the user.
- **Pass `formatInternalId` as soon as it is known.** With it the render carries, underneath
  the document, the format's own width, background, "Global (CSS) styles" («Глобальные (CSS) стили») and the email's shared styles — that
  is, it shows the email. Otherwise the picture shows only the document, and it looks every bit
  as convincing. This happens in two ways: the format is not named, or nothing has been saved
  in it yet — and the tool's response says plainly what exactly was drawn. Trust
  the response, not what you passed: a format with a typo falls into the same case.
- **If you inspect the rendering yourself, look at both snapshots.** The mobile layout differs from
  the desktop one in more than trifles: the columns of a row collapse into a stack by default, and what
  stands neatly on desktop may fall apart on mobile. A verdict based on one snapshot does not
  cover it.
- **HTML is downloaded only for HTML QA/diagnostics; PNGs are always links from the preview
  response, there is no separate call for them.** A local screenshot is not the standard
  path (the emergency Cowork fallback is Claude in Chrome, explicitly marked as a fallback). Return
  preview/editor/save errors to the generator verbatim; silent repair is forbidden.
- **Before save** show the target, the change and the freshness status of the preview; without
  a fresh preview, the confirmation must explicitly allow saving without a new
  preview. Preview/QA never means consent to write.
- Never build the email's HTML/preview by hand.

Close a simple "show/refresh the preview" request using the invariants above:
`visual_template_preview` → widget/fallback link; `references/preview-qa.md` is not needed
for that. **If an HTML download, PNG, mobile/desktop, QA/debug,
diagnostics of a specific rendering problem or a fallback is needed — you MUST read
`references/preview-qa.md`**: it has the download file name, working with MCP PNG links,
QA repeats and fallback branches; do not perform the extended procedure without the file.
## Editing the visual template

This is the only detailed workflow for reading, editing and writing JSX. Other sections
only choose the route and do not repeat the save algorithm.

### 1. Preparing the target

1. Call `campaign_get(mailingInternalId)`.
2. Record the campaign name, `mailingInternalId`, the current `mailingRowVersion`,
   variants, available formats and pre-existing validation errors.
3. Before choosing the target, check `kind` and `state` from `campaign_get`. Editing is allowed when
   `state: InDevelopment` or `ReadyForSend`, and also when `state: ActionTemplatesCreated`
   and `kind: Automatic`. Otherwise, and also when `transactional: yes`, stop the workflow,
   save nothing and tell the user that such a campaign is edited in the editor.
   `visual_template_preview` is allowed in that case — it writes nothing.
   In `ActionTemplatesCreated` the backend has the last word: it looks at the flow step,
   which is not in `campaign_get`. Its refusal means that nothing was written and a retry will not
   help.
4. Determine the exact write target: `variantInternalId` + `formatInternalId`.
5. The targets of a visual write are **all formats of the variant for which the visual workflow is allowed**:
   Draft, if it exists, and Active. A draft is a separate version of the same email, and it
   will replace the active one when it is activated, so an edit that went only into Active may not
   make it into the sent email. Do not offer a choice between them — write into all of them,
   **starting with Draft**: if something goes wrong with it, the active version stays
   untouched.
   A format for which the visual workflow is forbidden per item 6 (`Rawhtml` with `html: yes`) does not
   become a target: write into the compatible one and tell the user which one stayed unchanged.
   The skill cannot create a Draft — formats are created only in the editor.
6. Record the editor kind of each target format from `campaign_get`:
    - `Mindboxeditor` — the visual workflow is allowed;
    - `Rawhtml` with `html: yes` — this is existing raw HTML, the visual workflow is forbidden; handle a request
      to convert this email to the visual editor through the "Changing the template type" section;
    - `Rawhtml` with `html: no` — an empty-bootstrap candidate: do not ask for the UI, go to
      `visual_template_get` and choose the bootstrap/recovery branch based on its response;
      the bootstrap trigger is the verbatim response `No visual template was saved for
      format '…' yet`;
    - the editor kind is missing or not recognized — do not guess compatibility; stop.
7. If there is one suitable A/B variant, do not ask an unnecessary question, but show
   the choice in the final confirmation. If there are several variants,
   ask the user to choose the target before editing.
8. Do not invent `variantInternalId` or `formatInternalId`. Here
   `variantInternalId` means the campaign's A/B variant, not the JSX attribute `themeVariant`,
   which refers to the node's shared-styles variant.

### 2. Reading the initial state

1. Call `visual_template_get(formatInternalId)` for each target format —
   `Mindboxeditor` or `Rawhtml` with `html: no`. Each format has its own JSX snapshot and
   `visualTemplateRowVersion`; do not carry one format's version over to another. A draft may not yet
   have a visual template — then the bootstrap branch from item 3 applies to it, and the usual one to
   Active.
2. If the tool returned JSX and `visualTemplateRowVersion`, this is an existing visual
   template. With `Mindboxeditor + html: yes`, keep the exact JSX as the snapshot. With
   `html: no`, regardless of editor kind, first perform attachment recovery: after
   confirmation, save exactly this JSX with the current `mailingRowVersion` and
   `visualTemplateRowVersion`, then check the attachment as a first bootstrap save.
   Do not change the JSX and do not create a new version until recovery succeeds.
2a. An existing visual template, and the task is to build a new email (a mockup, a description, a prompt):
    before generation, ask what to do with what is already in the format — build a new one as a replacement or edit
    the existing one. A replacement is a different email, not an edit, and the user decides. Pass the answer
    to the generator together with the JSX snapshot.
    The gate is tied to the intent, not to this step: "let's start over", "redo everything", "from scratch" in the middle of
    editing — is the same replacement, and 2a–2b are gone through again, no matter how many iterations have passed. The snapshot and
    the styles are re-read at that point (`visual_template_get`, `visual_template_theme_get`): over
    the iterations they could have changed, including through your own saves.
2b. Replacement chosen — call `visual_template_theme_get(formatInternalId, jsx)` and see whether the
    email's shared styles are its own or the platform defaults. Its own — the second question is **mandatory**, and it has three answers:
    build the new content **in these styles** (headings and buttons take their look from them and from then on
    change together with the whole email), **give the email new ones**, or **reset to platform defaults**.
    Platform defaults already — do not ask.
    A full overwrite of the template does not erase the styles: they belong to the format and are carried over on
    save. That is why both "new" and "reset" mean an explicit `<Theme>` in the new document, not the
    absence of the tag; without it the new email inherits the previous styling. The values for a reset
    are taken from the same tool called **with neither argument** — this is the only way to get
    the platform defaults. And say it directly: a complete "as if nobody had configured it" is not available from markup — the email
    will look unconfigured, but its styles will remain its own, just equal to the platform defaults.
    Pass all the answers to the generator.
3. If the tool says verbatim `No visual template was saved for format '…' yet`,
   check the original `campaign_get`:
    - the chosen format has `html: no`, and the editor kind is `Mindboxeditor` or
      `Rawhtml` — this is a bootstrap of a new visual template: record the absence of
      stored JSX, snapshot and `visualTemplateRowVersion`; the first save is performed
      after explicit confirmation with the preview status and without
      `visualTemplateRowVersion`;
   - the format already has a body (`html: yes`) or its emptiness is not confirmed — do not treat
     this as a bootstrap and do not overwrite the format: there may be raw HTML in it.
4. If `visual_template_get` reports that the existing template cannot be expressed in JSX,
   do not continue editing blindly and do not overwrite it: offer the UI. JSX
   provided separately by the user can be edited/shown via the route below, but
   saved only into a confirmed compatible visual target through the usual gates.

### 3. Editing, freshness and optional preview/QA

1. Generator makes only the requested changes.
2. Any JSX change makes the previous preview/HTML/PNG links stale. Tell
   the user and offer a new preview or a full QA, but do not call preview
   automatically.
3. A request to redo the email from scratch in the middle of editing is a replacement, not an edit: go back to §2
   items 2a–2b, re-read the snapshot and the styles and ask both questions again. Silently starting a new email
   on top of the work done is not allowed, no matter how many iterations have passed.
4. If the user asks for preview/QA/debug, `visual_template_preview` renders
   the current JSX **with the target format's `formatInternalId`**; return an editor/preview error
   to the generator verbatim; silent repair is forbidden.
5. If the generator needs the current values of the email's shared styles — to detach a node from
   them or to write the set into `<Theme>` — call
   `visual_template_theme_get(formatInternalId, jsx)`. **Pass the current JSX**: without it
   the response will not take into account the document's not-yet-saved `<Theme>`, and replacing that tag with the response
   will erase the edits. The response is a ready fragment; give it to the generator as is. If there is no email
   yet — call it with neither argument (no `formatInternalId`, no `jsx`), and the platform default values will come back; the tool says so
   plainly, and they must not be relayed to the user as the styling of their email.
6. A request to change the look of nodes that follow the email's shared styles — the number first.
   Count in the document how many other elements an edit of the shared styles will affect: a node under them
   writes no styles of its own, so they are visible. Name the number and ask whether to edit the shared styles or
   detach the named nodes from them. A question without the number gives the user nothing to decide with.
7. For QA/debug, download the HTML if an HTML check is needed, and/or get the PNG links
   through MCP per the rules of the "Checks, preview and QA" section. For an ordinary edit
   without QA no download is performed.

The state of the shared styles needs to be known only when the document actually names a look (see
§4). It is found out without an extra call:

- the format is not named or does not exist yet — the styles are platform defaults; this follows from the fact itself;
- `visual_template_get` answered verbatim `No visual template was saved for format '…' yet` — the same:
  there was nothing to save, so the styles are platform defaults;
- the format contains an email — then this is already branch 2a–2b, where `visual_template_theme_get` is called for
  another reason; no separate call is needed.

Do not call the tool for this question alone: its response is the full set of styles, it is large, and pulling it into
the context for nothing is costly.

### 4. Confirmation

Before `visual_template_save`, one explicit confirmation from the user is needed. In it
always show the freshness status of the preview: fresh, stale after the last
edit, or not opened. If there is no fresh preview, the confirmation must explicitly
allow a save without a new preview:

```text
- campaign: <name and internalId if needed>
- A/B variant: <a human-readable description, show only if the campaign has several variants>
- formats: Active — or "Draft and Active" in write order, if the variant has a draft
- change: a short description
- preview: fresh / stale after the last edit / not opened
- shared styles: this line is mandatory in **every** confirmation and is read directly from the document, not
  from memory of the questions asked — three mutually exclusive cases:
  - the document has `<Theme>` → "the email's shared styles are set by this document";
  - there is no `<Theme>`, and the nodes name their own look → "the email is saved with its own styles on each
    element";
  - there is no `<Theme>` and the nodes name no look → "the email's shared styles stay as they are, the nodes follow
    them".

  No line — the confirmation is incomplete. Separately from the line: if, per the conditions of step 6 of
  `email/SKILL.md`, the offer to move the nodes' look into the shared styles is needed, and it has
  not been made in this conversation — make it right here, together with the other save parameters: "Save
  this email's styles as its shared styles? Then you'll be able to change headings and buttons across the whole
  email at once." One conversation — one such question: the answer holds until its end, and neither a second
  save nor edits re-trigger it
- existing email: only if the format already had a visual template and replacement was chosen —
  "the email in the format is replaced entirely"
- style move: only if the edit moves the nodes' styles into the email's shared styles — "the styles move
  into the email's shared styles, the look does not change; from then on they can be edited across the whole email at once"
- draft line: only if the variant has a Draft format — "The campaign has a
  draft started in the editor; the edit will be saved into it first, then into the active
  version"
```

If the target is ambiguous, the target choice question is asked earlier and separately from the
write confirmation.

### 5. Save

1. Call `visual_template_save` for one chosen `formatInternalId`.
2. One save — one `formatInternalId`; there is no batch write. If there are two targets, there are
   two calls as well, with the same JSX: **Draft first, then Active**. The order is not cosmetic —
   what does not go out to customers goes first.
3. For an existing visual template, pass the versions from discovery:
   `mailingRowVersion` and `visualTemplateRowVersion`. The second call gets the
   `mailingRowVersion` **from the response of the first one** — there is one campaign, and its version has grown —
   while `visualTemplateRowVersion` is taken from the format you are writing to: each has its own.
   If the first save (Draft) was refused, do not write into Active: stop, show the refusal and
   ask what to do. If the second one was refused, do not rewrite the first one and do not roll it back:
   tell the user which format was updated, which stayed unchanged, and why.
4. Only for the confirmed bootstrap branch from step 2, pass
   `mailingRowVersion`, and do not pass `visualTemplateRowVersion`.
5. From a successful response keep the new `mailingRowVersion`, `visualTemplateRowVersion`,
   state and validation errors. Use these versions for subsequent
   `campaign_edit`, `campaign_edit_content` and a repeated save.
6. After the first bootstrap save, you must independently re-read `campaign_get` and
   find the same `formatInternalId` that the save was made into, with its original
   Active or Draft status. Success is confirmed only if the format has become
   `Mindboxeditor` and `html: yes`. If it stayed `Rawhtml`, `html: no`, disappeared, or
   the response does not allow confirming both signs, do not say "saved" and do not repeat
   the same save automatically: report that the visual template did not attach to the email.
   Continuing is only possible via attachment recovery from item 8 after `visual_template_get` and
   a new confirmation.
7. Moving styles into the shared ones goes in **one** save: `<Theme>`, the nodes the styles were removed from, and
   `themeVariant` — one document. Do not save the theme separately from the nodes: between the two saves
   the email will be left without a look.
8. If the save explicitly reports that the visual template was saved but the campaign was not, keep
   the returned `visualTemplateRowVersion`: do not create a new template and do not repeat
   the save without this version. After a fresh `campaign_get`, attachment recovery uses
   exactly the same JSX and the version received.

### 6. Post-operation report

1. Briefly report the campaign, the format and the result. If there were two targets, name both and
   the result for each.
2. Separate validation errors into those pre-existing at the time of the original `campaign_get`
   and those introduced/changed by the current edit.
2b. Distinguish `visual template stored` from `campaign updated/attached`. A response with
    `The template itself WAS stored — only the campaign was not` is a partial persist,
    not a success: do not say "saved"; further actions — per §5 item 8.
3. Do not make a repeated `campaign_get` mandatory after every ordinary successful save:
   the save response is the main result. Re-read `campaign_get` and, if needed,
   `visual_template_get` only if the save response is incomplete, the version is lost, the next
   operation needs it, or an independent check is needed. The first bootstrap save is an exception:
   it always goes through the independent check from step 5.

### 7. `ChangeConflict`

`ChangeConflict` is a concurrency event, not a way to find out the current version.

On a conflict:

1. Re-read `campaign_get`.
2. Re-read `visual_template_get`.
3. Compare the fresh stored JSX with the original snapshot from step 2, not with the edited JSX.

If the fresh JSX matches the snapshot, the template body has not changed since reading, and
the conflict is related to another version of the campaign/metadata. **There is no automatic retry
here either:** say that the campaign was changed concurrently, name what exactly diverged, and
ask whether to apply the edit on top. Only after an explicit "yes" is one repeat
attempt with the same JSX and fresh versions allowed; a repeat preview is not needed if the JSX has not
changed. A second `ChangeConflict` — stop and tell the user.

If the fresh JSX does not match the snapshot, the template was changed concurrently. It is forbidden
to blindly repeat the save, overwrite the fresh JSX with the old edit, or merge automatically.
Stop and report:

> The template changed after it was read. Saving was stopped so as not to overwrite someone else's edit.

If the user wants to continue: fresh JSX → re-apply the requested
change → report that the previous preview is stale → optional preview/QA on request →
confirmation → save.

In the bootstrap branch, before the first successful save, there is no snapshot. On
`ChangeConflict` do not apply the usual comparison with the snapshot and do not repeat the save
automatically:

1. First re-read only `campaign_get` and find the same `formatInternalId` that
   the save was made into.
2. Continue the bootstrap check only if this format still explicitly
   has `html: no`, and the editor kind is `Mindboxeditor` or `Rawhtml`. With `html: yes`,
   an unknown editor kind or a target that disappeared/changed, stop before
   `visual_template_get` as with a concurrent edit.
3. Only after that call `visual_template_get`. If a visual template has appeared,
   do not repeat the bootstrap without a version: go to attachment recovery with the JSX found
   and `visualTemplateRowVersion`.
4. If the visual template is still missing and the same empty target is confirmed,
   show the user the unchanged target and request a new explicit confirmation.
5. After confirmation, exactly one bootstrap retry with a fresh
   `mailingRowVersion` and without `visualTemplateRowVersion` is allowed. A repeat preview is not needed
   only if the JSX has not changed since the previous save attempt; there is no need to run a preview
   until the user themselves asks to refresh the preview.
6. If the bootstrap retry again returned `ChangeConflict`, stop; there are no further
   automatic attempts.

## Routes

### New email

If the email is saved into a campaign: `campaign_get` → target choice →
`visual_template_get`/bootstrap discovery → if the format already contains an email, the question about
replacement and about its shared styles (§2 items 2a–2b) → Generator determines the needed images
→ Ops finds existing assets in the gallery or opens the upload panel, and the user
puts files from their machine into it → Generator creates the JSX → Ops reports that the previous preview
is stale/has not been opened yet, and offers a new preview or a full HTML/PNG QA through MCP →
Generator corrects the JSX if needed → the canonical visual workflow saves
the email after confirmation with the preview status. Images are still
resolved before JSX generation, but the upload panel is not opened before the target is checked.

### Editing an existing email by ID

`campaign_get` → check `kind`/`state`/`transactional` → choose
`variantInternalId` + target formats (Draft, if it exists, and Active) →
`visual_template_get` → the canonical visual workflow. If the template cannot be expressed in JSX,
stop and offer the UI; if the email is raw HTML and a visual one is requested — the "Changing the template
type" section. There is no longer a workaround conversion from serialized JSON.

### Changing the template type

"Rebuild it in the visual editor", "convert it to HTML" — the template type belongs to the format and
does not change through MCP: there is no tool, `visual_template_save` into `Rawhtml` with `html: yes` is forbidden,
and `htmlBody` can only do the reverse. The formats of one campaign are of the same type, so looking inside
it for a format for a different route is pointless. The refusal is known in advance, so iterating over tools,
saving "as a try" and retrying after a refusal are forbidden.

The only path is a new campaign in which the same email is built straight away in the needed
editor; the original stays untouched. Offer it and wait for explicit consent; without it
nothing is created. After consent: `campaign_create` per the "New campaign from scratch" section with
the same `mailingKind`, carrying over the subject/sender/preheader through `campaign_edit_content`,
the name/UTM — through `campaign_edit`, then the canonical visual workflow. Take the email's source
from `campaign_get` with `includeBodyForVariants`; do not reconstruct it from memory. Say in advance that
recipients, segment, subscription and schedule are not copied, and that table layout and non-standard
HTML are not carried over one-to-one into editor blocks.

### Hero vs subject

- «заголовок» ("heading"), hero, heading or content inside the email — the visual JSX workflow;
- «тема письма» ("email subject") / subject — `campaign_edit_content`, without JSX;
- «оформление», «дизайн», «общие стили», «стиль всех кнопок/заголовков» ("styling", "design", "shared styles", "the style of all buttons/headings") — this is **not**
  subject but the email's shared styles: `visual_template_theme_get` and the `<Theme>` tag in the JSX.

### Editing supplied JSX

Ops accepts the input → Generator makes only the requested changes → Ops marks
the previous preview as stale and runs preview/QA only on the user's request
or for diagnostics → the user confirms → Ops saves if there is a target mailing.
Existing personalization, the unsubscribe token, Html and other round-trip-sensitive
values do not change without a request from the user.

## `campaign_edit`, `campaign_edit_content`, `campaign_create`

For `campaign_edit` and `campaign_edit_content` no preview is performed. An explicit request
from the user with a specific campaign and exact new values counts as
confirmation of an ordinary metadata edit. Show the exact diff and request separate
confirmation if the target/value was inferred by the agent, the plan changed, or only
part of the request is being performed. Always confirm changes to the schedule, transactional/restriction flags and
ignoring of restrictions separately before the call.

**Before any metadata write, check `kind`/`state`/`transactional` from `campaign_get`, as in
"Editing the visual template" §1 item 3.** Do not edit a campaign in a state not allowed there, or a
transactional campaign, with either `campaign_edit` or `campaign_edit_content`: the subject, sender,
preheader and schedule of a live campaign reach customers just like the email body. Stop the workflow and
tell the user to edit such a campaign in the editor.

### Campaign version chain

`rowVersion` in `campaign_get`/`campaign_edit`/`campaign_edit_content` and
`mailingRowVersion` in `visual_template_save` are the same optimistic locking token
of the campaign; do not confuse it with the separate `visualTemplateRowVersion`.

1. Start with `currentMailingRowVersion = campaign_get.rowVersion`.
2. Each `campaign_edit` or `campaign_edit_content` gets the current token and on
   success returns a new `rowVersion`; immediately replace
   `currentMailingRowVersion` with it.
3. The next write operation gets only the updated token. Do not reuse the old version after
   a successful edit/save.
4. `visual_template_save` gets this token as `mailingRowVersion`, and its
   successful response again replaces `currentMailingRowVersion`.
5. If the token is lost, the response was incomplete, or there is any doubt — re-read
   `campaign_get` before the next write operation.

### New campaign from scratch

If the user wants to create and prepare a campaign without an existing mailingId,
follow a single route:

1. Ask the user for **the folder and the brand in words, not as identifiers**, and the time zone. Find
   the values for `campaign_create` yourself: `entities_list(entityType: "Folder")` returns
   the folder's `internalId` and its brands, `entities_list(entityType: "Brand")` — the brand's system name
   (in this tool the brand is specified by it, not by GUID). Nothing found or several candidates —
   show what was found to the user and let them choose, but do not ask them to dictate a GUID. To prepare for
   sending, collect the confirmed `name`, subject/sender/reply-to/preheader, UTM,
   schedule/timezone/rate and restriction flags. Do not invent values.

   **`mailingKind` is a decision, not an optional field.** The tool creates
   `Manual` by default, and **the campaign kind does not change after creation**: `campaign_edit`
   does not accept it; it can only be fixed with a new campaign. Determine the kind from what
   the user described:

   - **`Automatic`** — an automated campaign ("Automated campaigns" in the UI): the email is sent by a flow in response to an event: an abandoned
     cart, viewed products, welcome, order status, reactivation, a
     birthday, any "when the customer …". Such an email has access to the event's data —
     the order, the cart, the views.
   - **`Manual`** — a bulk campaign ("Bulk campaigns" in the UI): a one-off send to a segment that the user launches from
     the interface: a promotion, an announcement, a digest.

   Do not guess by default: if the kind cannot be read unambiguously from the request, ask
   one question before creation. The content depends on the kind — see below.
2. `campaign_create` → immediately `campaign_get`; from then on track `currentMailingRowVersion`.
3. `campaign_edit_content` — subject/sender/reply-to/preheader;
   `campaign_edit` — name/UTM/schedule/timezone/rate/restrictions. After each
   write, update rowVersion through the chain above.
4. The email body: Generator creates JSX → canonical visual workflow/bootstrap below →
   optional preview/QA → a confirmed `visual_template_save`.
5. Set the recipients per `references/recipients.md`. Topic/subscription and live
   send/activate/delete are not configured through MCP: at the end, say explicitly that this has to be
   done in the UI. Offer a test
   send of the finished email per the "Test send" section.

**The campaign kind restricts the email's content.** A `Manual` campaign has no event that could be
read, so the following make no sense in it:

- personalization parameters marked *(automated only)* — they read the order;
- product rows on mechanics that look at the customer's action — the cart,
  viewed products, order items. Recommendations and product lists work in
  both: they do not depend on an event.

If the email the user asks for relies on such data, the kind must be
`Automatic` — say so before creation, not after the email has been built in `Manual`.

### `campaign_edit`

Use it for campaign settings: name, UTM, schedule, rate limit, timezone and
sending restrictions; recipients — per `references/recipients.md`. Always pass a fresh `mailingRowVersion`: from a fresh
`campaign_get` or from the last successful save/edit. If the version is lost or
doubtful, re-read `campaign_get`. UTM, schedule and rate are supported by the tool's
contract; only a name change is considered live-verified.

### `campaign_edit_content`

Use it for subject, senderName/senderEmail, replyToName/replyToEmail, preheader,
and raw HTML only on an explicit request for raw HTML. A fresh
`mailingRowVersion` is needed before the call. v1 supports only a single-variant email campaign: if
`campaign_get` returned several A/B variants, stop and report the limitation.
Do not ask to choose `variantInternalId` — `campaign_edit_content` does not accept it, and
such a choice will not make the call valid; direct the user to the UI or another
confirmed route. Subject/preheader can be set to non-empty values;
do not clear subject/preheader to empty/null through the current MCP — report the
limitation and direct to the UI. Nullable behavior and `preHeaderPresentationMode`
are not described by the current contract — do not use them.

`htmlBody` is not a fallback for `visual_template_save`; switching to raw HTML
is allowed only on an explicit request from the user.

### `campaign_create`

1. Call `campaign_create` only after the user has explicitly chosen folder/brand/timezone;
   do not invent values.
2. After creation you must run `campaign_get(new internalId)` and discover
   the variants/formats.
3. An empty format (`html: no`) may exist before the visual template is initialized,
   and allows a bootstrap with editor kind `Mindboxeditor` or `Rawhtml`.
4. For the empty format found, call `visual_template_get`. The response
   `No visual template was saved for format '…' yet` together with `html: no` turns on
   the bootstrap branch of the canonical workflow: new JSX → stale-preview notice →
   optional preview/HTML/PNG QA on request → confirmation with the preview status
   → the first `visual_template_save` without `visualTemplateRowVersion`.
5. If no format is found, do not invent an ID and do not save "as a try"; report that
   the format needs to be initialized in the UI or another confirmed route is required.
6. Do not overwrite `Rawhtml` with `html: yes` with a visual save. Handle `Rawhtml` with `html: no`
   as bootstrap/recovery based on the result of `visual_template_get`.

## Test send

A test email is a real send to staff test recipients, not a preview.
The published (Active) content of the chosen variant goes out.

1. Call `campaign_test_recipients`: it returns the test recipients of the
   staff account — `recipientId`, name, email and phone with the platform's verdict on
   the validity of each contact.
2. Show the list to the user by names and contacts, never by id, and wait for
   an explicit choice of recipients and consent to send. Do not choose recipients silently,
   even if there is one candidate.
3. Call `campaign_send_test` with `campaignInternalId` and `variantInternalId` from
   `campaign_get`, a fresh `rowVersion` per the campaign version chain, and `recipientIds`
   only from a fresh `campaign_test_recipients` response — never a customer id and
   never an email. If there are several A/B variants, the user chooses the variant.
4. Report the result by contacts, not by id.
5. If the backend answered "is not ready for test sending", the variant is not fully filled in
   (subject/sender/content/DKIM): return the error verbatim and do not retry.
6. On a timeout or no response, do not repeat the call: the send may already have
   happened. Tell the user; a repeat — only on their explicit request.

## Safety boundaries

1. Do not invent `mailingInternalId`, `variantInternalId` or `formatInternalId`.
2. Do not choose an ambiguous A/B variant silently. The targets of a visual write are all formats of the variant for which the visual workflow is allowed: Draft first, if it exists, then Active, with the same JSX and two calls.
3. Editing — only in the states from "Editing the visual template" §1 item 3; everything else and `transactional: yes` — preview only; editing is done in the editor.
4. Preview is not performed automatically after each edit and is not a
   mandatory save gate. Before save the user must see the preview
   status; if it is stale or not opened, the confirmation explicitly allows
   saving without a new preview or runs preview/QA first.
5. One save — one `formatInternalId`; there is no batch write.
6. Before `visual_template_save` — one explicit confirmation with the preview status;
   for `campaign_edit` and `campaign_edit_content` a separate metadata policy
   from the corresponding section applies.
7. On `ChangeConflict` the fresh stored JSX is compared with the original snapshot, not with the edited JSX; there is no automatic retry: the agent names the divergence and asks the user whether to apply the edit on top, and only after explicit consent makes one attempt. In the bootstrap branch a retry is allowed only after re-confirming the same `formatInternalId` that the save was made into, with `html: no` and editor kind `Mindboxeditor` or `Rawhtml`, the result of `visual_template_get` and new consent from the user; a repeated conflict always stops the workflow.
8. `htmlBody` is not a fallback of the visual save; switching to raw HTML is allowed only on an explicit request from the user.
9. Do not clear subject/preheader to empty/null through MCP — nullable behavior is not described by the contract; report the limitation and direct the user to the UI.
10. Live send/activate/delete are not available through MCP — UI only; recipients — only per `references/recipients.md`, after separate confirmation. A test send (`campaign_send_test`) — only to `recipientId`s from a fresh `campaign_test_recipients`, after the user has explicitly chosen the recipients and consented to the send; on a timeout the call is not repeated.
11. Preview/save/editor errors are returned verbatim; silent repair is forbidden.
12. Validation errors are separated into pre-existing ones and those introduced by the current edit.
13. For images use only the `url` from `gallery_images_list`, the address from the panel context / the `gallery_image_upload` response, or an external HTTPS URL that the user asks to leave as is; do not build URLs by hand and do not encode images yourself.
14. The format's template type does not change through MCP: there is no workaround; retrying a refused save and `htmlBody` are not workarounds. The route is the "Changing the template type" section: a visual format of the same campaign, otherwise a new campaign after the user's consent.
15. Anything not available in the MCP tools (activation, live send, deletion, creating a format, changing the template type, recipients from a file), the agent does not do on the user's behalf: neither in the editor nor through browser automation. The browser is allowed only for reading — preview and contact sheet. If you run into such an action — name it, say where the user can do it, and stop.
16. `Rawhtml/html:yes` is never a target of a visual save. `Rawhtml/html:no`
    allows bootstrap/recovery based on the result of `visual_template_get`; the success of the first
    save is confirmed by a fresh `campaign_get`: the `formatInternalId` that was
    written to, with its original status, `Mindboxeditor` and `html: yes`.

## References

| Situation / signal | File | What's there |
|---|---|---|
| You are composing MCP `feedback` text and it is unclear how to write `problem`/`context` | `references/feedback-examples.md` | Examples of user-report and agent-observation from real runs + anti-examples |
| The user asks to view/choose images from the gallery visually | `references/visual-gallery-selection.md` | The full Codex/Cowork contact sheet procedure, the card template, the result format |
| HTML download, PNG, mobile/desktop, QA/debug, rendering diagnostics or fallback (a simple preview stays in the core) | `references/preview-qa.md` | The extended QA procedure: download HTML, MCP PNG links, diagnosing complaints, repeats and fallback |
| Set or change the campaign's recipients: a segment or a filter | `references/recipients.md` | Choosing the method, `segments_list` / the filter-building skill (`filter-build`), confirmation, writing |
