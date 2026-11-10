---
name: email-ops
description: Operational actions for Mindbox emails — preview, reading and saving the visual template, saved templates (an email by one, changing, creating, copying), reading/editing campaign metadata and recipients (a segment or a filter), finding gallery assets via MCP, uploading images from the user's machine through the MCP App panel, PNG preview. Use together with the email skill when images, a preview, a check or a write to Mindbox are needed.
metadata:
  version: 1.1.2
---

# email-ops — Mindbox operations for emails

The project-aware executor of all integration actions. The JSX generator is
`email`; this skill does not invent email content and **does not change JSX
silently**. Preview, editor and save errors go back verbatim — to the generator when they are about the
JSX (they are the editor's text, with line numbers), to the user otherwise; do not rephrase, classify or
repair them silently.

## Communicating with the user

JSX, internal IDs and rowVersion are internal execution details. In normal conversation with the
user say "email", "email layout", "content", "preview" and "saving",
not JSX, the campaign's `internalId`, `variantInternalId`, `formatInternalId`, `rowVersion`
or `visualTemplateRowVersion`. Do not show raw JSX or GUIDs unless the user explicitly asked for code,
export, markup or technical details.

A working automated campaign keeps its email in two versions, and the user sees both: the "active" one,
which goes out to customers, and the "draft", which the user will apply in the editor. Call them exactly
that. A campaign that is not working has one version: do not call it a "draft" and do not name the
version at all — the user sees one email. The tools call the draft "version being prepared" — that is
their language for the agent, not addressed to the user. "Working" is what `campaign_get` says in its
editability line ("This campaign is working, and …"), not a guess from `kind`: an automated campaign in
`in development` or `ready to send` has not gone anywhere yet and is not working.

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
   `[skill-feedback:v1]`, followed by the fields `skill=<email|email-ops>` (the skill whose
   instructions the problem is about), `skillVersion=<metadata.version of that skill's SKILL.md>`, `source=user-report|agent-observation`,
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

## Quickstart: pick a route

| Task | Route |
|---|---|
| New JSX email | target discovery (§1–§2) if a save is needed → images → Generator → preview → confirmation and save (§4–§5) |
| Editing an existing email by ID | `campaign_get` → §1–§7 |
| Editing JSX the user supplied | Generator makes only the requested changes → preview → confirmation and save, if there is a target campaign |
| Saved templates: an email "by template X", "show my templates", change, rename, create ("save this email as a template"), copy | `references/saved-template.md` |
| A saved editor block | `visual_template_saved_block_list` → the user chooses in the panel → `visual_template_saved_block_get` with the `internalId` from their message → the Generator places it; a row the get refuses — ask for another block, no look-alike |
| Change the content inside a `<QuokkaBlock>` of an email that was read or of a library block | `visual_template_quokka_block_describe` by the block's `templateId` → the Generator → §4–§5 |
| Change the content inside a `<SyncBlock>` | not through MCP: say it is a «Сквозной блок» (in English «Universal block»), edited only in the editor |
| «заголовок» ("heading"), hero, content in the email body | the visual workflow, not subject |
| «оформление», «дизайн», «общие стили», «стиль всех кнопок/заголовков» ("styling", "shared styles", "the style of all buttons/headings") | the email's shared styles: `visual_template_theme_get` and `<Theme>` in the JSX (§3), not subject |
| Change the template type: "rebuild it in the visual editor", "convert it to HTML" | "Changing the template type" |
| «тема письма» ("email subject"), sender, preheader | `campaign_get` → `campaign_edit_content` |
| Name, UTM, schedule | `campaign_get` → `campaign_edit` |
| A block or a row only for a segment, or hidden from one | `segments_list` with `entityType: User`, only segments of `Active` segmentations, the user picks by name → show or hide → the Generator writes `targeting` → the line in the save confirmation |
| Set recipients: a segment or a filter | `references/recipients.md` |
| A new campaign from scratch | "New campaign from scratch" |
| Test send of an email | "Test send" |
| An image: in the gallery / from the user's machine / by an HTTPS link | "Images" |
| PNG, desktop/mobile, HTML check | "Checks, preview and QA" |

If data is missing:

- there is no mailing internalId for reading/editing by ID — stop and ask;
- there is no exact target for the save (`variantInternalId` + `formatInternalId` of the version you write into) or no save confirmation — stop and ask;
- do not invent the campaign's `internalId`, `variantInternalId`, `formatInternalId`, a URL, a gallery asset or a `fileName`.

Live send/activate/delete, applying the draft, creating a format and recipients from a one-off file are not
available in MCP. The agent does not do them on the user's behalf — neither in the editor nor through browser
automation: name the action, say it is done in the campaign UI, and stop.

## Images

Ops chooses nothing silently; target discovery comes first (Quickstart, "New JSX email"). Generator always gets the
pair `{url, fileName}` from the tool output or the panel line.

### Finding an already uploaded image

1. `gallery_images_list` by the name from the user's request.
2. No candidates — say so. More often a miss means a different spelling than a missing file: if the user
   expected the image to be there, ask about the name rather than uploading a second copy. Otherwise offer an
   HTTPS link or the upload panel.
3. Whenever an image is offered, say whether it is a platform system icon or a project asset.
4. One candidate — return `{url, fileName}`, `fileName` as printed in the same row.
5. Several candidates — do not choose. Show `fileName`; size/date only if present, the URL only if the options cannot be told apart without it. One social
   network often has several icon designs — ask. A missing network — ask for a link or a file; no look-alike.

### Visual image selection from the gallery

In a client that supports MCP Apps, every `gallery_images_list` call also opens a picker panel with a search
box and thumbnails; the user marks images and presses "send to chat", and the message carries one
`file name — URL` line per image. The text answer is the same with or without the panel.

- With several similar candidates, or when the user wants to see the images, suggest choosing in the panel.
  One call is enough: the panel pages and searches on its own.
- Take `url` from the line and the file name as `fileName`. For an image stored in WebP this is a link to its
  PNG copy — that is normal.
- Several images came while the email needs one, or it is unclear which line goes where — ask.
- The user sees no panel — their client does not support it: say so and continue with the text list.

### Uploading images

`gallery_image_upload` — how to use it is in its description and responses. The skill adds:

- **When to upload.** The trigger is the intent to put an image into the email, not the presence of an
  attachment. A bug screenshot, a mockup reference, "this is how it looks now, fix it" — just look at the
  image and upload nothing. If it is unclear what is wanted, ask.
- **What goes where is the user's decision.** A file name is a label for the conversation, not a description
  of the content: do not guess by name or order, ask.
- **A link from the user** that is not https: `http://` — ask for an https one or a file; a
  `data:`/`blob:`/`file:`/`cid:` string is not a link — ask to save the picture as a file and use the panel.

### Gallery errors

If a gallery tool returned an error, pass its meaning to the user and stop the image-dependent part. Do not
invent a cause, do not ask for local credentials, do not substitute a similar image. Without a URL from the
tool the email can be built only without this image, by agreement.

## Checks, preview and QA

- **Every generation or edit of the JSX ends with one `visual_template_preview`** of the turn's final JSX — the
  user has to see what was built. Open both snapshots yourself first (with a mockup — compare them with it,
  `email/SKILL.md` step 4), then give the user the widget and the `htmlUrl` link. Operations without JSX —
  metadata, recipients, a test send — get none; a campaign that is not edited from here may get one as read-only.
- **A statement about the layout is made only after both snapshots were opened.** The columns of a row collapse
  into a stack on mobile by default, and what stands neatly on desktop may fall apart. Could not open them — say
  so instead of describing the layout: the user opens the links.
- Each call opens a new widget; old ones are not updated. If the user asks which one to trust — the last one;
  earlier ones show earlier versions.
- **The preview is the editor canvas, not the email that is sent**: sample values are not customer data, and
  where the canvas differs from the sent email is in the wiki page `email.tags.template`, "In the preview".
- **The preview takes the settings of the template the email will have once saved, when there is one.** Editing a
  campaign's email — the campaign's template of the version being edited, once it has a saved email; building by a
  saved template — the saved template; a campaign without a saved email that is not built by a saved template — none.
  Without them the picture shows only the document, and looks every bit as convincing. Take the version you are
  editing, unless the user asked to show the other one.
- **Preview errors.** A bare `Internal server error` without a line number is not a value-shape error (those come
  back named): pass it on, do not retry blindly. A sync block not found on preview or save: after the editor's
  text, offer to remove its `<SyncBlock>` or to open the email in the editor.
- Never build the email's HTML or preview by hand. Preview/QA never means consent to write.

**HTML download, diagnostics of a specific rendering problem, PNG links missing — you MUST read
`references/preview-qa.md` first.**

## Editing the visual template

The only detailed workflow for reading, editing and writing JSX.

### 1. Preparing the target

1. Read the campaign with `campaign_get`. Record its name, `internalId` and `rowVersion`,
   variants, formats and pre-existing validation errors.
2. Read the editability line above the variants. If it says the email is not edited from here, save nothing
   and tell the user such a campaign is edited in the editor; a preview is still allowed.
3. The write target is **one version of the email**: `variantInternalId` + `formatInternalId` from those
   `campaign_get` lists. A working automated campaign lists two: the edit goes into the draft; if there is none
   yet, write into the active one — the save itself creates the draft and says so.
4. Record the editor kind of the target format:
    - `Mindboxeditor` — the visual workflow is allowed;
    - `Rawhtml` with `html: yes` — existing raw HTML, never a visual target: say the email is edited in the
      editor; a request to convert it — "Changing the template type";
    - `Rawhtml` with `html: no` — an empty-bootstrap candidate: go to `visual_template_get` (§2);
    - missing or unrecognized — do not guess; stop.
5. One A/B variant — no question, but show it in the confirmation. Several — ask the user to choose before
   editing. `variantInternalId` is the A/B variant, not the JSX attribute `themeVariant`.

### 2. Reading the initial state

1. `visual_template_get` of the campaign's template of the target version.
   `visualTemplateRowVersion` belongs to exactly that version: never pass it into another one.
2. JSX and `visualTemplateRowVersion` came back — an existing visual template; keep the exact JSX as the
   snapshot. If the format has `html: no`, first do attachment recovery: after confirmation, save exactly this
   JSX with the campaign's current `rowVersion` and the template's `visualTemplateRowVersion`, then check it as a
   first bootstrap save (§5 item 6). No edits and no new version until recovery succeeds.
   - **2a.** The task is to build a new email (a mockup, a description, a prompt, an email by a saved template)
     over an existing one — ask first: a replacement or an edit of the existing one. The user decides; pass the
     answer to the generator with the snapshot. The gate is tied to the intent: "let's start over", "redo
     everything" in the middle of editing is the same replacement, however many iterations have passed — and
     the snapshot and the styles are re-read then (`visual_template_get`, `visual_template_theme_get`).
   - **2b.** Replacement chosen, not by a saved template — `visual_template_theme_get` on the campaign's
     template with its JSX. The email's styles are its own — the second question is **mandatory**, with
     three answers: build in **these styles** (headings and buttons follow them and change with the whole email),
     **new styles**, or **reset to platform defaults**. Already platform defaults — do not ask.
     The question is mandatory because a full overwrite does not erase the styles: they belong to the format and
     survive the save. Reset values come from the same tool **with no template**. Say plainly that a complete
     "as if nobody had configured it" is not available: the styles stay the email's own, just equal to the
     platform defaults.
3. `No visual template was saved for` — check the original `campaign_get`:
    - `html: no` with `Mindboxeditor` or `Rawhtml` — a bootstrap: there is no snapshot and no
      `visualTemplateRowVersion`; the first save goes after confirmation, without `visualTemplateRowVersion`;
    - `html: yes` or emptiness not confirmed — not a bootstrap, do not overwrite: there may be raw HTML. **The
      exception** is a draft the save itself has just created (§5 item 5): it has a body as a copy of the
      active version, but no editor template yet — the retry goes per §5 item 5.
4. The template cannot be expressed in JSX — do not overwrite it.

### 3. Editing

1. Generator makes only the requested changes; each change ends with a preview.
2. The generator needs the current shared styles — to move a look into `<Theme>` or check values after the
   move — `visual_template_theme_get` on the campaign's template with the current JSX; give the fragment to the
   generator as is. Platform defaults (no template yet) are never relayed to the user as their email's styling.
3. A request to restyle nodes that follow the shared styles — the number first. Count how many other elements
   an edit of the shared styles will affect, name the number and ask: edit the shared styles or detach the named
   nodes. "Throughout the email", "all the buttons" — the choice is made: edit the shared styles and name the
   number without asking.

The state of the shared styles is needed only when the document names a look, and needs no separate call: no
format or `No visual template was saved for …` — platform defaults; an email in the format — item 2, only when the
generator needs the values. The tool's response is large; do not call it for this question alone.

### 4. Confirmation

A meaningful change to the email — a new email, or an edit the user asked for is done — ends with the offer to
save it, in the same reply as the preview. Before `visual_template_save`, one explicit confirmation of the version
the user saw in the last preview:

```text
- campaign: <name and internalId if needed>
- A/B variant: <a human-readable description, show only if the campaign has several variants>
- where the edit goes: for a campaign that is not working, do not name it — there is one email; for a
  working one, say plainly that the edit goes into the draft, not into the email customers receive now
- change: a short description
- targeting: per targeted block, in words — «<block>» only to «<segment>» / to everyone except «<segment>»; a new
  email (the first save into an empty format, or one replacing the format's email) with none, and segments never
  touched in the conversation — ask once whether some block should go only to a segment; "no" holds to the end of the
  conversation
- shared styles: this line is mandatory in **every** confirmation and is read directly from the document, not
  from memory of the questions asked — three mutually exclusive cases:
  - the document has `<Theme>` → "the email's shared styles are set by this document";
  - there is no `<Theme>`, and the nodes name their own look → "the email is saved with its own styles on each
    element";
  - there is no `<Theme>` and the nodes name no look → "the email's shared styles stay as they are, the nodes follow
    them".

  No line — the confirmation is incomplete. Separately from the line: if the document meets the condition of
  step 5 of `email/SKILL.md`, and the offer has not been made in this
  conversation — make it right here, together with the other save parameters: "Save
  this email's styles as its shared styles? Then you'll be able to change headings and buttons across the whole
  email at once." One conversation — one such question: the answer holds until its end, and neither a second
  save nor edits re-trigger it
- existing email: only if the format already had a visual template and replacement was chosen —
  "the email in the format is replaced entirely"
- style move: only if the edit moves the nodes' styles into the email's shared styles — "the styles move
  into the email's shared styles, the look does not change; from then on they can be edited across the whole email at once"
- applying line: only for a working campaign — "customers will see the edit after it is applied in
  the editor"
```

An ambiguous target is asked about earlier and separately from the write confirmation.

### 5. Save

1. **One edit — one write** into the chosen version; no second call with the same JSX to another
   version. A saved template's settings go only into the save that applies it (`references/saved-template.md`).
   Moving styles into the shared ones is **one** save: `<Theme>`, the nodes the styles left and `themeVariant` in
   one document — saved apart, the email is left without a look in between.
2. An existing template: the campaign's `rowVersion` and the `visualTemplateRowVersion` of the version you write into.
   A bootstrap: the campaign's `rowVersion` only. A draft the save itself creates needs no template version either.
3. Take from the response the new versions, the state and validation errors; the campaign version — per
   "Campaign version chain": it moves on a refusal too.
4. A refusal "the version you named is not the one the user sees" is the same concurrency event as
   `ChangeConflict` (§7). The tool names the right version — take it from the refusal text, not from memory.
5. **Partial failure on a working campaign:** the save created a draft but writing the email into it failed;
   the tool names the draft version and the campaign's new `rowVersion`. The retry goes into that version with
   that `rowVersion` and **without** `visualTemplateRowVersion`. Only the save's own response confirms
   success here: the draft is a copy of the active version and shows `Mindboxeditor`/`html: yes` before any write.
6. **After the first bootstrap save** re-read `campaign_get` and find the same `formatInternalId`. Success only
   if it became `Mindboxeditor` with `html: yes`. Stayed `Rawhtml`/`html: no`, disappeared, or unclear — do not
   say "saved", do not repeat automatically: report that the template did not attach; continue only via
   attachment recovery (item 7) after `visual_template_get` and a new confirmation.
7. **The template was stored but the campaign was not** (`The template itself WAS stored — only the campaign
   was not`): keep the returned `visualTemplateRowVersion`, do not create a new template. After a fresh
   `campaign_get` and §1 items 2–3 again, attachment recovery saves exactly the same JSX with that version.

### 6. Post-operation report

1. The campaign and the result. A working campaign — name the consequence: the edit went into the draft,
   customers still receive the previous email and will see the new one when the draft is applied in the editor.
2. Separate validation errors that existed at the original `campaign_get` from those the edit introduced.
3. A partial outcome is not "saved": the template stored without the campaign (§5 item 7); a draft created with
   the previous email in it (§5 item 5) — say a draft appeared and customers are not affected.
4. A campaign that is not ready to send — "What the campaign still needs to go out".
5. The save response is the main result. Re-read `campaign_get`/`visual_template_get` only if the response is
   incomplete, the version is lost or the next operation needs it — and always after the first bootstrap save.

### 7. `ChangeConflict`

`ChangeConflict` is a concurrency event, not a way to find out the current version. On a conflict re-read
`campaign_get` (§1 items 2–3: still edited from here, same target), then `visual_template_get`, and compare the
fresh JSX with the original snapshot from §2, not with the edited JSX.

- **Matches the snapshot** — the conflict is about the campaign's metadata. No automatic retry: say the campaign
  was changed concurrently, name what diverged, ask whether to apply the edit on top. After an explicit "yes" —
  one attempt with the same JSX and fresh versions; no new preview if the JSX has not changed. A second conflict —
  stop.
- **Does not match** — stop: "The template changed after it was read. Saving was stopped so as not to overwrite
  someone else's edit." Never repeat, overwrite or merge. If the user wants to continue: fresh JSX → re-apply the
  change → preview → confirmation → save.
- **Bootstrap, before the first successful save** — there is no snapshot. Re-read only `campaign_get` and find
  the same `formatInternalId`: continue only if it still has `html: no` with `Mindboxeditor` or `Rawhtml`,
  otherwise stop as above. Then `visual_template_get`: a template has appeared — attachment recovery with it and
  its `visualTemplateRowVersion`; still none — show the unchanged target, get a new confirmation, one bootstrap
  retry with the campaign's fresh `rowVersion`. A second conflict — stop.

## Changing the template type

"Rebuild it in the visual editor", "convert it to HTML" — the template type belongs to the format and
does not change through MCP: there is no tool, `visual_template_save` into `Rawhtml` with `html: yes` is forbidden,
and `htmlBody` can only do the reverse. The formats of one campaign are of the same type, so looking inside
it for a format for a different route is pointless. The refusal is known in advance, so iterating over tools,
saving "as a try" and retrying after a refusal are forbidden.

The only path is a new campaign in which the same email is built straight away in the needed
editor; the original stays untouched. Offer it and wait for explicit consent; without it
nothing is created. After consent: "New campaign from scratch" with
the same `mailingKind`, carrying over the subject/sender/preheader through `campaign_edit_content`,
the name/UTM — through `campaign_edit`, then the visual workflow. Take the email's source
from `campaign_get` with `includeBodyForVariants`; do not reconstruct it from memory. Say in advance that
recipients, segment, subscription and schedule are not copied, and that table layout and non-standard
HTML are not carried over one-to-one into editor blocks.

## `campaign_edit`, `campaign_edit_content`, `campaign_create`

No preview for metadata edits. An explicit request with a specific campaign and exact new values counts as
confirmation. Show the exact diff and ask separately if the target/value was inferred by the agent, the plan
changed, or only part of the request is being performed. Always confirm changes to the schedule,
transactional/restriction flags and ignoring of restrictions separately.

**Before any metadata write, read the editability line in `campaign_get`.** Neither tool is used when:

- the line says the email is not edited from here — such a campaign is managed entirely in the editor;
- the campaign is working ("This campaign is working, and …"): the subject, sender, preheader and schedule exist
  in a single copy — they have no draft, and an edit reaches customers at once.

Then stop and tell the user to edit such a campaign in the editor, where it is visible what exactly will go out.
The restriction lives here, not in the tool: the tool will accept such an edit. `htmlBody` is the email, not a
field: it has a draft, and on a working campaign the edit goes there without changing what is being sent.

### Campaign version chain

The campaign's `rowVersion` is one token for the campaign tools and `visual_template_save`, not
`visualTemplateRowVersion`; each write returns the next one.

1. **Not only success moves the version.** A save that created a draft on a working campaign, and a test send
   from the draft, rewrite the campaign before the outcome is known: if a tool named the campaign's new `rowVersion`,
   take it, success or not. So `campaign_send_test` from the draft is a write in this chain too.
2. The token lost, the response incomplete, any doubt — re-read `campaign_get` before the next write.
3. **A version conflict** on `campaign_edit` or `campaign_edit_content` — as `ChangeConflict` (§7, "matches the
   snapshot"), comparing the fields you are changing with the ones read before.

### New campaign from scratch

1. Ask for **the folder and the brand in words, not as identifiers**, and the time zone. Find
   the values for `campaign_create` yourself: `entities_list(entityType: "Folder")` returns
   the folder's `internalId` and its brands, `entities_list(entityType: "Brand")` — the brand's system name.
   Nothing found or several candidates —
   show what was found and let the user choose, but do not ask them to dictate a GUID. Settings the user names now
   (`name`, UTM, schedule, restrictions) go in after creation; the rest — "What the campaign still needs to go
   out".

   **`mailingKind` is a decision, not an optional field: the kind does not change after creation** — only a new
   campaign fixes it:

   - **`Automatic`** — an automated campaign ("Automated campaigns" in the UI): sent by a flow in response to an
     event — an abandoned cart, viewed products, welcome, order status, reactivation, a birthday, any "when the
     customer …". Such an email has the event's data — the order, the cart, the views.
   - **`Manual`** — a bulk campaign ("Bulk campaigns" in the UI): a one-off send to a segment launched from the
     interface — a promotion, an announcement, a digest.

   The kind cannot be read unambiguously from the request — ask one question before creation. A `Manual`
   campaign has no event to read, so parameters marked *(automated only)* and product rows / editor-template
   collections on mechanics that look at the customer's action (the cart, viewed products, order items) make no
   sense there; recommendations and product lists work in both. An email that relies on event data needs
   `Automatic` — say so before creation.
2. `campaign_create` → immediately `campaign_get`; the collected fields go in through the tools its description
   names; from then on follow the version chain.
3. The email body: the empty format (`html: no`) goes through `visual_template_get` and the bootstrap branch of
   §2 item 3. No format found — do not invent an ID and do not save "as a try": say the format needs to be
   initialized in the UI. The campaign link comes back from the first save; until then report the `internalId`
   and the name, never an address built by hand.
4. After the email is saved — "What the campaign still needs to go out".

### What the campaign still needs to go out

After the first save of an email into a campaign that is not ready to send (`readyForSending: no` in
`campaign_get`), and whenever the user asks to prepare it for sending, read `campaign_get` and name what is still
missing, in words, offering to fill in what MCP can:

- **subject** — `subject: not set`: offer one written from the email's content, and a preheader with it; set
  them after the user agrees or gives their own;
- **sender** — a new campaign usually has the brand's sender already: name it and keep it unless the user wants
  another; `not set` — ask for the name and the address, there is no list of senders to pick from;
- **recipients** of a bulk campaign — `Recipients: not selected`: offer a segment or a filter
  (`references/recipients.md`); an automated campaign takes its audience from the flow;
- **validation errors** of the campaign — name each one in words; an error about the email goes to the
  Generator;
- the schedule, if the user wants a date.

Fields go in through `campaign_edit_content` and `campaign_edit` by the version chain. Subscription topic and the
launch itself are done in the UI — say so last, and offer a test send of the email.

### `campaign_edit_content`

`htmlBody` only on an explicit request for raw HTML — never as a fallback for `visual_template_save`, and never
offered for an email built in the visual editor. Through the skill only a single-variant campaign is edited:
several A/B variants — stop and direct to the UI.

Write the report as a consequence, not as a version: the tool names it on a separate line — relay it and do
not make up the other case. A campaign closed for editing refuses `htmlBody`; do not offer "the subject
instead" — such a campaign is not edited at all, per the prohibition above. Do not clear subject/preheader to
empty/null — not described by the contract: report the limitation and direct to the UI; do not use
`preHeaderPresentationMode`.

## Test send

A test email is a real send to staff test recipients, not a preview. The saved content of the chosen version
goes out, not what is in your context.

1. The edit is not saved yet — save it first, or say plainly that the test goes out without it.
2. `campaign_test_recipients` → the user explicitly chooses the recipients, even with one candidate.
3. `campaign_send_test`. Several A/B variants — the user chooses. A working campaign — ask what to show: what
   customers receive now, or the draft, and send from that version. Relay where the email went
   out from.
4. "Is not ready for test sending", or the draft is not ready — do not retry: return the error verbatim and look
   up what is missing (subject/sender/content/DKIM) in `campaign_get`. Offer a test from the active version only
   on a working campaign, where it is visible.
5. "Ready" is not proof, especially with `transactional: yes`: the platform then checks neither the unsubscribe
   link nor synchronization. Do not present it as "the email is fine": the real check is the preview and the test
   email by eye. A transactional campaign is edited by the general rules.
6. The campaign could not be read — the tool refuses; do not call it again without an address to "send anyway":
   the test would go out from the active version, not the draft the user edited. Say so and come back to it later.

## References

| Situation / signal | File | What's there |
|---|---|---|
| You are composing MCP `feedback` text and it is unclear how to write `problem`/`context` | `references/feedback-examples.md` | Examples of user-report and agent-observation from real runs + anti-examples |
| HTML download, rendering diagnostics, PNG links missing | `references/preview-qa.md` | The extended QA procedure: download HTML, MCP PNG links, diagnosing complaints, repeats and fallback |
| Saved templates | `references/saved-template.md` | Picking a template; an email by it; changing, creating, copying |
| Set or change the campaign's recipients: a segment or a filter | `references/recipients.md` | Choosing the method, `segments_list` / the filter-building skill (`filter-build`), confirmation, writing |
