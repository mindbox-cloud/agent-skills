---
name: email
description: Creates and edits emails for the Mindbox email editor from a text description. Use when the user asks to create or change an email, describes an email template, or says 'сделай рассылку' / 'make a campaign'. The technical layout is handed over for preview and saving through the companion skill email-ops.
metadata:
  version: 2.0.1
---

# email — email generation

The Mindbox email editor stores an email as a tree of blocks; this skill writes that tree as JSX — `<Template>`, `<Block>`, `<FlexRow>`, `<Column>` and the elements inside — from the user's words, a mockup or an email read back from a campaign. The converter turns the JSX into the editor's JSON and refuses what it cannot read; each tag's page in the wiki holds its attributes and defaults, this file holds the procedure and the rules that span tags.

The wiki is read with the MCP tool `wiki` in the domain `visual-template-editor`: a page this file names as `email.tags.button` is read by that id, several in one call; a page's own links name the next ones the same way.

## Skill boundaries

Every operational action — gallery and uploads, conversion, preview, reading a campaign or
its template, saving — belongs to the companion skill **email-ops** ("Ops" below); this skill (the Generator) writes the JSX and corrects it from Ops's
feedback; it fixes errors about the JSX and none about the operation (gallery, ids, network). Campaigns, recipients, test sends are Ops's. A change of the template type ("rebuild it in the visual
editor", "convert it to HTML") is Ops's route too: the JSX is assembled after Ops decides where it will be saved.

## Communicating with the user

As `email-ops`, "Communicating with the user".

## Feedback to developers

A critical problem in these rules, or the user asking to pass feedback to the developers — `email-ops`, "Skill feedback via MCP feedback".

## Workflow

1. **Read the request.** Identify the blocks (heading, body, CTA, image, divider); a new email gets a footer with an unsubscribe link (`email.topics.unsubscribe_link`) unless it is transactional or its wrapper or campaign settings carry the link — not asked about, not left for later. The situation changes what step 3 writes:
   - **Adding headings or buttons to an existing email** ("Shared styles" below) — look named (in the request, the mockup, by example): write it inline, do not ask whether to take the email's look instead. Look not named: write no styles, the nodes follow the shared styles. "Like the heading above": copy that node; if it follows the styles, the new one stays empty.
   - **Editing an existing email** — everything the request did not name is written back as read, even when it looks odd (*Read email* below); list to the user only what breaks the sent email: a placeholder link, an image without a source.
   - **Building a new email into a format that already contains one** — Ops asks the user before generation; build by their answer. "In the existing styles" — nodes without styles, the variant by `themeVariant`. "New styles" or "reset to platform defaults" — an explicit `<Theme>` (`email.tags.theme`, "An email without its own shared styles"); without it the previous styles remain.
   - **An email by a saved template, or a change to one** — Ops reads the template and hands over its JSX: work on it as on an email that was read (the "Editing an existing email" case). Ops saves it with the template's shared styles — no `<Theme>` unless the user asks to change the styles.
   - **A mockup is given** (a design file, a picture of a layout, an email from another platform) — it shows where each element ended up on one screen, not how it is built. The tag follows the element's role, not its drawing: what the reader presses to act is a `Button`, however the mockup draws it, and so for every element — what it does in the email picks the tag from "Tags and pages", not the shapes it is drawn with. The layout follows the arrangement: what stands side by side in one line is the columns of one row, spacing is the distance between neighbours, never a position — and the layout must hold on the desktop and on the phone (`email.topics.containers`). Before generation, name to the user the mockup's width when it is not the width of the email being built (600 for a new one) and ask which to take: blocks of a wider email do not fit a narrower one.
   - **A reference is given** (a website, a brand, a screenshot) — fix before generation: 2–3 brand colors in `#RRGGBB`, a contrasting pair of sections, three levels of text size, and apply them through the structure of sections, background and color, not only text. The default styles are a stub for when nothing is known, not the target look.
   - **Restyling an existing node that follows the shared styles** — after Ops's question (`email-ops`, §3): "this one" — on the node, "all of them" — in the styles.
   - **Blocks for different segments** ("this banner for the «Men» segment, that one for «Women»") — the segments are an input of step 2; each such block or row gets `targeting` (*`targeting`* below). Nothing said about segments — write none.
   - **A saved editor block is asked for** ("our footer from the editor") — Ops finds it and the user picks it (`email-ops`, Quickstart, "A saved editor block"); place it instead of the named section and leave the rest untouched: `kind` `block` comes as ordinary `<Block>` JSX and is placed as it came, `quokka` and `sync` as their tags. If the email uses a theme, warn: the block's styles are fixed values and no longer follow it. Nothing found — say so, then offer to build it.

2. **Get the inputs through Ops before generating:** every image address, the theme fragment when a look moves into the shared styles, the product mechanic and tenant names from lookups, the segment of every block shown to or hidden from one, a project font's `internalId`.

3. **Generate JSX** along `Template → Block → FlexRow → Column → elements`; before the first node of any tag read every page its row in "Tags and pages" names — the first is not enough; a page the row gives "for each … written" is read for the tags actually written. A row may take more than one `wiki` call: an answer that ends with "Not shown" is finished with the call it names before the node is written. A page not found is looked up once in the domain's index under its current id. A page that still cannot be read — no `wiki` tool, the domain not found, the page in neither place, the service failing after the retry its answer allows — stops the generation: no node is written without its page, and the user is told the editor's reference is unavailable. A styled attribute as "Shared styles" says; where nothing was said about the look, write no styles (step 1), but economy of edits is not giving up styling — set color, size and background where the design requires them; heading levels by `themeVariant`. Enumerations of features, plans, benefits, metrics are a **grid** — `FlexRow` 6+6 or 4+4+4 with `background` and `borderRadius` on `Column` — not a list, unless the user asked for a list.

4. **Check the output against "Rules across elements" and "Shared styles"** — the converter does not catch them. With a mockup, compare it element by element — what each one is, what stands beside what, the distances between neighbours — on the desktop and on the phone, against the two snapshots of the preview Ops renders (step 6); whatever is not carried over is named to the user, and pixels rounded to twelfths are named with the cause (`email.topics.containers`, "The grid").

5. **The move into the shared styles.** When two or more nodes you wrote in this conversation — a new email, or nodes you added or restyled — repeat the same covered look (a read email's untouched nodes do not count; spacing and the backgrounds of blocks and rows are not a look), Ops offers to move it in the save confirmation (`email-ops`, §4); with `<Theme>` already in the document — into that tag. Consent, or an explicit "make these styles the email's design" — move them by `email.tags.theme`; a node whose look differs on mobile is moved only after its own question — the difference is lost (`email.tags.theme`, "Moving styles"). A refusal is a legitimate outcome.

6. **Output to Ops: bare JSX** — no fences, no comments, no prose. Ops renders a preview of every iteration and saves (`email-ops`, "Checks, preview and QA"); an error about the JSX comes back to you — fix it and repeat only the requested check.

## Tags and pages

```text
Template → Block → FlexRow → Column → elements        Block → CollectionRow (the product row) → cards → Column
```

`Template` holds at least one of `Block`, `QuokkaBlock`, `SyncBlock`; a `Block` at least one row; a row at least one `Column`; the converter refuses a broken chain and names the tag. A row is a `FlexRow` or the product `CollectionRow`. The page of a tag holds every attribute with its default and the snippets; here only what decides which tag to pick.

| tag | where it stands / what it holds | what it is for, and when to pick it | page |
|---|---|---|---|
| `Template` | the root, exactly one; `<Theme>` first, then the blocks | the email's width and background | `email.tags.template`; nesting `common.structure`, `email.topics.containers` |
| `Block` | an email section; rows or a `CollectionRow` | one section with its background and spacing; a strip of the page between sections (`gapAfterBlock`) only when asked for | `email.tags.block` |
| `FlexRow` | columns | the row of a section; decides whether its columns stack on the phone | `email.tags.flex_row` |
| `Column` | elements; empty only for the grid, never to centre or to space | `size` in twelfths, one row's sizes sum to 12; a card is `background`, `borderRadius`, `innerSpacing` on the column; cards side by side with buttons at one height — "Synchronized rows" | `email.tags.column`; the twelfths `common.structure`; "Synchronized rows", "The grid" — `email.topics.containers` |
| `Text` | | words and markup: a heading, a paragraph, a link in a line, a plain list inside a paragraph (`<ul>` markup, no marker image) | `email.tags.text`; the shared typography fields `email.topics.typography` |
| `Button` | | the call to action — whatever the reader presses to act, however a mockup draws it: a plain caption and an address | `email.tags.button` |
| `Divider` | self-closing | a horizontal line | `email.tags.divider` |
| `Html` | a quoted string between the tags | the last resort — a table, Outlook conditionals — never for what the elements above express | `email.tags.html` |
| `Image` | self-closing; also a line of `Socials` | a picture from the gallery, or a dynamic one whose address comes from a parameter — a customer field, a product picture in a card (`mode: "dynamic"`: a chip, not a file) | `email.tags.image` |
| `Menu` | lines without `<Column>`: only `Text` or only `Button` | a horizontal row of links; each line is one `Text` or one `Button`, nothing else — a badge beside a menu item is `email.tags.labels` | `email.tags.menu`; lines `email.topics.groups` |
| `BulletList` → `BulletItem` | lines without `<Column>` | short one-line points with one marker image for the whole list, or any list the user asked for by name; a different marker per line — "The bullet icon" | `email.tags.bullet_list`, `email.tags.bullet_item`; lines `email.topics.groups` |
| `Socials` | lines: `Image` | a row of icons of one size | `email.tags.socials`; lines `email.topics.groups` |
| `Split` | an element, not a row of the section: stands inside a column like `Text` or `Button` and lays its own `Column size={N}` side by side, 0..1 element each; no `Split` inside | things side by side within one column — an icon beside text, a price beside its old price; never stacks on the phone | `email.tags.split`; `email.tags.text`, "Two values in one text" |
| `Labels` → `Label` | **only inside a product row card**; lines: `Label` | badges on a product card; a badge anywhere else — a word nobody presses — is a `<Text>` with its own `background` in a narrow column | `email.tags.labels`, `email.tags.label` |
| `CollectionRow` → `CardTemplate` / `CollectionCard` → `Column` | the product row, a child of `Block`; a card's columns hold the same elements plus `Labels`, minus `Menu` / `BulletList` / `Socials` | products from a mechanic — recommendations, a list, order items, viewed products, a manual selection; the mechanic, its required settings and the lookup names come first | `email.tags.collection_row`, `email.tags.card_template`, a manual selection `email.tags.collection_card`; the mechanic `email.topics.product_row_mechanics` |
| `Var` | inside `<Text>` markup, in a value's segment list (a personal link, a dynamic image, button words, a label), between `<Button>` or `<Label>` tags | the editor's personalization chip: every personal value — a name, a promo code, a price, a field of the customer; the parameter's entity decides where the chip may stand; a few chips print their own sign or word — the currency after a price, the bonus word, the greeting — never written next to them again; an unread `<personalization-parameter>` is carried over and named | `email.topics.personalization`; a parameter not among the common ones `email.topics.personalization_parameters`, a product value `email.topics.product_parameters` |
| `QuokkaBlock` → `Collection` → `Item` | a block laid out in an editor template | **only carried over** from a read email or the library, never generated; its settings via Ops's `visual_template_quokka_block_describe` | `email.tags.quokka_block`, `email.tags.collection`, `email.tags.item` |
| `SyncBlock` | self-closing | a block shared between emails ("universal block" to the user): **only carried over**, never generated | `email.tags.sync_block` |
| `Theme` → `TextStyle`, `ButtonStyle`, `LabelStyle`, `BlockStyle`, `RowStyle` | not an element: inside `<Template>` before the blocks, one at most | the shared styles ("Shared styles", *Styles*) | `email.tags.theme`, how a node follows them `common.shared_styles`; for each style tag written, its page — `email.tags.text_style`, `email.tags.button_style`, `email.tags.label_style`, `email.tags.block_style`, `email.tags.row_style` |

**Forbidden:** any tag not in the table (`MenuItem`, `SplitColumn`, …).

Pages that belong to no single tag:

| signal | page |
|---|---|
| The unsubscribe link (step 1): where it stands and its forms, the standard caption, one hidden inside a `<SyncBlock>` | `email.topics.unsubscribe_link` |
| A pixel mockup onto the twelfths grid, what spacing adds up from; «tighter», «more air», «stuck together», «too much air» | `email.topics.containers`; mobile stacking and order `email.tags.flex_row` |

## Shared styles

An email carries a set of styles per variant — `themeVariant` `h1`/`h2`/`h3`/`text` for text, `primary`/`secondary` for buttons and labels. A node that writes none of the attributes those styles cover *follows* them and changes with them. Writing a covered attribute gives the node its *own look*; write only the fields you change — the ones left out, the phone side included, come from the styles of the node's variant (`Block` and `FlexRow` have one set each, no variant), so a button with a size of its own writes `widthMobile`/`heightMobile` next to `width`/`height`. The current values are the *fragment* `visual_template_theme_get` returns through Ops — for moving a look into the styles and checking the values after the move. How a node follows, leaves and is filled in from the styles — `common.shared_styles`; the `<Theme>` tag, the move of a look into the styles — `email.tags.theme`.

**Moving a look into the styles** (step 5): the email looks the same after the move, pixel for pixel. Different values are not merged into one variant and not rounded toward one; the mobile fields move with the desktop ones. More looks than variants — say the number, move the most common, leave the rest unique: "I found six text sizes, there are four variants; I'll move the four most common and leave two unique — they are in the footer". After the move, compare what was removed from each node with its variant's set in the fragment, field by field: a look that drifted is a field missing from the set — add it, or leave the node unique.

## Rules across elements

- **Image sources.** Every address — `Image`, `Template.background`, `<img>` inside `<Html>`, `bulletIcon`, `iconSrc` — is the `{url, fileName}` pair of an image in the project's image gallery, found or uploaded through Ops — never an address of your own or a link the user gave; the image is not in the gallery yet — ask before generating; no gallery address — no image (`email.tags.image`, "Where the address comes from").
- **No address.** A new node never gets a placeholder URL. A button, an image or a menu line that needs an address and has none: ask once, for all such elements together; then write it without `url` and name it in the response (`email.tags.button`, "`url`").
- **Personal values.** A personal value is `<Var>`, never a `${…}` expression written as text. `param` and every tenant-specific setting come from `entities_list` lookups or from the user, carried as the row returned them — `systemName`, `name`, `internalId`, `type` for a custom field — never invented.
- **Read email.** Everything the request did not name — in a read email or in a library block taken via Ops — is written back as read: markup with its `data-rich-*` markers, `font` objects, `<QuokkaBlock>`, `<SyncBlock>`, placeholders and the editor's storage forms. A carried-over node gets no `style.mobile`, no number converted from a string, no attribute completed or trimmed — trimmed, it would fall under the shared styles on the next read.
- **Fonts.** A new `font.family` only from the list in the `style` / `simpleTextStyles` row of the tag's table. A font not on it may be the project's own: `font: { internalId }` from the row of `visual_template_custom_font_list` whose `name` matches the requested font — several fit, the user picks; the row's `variants` lack what the text asks for — 400, 700 with `bold`, the italic style with `italic` — or the user wants another weight, such as Light: tell the user. Not there either — the closest listed family, and tell the user the font is added in the editor.
- **`targeting`** only with a segment Ops resolved and the user confirmed: all three values from one `segments_list` row, written as JSON with double quotes; never a segment from memory or by a likely name. One read with the email is carried over unchanged — dropped, the block goes to everyone (`email.tags.block`, "Targeting").
- **Styles.** A look asked for the whole email ("Montserrat everywhere", brand colors for all buttons) is `<Theme>` at once; a look given per element stays on the element until step 5 offers the move.

- **The preview is the canvas** (`email.tags.template`, "In the preview"; what each element draws there — on its page): device-hidden duplicates are not removed for a clean canvas — that silently sacrifices the mobile layout, and a user alarmed by "blurred cards" is told they are the hidden device variants, and giving up responsive variants for one layout is a product decision made with the user; a successful preview proves the layout, not the data — an invented `systemName`, a missing lookup value or a string field in a boolean filter draw just as well.
- **Page rules a preview passes silently** — `url` on every `<CardTemplate>` and on every `<CollectionCard>` with its own columns (`email.tags.card_template`, `email.tags.collection_card`); every chip in a card that takes `formatString` — a name, a description, a string field — carries it (numbers, prices and discounts take none), *(automated only)* parameters absent from a bulk campaign, product chips only inside a card (`email.topics.personalization`); `<Labels>` only inside a product row card (`email.tags.labels`); `mobile` written where the phone must differ, not as a copy of the desktop, and for every `FlexRow` with 2+ columns decided whether they stack (`email.topics.typography`, `email.tags.flex_row`).

## Value rules

- Strings in quotes; numbers, objects and lists in `{}`. Literals only: an expression, a variable or a comment is refused.
- **On a new node write only what you change.** An object is completed field by field from the node's own defaults — or, for a covered attribute, from the shared styles — never from the previously saved email. A value of another kind is new rather than an edit — a `border` of `type: "none"`, a background of another `type`, a `font` replace the one there whole. Units are written with their width — `widthType` in `buttonSize`, a side's `type` in an image's `size`. A list is replaced whole; to reset a setting write its empty value — `0`, `""`, `{ type: "none" }`, `{ type: "transparent" }` — not nothing. A node read from the email is written back whole (*Read email*). `mobile={{…}}` is not an attribute: mobile values live inside the attribute's object (`innerSpacing={{ top: 24, mobile: { top: 12 } }}`). A covered attribute — "Shared styles".
- **A setting keeps the shape of its default.** A wrong shape is refused with the expected one shown — read the message. An unknown attribute on a tag is refused; an unknown field inside a setting's object is stored and ignored silently, so field names come from the element pages, not from memory.

## Refuse or simplify

- A backend error about parameters — “The campaign contains unknown parameters: …” on the campaign card or at save (an invented or wrong name in a chip or a mechanic setting), or `Line N, symbol M: Invalid symbol` (an empty setting) → re-check every `<Var>` and every mechanic setting against the lookups — `email.topics.personalization`, "Errors from the backend".
- Dynamic content (RSS, a feed) → refuse. A countdown timer or a video block → not writable in JSX: say so and build the rest; the user adds the block in the editor after the save. Meanwhile a video is an `<Image>` preview with the link in `url` — that is what the sent email carries anyway; a deadline is plain text.
- A request the DSL cannot express, as the tag's page states it → say so and offer the ways out that page gives; never an approximation passed off as the request.
