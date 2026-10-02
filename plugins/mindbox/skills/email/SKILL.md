---
name: email
description: Creates and edits emails for the Mindbox email editor from a text description. Use when the user asks to create or change an email, describes an email template, or says 'сделай рассылку' / 'make a campaign'. The technical layout is handed over for preview and saving through the companion skill email-ops.
metadata:
  author: Personalization + AI tribe
  version: 1.0.0
---

# email — email generation

Generates the technical layout of an email from a text description. The internal output for Ops is valid JSX, which is then converted into Mindbox editor JSON.

## Skill boundaries

This is a portable JSX generator. It **does not work directly** with gallery/upload tools,
mailing ID and rowVersion, VNet, or saving to Mindbox. All operational
actions — uploading and finding images via Ops, JSX → JSON conversion, HTML rendering, PNG preview,
reading campaign metadata/visual template, and saving — are performed by the companion skill **email-ops**.
Generator uses JSON/HTML/PNG feedback from Ops to self-correct the JSX, but does not fix
operational errors on its own and does not invent URLs.
A request to change the template type ("rebuild it in the visual editor", "convert it to HTML") is not
generation either: Ops leads the route, and the JSX is assembled after Ops decides where to save.

## Communicating with the user

JSX, internal IDs and rowVersion are internal details of the Generator–Ops interaction.
In normal conversation with a CSM say "email", "email layout", "email content",
"preview" and "saving", not JSX, `mailingInternalId`, `variantInternalId`,
`formatInternalId`, `rowVersion` or `visualTemplateRowVersion`. Do not replace the terms Active/Draft:
if they are needed to pick a version precisely, use exactly Active/Draft.
Do not show raw JSX or GUIDs unless the user explicitly asked for code, export,
markup or technical details. Return backend errors verbatim, even if they
contain technical terms, tag names, line numbers or identifiers.

## Feedback to developers

Pass feedback about the skill's work to the companion operational skill for Mindbox email —
the one whose description covers preview, save, gallery and Mindbox MCP calls
(`email-ops` in this package). It is the one that calls the MCP tools of the current
Mindbox/email MCP server. If you see a critical problem in the Generator rules
or the user asks to pass feedback to the developers, hand it to Ops and follow
its protocol: first show the full feedback text to the user, then get
explicit confirmation, and only then send it.

## Not implemented (on purpose)

- **Do not set** `targeting` on Block/Row — it is an external setting not available from the DSL.
- **Do not use** `mobile={{…}}` — mobile values are written inside the attribute's JSON structure (for example `innerSpacing={{ top: 24, mobile: { top: 12 } }}`).
- **Do not write personalization as quokka expressions** (`{{name}}`, `${Customer...}`, `${Order...}`) — there is the `<Var>` tag for it, see the "Personalization" section. The only remaining exception is the exact token `${Message.UnsubscribeLink}` in the `href` of an `<a>` link inside `<Text>`, per the "Unsubscribe link" section: a chip cannot be assembled inside an HTML attribute of markup, and the converter escapes it into text. In prop values (`Button.url`, `Image.url`) the chip works — the token is not needed there.
- **`<Image>` is allowed only with a source.** There are two kinds of source: a file — `image={{ mode: "static", static: { url, fileName } }}`, and a personal one — `image={{ mode: "dynamic", dynamic: [<Var … />] }}`, when the image address lives in the customer's or order's data. Bare `<Image />` is forbidden because the backend substitutes a base64 placeholder. Source rules are in the "Images" section.
- **`<BulletList>` without `bulletIcon` breaks the layout.** The default system marker renders as a broken placeholder icon: the backend inserts an SVG into `src` without escaping quotes. Always set an HTTPS icon explicitly. For a small dot use the live-verified flat form `bulletIcon={{ url, fileName }}` — it renders a 4px-wide marker. For a large editable marker/number badge use the constructor-compatible form `bulletIcon={{ type: "custom", url, fileName, size: N }}` — the stored template and the live preview/HTML confirmed the actual width `N`. The form `{{ mode: "static", static: {...} }}` is silently ignored, and a string crashes the preview with `Internal server error`.
- **Standalone** `<Timer>`, `<Video>`, `<BulletItem>` have no safe shorthand. `<BulletItem>…</BulletItem>` works as a line only inside `<BulletList>`; simplify a timer or video to `<Text>` or `<Button>`.
- **`<Html>` is a fallback only.** Always first try to build the design from standard flexible blocks (`Text`, `Button`, `Image`, `Menu`, `BulletList`, `Socials`, `Split` with spacing/backgrounds/grid) — they are responsive and survive manual edits in the editor. Switch to `<Html>` only if what is needed cannot be expressed with standard blocks (table layout, non-standard entities) **or the user explicitly asks for arbitrary HTML**. The content is a single quoted string only: `<Html>{"<table>…</table>"}</Html>`; the live backend rejects direct JSX inside `<Html>` (`<Html> may only contain text`). Warn that manual editing of such a block is unsafe.

## Workflow

1. Analyze the text description: identify the blocks (heading, body, CTA, image, divider) and add a footer with an unsubscribe link — it is needed in any marketing email, even when the description does not mention it. Apply the "Unsubscribe link" section and do not disguise it as a regular URL.
   **Adding headings or buttons to an existing email** — the look of the new elements is decided not by the state of the email but by whether anything was said about it. If nothing was said — not in the request, not in the mockup, not by example — write no styles: the nodes follow the email's shared styles, whatever they are, and from then on change together with them. If something was said — inline it as asked, and do not ask whether to take the look from the email instead: that would mean overriding what was said. A reference to a neighboring element ("like the heading above") does not count as a specified look — the look is taken from that node, and if it follows the shared styles, the new one stays empty.
   **Building a new email into a format that already contains an email** — Ops sees this from `visual_template_get` and asks the user before generation; build according to their answer. "In the existing styles" — the new nodes write no styles of their own, the variant is set with `themeVariant`, and the look comes from the email's styles. "New styles" and "reset to platform defaults" — write `<Theme>` explicitly: without it the previous styles remain (§8 `dsl-surface.md`), and the new nodes take their look from them. The same applies when the user says "let's start over" in the middle of editing: this is a replacement, not an edit, and Ops asks the same questions again — do not start a new email until they have been asked.
   **If a reference is given** (a website, a brand, a screenshot, "like on our landing page"), fix before generation: 2–3 brand colors in `#RRGGBB`, a contrasting pair of sections (dark/light) and three levels of text size — heading, subheading, body — and apply them throughout the email. A reference carries over through the structure of sections, background and color, not only text: an on-brand email cannot consist of sections without background and text of a single size. The default styles are a stub for when nothing is known about the style, not the target look.
2. **If the user asks for a saved editor block — take it, do not redraw it.** The signal: words about the library ("saved block", "our footer from the editor"), with or without a name. Through Ops call `visual_template_saved_block_list` — with `nameSubstring` if the block is named. The user chooses in the panel: do not choose yourself, do not ask them to name the block in words, do not iterate over name variants and do not call the tool again — wait for the message with the choice, then `visual_template_saved_block_get` with the `internalId` from it. No panel (Claude Code, CLI) — show the names from the response, the user chooses by name, take `internalId` from the same row. Nothing found — say so first, then offer to build it yourself.
   When editing, the block takes the place of the named section; leave the rest of the email untouched. If the email uses a theme, warn the user: the block's styles are fixed as explicit values, and it no longer follows the theme.
3. **Resolve all images.** Every `<Image>` needs a confirmed HTTPS URL: a URL from the gallery/search MCP workflow returned by Ops (Ops also puts an image given by the user as a link into the gallery), or a URL from the JSX of a library block. If there is no source, request an upload/search through `email-ops` before generating JSX. If the result has to be saved into a specific campaign, Ops starts image lookup/upload only after `campaign_get`, target selection and `visual_template_get`/bootstrap discovery; the "images before JSX" rule still holds.
   A personal (dynamic) image is an exception: its source is not a file but a parameter, so it needs neither the gallery nor an upload. What has to be resolved before generation is something else — `param` and, if it is tenant-specific, `systemName` from `entities_list`. See "Images" → "Personal image for the customer".
4. Generate JSX following the hierarchy: `Template → Block → FlexRow → Column → Text|Button|Divider|Html|Image|Menu|BulletList|Socials|Split`. `Template` must contain at least one `Block`, a `Block` at least one row, a row at least one `Column`; an empty `Column` is allowed. The second kind of row is the product `CollectionRow` with cards instead of columns (`references/product-rows.md`). For groups, follow the line types and the Split grid rule. Try to express every visual device with standard blocks first; `<Html>` is a fallback only (see "Not implemented").
5. For text styles use `style={{...}}` with partial merge — write only the fields you change. For a button — `simpleTextStyles={{...}}`. Partial merge is about economy of edits, not about giving up styling: set color, size and background where the design requires them. But for an element whose look is not mentioned in the description, the examples or the mockup, and nothing similar exists in the email yet, write no styles at all: the look will come from the email's shared styles and will keep changing together with them (§8 `dsl-surface.md`). Set heading levels with `themeVariant`, not with invented sizes.
   Build enumerations of features, plans, benefits, metrics as a **grid** — `FlexRow` 6+6 or 4+4+4 with `background` and `borderRadius` on `Column` — not as a vertical list. Keep `BulletList` for short one-liners and explicitly styled numbered-point/badge elements per the special pattern below.
6. Check yourself against the Self-check list.
7. **Before the first save** of the email — not before handing it to Ops — offer to move the nodes' look into the shared styles. Both conditions are checked against the document itself, not against the state of the email:
   - **there is something to move**: at least one node itself names a setting covered by the shared styles — color, size, font style, background, radius (§8 `dsl-surface.md`);
   - **moving would achieve something**: the same look repeats on two or more nodes. If every node has its own look, there is nothing to move: the number of variants is finite, and the family resemblance that the move is for will not arise.

   Both hold — ask: "Save this email's styles as its shared styles? Then you'll be able to change headings and buttons across the whole email at once." If the document already carries `<Theme>` — for example, the styles were just reset to platform defaults — the question means the same but is about that tag: whether to move the repeating look of the nodes into it. On consent, move them following the procedure in §8; a refusal is a legitimate outcome, and the email is saved with its own styles on each element. **One conversation — one such question:** any answer holds until the end of the conversation and is not re-triggered by a second save or by edits between saves.
   Going **only to preview** does not close this step: the question is triggered by the turn in which saving is first requested. The state of the email ("platform default styles", "own styles") is irrelevant here: after a reset to platform defaults they are formally its own, but there is still somewhere to move the nodes' look.
   **A node with a mobile difference is moved only with the user's consent:** the shared styles have one mobile side per variant, so such a node either stays unique or loses the difference. Name the element and ask — §8 `dsl-surface.md`.
8. For handing over to Ops, the Generator's result is **JSX only** — no Markdown fences, no comments, no explanations. Do not output this JSX to the user in normal conversation; show raw code only on an explicit request for code/export/markup/technical diagnostics. Clarifying questions and messages about limitations — in the user's language.
9. If the request includes a check, preview, diagnostics or saving, step 8 is not the end: hand the JSX to Ops. Ops does not call preview automatically after each edit. After a JSX change the previous widget/link is stale: the user gets the message "the previous preview no longer matches the current version; if you like, I'll show a new one". `visual_template_preview` is called only when the user asks for a preview, HTML/PNG QA, desktop/mobile/debug, or comments on specific content/rendering; it renders the editor canvas, where personalization is filled with sample values, so the substituted values and `#` links are not defects. In a supporting host, the HTML preview opens an MCP App widget and gives a fallback link; local HTML is downloaded only for HTML QA/debug, and the desktop/mobile PNG links come in the same preview response — there is no separate call for them. Fix backend preview/save errors in the JSX and repeat only the requested check. Never build HTML/preview by hand. The main preview for the user is the MCP App widget/fallback link before save and the editor canvas after save; save is allowed after explicit confirmation with the preview status.
10. If the user asks to create/save a new campaign with metadata, do not handle
   it in Generator: hand it to Ops. Generator is responsible for the JSX; Ops runs
   `campaign_create`/`campaign_edit_content`/`campaign_edit` and separately reports that
   send/activate are configured in the UI; recipients — a segment or a filter — are set by Ops.
11. Accept a "backend unavailable" status only from Ops after a call to the specific MCP
   tool of the current operation (`campaign_get`, `visual_template_get`,
   `visual_template_preview` or `visual_template_save`) and an actual error/status.
   Do not carry the result of a ping, the root or a neighboring host over to the required MCP route.

## Hierarchy (22 tags)

```text
Template         exactly one, the root
├─ Block         1..n, an email section
│  ├─ FlexRow    1..n, a row (grid container)
│  │   └─ Column  1..n, size is required; sum of size in one FlexRow == 12
│  │         └─ Text | Button | Divider | Html | Image | Menu | BulletList | Socials | Split | Labels    0..n elements
│  └─ CollectionRow    product row: cards instead of columns, rules in product-rows.md
│      ├─ CardTemplate     0..1, the card that renders every product
│      └─ CollectionCard   0..n, the card of a manually selected product
│            └─ Column  sum of size == 12; inside — the same elements
└─ QuokkaBlock   a block from an editor template: inside only Collection → Item
```

`Menu`, `BulletList`, `Socials` and `Labels` contain direct line elements without `<Column>`; for `Split` the lines are `<Column size={N}>`. A group can sit in a regular column or in a `Split` column, except `Split` inside `Split`.

## Allowlist

| Tag | Required attribute | Optional attributes | Purpose |
|-----|----------------------|----------------------|------------|
| `Template` | — | `containerWidth`, `background` | Root (exactly one) |
| `Block` | — | `background`, `border`, `borderRadius`, `innerSpacing`, `gapAfterBlock`, `externalBackground`, `visibilityOnDevices` | Email section |
| `FlexRow` | — | `background`, `border`, `borderRadius`, `innerSpacing`, `columnsGap`, `rowsGap`, `verticalAlign`, `isColumnsMobileAdaptive`, `columnVerticalDirection`, `emptyVariableBehavior`, `visibilityOnDevices` | Row (grid container) |
| `Column` | `size={1..12}` | `background`, `border`, `borderRadius`, `innerSpacing` | Grid column |
| `CollectionRow` / `CardTemplate` / `CollectionCard` | see reference | see reference | Product row. The full allowlist, card attributes and limitations — only in `references/product-rows.md` |
| `Text` | — | `style`, `themeVariant` (`h1`/`h2`/`h3`/`text`), `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` | Text block (supports HTML markup inside) |
| `Button` | — | `url`, `align`, `themeVariant` (`primary`/`secondary`), `innerSpacing`, `buttonSize`, `background`, `simpleTextStyles`, `border`, `borderRadius`, `iconSrc`, `iconAlt`, `iconDisplay`, `iconSizeInPercents`, `visibilityOnDevices` | CTA button; `url` is required for generation by policy |
| `Divider` | — | `innerSpacing`, `border`, `visibilityOnDevices` | Divider line (self-closing) |
| `Html` | — | `visibilityOnDevices` | Arbitrary HTML block |
| `Image` | `image` | `url`, `size`, `align`, `innerSpacing`, `border`, `borderRadius`, `visibilityOnDevices` | Image with a confirmed source URL (self-closing); `size`/`align` control the image itself, `url` makes it clickable; do not set `background` — the backend rejects it |
| `Label` | — | `url`, `background`, `border`, `borderRadius`, `innerSpacing`, `contentSpacing`, `simpleTextStyles`, `iconSrc`, `iconAlt`, `iconDisplay`, `iconSizeInPercents`, `emptyVariableBehavior`, `boolFieldVisibility`, `themeVariant` (`primary`/`secondary`) | Label: short text on a background with an optional icon. **Place it only in a product row card and only as a line of `<Labels>`**: the restriction lives in the editor canvas, the converter will let a label through in a regular row — a successful preview proves nothing here. Placed directly in `<Column>`, it is rejected by the editor. The label has no `visibilityOnDevices` (`Unknown attribute`); it is set on the `<Labels>` group. Content — text or a list of segments `{["−", <Var … />]}`; a chip as a child element is rejected: `<Label> may only contain text`. It disappears entirely if a variable in it has no value — `references/dsl-surface.md`, §6 |
| `Menu` | — | `align`, `itemsGap`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` | Menu; children — only `<Text>` or only `<Button>` |
| `Labels` | — | `align`, `itemsGap`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` | A row of labels in a product row card; children — only `<Label>…</Label>` |
| `BulletList` | `bulletIcon` | `background`, `border`, `borderRadius`, `innerSpacing`, `itemsGap`, `iconTextGap`, `iconTopPadding`, `visibilityOnDevices` | Short one-line enumerations; children — `<BulletItem>…</BulletItem>`. Build features with a heading and a description as a grid, not a list |
| `Socials` | — | `background`, `align`, `innerSpacing`, `itemsGap`, `imageSize`, `border`, `borderRadius`, `visibilityOnDevices` | Social networks; children — `<Image image={{...}} />` with the URL of each icon |
| `Split` | — | `columnsGap`, `verticalAlign`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` | Children — `<Column size={N}>`; sum of `size` == 12, 0..1 element per column |
| `Var` | `param` | settings, formatters and affixes of its parameter | Personalization chip (self-closing); sits inside `<Text>` markup, in a value's segment list, or between `<Button>` tags. Parameter list — `references/personalization.md` |
| `QuokkaBlock` → `Collection` → `Item` | `templateId` on the block, `name` on the collection | see reference | **Only when carrying over an email that was read, not for generation.** A block laid out in an editor template: a direct child of `<Template>`, holds only `<Collection>`, which holds only `<Item>`. Return it as it came; the full rules — `references/dsl-surface.md` §13 |

If the email contains products — recommendations, a product list, order items, viewed products, products
from an event or a manual selection — you MUST read `references/product-rows.md` before generation.
`<CollectionRow>` is the second kind of row and a direct child of `<Block>`; do not generate it before reading the reference.

Fill in the mechanic's source settings completely before generation: the current preview already rejects an incomplete
`RECIPIENT_RECOMMENDATIONS` without the required `recoMechanicSystemName` (`RECIPIENT_RECOMMENDATIONS needs a value in its settings`);
save behavior on an incomplete mechanic has not been confirmed separately, so do not treat a successful save as proof of completeness.

Product-row safeguards:
1. Do not proceed until the mechanic, all its required settings and the project's real values have been chosen.
2. Do not invent `systemName` or identifiers: for recommendations use `RecommendationMechanic`, for a list — `ProductList`, for a manually selected product — `Product`.
3. Place product `<Var>` only inside a product row card and only from the family that matches the chosen mechanic.

**Forbidden:** `MenuItem`, `SplitColumn` and any other tags outside this table. The exception is `<Theme>` and the style tags inside it (`TextStyle`, `ButtonStyle`, `LabelStyle`, `BlockStyle`, `RowStyle`): they are not email elements, they live directly inside `<Template>` and are described in §8 `dsl-surface.md`. `<QuokkaBlock>` with its `<Collection>` and `<Item>` is in the table, but only for carrying over: do not write them yourself and do not remove them from an email you read. Do not generate bare `<Image />`, standalone `<Timer>`, `<Video>`, `<BulletItem>`, `<Label>`.

## Images

For a new `<Image>` always write the self-closing form:

```jsx
<Image image={{ mode: "static", static: { url: "https://cdn.example.com/banner.png", fileName: "banner.png" } }} />
```

`https://cdn.example.com/banner.png` is only an illustration here: in the response use a URL
only from the user or one returned by Ops from the gallery. Do not convert an external URL into an
internal one yourself: Ops puts the image from the user's link into the gallery and returns `{url, fileName}`.

Set the size and alignment of a standalone image on the `Image` itself; do not imitate them
with extra columns or side `innerSpacing`:

```jsx
<Image
  align={{ align: "center", mobile: { align: "center" } }}
  size={{ type: "fixed", width: 120, mobile: { type: "fixed", width: 80 } }}
  image={{ mode: "static", static: { url: "https://cdn.example.com/icon.png", fileName: "icon.png" } }}
/>
```

Preserve `size.equalizedImageMaxWidth` from stored templates on round-trip, but do not compute or
invent it for a new fixed-size Image. Inside `<Socials>` the size of the lines is set by `Socials.imageSize`,
and alignment by `Socials.align`.

`Image.url` is an optional HTTPS link opened when the image is clicked; it is not
the image source. Create it only from an explicit URL from the user — but in a product row
card it is mandatory and always a product chip, see `references/product-rows.md`.
Do not substitute a placeholder. Preserve an existing `Image.url` on round-trip unless the
user asked to change the link. Stored JSX may contain serialization artifacts:
`Image.url` with a bare `https://` without a path, or a URL inside `fileName` — do not treat them
as defects and do not "fix" them without a request from the user.

### Personal image for the customer

If the user asks for a "personal image", a "dynamic image link", an image "from the customer's
data" or "put a field value into the image" — this is `mode: "dynamic"`, the same mode as the
"Personalised picture for the customer" radio button («Персональная картинка для клиента») in the editor. The image address in the email is taken from
the parameter's value:

```jsx
<Image image={{ mode: "dynamic", dynamic: [<Var param="RecipientCustomFieldString" customFieldType={{ "systemName": "<CUSTOM_FIELD_SYSTEM_NAME_FROM_LOOKUP>" }} />] }} />
```

- **Do not ask for a file, the gallery or a base domain.** The field value is the whole address. Add a prefix
  only if the user named it themselves:
  `dynamic: ["https://cdn.shop.ru/", <Var … />]`. An invented domain breaks the image for all recipients.
- **There is no need to write `static`** — fields missing from `image` are merged with the default, and in dynamic
  mode a static source is not rendered.
- What to ask here is not the image but the parameter: which field to substitute. After that, the usual rules of
  the "Personalization" section apply: `param` from `references/personalization.md`, the tenant-specific `systemName` from
  `entities_list`.
- **The field's content is not checked.** If it is empty for a recipient or is not a URL, the email goes out with a broken
  image. The behavior of an empty variable is configured in the editor on the "Display conditions" tab («Отображение») and cannot be
  set from the DSL — warn the user rather than substituting a stub.
- The click address is personalized the same way: `url={["https://shop.ru/points/", <Var … />]}` or
  a single `<Var>` as a whole if the link is entirely stored in the field. The converter does not apply `https://` validation
  to such a value.

### External URL from the user

- Accept only a direct HTTPS URL and pass it to Ops: it will put the image into the gallery and return
  `{url, fileName}` — put those into the JSX.
- Insert the link as is only if the user explicitly asks not to copy it; then
  `fileName` comes from the name given by the user, or from a non-empty unambiguous basename of the URL path.
  If the name cannot be determined unambiguously, ask; do not invent it.

### Attached file or gallery image — via Ops

Generator does not upload files and does not go to gallery/search tools itself. Delegate to `email-ops`:

- **Attached file:** ask Ops to upload it to the gallery (`gallery_image_upload`). Ops
  returns the pair `{url, fileName}`: `url` is the image source, `fileName` is only a service name.
  Do not build a URL from the file name and do not present a partial upload as complete: if the file is not among
  the uploaded ones, stop and tell the user.
- **An already uploaded image:** ask Ops to find it by name. With one candidate Ops
  returns `{url, fileName}`; with several — show the user safe metadata
  (`name`, extension, size/date if available, system/project source) and ask. Do not choose by name automatically.
- If Ops reports an upload/list error, stop; do not generate JSX with the image and do not
  pick an image by name.

An image is set as the email background following the same source rules; the value shape and modes are in §4 `dsl-surface.md`, "Image background".

### Hard rules

- Do not invent URLs. Author-supplied `data:`, `base64`, `blob:`, `file:` and `cid:` are forbidden.
- An image URL must be HTTPS per the skill's policy. This is not a claim about the converter's runtime
  validation for Image.
- Bare `<Image />` is forbidden in both new and edited JSX: replace it with a confirmed URL
  (via Ops) before writing again — or with a personal source if the image is taken from data.
- The invented-URL rules extend to a personal image as a ban on an invented prefix: the address
  itself comes from the project's data, not from the model's response.
- `<Html>` does not bypass the image rules: if the quoted HTML contains `<img>`, its `src` must
  be a confirmed HTTPS URL; prefer `<Image>`.
- Do not put important text, a price, a CTA or terms only inside an image.

## Fonts

- Take `font.family` and `fallbackFontFamily` only from the two lists in `formats.md` §style —
  they are the same for all projects. A name outside the lists will silently not apply.
- A font name from a mockup, website or request may not be in the lists: then pick
  the closest one in design and tell the user what you replaced it with. When editing an email, preserve the already stored
  `font.family` unless asked to change the font.
- If the client's brand font is requested — the project's custom fonts cannot be set through generation.
  Say so directly and approximate the character of the typography with size, font style, color,
  `lineHeight` and letter case.

## Editable numbered points / badges

- If a number, dot or badge must remain a separate editable editor
  element, do not draw it with a styled `<span>` inside `<Text>` and do not use
  `<Html>`. Such a rich-text/CSS trick may look right in the email, but it does not give
  the user a proper separate editor element.
- Use only separate editor elements: `<BulletList>` or `<Image>` +
  `<Text>`; the choice of form, the different-digits rule and mobile settings are not duplicated in the core.
- If the digit/icon asset is missing, first find/upload it via Ops or offer
  the user a simple confirmed bullet; do not draw a replacement with inline CSS.
- **Before any numbered-point block you MUST read
  `references/examples.md` §14**: without it it is easy to apply one icon to all digits,
  lose the mobile setting, or end up again with a non-editable CSS badge.

## Spacing between blocks

- In a new or modified layout, by default do not use `gapAfterBlock` for
  ordinary vertical spacing. Create spacing through `innerSpacing.top/bottom`
  of the `Block`, `FlexRow` or `Column` itself, so that the section background continues under the spacing.
- `gapAfterBlock` creates an external gap between blocks. Use it only when
  the user explicitly asks for an external gap or the mockup clearly shows a strip/gap
  in an outer background color different from the background of the neighboring blocks.
- On round-trip, do not remove an existing `gapAfterBlock` in a part of the email that
  the user did not ask to change. The rule governs the new/modified layout; it does not
  permit a side rework of the whole email.
- **If a mockup is given, take the spacing from it rather than eyeballing it:** it is exactly
  in spacing that an email diverges from the original most often, and such a divergence is
  almost invisible in a preview. Saying nothing about spacing means taking the element's default, not "flush": zero
  is written explicitly, and the defaults themselves are collected in §6 `dsl-surface.md`, "Defaults that appear on their own".

## Personalization

A personalized value is written with the `<Var>` tag: `param` is the parameter name, the other attributes are its
settings and formatters. The converter builds a real chip from this, so the email stays editable in the
editor.

```jsx
<Text><p style="margin: 0;"><Var param="RecipientGreeting" /></p></Text>
```

This chip carries the whole greeting — "Hello, Anya!": do not write your own words around it, otherwise you get
"Hello, Hello, Anya!!". Your own words are set with the `affixVariableText` attribute — and then both
its parts at once, otherwise a customer without a name gets the default phrase. Details and the same approach for
bonuses — in `references/personalization.md`.

- **Rules and common parameters — `references/personalization.md`.** Open it before the first personalization
  in an email: `param` cannot be guessed, and the parameter's entity decides where it resolves. If the one you need is not in
  the list of common ones — open `references/personalization-parameters.md` (email parameters) or
  `references/product-parameters.md` (product ones, by entity), rather than picking a similar one. **Order data are email parameters:** in an automated email `OrderTotalAmount`,
  `OrderExternalId`, `OrderDateTime`, order custom fields and other `Order*` can be placed anywhere in the email.
  But product parameters and list-row parameters (`ProductName`, `ProductListItemProductPrice`, `OrderItemProductName`,
  …) resolve only inside a product row card, and which of the five families to use is decided by its
  mechanic: see `references/product-rows.md`. Do not use a product chip in a regular column: for products,
  build a product row.
- **Take tenant-specific names only from `entities_list`** — promo code pools (`PromoCodePool`), custom fields
  (`CustomField` + `ownerEntityType` by the field's owner), bonus point balances (`Balance`), external systems
  (`ExternalSystem`), product lists (`ProductList`) for the product row mechanic, the products themselves
  (`Product`) for a manually selected row, recommendation mechanics (`RecommendationMechanic`) for
  a row with recommendations. The order number is a special case: its reference list is not `ExternalSystem` but
  `CustomField` with `ownerEntityType: OrderExternalId`; which setting is looked up with what — see the table in
  `references/personalization.md`. Carry all fields from the response into the JSX: `systemName` goes into the email,
  `name` and `internalId` are needed by the editor for the preview and the select control.
- **Do not invent `systemName` and do not leave it empty.** Preview/save do not prove that a
  tenant-specific value exists and do not always catch an unfilled source setting. A non-existent or empty name
  causes validation/runtime problems, so take every `systemName` from a lookup or from the user, and if
  there is no name, do not write the parameter and ask. Values are required for `customFieldType`, `externalSystem`,
  `orderExternalIdSelection`, `promoCodePool`, `balance`, `hostname`, and an enabled `discountQuery`.
- **Do not carry the custom field's value type into the chip.** It determines `param`, and the converter infers the `type` field in
  `customFieldType` itself — one written by hand will not be rejected but saved incorrectly.
- **Personalization lives not only in text.** A list of segments is accepted by any value string: the address of
  a personal image (`image.dynamic`), a link (`url`), the button's words, the content of `<Label>`. If the user asks for
  a "dynamic" image or link — this is it, not a request for a file and not a quokka expression. Forms — in
  `references/personalization.md`, section "Where it can go"; for an image — "Images" → "Personal
  image for the customer".
- **A chip the converter did not recognize** arrives in the stored form `<personalization-parameter model="…">`.
  Leave it as is.
- If the user asks for personalization for which a name from a reference list is missing — ask for the name, do
  not offer static text: that used to be the only way out, it no longer is.

## Unsubscribe link

- **An unsubscribe link is needed by default, not on request.** A marketing email is not handed over without it:
  add the canonical rich-text anchor to the email's footer — as the last `<Block>`, in small gray text —
  even if the user said not a word about unsubscribing. There is no need to ask for permission.
- Do not add it in two cases: the user said that the unsubscribe link comes from the email wrapper or the
  campaign settings, or the email is transactional (password, receipt, order status). **Name either case in
  the response** — "I didn't add an unsubscribe link because …" — so that its absence is a choice, not a
  forgotten step.
- On an explicit request for an unsubscribe link/button — the same canonical rich-text anchor inside `<Text>`.
- Use only the exact `href="${Message.UnsubscribeLink}"`. The correct form is
  `<p style="margin: 0;">… <a href="${Message.UnsubscribeLink}">unsubscribe</a></p>`
  inside `<Text>`; if the link text is not specified, use the standard text in the email's language:
  Russian «Если письмо больше не нужно — отписаться от рассылки», English
  "If you no longer want these emails — unsubscribe".
- A full example and the explanation of why a literal token in the preview is not a defect —
  `references/dsl-surface.md` §"Unsubscribe link"; open it if diagnostics are needed.
- Do not write the token in a prop value — neither `<Button url="${Message.UnsubscribeLink}">` nor in `Image.url`.
  A chip is written there: the unsubscribe link has its own parameter, and it yields the same expression.
- **A button or a clickable image for unsubscribing** is the `SpecialLinkUnsubscribeLink` chip in `url`:
  `<Button url={[<Var param="SpecialLinkUnsubscribeLink" />]}>Unsubscribe</Button>`. Such a chip stays
  editable in the editor, unlike the token. Do this if the user asks specifically for a button;
  by default the footer is a text link.
- Fake URLs (`/unsubscribe`, `#`, `https://example.com/unsubscribe`) and other `${...}` tokens are forbidden.
- If the canonical link is already present, do not add a second one. When editing, preserve the existing
  text, placement and styling unless the user asked to change them.
- **Having a footer does not replace the link:** "present" means the exact `${Message.UnsubscribeLink}` or the chip, not
  a block with a caption about unsubscribing. The same for a footer from the library and for the footer of an email you are editing;
  adding the anchor to such a footer is allowed — it is an edit of text and a link.

A ready-made footer that can be used as is (Russian email):

```jsx
<Block innerSpacing={{ "top": 20, "bottom": 20, "left": 24, "right": 24, "mobile": { "top": 16, "bottom": 16, "left": 16, "right": 16 } }}>
  <FlexRow><Column size={12}>
    <Text style={{ "fontSize": 12, "color": "#888888", "mobile": { "fontSize": 11 } }}>
      <p style="margin: 0;">Если письмо больше не нужно — <a href="${Message.UnsubscribeLink}">отписаться от рассылки</a></p>
    </Text>
  </Column></FlexRow>
</Block>
```

For an English email the paragraph is
`<p style="margin: 0;">If you no longer want these emails — <a href="${Message.UnsubscribeLink}">unsubscribe</a></p>`.

## Divider and placeholder links

- `<Divider />` is fully supported. But set `border` completely (`type`, `color`, all four
  sides of `size` with the same thickness, usually `1`) and do not zero out `innerSpacing` — otherwise the line is in the
  markup but not visible in the email. A thickness only in `size.top` does not draw the line: with unequal sides
  it is not visible at all, and the converter accepts such a value silently, without an error — set all four
  sides to the same thickness yourself. A full-width divider is its own `FlexRow` with `<Column size={12}>`.
- If the divider was in the description or reference and the user requested preview/QA — check
  that it is **visible in the preview**, not just present in the JSX.
- Do not generate placeholder URLs (`https://`, `#`, `/`, `about:blank`, `https://example.com`).
  A button has no address — first ask for it with one question covering all such buttons at once. If the user has no address, asks to leave it as is, or did not answer about it while continuing the task — use `<Button>` without `url`: this is a button with an empty link, like a fresh one in the editor; it is visible in the email but not clickable; name such buttons in the response. Do not imitate a button with a column with a background and text. No click address for an image — do not set `Image.url`.
  In a product row card the product address is provided by the mechanic as a chip — it is not asked for.
- On round-trip of an existing email, preserve placeholders as is, but list them to the user.

```jsx
<FlexRow><Column size={12}>
  <Divider innerSpacing={{ top: 18, bottom: 18, mobile: { top: 12, bottom: 12 } }} border={{ type: "solid", color: "#cfcfcf", size: { top: 1, right: 1, bottom: 1, left: 1 } }} />
</Column></FlexRow>
```

## Value rules (partial merge)

**Write only what you change.** Objects are filled in from the defaults of the new node, not from the previously
saved email; lists are replaced entirely. Write strings in quotes, numbers/objects in
`{}`.

Copy-safe forms of common attributes (the numbers and colors below are illustrative, the syntax is valid):

```jsx
background={{ type: "color", value: "#F5F5F5" }}
background={{ type: "transparent" }}
border={{ type: "solid", color: "#CCCCCC", size: { top: 1, right: 1, bottom: 1, left: 1 } }}
border={{ type: "none" }}
borderRadius={{ topLeft: 12, topRight: 12, bottomLeft: 12, bottomRight: 12, mobile: { topLeft: 8, topRight: 8, bottomLeft: 8, bottomRight: 8 } }}
innerSpacing={{ top: 24, bottom: 24, left: 24, right: 24, mobile: { top: 16, bottom: 16, left: 16, right: 16 } }}
style={{ fontSize: 16, color: "#111111", inscription: ["bold"], align: "left", mobile: { fontSize: 14, align: "left" } }}
simpleTextStyles={{ fontSize: 16, color: "#FFFFFF", inscription: ["bold"], mobile: { fontSize: 14 } }}
```

The other forms — `buttonSize`, a standalone `align`, `gapAfterBlock`, `verticalAlign`,
as well as `Image.size`/`align`/`url`, `image` and `bulletIcon` — **you MUST check against
`references/formats.md` before generation**: their exact value shapes are not
duplicated in the core, and a guessed shape yields a bare `Internal server error` on
convert/preview without a line number.

**A setting keeps the shape of its default value.** An object is written as an object, a single
value as a single value. This rule is general, and `*Gap` is only the most frequent case: `itemsGap`,
`columnsGap`, `rowsGap`, `iconTextGap`, `cardGap`, `rowGap` accept `{ size: N, mobile: { size: N } }`,
and `columnCount` and `rowCount` of a product row take `{ value: N }`, not a number. A product grid requires
`orientation="vertical"`: a horizontal row does not read the column count.

The converter rejects anything written in another shape and **shows the whole default** — that is, the response itself
names the required shape, there is no need to look it up in the reference:

```
`cardGap` takes an object like {"size":16,"mobile":{"size":16}}, and states a number
```

So you do not have to memorize the shape of `*Gap` and the counters: if unsure, write it as you understand it and read
the response. This does not work for the forms in the list above: there the converter replies with a bare `Internal server error`
without a hint, which is why they are checked in advance.

## Text — markup inside text

`<Text>` supports HTML markup inside the tags. Write the markup directly:

```jsx
<Text>
  <p style="margin: 0;">Summer <strong>sale</strong>, <a href="https://shop.example">see more</a></p>
</Text>
```

Available tags: `p`, `h1`–`h6`, `blockquote`, `ul`, `ol`, `li`, `strong`, `em`, `u`, `s`, `span`, `a`, `code`, `pre`, `br`, `hr`, `personalization-parameter`. The live backend rejects `div` and tables (`table`, `thead`, `tbody`, `tr`, `td`, `th`) in `<Text>` (`<div> is not allowed in a text`) — move table layout into `<Html>{"<table>…</table>"}</Html>`. In explicit JSX markup write `<br/>` and `<hr/>`. Do not generate `img`, `iframe`, `script`, `style` without a separate live check; prefer an image via standalone `<Image>` or inside `<Socials>`.
Available attributes: `style`, `align`, `href`, `target`, `rel`, `model`, `data-rich-*`. Attributes inside rich-text markup are only a plain string or a marker without a value: correct is `<p style="margin: 0;">`, `<a href="https://...">`, `<span data-rich-text>`. Not allowed: `<p style={{ margin: 0 }}>`, spread attributes, expressions — the live backend returns the error `Attribute \`...\` on <...> must be a plain string`.

Plain text (without its own markup) is automatically wrapped in `<p style="margin: 0">…</p>`.

The style of the text as a whole (font, size, color) is set through `style={{...}}` on `<Text>` — not to be confused with inline `<strong>` markup inside.

## Cards with aligned CTAs

This is about cards made of **static** content that you write yourself. Product cards from a mechanic — recommendations, viewed products, order items — are a product row `<CollectionRow>` (`references/product-rows.md`); there the heights are synchronized on their own. The full list of mechanics is in `references/product-rows.md`; what people call a "cart" is not among them: the closest is `SessionGetAddedToListProducts`, products added to a list during the session.

If you need cards in a row with buttons at the same height, do not build each card entirely in its own `Column` — column heights are independent, and the CTAs will drift. Build **synchronized rows**: separate `FlexRow`s (4+4+4 or 6+6) for images, headings, descriptions and buttons — then all CTAs sit in one row with the same top coordinate. The cost: with mobile stacking the order becomes "all images → all headings → …" rather than card by card; if the card-by-card mobile order matters, it is a product decision (separate mobile rows via `visibilityOnDevices`) — discuss it with the user. Full example: `references/examples.md` §12.

## visibilityOnDevices and "blurred" blocks on the canvas

Blocks hidden for the current device (`visibilityOnDevices="mobile"`/`"desktop"`) are shown by the Mindbox canvas editor as **blurred — this is the editor's visualization, not a defect of the email**: in the sent HTML hiding works through media queries, and the PNG preview in the desktop viewport shows only the desktop variant. Therefore:

- do not remove device variants just for the sake of a clean canvas — by doing so you silently sacrifice the mobile layout;
- if the user is alarmed by "blurred cards" in the editor, explain that these are device-hidden duplicates, not broken images;
- the decision to give up responsive variants and keep a single layout is a product decision; discuss it with the user explicitly.

## Self-check (mandatory before output)

**Not applied to a library block or to `<QuokkaBlock>`:** "Values", "Gap attributes" and "Mobile typography": do not add `style.mobile`, do not convert string numbers into numbers, do not complete partial attributes. Check everything else as usual.

- **Visual hierarchy** (for emails longer than one section): at least two sections with an opaque `background`, at least one of which contrasts with the others; three levels of text size — provided either by different `fontSize` or by different `themeVariant` (`h1`/`h2`/`text`), and the latter is no worse: the sizes live in the email's shared styles; the CTA has a brand `background` if the brand or reference is known; enumerations are built as a grid, not as a single column. An email of sections without background, with text of one size and a black-and-white button is unfinished, even if valid.
- **Grid:** the sum of `size` of all columns in each `FlexRow` and `<Split>` **equals 12** (strictly ==, not ≤). For example: `12` (one), `6+6` (two), `4+4+4` (three), `8+4`, `3+3+6`. Empty columns fill up the grid or narrow the composition when a narrow one is visible in the mockup or named by the user — but they do not center: a single centered line is given by `Column size={12}` with `style.align`, not by margins of empty columns on the sides.
- **Carry-over:** elements adjacent in one source HTML/Klaviyo container
  (table row/div-row) are in one `FlexRow`, not spread across several `12` rows;
  the container's proportions are converted into integer `Column.size` summing to 12 (`600+600` of `1200` → `6+6`).
- **Completeness of carrying over from a mockup** — any kind of source: an image, Figma, HTML, a link to a page.
  Checked against the source: images and logos are in place, all text blocks, spacing and sizes,
  buttons with their links, the footer's contents, and when carrying over a whole email — the background and width on
  `<Template>`. The goal is one-to-one, even if the request contains nothing but "make it per the mockup".
  Whatever could not be carried over is named to the user as a list: there must be no silently dropped elements and no silently
  simplified styling.
- **Pixel → grid:** `Column.size` is an integer 1..12 only. A pixel width does not map
  onto the grid directly: 480px of 1200px = 4.8/12 and will be rejected by the backend. Choose the strip cut
  along twelfths boundaries: 480 → 500 (`5+7`). Do not write fractional sizes —
  the backend rejects them rather than rounding.
- **Spacing between blocks:** in a new/modified layout ordinary spacing is done through
  `innerSpacing`, so that the section background is preserved. Each new `gapAfterBlock` is justified
  by an explicit request or by an outer color gap visible in the mockup; an untouched existing
  gap is preserved during an unrelated edit.
- **Button width:** if `buttonSize.width` is set, `widthType` is next to it — without it the number is read
  as percent (the default), and `{ width: 240 }` becomes 240%, not 240px. Both values are accepted: `"percent"` and
  `"pixels"`. Pixels — for a mockup with a specific width, keeping in mind that they do not shrink on
  mobile and will be clipped in a column or card narrower than their value.
- **Runtime-required attribute:** `Column.size`. `Button.url` is optional: without it the button is saved with an empty link, and a specified address is validated by the converter.
- **Minimal nesting:** `Template` contains at least one `Block`; each `Block` contains at least one row — `FlexRow` or `CollectionRow`. Each `FlexRow` contains at least one `Column`. For the card structure of `CollectionRow`, apply the mandatory `references/product-rows.md`.
- **Root tag:** exactly one `<Template>`, nothing outside it. `<Theme>`, if present, is one and directly inside `<Template>`.
- **Button.url is validated** by the converter: `https://`, `tel:`, `mailto:`; survey links are an error. For Image a separate HTTPS policy from the "Images" section applies.
- **Values:** strings in quotes, numbers and objects in `{}`. Objects — partial merge (only changed fields). Lists — full replacement.
- **Value shape:** every `*Gap` (including `cardGap`, `rowGap`) has an object value `{ size: N }`, not a number; `columnCount` and `rowCount` have `{ value: N }`. In general, a setting keeps the shape of its default.
- **Mobile typography:** every `<Text>` where `style.fontSize` or `style.align` is set has `style.mobile` with deliberate `fontSize`/`align`. A `<Button>` with `simpleTextStyles.fontSize` set has `simpleTextStyles.mobile.fontSize`. After convert there are no unexpected `{ fontSize: 18, align: "left" }` for important desktop texts. For a `FlexRow` with 2+ columns it has been decided whether to collapse them on mobile: by default the columns stack vertically; a row of tiles or icons is kept in one line via `isColumnsMobileAdaptive={false}` (§4 `dsl-surface.md`).
- **Fonts:** every `font.family` is from the list of 19 names (`formats.md` §style), none
  invented; every Google Fonts font has a `fallbackFontFamily` from the web-safe ten;
  an existing family was not changed on round-trip without a request.
- **Escaping in text:** write markup inside `<Text>` only as explicit JSX tags (`<br/>`, `<strong>…</strong>`). A quoted string in `<Text>` is **literal text**: the live backend escapes it (`<p>` becomes visible text, not a paragraph). Move raw-HTML fragments (an unclosed `<br>`, a comment, the `&nbsp;` entity, a table) into `<Html>{"…"}</Html>`. Do not mix bare text and a quoted string in one element — live rejects it (`<Text> may only contain text`).
- **Button content:** plain text only, no markup.
- **Divider:** self-closing `<Divider />`, content forbidden; `border` is set completely with the same thickness on all four sides, `innerSpacing` is not zeroed, a full-width divider is in its own `FlexRow`; if preview/QA is performed, the line is visible in the preview.
- **Links:** not a single placeholder URL in new `Button`/`<a>`/`Image.url`; the button's address was asked for, and without an answer the button is `<Button>` without `url` and is named in the response; no button is imitated with a column with a background; a clickable
  `Image.url` is set only from an explicit HTTPS URL from the user; placeholders from an existing
  email are preserved and listed to the user. In a product row card every link is
  a product chip: `CardTemplate.url`, `Image.url`, `Button.url`.
- **Groups:** at least one line. In `<Menu>` all lines are only `<Text>` or all only `<Button>` (do not mix); in `<BulletList>` — only `<BulletItem>`; in `<Socials>` — only `<Image image={{...}} />` with a confirmed URL.
- **`BulletList`:** a supported `bulletIcon` is set: flat `{ url, fileName }` for
  a small 4px dot or `{ type: "custom", url, fileName, size: N }` for a large
  marker/badge. Without it the default marker breaks the layout. Different numbered icons are not
  placed as lines in one list with a shared icon; an enumeration of ordinary features with a heading
  and description is built as a grid, not a list.
- **Split:** contains only `<Column size={N}>`; a column has only `size` and 0..1 element. Do not nest `<Split>` in `<Split>`.
- **The format already had an email:** the replacement was confirmed by the user — including when "let's start over" was said in the middle of editing — and the look of the new content is built according to their answer: "in the existing styles" means nodes without their own styles and with `themeVariant`; "new" and "reset to platform defaults" mean an explicit `<Theme>`; there is no silently inherited styling.
- **Shared styles** (§8 `dsl-surface.md`): a node names either nothing from the covered set or all of it; part of the set — only where the node's look must not change together with the email's styles. The fields inside `<Theme>` are taken from the `visual_template_theme_get` fragment and only from there. `<Theme>` itself is written only on a request to change the styling of the whole email: exactly one, inside `<Template>` before the blocks.
- **Moving into the shared styles** (if it happened) — checked against §8 `dsl-surface.md`: the set from the variant matches the removed one field by field, by values, not by the preview; mobile sides are carried over; extra looks are left unique and their number is named; `<Theme>`, the removed styles and `themeVariant` went in one document.
- **Editor-template blocks:** every `<QuokkaBlock>` of the email that was read is in place and untouched — the same `templateId`, the same attributes, the same `<Collection>` with their shape (`count`, `count` + one card, a list of `<Item>`). None has been rewritten with flex blocks or removed "as unnecessary"; removal happens only where it was asked for.
- **Group visibility:** set `visibilityOnDevices` on `<Menu>`, `<BulletList>`, `<Socials>`, `<Labels>` or `<Split>`, but not on their lines: `<Label>` has no such attribute at all.
- **Images:** every authored `<Image>` has a source — `image={{ mode: "static", static: { url, fileName } }}` or `image={{ mode: "dynamic", dynamic: [<Var … />] }}` without an invented prefix. The source URL starts with `https://` and came from the user or was returned by Ops from a confirmed gallery DTO; there are no bare Image, no author-supplied forbidden schemes and no invented Socials URLs. If the size/alignment of a standalone image was requested, they are set through `Image.size`/`Image.align`, not with extra columns or side `innerSpacing`; a fixed size has a deliberate mobile branch. A source that arrived inside a library block is already confirmed.
- **Unsubscribe:** the email **has** a canonical unsubscribe link — or the response says why it is absent (a transactional email, or unsubscribe comes from the wrapper/campaign settings). Then, as to form: the canonical link is not duplicated; a new `${Message.UnsubscribeLink}` is allowed only as the exact `href` of an `<a>` inside `<Text>`, and in the `url` of a button or image the `SpecialLinkUnsubscribeLink` chip is used instead of the token. There is no `<Button url="${Message.UnsubscribeLink}">`, fake `/unsubscribe`, preferences-as-unsubscribe or other new `${...}`/`{{...}}` variables — personalization is written with `<Var>`, not with a quokka string.
- **Personalization:** every `<Var>` has a `param` from `references/personalization.md`; no tenant-specific setting (`customFieldType`, `externalSystem`, `orderExternalIdSelection`, `promoCodePool`, `balance`, `hostname`) is empty or invented; product families (`Product*`, `ProductView*`, `ProductListItem*`, `SingleProductListItem*`, `OrderItem*`) are only inside a product row card, not in a regular column; *(automated only)* parameters did not end up in a bulk (`Manual`) campaign. Every tenant-specific `systemName` came from `entities_list` or from the user, none is invented; `name` and `internalId` are carried over next to it if they are in the response. `type` is not written in `customFieldType`. Existing `<personalization-parameter model="…">` are not rewritten. Every text chip in a product row card (`ProductName`, `ProductDescription`, `ProductVendorName` and their counterparts in the other families) has `formatString` set: a limit from the presets `40`/`80`/`120`/`200` for products from a mechanic, `isTruncateEnabled: false` for manually selected ones. The default 150 breaks card heights.

## Unsupported → refuse or simplify

The bans themselves are stated in "Not implemented" and the topical sections; here is what
to do instead when a request runs into them.

- `<Var>` is not a blanket-unsupported tag: use it only per the rules of `references/personalization.md`, with a known `param` and the required settings filled in. Do not invent an unknown `param` or its settings. If the parameter is missing from the static list but the user names it exactly, check it through the converter; pass a converter refusal on verbatim.
- A request to change the look of one node when the node follows the email's shared styles → detach
  it from them by writing out the **whole** set explicitly (§8 `dsl-surface.md`), rather than editing the shared styles:
  an edit of the shared styles will spread across the whole email. Take the missing values from
  `visual_template_theme_get`; do not invent them.
- `targeting` on Block/Row → explain that it is an external setting: it cannot be
  emulated via JSX and is configured outside the email.
- Dynamic content (RSS, feed) → refuse.
- An image without a confirmed HTTPS URL (rules — "Images") → ask for an HTTPS URL or pass the file to Ops for the gallery; substitute nothing from forbidden schemes.
- A request for a "personal" or "dynamic" image, or an image from a customer/order field → this is `mode: "dynamic"`, not a missing source: ask for the parameter, not for a file, the gallery or a base domain.
- Bare `<Image />` in existing JSX → ask for a URL or get it via Ops per the gallery rules and replace the Image before writing again.
- Converter error `needs a value in ...` → the chip has the named setting unfilled: take the name through `entities_list` or from the user; do not remove the attribute and do not substitute a stub.
- The editor error **«В рассылке содержатся неизвестные параметры»** ("The campaign contains unknown parameters") has several causes. First read the expression itself: `Recipient.Recommendations...`, `GetProductList(...)`, `Products.GetBySegment(...)` → open `product-rows.md` and check the mechanic, source settings and family; `Order.IDs...`, promo-code, balance, custom field or external-system path → open `personalization.md` and check the exact reference list and `systemName`; the expression does not belong to any known pattern → do not diagnose from memory, show it verbatim and check the corresponding reference/lookup.
- Error `Line N, symbol M: Invalid symbol` on link enrichment or saving → the email has a chip with an empty tenant-specific setting: the expression ended up without a name (`Order.IDs.`). Find the `<Var>` whose lookup setting is not filled, take the name through `entities_list` or from the user and rewrite the JSX. Number of errors = number of broken places.
- Timer or video → explain the limitation and offer Text/Button.
- A CTA without an address → one question about the address; no address → `<Button>` without `url`, and a list of buttons without a link in the response; a clickable image without an address → without `Image.url`. No `https://example.com`, `https://`, `#`.
- A divider with a zeroed `border` or `innerSpacing` → do not hand it over as a finished result: the line will not be visible.
- The user asks for an unsubscribe button → make it the `SpecialLinkUnsubscribeLink` chip in `url`, not a token in a string. Do not generate fake URLs or arbitrary tokens.
- Editing an existing campaign "by ID only" → hand the task to Ops:
  `campaign_get` → `visual_template_get`. If the template cannot be expressed in JSX or there is no visual format,
  do not start a workaround JSON conversion: offer the UI. Separately supplied
  JSX can be edited/shown, but do not write it into an unconfirmed or
  incompatible format. Do not invent `mailingInternalId`, `variantInternalId` or
  `formatInternalId`.

## Where to go for details

| Situation / signal | File | What's there |
|---|---|---|
| Any value shapes beyond the copy-safe forms of the "Value rules" section — MUST before generation | `references/formats.md` | Exact value shapes: `buttonSize`, standalone `align`, `gapAfterBlock`, `verticalAlign`, `Image.size`/`align`/`url`, `bulletIcon` |
| Unclear which tag/attribute to apply; the full markup grammar is needed | `references/dsl-surface.md` | The full DSL specification: attributes, hierarchy, grid, text markup |
| A `<QuokkaBlock>` appeared in an email that was read — or someone asks to change something inside it | `references/dsl-surface.md` §13 | A block laid out in an editor template: what is carried over as is, why `templateId` is not invented, what to answer to a request to edit it |
| The layout's rhythm is needed: which spacing, height or size appears on its own — MUST before generation | `references/dsl-surface.md` §6 | "Defaults that appear on their own": the only place with default values |
| The email has products — recommendations, cart, viewed products, order items, a product list — MUST before generation | `references/product-rows.md` | The product row in full: mechanics and their required settings, which chip family goes with what, the card and its heights, manually selected products |
| A request about the email's shared styles ("Design" / «Дизайн»), the email's width or background, `themeVariant` — MUST before generation | `references/dsl-surface.md` §8 and §4 | How a node is detached from the shared styles, why the set is written in full, the `<Theme>` tags and root attributes |
| The first personalization in an email — MUST before generation | `references/personalization.md` | `<Var>` forms in text / value / words / structure, what comes from a lookup, common parameters |
| A personal image or a personal link | `references/personalization.md` § "Where it can go" | The form `image={{ mode: "dynamic", … }}` and the segmented `url` |
| The needed parameter is not among the common ones | `references/personalization-parameters.md` | 49 parameters of the email itself with attributes: recipient, promo code, points, custom fields, order data |
| The needed product value is not in the "value → name" table | `references/product-parameters.md` | 127 product parameters by entity: `product`, `productListItem`, `singleProductListItem`, `orderItem`, `productView` |
| A label, a badge, "discount on a background", a "bestseller" corner — MUST before generation | `references/dsl-surface.md` §6 "Labels and Label" | That `<Label>` lives only inside `<Labels>`, the content shape with a segment list, the behavior with an empty variable and the "display filter" («фильтр отображения») |
| Any numbered-point block — MUST before generation | `references/examples.md` §14 | A sample `BulletList` with a custom icon and notes on mobile |
| A typical written fragment is needed as a sample | `references/examples.md` | Input→output examples by block type |
