# DSL surface — what can be written in JSX

**When to read:** it is unclear which tag/attribute to use or how the allowed
JSX/rich-text markup works. Exact value shapes are chosen separately via the `SKILL.md` router.
**Return to:** Workflow / Self-check in `SKILL.md`.

This is a reference for JSX markup used to generate and edit Mindbox emails. It covers only what can be expressed in markup; everything else (UUID, envelope, subject, targeting, materialization, etc.) the serializer adds on its own or ignores.

This reference describes the currently accepted DSL; the mandatory backend check (JSX conversion and validation) runs inside `visual_template_preview`/`visual_template_save` in `email-ops` — there is no separate `convert` tool.

## 1. Structure

The four outer containers always come in this order; after `<Column>` comes an element, including a group:

```
Template  →  Block  →  FlexRow  →  Column  →  Text | Button | Divider | Html | Image | Menu | BulletList | Socials | Split
```

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={8}><Text>Main column</Text></Column>
      <Column size={4}><Text>Side column</Text></Column>
    </FlexRow>
  </Block>
</Template>
```

Rules:

- **Exactly one `<Template>`** per document, and nothing outside it. Use only `<Template>` as the root. `<Email>`, `<Root>` or any other root tag is rejected: `The root element must be <Template>`.
- **Each container holds only its own child.** `<Template>` — blocks; `<Block>` — rows; `<FlexRow>` — columns; only `<Column>` — elements or groups. Putting `<Text>` directly into `<FlexRow>` is an error, not a shortcut.
- **An email has a second kind of block — `<QuokkaBlock>`**, laid out not in the document but in an editor template. It sits where a `<Block>` sits, but holds collections that its template repeats rather than rows — §13.
- **A block has a second kind of row — `<CollectionRow>`**, the product row. It holds product cards rather than columns and is described in [product-rows.md](./product-rows.md). Everything else in this file is about `<FlexRow>`.
- **Use `<FlexRow>`, not `<Row>`.** The row tag in the production converter is `FlexRow`. `<Row>` is rejected as `Unknown element <Row>`.
- **Every container requires at least one child** — except the column. An empty `<Template>`, `<Block>` or `<FlexRow>` is an error. An empty `<Column>` is allowed.
- **A column requires `size`** — an integer from 1 to 12. The `size` values of the columns in one `<FlexRow>` must sum to 12 (see §5).
- **A group requires at least one line.** `<Menu>` contains only `<Text>` or only `<Button>`; `<BulletList>` — only `<BulletItem>`; `<Socials>` — only `<Image image={{...}} />`.
- **`<Split>` contains only `<Column size={N}>`.** Their `size` values sum to 12; a Split column carries only `size` and contains 0..1 element. Any group is allowed inside it except `<Split>`.

## 2. Value grammar — literals only

Attributes accept only strings, numbers, JSON objects and arrays; do not generate expressions, spread,
imports, executable code, comments or Markdown fences. This file covers
the allowed tags/attributes and markup structure; for value shapes go back to the
`SKILL.md` router.

## 3. Write only what you change

JSX → template creates a new node with defaults. An attribute changes the specified setting; omitting it
does not restore the value from a previously saved email.

```jsx
<Divider innerSpacing={{ top: 30 }} />
```

Objects are filled in from defaults key by key; lists are replaced whole. To reset, give
an explicit empty/zero value, `{ type: "none" }` for a border or `{ type: "transparent" }`
for a background.

A rule about you, not about the converter: **reading an email prints, for a node that does not follow the shared
styles, the whole look set** — for text that is all nine `style` fields. All nine are printed,
even though the shared styles do not control every one of them: alignment is part of the printed set but is not
covered by the email's styles (§8). This is
intentional: a node that said nothing about these fields would, on the next read, end up under the email's
styles, and its look would shift. Do not trim such sets when editing — keep them as they are and change
in them only what was asked.

## 4. Containers — attributes

Only these attributes. Any other one is `Unknown attribute` (not a silent drop).

| Container | Attribute | What it does |
|---|---|---|
| `Template` | `containerWidth` | Email width in pixels. A number: `700` |
| `Template` | `background` | Email background — what is visible behind the column. JSON: same as `Block` |
| `Block` | `background` | Background. JSON: `{ type: "color", value: "#f5f5f5" }` or `{ type: "transparent" }` |
| `Block` | `border` | Border. JSON: `{ type: "solid", color: "#cccccc", size: { top: 1, right: 1, bottom: 1, left: 1 } }` or `{ type: "none" }` |
| `Block` | `borderRadius` | Corner rounding. JSON: `{ topLeft: 0, topRight: 0, bottomLeft: 0, bottomRight: 0, mobile: { … } }` |
| `Block` | `innerSpacing` | Inner padding. JSON: `{ top, bottom, left, right, mobile: { … } }`. **Do not write `padding`** — there is no such attribute, use `innerSpacing` |
| `Block` | `gapAfterBlock` | Outer gap after the block. JSON: `{ desktop, mobile }`. For ordinary spacing inside a colored section, use `innerSpacing` by default; a new `gapAfterBlock` — only on explicit request or when a gap in the outer background color is visible |
| `Block` | `externalBackground` | Outer background |
| `Block` | `visibilityOnDevices` | Visibility. A string: `"all"` (default), `"desktop"`, `"mobile"` |
| `FlexRow` | `background` | same as `Block` |
| `FlexRow` | `border` | same as `Block` |
| `FlexRow` | `borderRadius` | same as `Block` |
| `FlexRow` | `innerSpacing` | same as `Block`. **Not `padding`** |
| `FlexRow` | `columnsGap` | Gap between columns. JSON: `{ size: 20, mobile: { size: 20 } }` |
| `FlexRow` | `rowsGap` | Gap between rows (mobile-stacked) |
| `FlexRow` | `verticalAlign` | Vertical alignment. JSON: `{ align: "middle", mobile: { align: "middle" } }` |
| `FlexRow` | `isColumnsMobileAdaptive` | Whether to collapse the columns into a stack on screens narrower than the email. Boolean, default `true`: each column takes the full width, and `columnsGap` turns from horizontal into vertical. `false` — the columns stay in a row at their share of the width, and a long caption in a narrow column wraps by words |
| `FlexRow` | `columnVerticalDirection` | Column order when collapsed. JSON: `{ direction: "ttb" }` (default) or `{ direction: "btt" }` — on mobile the last column comes first; the desktop order does not change. Works only with `isColumnsMobileAdaptive: true`; otherwise there is no collapsing and no order to change |
| `FlexRow` | `visibilityOnDevices` | same as `Block` |
| `FlexRow` | `emptyVariableBehavior` | A string: `"show"` (default) or `"hide"`. `hide` removes the row from the **sent** email if a variable in it is empty; in the preview the row is always visible |
| `Column` | `size` | **Required.** Integer 1..12 |
| `Column` | `background` | same as `Block` |
| `Column` | `border` | same as `Block` |
| `Column` | `borderRadius` | same as `Block` |
| `Column` | `innerSpacing` | same as `Block`. **Not `padding`** |

The root attributes concern the email as a whole, and both are optional: if you do not name one, it stays as it was. “Global (CSS) styles” («Глобальные (CSS) стили») and the Gmail annotation have no attributes and are edited only in the editor; they are carried over on save.

`Column.size` is the only required attribute among containers. The `flexColumnSize` attribute must not be used — it is the internal prop name; write `size`.

`innerSpacing` and `gapAfterBlock` are not interchangeable: `innerSpacing` keeps the container's
background under the spacing, `gapAfterBlock` shows the outer background between sections. When
creating/changing a layout, make ordinary vertical spacing with `innerSpacing`; do not remove an untouched
existing `gapAfterBlock` during an unrelated round-trip edit.

### Image background

The email background is not only a color: `Template.background` accepts an image.

```jsx
<Template background={{
  type: "image",
  url: "https://cdn.example.com/background.jpeg",
  fileName: "background.jpeg",
  fallbackColor: "#ffffff",
  mode: "cover"
}}>
```

The image URL is obtained the same way as for `<Image>` — the "Images" section in `SKILL.md`.

`mode` decides how the image fills the background area:

| `mode` | what it does |
|---|---|
| `cover` | fills it, cropping the excess along one side |
| `contain` | fits it whole, leaving empty space along the other side |
| `stretch` | stretches it over the whole area, distorting the proportions |
| `repeat` | tiles it with copies |

`fallbackColor` is not a decorative field: some email clients will not show a background image at all, so
the text on top must be readable on the fallback color too, and an image that carries meaning goes into `<Image>`, not into the background.

**Targeting** is deliberately not part of the DSL: it points outside the email, stays as it is, and writing it is an error.

There is no separate attribute for a node's subscription to the email's shared styles either, and it is **not written but inferred**: a node that names none of the settings covered by the shared styles follows them; a node that names all of them is detached from them. Which settings these are and why the set cannot be written halfway — see §8.

## 5. Grid

The `size` values of the columns in every `<FlexRow>` and `<Split>` **must sum to 12**.
`Column.size` is an integer 1..12 only: the backend rejects fractional values rather than rounding them.

```jsx
<FlexRow><Column size={6}>…</Column><Column size={6}>…</Column></FlexRow>
<FlexRow><Column size={4}>…</Column><Column size={4}>…</Column><Column size={4}>…</Column></FlexRow>
```

Do not write `size={6}` for a lone column of a multi-column row — the sum will not reach 12 and you will get an error. For one column spanning the whole row use `size={12}`. Rows and Splits are independent: each sums to 12 separately.

Pixel → grid is computed in twelfths of the container, not from an arbitrary
pixel width. For example, 480px in a 1200px container is 4.8/12; such a
`size` cannot be expressed; the nearest valid cut is 500px = `5/12`, i.e. `5+7`.

## 6. Elements

Elements live inside `<Column>`. Fully supported: `<Text>`, `<Button>`, `<Divider>`, `<Html>`, `<Image>`, `<Menu>`, `<BulletList>`, `<Socials>`, `<Split>`, `<Labels>`. Do not generate `<Timer>` and `<Video>` without a separately verified stored schema; `<BulletItem>…</BulletItem>` is allowed only as a `<BulletList>` line, `<Label>…</Label>` — only as a `<Labels>` line.

Empty nodes are emitted self-closing: `<Column size={4} />`, `<Html />`, `<Divider />`, `<Image ... />`. On write, both forms (`<Column size={4}></Column>` and `<Column size={4} />`) are read the same.

**Inside a product row card the attribute set is wider** — an element there has settings that
the same element in an ordinary column does not: `height` on `<Text>`, `<Image>` and `<Split>`, `url` on `<Text>` and
`<Divider>`. The tables below are about an ordinary column; the card additions are in [product-rows.md](./product-rows.md).

**Several values side by side are better kept as separate elements.** Two `<Var>` in one `<Text>` are one
cell: the values cannot be separated either by look or by empty-value behavior, and in the editor they can be edited
only together. Price and old price, price and discount, name and SKU are usually worth
separating.

Layout here is decided by the column and the spacing: one under another — two `<Text>` in a row in one column
(the distance between them comes from their own `innerSpacing`), side by side — the columns of a row, cards or
`<Split>`. One under another inside `<Split>` cannot be built: its column holds only one element.

### Defaults that appear on their own

An unwritten setting is not zero but the element's default, and the distance in the email adds up from
neighbors. **Flush is written with explicit zeros**: `innerSpacing={{ top: 0, bottom: 0 }}`. When taking the rhythm from
a mockup, set spacing on every element involved in it and check against this table —
the defaults live only here; they are not duplicated anywhere else.

| setting | on what | default |
|---|---|---|
| `innerSpacing` | `Text`, `Button` | 10 top and bottom |
| | `Divider` | 20 top and bottom |
| | `Label` | 8 at the bottom; for a label this is the **outer** spacing |
| | `Image`, `Menu`, `BulletList`, `Socials`, `Labels`, `Split` | 0 |
| `contentSpacing` | `Label` | 8 on all sides — padding inside the label |
| `itemsGap` | `Labels`, `BulletList` | 8 |
| | `Socials` | 12 |
| | `Menu` | 20 |
| `buttonSize` | `Button` | `{ height: 50, widthType: "percent", width: 100 }` |
| `height` — **only inside a product row card** | `Image` | 220 px, `manual` |
| | `Split` | 58 px, `manual` |
| | `Text` | by content, `original` |

A height container reserves space even when its content is shorter: in a card this is the main source
of a broken rhythm. What to do about it — [product-rows.md](./product-rows.md), "Card heights and `autosizeKey`".

### Text

Content: the text itself or HTML markup, between the tags (see §7). The `style` attribute is the typography of the whole text.

```jsx
<Text>Summer sale starts today</Text>
<Text innerSpacing={{ top: 24, bottom: 24 }} background={{ type: "color", value: "#f5f5f5" }}>
  Text with padding and background
</Text>
<Text style={{ fontSize: 24, inscription: ["bold"] }}>
  <p style="margin: 0;">Summer <strong>sale</strong>, <a href="https://shop.example">see the offers</a></p>
</Text>
```

| Attribute | What it does |
|---|---|
| `style` | Typography of the whole text (font, fontSize, color, inscription, link, align, mobile). JSON. **Do not write `fontSize`, `color`, `align` as separate attributes** — they do not exist; nest them in `style={{ … }}`. If `fontSize` or `align` is set, set `style.mobile` deliberately. `font.family` — only from the list of 19 names (`formats.md` §style); for a Google Fonts font set `fallbackFontFamily` from the web-safe ten; an existing family is preserved on round-trip |
| `innerSpacing` | same as `Block` |
| `background` | same as `Block` |
| `border` | same as `Block` |
| `borderRadius` | same as `Block`. **Not `radius`** |
| `visibilityOnDevices` | same as `Block` |

### Button

Content: the button caption, between the tags. **Plain text only.** Markup inside `<Button>` is forbidden — `<strong>…</strong>` gives `<Button> may only contain text`.

```jsx
<Button url="https://shop.example/sale">Shop now</Button>
<Button url="mailto:hi@example.com" buttonSize={{ widthType: "percent", width: 60 }} borderRadius={{ topLeft: 24, topRight: 24, bottomLeft: 24, bottomRight: 24 }}>
  Write to us
</Button>
```

**Width comes in two kinds, and the kind is chosen explicitly.** `widthType` — `"percent"` (default) or `"pixels"`,
both accepted by the editor. If you write `width`, write `widthType` next to it; otherwise the number is merged onto the default
`percent`, and `{{ width: 240 }}` will turn out to be 240 percent, not pixels. The whole default is
`{ height: 50, widthType: "percent", width: 100 }`: for a full-width button write nothing.

Percent is safer: a pixel width stays in pixels on mobile too (`240px !important`), and in
a column or card narrower than its value the button gets cut off. So `"pixels"` is a deliberate choice for
a mockup with a specific width, not the default form. `widthMobile` and `heightMobile` are optional:
without them the mobile side repeats the desktop one — this is synchronization, not an omission. Write them when
the mobile value must differ.

| Attribute | What it does |
|---|---|
| `url` | Link. Write `url`, **not `href`** — `href` is rejected as `Unknown attribute`. Without the attribute the button is saved with an empty link and is not clickable in the email — this is how you leave a button whose address you asked for when the person does not know it. `https://`, `tel:`, `mailto:` (the type is inferred from the value) |
| `align` | Alignment. JSON: `{ align: "center", mobile: { align: "center" } }` |
| `innerSpacing` | same as `Block` |
| `buttonSize` | Size. JSON: `{ height, heightMobile, widthType: "percent", width: 100, widthMobile: 100 }`. Width is in percent; for pixel width and the mandatory `widthType` see above |
| `background` | same as `Block`. Default `{ type: "color", value: "#000000" }` |
| `simpleTextStyles` | Caption typography. JSON: `{ font: { family: "Arial" }, fontSize: 14, color: "#ffffff", inscription: [], mobile: { fontSize: 14 } }`. Font and `fallbackFontFamily` — by the same rules as for `Text.style`. If a desktop `fontSize` is set, set `mobile.fontSize` deliberately |
| `border` | same as `Block` |
| `borderRadius` | same as `Block`. Default 8 on each corner |
| `iconSrc` | Icon next to the caption. JSON: `{ url: "https://cdn.example.com/cart.png" }` |
| `iconAlt` | Icon alt text |
| `iconDisplay` | JSON: `{ position: "EMPTY" }` (default — no icon) |
| `iconSizeInPercents` | JSON: `{ widthInPercents: 30 }` |
| `visibilityOnDevices` | same as `Block` |

### Divider

A horizontal line. No content, always a self-closing tag: `<Divider />`.

```jsx
<Divider />
<Divider innerSpacing={{ top: 18, bottom: 18 }} border={{ type: "solid", color: "#cccccc", size: { top: 1, right: 1, bottom: 1, left: 1 } }} />
```

| Attribute | What it does |
|---|---|
| `innerSpacing` | same as `Block`. Default `{ top: 20, bottom: 20, left: 0, right: 0, mobile: { … } }` |
| `border` | The whole line. JSON: `{ type: "solid", color: "#000000", size: { top: 1, right: 1, bottom: 1, left: 1 } }`. **Do not write `color` and `thickness` as separate attributes** — they do not exist; nest `color` and `size` inside `border={{ … }}`. **Set the whole object** — with partial merge, `border={{ color: "…" }}` takes `type` and `size` from the default. **One thickness on all four sides** (usually `1`): the line is drawn only from a value with equal sides, and with unequal sides it is not drawn at all; the converter accepts such a value silently (no error), so set all four sides to one thickness yourself |
| `visibilityOnDevices` | same as `Block` |

### Html

Arbitrary HTML between the tags. Order of choice: first the standard flexible blocks (`Text`/`Button`/`Image`/`Menu`/`BulletList`/`Socials`/`Split`); `<Html>` — only as a fallback, when what is needed cannot be expressed otherwise, or on the user's direct request. Text, button and image survive manual edits in the editor; an html block does not.

```jsx
<Html>{"<table><tr><td>Raw table markup</td></tr></table>"}</Html>
```

There is one form: a single quoted string (`<Html>{"<table>…</table>"}</Html>`). Use it for
required raw markup that cannot be expressed with standard blocks. The backend rejects direct JSX inside
`<Html>`; an empty `<Html />` creates a library placeholder.

| Attribute | What it does |
|---|---|
| `visibilityOnDevices` | same as `Block` |

### Image

`<Image>` is a standalone image or a `<Socials>` line. A complete source is required:

```jsx
<Image image={{ mode: "static", static: { url: "https://cdn.example.com/banner.png", fileName: "banner.png" } }} />
<Image
  url="https://shop.example/item"
  align={{ align: "center", mobile: { align: "center" } }}
  size={{ type: "fixed", width: 120, mobile: { type: "fixed", width: 80 } }}
  image={{ mode: "static", static: { url: "https://cdn.example.com/icon.png", fileName: "icon.png" } }}
/>
```

The URL in the example is illustrative. The URL source and the rules for choosing a gallery asset are defined in
`SKILL.md`; the allowed form of `image` and `fileName` is shown in the example and the table below.

The second kind of source is a personal image: the address comes from customer or order data,
and instead of `static`, `dynamic` is filled with a list of segments containing a chip. `static` need not be written
in that case; missing fields are merged onto the default:

```jsx
<Image image={{ mode: "dynamic", dynamic: [<Var param="RecipientCustomFieldString" customFieldType={{ "systemName": "<CUSTOM_FIELD_SYSTEM_NAME_FROM_LOOKUP>" }} />] }} />
```

The click is personalized the same way: `url={["https://shop.example/points/", <Var … />]}` or a single `<Var>`
as the whole value. Rules — in `SKILL.md`, section "Images" → "Personal image for the customer".

| Attribute | What it does |
|---|---|
| `image` | **Required.** File: `{ mode: "static", static: { url: "<HTTPS URL>", fileName: "<name>" } }`. Personal image: `{ mode: "dynamic", dynamic: [<Var … />] }` |
| `url` | Optional HTTPS link for clicking the image. This is not the image source; do not write a placeholder. Personalized with a list of segments |
| `size` | Size of a standalone image. Fixed: `{ type: "fixed", width: N, mobile: { type: "fixed", width: N } }`. Stored editor templates also round-trip `{ equalizedImageMaxWidth: "N" }`; preserve this field, but do not invent it |
| `align` | Alignment of a standalone image: `{ align: "left"|"center"|"right", mobile: { align: "…" } }` |
| `innerSpacing` | same as `Block` |
| `border` | same as `Block` |
| `borderRadius` | same as `Block` |
| `visibilityOnDevices` | same as `Block` |
| `alt` | Caption shown in place of an image that failed to load: a string or a list of segments with a chip. In a product row card the editor puts the product name chip here — `references/product-rows.md` |

Do not generate `background` (the backend rejects it), or the HTML attributes `src`, `href`,
`width` or `radius`. For the source use `image`, for the click — the editor attribute
`url`, for size — `size`, for alignment — `align`. For `<Socials>` lines, size and
alignment are set on the parent's `Socials.imageSize`/`Socials.align`.
Bare `<Image />` is forbidden by policy, because the backend substitutes a base64 placeholder.

### Labels and Label

A label: short text on a rounded background with an optional icon.

**Put labels only in a product row card.** The restriction lives in the editor canvas, not in the converter: it will both accept and render an ordinary row with a label, but the editor has no such label — so a successful preview here is not permission; it is exactly the case where it proves nothing.
And **`<Label>` lives only inside `<Labels>`**.

Content — text or a list of segments with a chip; a chip as a child element is rejected
(`<Label> may only contain text`).

```jsx
<Labels itemsGap={{ size: 8 }}>
  <Label background={{ type: "color", value: "#ffe8e8" }} borderRadius={{ topLeft: 12, topRight: 12, bottomLeft: 12, bottomRight: 12 }} simpleTextStyles={{ fontSize: 12 }}>Bestseller</Label>
  <Label iconDisplay={{ position: "LEFT" }} iconSrc={{ url: "https://cdn.shop.example/fire.png", fileName: "fire.png" }}>{["−", <Var param="ProductDiscount" />]}</Label>
</Labels>
```

| Attribute | Value |
|---|---|
| `Labels` | `align`, `itemsGap`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` |
| `url` | A string, as on `<Button>`: makes the label clickable |
| `contentSpacing` | Padding inside the label, around the text. Default — 8 on all sides |
| `simpleTextStyles` | Caption typography, as on `<Button>` |
| `iconSrc`, `iconAlt`, `iconSizeInPercents` | Icon, its alt and size |
| `iconDisplay` | Object `{ "position": "LEFT" \| "RIGHT" \| "EMPTY" }`. **Not a string** |
| `background`, `border`, `borderRadius` | same as other elements |
| `innerSpacing` | The label's **outer** spacing — that is what the editor calls it. Default — 8 at the bottom; inner padding is set by `contentSpacing` |
| `themeVariant` | Label variant from the shared styles: `primary` or `secondary` (§8, the `LabelStyle` tag) |
| ~~`visibilityOnDevices`~~ | The label itself does not have it — the converter answers `Unknown attribute`. Only the `<Labels>` group can be hidden by device |

**A label with an empty variable disappears entirely** — not just the chip, but the whole node with its background and icon. This is
the default; nothing needs to be written; in the editor it is the “Display conditions” tab («Отображение») → “If element variables are empty” («Если переменные в элементе пустые»).
`emptyVariableBehavior` changes it: `"hide"` (default), `"show_height_only"` — remove the content while keeping
the space, `"show"`. It works on send; in the preview the label is always visible. On `<FlexRow>` the default is the opposite,
`"show"` (§4).

**“Display filter” («Фильтр отображения»; in the editor, the “Boolean field filter” section, «Фильтр по логическому полю») — `boolFieldVisibility`.** Shows the label based on a boolean custom field, separately from an empty
variable:

```jsx
<Label boolFieldVisibility={{ "enabled": true, "boolField": { "systemName": "<FROM_LOOKUP>", "internalId": "<FROM_LOOKUP>", "fieldFor": "product" }, "mode": { "type": "FILLED", "value": true } }}>Hit</Label>
```

`mode.type` — `FILLED` with `value` (in the editor “Filled and” («Заполнен и») → yes/no), `FILLED_ANY` (“Filled and any”, «Заполнен и любой»),
`NOT_FILLED` (“Not filled”, «Не заполнен»). `fieldFor` — `recipient`, `product`, `productListItem`, `order`,
`orderItem`; `systemName` and `internalId` come from `entities_list(entityType: "CustomField")`; invented ones will not be rejected
on conversion.

**The field must be boolean, and checking that is on you.** The lookup row has `valueType` and `isMultiple` —
only `valueType: Bool` with `isMultiple: false` fits. The converter and the preview will accept a string field
silently: the condition simply will not work for the recipient, and there is no way to tell from the preview. If there is no
suitable field, tell the person, rather than substituting a similar one. The condition goes only into the sent email and works only where the field's entity
is in scope: a `product` field works inside a card; in an ordinary column the prop is saved and
does nothing.

### Timer, Video, BulletItem

Do not generate `<Timer>` and `<Video>` without a verified stored schema. `<BulletItem>text</BulletItem>`
works only inside `<BulletList>`.

### Group elements: Menu, BulletList, Socials, Split, Labels

A group is one element with direct lines. It requires at least one line; `visibilityOnDevices` is set only on the group, not on a line. `globalThemeSync` may appear in a JSON fixture, but it is not part of the DSL.

#### Menu

All lines in a menu are of one kind: only `<Text>` or only `<Button>`, no mixing.

| Menu attributes |
|---|
| `align`, `itemsGap`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` |

#### BulletList

`<BulletList>` contains only `<BulletItem>…</BulletItem>`. `align` on the list is not confirmed and is not part of the DSL.

Set `bulletIcon` **always**: the system default is serialized as a `data:` SVG, but the backend does not
escape the quotes inside `src`, the attribute breaks off, and the marker renders as a broken placeholder icon.
There are two confirmed authored forms:

```jsx
bulletIcon={{ url: "https://cdn.example.com/dot.png", fileName: "dot.png" }}
bulletIcon={{ type: "custom", url: "https://cdn.example.com/one.png", fileName: "one.png", size: 40 }}
```

The flat form renders a small 4px-wide marker and suits a dot/circle.
`type: "custom"` + `size` round-trips from the editor, and live preview/HTML renders
the real `size` width; use it for a separate editable numbered badge.
`{{ mode: "static", static: {...} }}` is silently ignored, and a string crashes the preview with
`Internal server error`. All URLs must be confirmed HTTPS assets.

One group applies a shared `bulletIcon` to all lines. For different icons 1/2/3
use a separate BulletList per item, or a standalone fixed-size Image + Text. Do not
draw a structural badge with a styled span inside Text or with Html if it must be
editable in the editor. A custom marker has one `size` for desktop/mobile,
so check mobile visually; if you need a separate mobile size, use
a standalone Image with `size.mobile.width`.

| BulletList attributes |
|---|
| `bulletIcon` (always set; flat 4px dot or custom `size` badge), `background`, `border`, `borderRadius`, `innerSpacing`, `itemsGap`, `iconTextGap`, `iconTopPadding`, `visibilityOnDevices` |

#### Socials

`<Socials>` contains only `<Image image={{...}} />` lines. The URLs below are illustrative;
the source rules are the same as for an ordinary Image.

| Socials attributes |
|---|
| `background`, `align`, `innerSpacing`, `itemsGap`, `imageSize`, `border`, `borderRadius`, `visibilityOnDevices` |

#### Split

`<Split>` contains only `<Column size={N}>`; the sum of all `size` values is strictly 12. A Split column carries only `size`, contains 0..1 element and may be empty. Simple elements and groups are allowed in it, except `<Split>`.

| Split attributes |
|---|
| `columnsGap`, `verticalAlign`, `innerSpacing`, `background`, `border`, `borderRadius`, `visibilityOnDevices` |

Each group has its own default `itemsGap` — the values are in §6, "Defaults that appear on their own"; specify your own only when you need a different gap.

## 7. Text — content and inline markup

`<Text>` is two places at once: what is written (with markup) between the tags, and how it looks as a whole (font, size, color, links) — in the `style` attribute.

### Plain text — auto-wrapped in `<p>`

Text without its own markup is written as words:

```jsx
<Text>Summer sale starts today</Text>
```

The converter wraps such words into a paragraph by itself: the result becomes `<p style="margin: 0">Summer sale starts today</p>`. Such a paragraph comes back in the same form on read. Auto-wrapping triggers only if there is no block tag among the top-level children: if the markup already contains a block element, the converter wraps nothing.

### Full HTML markup

Markup is written as markup, exactly as the email stores it. **Do not rewrite what you did not come to change**: the email wraps text in its own way, and replacing that wrapper edits the email invisibly.

```jsx
<Text>
  <p style="margin: 0;"><span data-rich-text>Inspiring <u>stories</u> of people, </span><a data-rich-link href="https://example.com/stories">read them</a></p>
</Text>
```

**Tags you can use:** paragraphs and headings (`p`, `h1`–`h6`, `blockquote`), lists (`ul`, `ol`, `li`), inline marks (`strong`, `em`, `u`, `s`, `span`, `a`, `code`, `pre`), breaks (`br`, `hr`) and `personalization-parameter`. The live backend rejects `div` and tables (`table`, `thead`, `tbody`, `tr`, `td`, `th`) in `<Text>` (`<div> is not allowed in a text`) — move table and div layout into `<Html>` as a quoted string (§6). Write explicit JSX void tags self-closing: `<br/>`, `<hr/>`. Do not generate `img`, `iframe`, `script`, `style` without a separate live check. An image goes into a standalone `<Image>` or inside `<Socials>`.

**Attributes you can use:** `style`, `align`, `href`, `target`, `rel`, `model`, `data-name` and the `data-rich-*` markers the editor sets. Attributes inside rich-text markup are only a plain string or a marker without a value: correct — `<p style="margin: 0;">`, `<a href="https://...">`, `<span data-rich-text>`. Not allowed: `<p style={{ margin: 0 }}>`, spread attributes, expressions — the live backend returns `Attribute \`...\` on <...> must be a plain string`.

### Quoted string in `<Text>` — literal text

A quoted string in `<Text>` is **literal text, not markup**. The live backend escapes it: `<Text>{"<p>One<br>Next</p>"}</Text>` is serialized as `<p style="margin: 0"><span data-rich-text>&lt;p&gt;One&lt;br&gt;Next&lt;/p&gt;</span></p>` — the tags become visible text in the email. So write markup only with explicit JSX tags from the allowlist above, and move raw HTML (tables, an unclosed `<br>`, comments, entities) into `<Html>{"…"}</Html>`. Do not mix bare text and a quoted string in one element — live rejects it (`<Text> may only contain text`). A quoted string is still needed for literal text with special characters (`"`, `{`, `}`, a line break) and for existing personalization (§7).

### `data-rich-*` markers

The markers `data-rich-text` (on `<span>`), `data-rich-link` (on `<a>`), `data-rich-list-item` (on `<li>`) are placed by the converter — **do not write them when creating**. When reading existing text — **leave them in place, do not move them**: a missing marker is restored, but a moved marker is not removed and breaks the styling (for example, `data-rich-text` on `<a>` styles the link both as text and as a link). A marker without a value is stored as `data-rich-text=""`.

### Personalization

New personalization is written with the `<Var>` tag — rules and the parameter list are in
`references/personalization.md`:

```jsx
<Text><p style="margin: 0;"><Var param="RecipientGreeting" /></p></Text>
```

Do not write Quokka expressions in text: a parameter written as a string stays literal text.
Preserve existing fragments in that form unchanged unless the user asked to
redo them:

```jsx
<Text>{"Hello, ${Customer.FirstName}!"}</Text>
```

A chip the converter did not recognize (the parameter is not in this campaign's registry, the model cannot be
decoded) comes in its stored form — leave it as is:

```jsx
<Text><p style="margin: 0;">Hello, <personalization-parameter model="bm90LWEtbW9kZWw="></personalization-parameter>!</p></Text>
```

#### Unsubscribe link

The unsubscribe link is included in a marketing email by default — the rules for when it is not added are in `SKILL.md`
§"Unsubscribe link".

A chip in an html attribute of the markup is not assembled: the converter escapes `<a href="<personalization-parameter …>">`,
and text, not a parameter, goes into the email. That is why the href of a text unsubscribe link remains
an exception to the ban on Quokka expressions: the exact `${Message.UnsubscribeLink}` as the plain-string value
of `href` on `<a>` inside `<Text>`.

In prop values the chip works, and unsubscribe has its own parameter — `SpecialLinkUnsubscribeLink`; its
expression is exactly `Message.UnsubscribeLink`. This is how an unsubscribe button or clickable image is made:

```jsx
<Button url={[<Var param="SpecialLinkUnsubscribeLink" />]}>Unsubscribe</Button>
```

Correct:

```jsx
<Text>
  <p style="margin: 0;">If you no longer want these emails — <a href="${Message.UnsubscribeLink}">unsubscribe</a></p>
</Text>
```

Wrong:

```jsx
<Button url="${Message.UnsubscribeLink}">Unsubscribe</Button>
```

```jsx
<Text><p style="margin: 0;">Hello, ${Customer.FirstName}</p></Text>
```

Correct — with the tag: `<Text><p style="margin: 0;"><Var param="RecipientGreeting" affixVariableText={{ "aroundText": { "before": "Hello, ", "after": "!" }, "emptyBehavior": { "mode": "fallback", "fallbackText": "Hello!" } }} /></p></Text>` (the chip carries the whole phrase — no words are written around it, and the words inside it are set explicitly)

```jsx
<Text><p style="margin: 0;"><a href="/unsubscribe">Unsubscribe</a></p></Text>
```

The preview HTML will not contain the token: the preview renders the editor canvas, and in it
all links inside `<Text>` have `href="#"`. This is not a defect and not a reason to replace the token with
a fake URL — the canonical `${Message.UnsubscribeLink}` lives in the JSX; the substitution happens at
send time. Preserve an existing canonical token, its caption and placement unchanged
unless the user asked to change the link.

### `style` — typography of the whole text

The `style` attribute holds what the editor's bottom panel edits — it applies to the whole text, not to a fragment:

```jsx
<Text style={{ fontSize: 16, color: "#111111", link: { color: "#0000E7", inscription: ["underlined"] }, mobile: { fontSize: 16, align: "left" } }}>
  Styled from end to end
</Text>
```

The same rule as everywhere: write the fields you change; the rest stay. If you change
`fontSize` or `align`, set `style.mobile`: without it the backend may substitute
`{ fontSize: 18, align: "left" }` and break the mobile hierarchy.

Note the two kinds of "bold". `<strong>` inside the markup makes one word bold; `inscription: ["bold"]` in `style` makes the whole text bold. They are stored in different places and do not replace each other.

`inscription` values — `bold`, `italic`, `underlined`, `crossed`; strikethrough is `crossed`.

## 8. The email's shared styles

An email has its own set of styles — what a heading looks like, what the primary
button looks like, what background a block has. This is the “Design” («Дизайн») tab in the editor. Do not confuse it with the subject: the subject
is edited via `campaign_edit_content` and has nothing to do with markup.

### A node either follows the shared styles or has its own look

This is not declared anywhere; it is **inferred** from what the node wrote:

```jsx
<Text themeVariant="h1">Summer sale</Text>
<Button url="https://shop.example">Buy</Button>
```

Neither said anything about its look — both follow the email's shared styles. Name a value
that the shared styles set, and the node is detached from them.

Only what is named from the set counts. Text alignment lives in the same `style` prop as
typography, but the shared styles do not set alignment — so the heading below stays under them
and is centered at the same time:

```jsx
<Text themeVariant="h1" style={{ align: "center" }}>Summer sale</Text>
```

The shared styles cover only this, and only on these nodes:

| node | what the shared styles cover |
|---|---|
| `Block`, `FlexRow` | `background`, `innerSpacing` |
| `Text`, `BulletItem` | inside `style`: font, size, spacing, color, link styling — not the words and not `align` |
| `Button` | `background`, `border`, `borderRadius`, `buttonSize`, `simpleTextStyles` |

The same name is covered on one node and not on another: a `<Text>` with its own `background`
still takes its typography from the shared styles, because the text background is not part of the set.

**Nothing said about the look — write no styles.** You are building an email from scratch, the look of this element is not mentioned in the
description, the examples or the mockup, and there is no similar element in the email yet — leave it without style
attributes: the look will come from the email's shared styles and will keep changing with them. An invented
`fontSize` instead detaches the node from the styles and freezes the rest of the set. If a similar element already
exists — take the look from it; if there is a mockup or reference — write the styles, the look is stated there. A reference to a neighboring
element ("like the heading above") does not count as a specified look: it is the same case; the look is taken from that
node, and if it follows the shared styles, the new one also stays empty. Heading levels
are set by `themeVariant`, not by different numbers.

### Half a set is filled in from the shared styles

An unwritten prop reads as "take it from the shared styles", so a node that wrote three of
five takes the other two from the styles it is leaving — as they were at the moment the document
was read. From then on it keeps them: the node is already detached, and edits to the styles no longer reach it.

The read itself depends on the styles: the same document, read before and after they are edited,
yields nodes with a different look. So the choice is binary: either name nothing from the set — the node
follows the email's styles and their edits — or name the whole set, with values from
`visual_template_theme_get` (Ops). The converter will accept part of a set, but the result is a look that
nobody wrote: what you named is yours, the rest is frozen at the styles of that moment.

A short form like `<Text style={{ fontSize: 20 }}>` is legitimate and common: you said
"this text is larger", and the node carried the font and color over from the styles. Just know that from then on they will not follow
them, and do not write it this way where the node must change together with the email.

### `themeVariant`

Says which variant the node is assigned to: `h1`, `h2`, `h3`, `text` for text, `primary` or
`secondary` for a button. Blocks and rows have a single variant, and it is not written. Only
a variant different from the default is written.

**The names are not interchangeable.** On a node the attribute is called `themeVariant`; inside `<Theme>` —
`variant`. The converter rejects both mixed-up names, but differently: `variant` on a node — as
`Unknown attribute`, and `themeVariant` inside `<Theme>` — with a message that `variant` must be named
(«come in 4 variants, so one has to be named»). This does not mean there is no way to assign a node to a variant:
it means the wrong name was used.

```jsx
<Theme><TextStyle variant="h1" simpleTextStyles={{ fontSize: 32 }} /></Theme>
<Text themeVariant="h1">Summer sale</Text>
```

Having a variant says nothing about whether the node follows the shared styles: these are read from
different places.

### Editing the shared styles themselves — `<Theme>`

Written once, directly inside `<Template>`, before the blocks, one tag per style kind:

```jsx
<Template>
  <Theme>
    <TextStyle variant="h1" simpleTextStyles={{ fontSize: 32, inscription: ["bold"] }} />
    <ButtonStyle variant="primary" background={{ type: "color", value: "#ff6600" }} />
  </Theme>
  <Block><FlexRow><Column size={12}><Text>Body</Text></Column></FlexRow></Block>
</Template>
```

It reads as a patch: what is named replaces the email's value; what is not named stays as it was.
A complete set is **not required** here — that rule is about nodes, not about styles.

| tag | variants | props |
|---|---|---|
| `BlockStyle`, `RowStyle` | one, `variant` is not written | `background`, `innerSpacing` |
| `TextStyle` | `h1`, `h2`, `h3`, `text` | `simpleTextStyles` |
| `ButtonStyle` | `primary`, `secondary` | `background`, `border`, `borderRadius`, `buttonSize`, `simpleTextStyles` |
| `LabelStyle` | `primary`, `secondary` | same as `ButtonStyle`, but `contentSpacing` instead of `buttonSize` |

The typography inside `<TextStyle>` is called `simpleTextStyles`, while on `<Text>` itself the same
set is called `style`. The name in the style set and the name on the node do not always match — check against
this table and the table above, not from memory.

**Inside a style, write only fields that are present in the fragment from `visual_template_theme_get`** —
it is the complete list of what the styles set. Fields outside it cannot come from the shared styles:
the converter rejects such an entry rather than silently accepting it. Most often this is an attempt to set
alignment — it is not in the set; it lives on the node:

```jsx
<Theme><TextStyle variant="h2" simpleTextStyles={{ fontSize: 14, color: "#3C4043" }} /></Theme>
<Text themeVariant="h2" style={{ align: "right" }}>Gmail</Text>
```

`LabelStyle` — the shared styles for `<Label>` labels; it arrives in the fragment from `visual_template_theme_get`.
Insert it together with the rest.

Editing a shared style changes **all** nodes that follow it, including those the document did not
touch. So change the shared styles only on an explicit request — "make the buttons orange throughout
the email", not "make this button orange".

Reading an email does not return `<Theme>`: the document carries it only when it changes something.
Ask `visual_template_theme_get` for the current values — it returns a ready fragment
in exactly this form. Pass it the current document: otherwise the response will not account for a not-yet-saved
`<Theme>`, and replacing the tag with such a response will wipe out the edits.

### Moving styles from nodes into `<Theme>`

A request to "move the styles into the shared ones" is three actions in one document:

1. write the values into `<Theme>`;
2. **remove** these styles from the nodes that move under them;
3. add `themeVariant` where the variant is not the default.

Skipping the second is the most common mistake, and it is invisible: the look does not shift, the preview shows nothing, and
the shared styles end up applied to nothing. The person finds out at their first edit of them.

**The look after the move matches the previous one pixel for pixel.** Hence:

- **the number of variants is finite** — text `h1`/`h2`/`h3`/`text`, button and label
  `primary`/`secondary`, block and row one each. Different values are not merged into one variant;
- **values are not adjusted**: 12px stays 12px, no rounding, no "almost the same";
- **a different mobile size is a different look**, even if the desktop one matches;
- **more looks than variants** — state it as a number and leave the extra ones unique: "I found six
  text sizes, there are four variants; I'll move the four most common, and leave two unique — they are in
  the footer". Pulling a look toward the nearest variant is not allowed;
- **alignment stays on the node** — it is not part of the set and is not removed from the node during the move.

**The mobile side moves together with the rest.** The shared style set holds it, in two forms:
nested (`mobile: { height: 48 }`) and as a flat sibling field (`widthMobile`). Both are written inside
`<ButtonStyle>` / `<TextStyle>` on par with the desktop value. So "the shared styles do not store
a separate mobile button height" is not a limitation but an unwritten field: look in the
`visual_template_theme_get` fragment at what the set defines, and move everything the node had.

**But mobile values live in the variant itself, not on the node:** all buttons on `primary` have the same mobile
height, from the styles. So a node that needs on mobile something other than what is in the
variant cannot keep it under the shared styles — for a button, label, block and row the prop comes from
the styles as a whole, including its mobile part. Text is different: of the mobile values, the styles set only the font
size; the rest of the mobile values stay on the node.

Such a node is not moved silently: name the element and the difference, and ask whether it is fine for it to follow the shared
styles on mobile. Yes — it moves together with the rest, the difference is lost; no — it stays
unique. Rewriting a variant for the sake of one node is not allowed: all nodes of the variant would get the new values.

**A look that drifted after the move is always an incomplete set, not a platform limit.** You must not explain
the shift that way: either add the missing side, or leave this node unique and say that it did not
move and why.

This is verified by values, not by the picture: for each node you removed styles from, the variant's set
matches what was removed, field by field. Before the move the values are written in the document; after it — in the
`visual_template_theme_get` fragment. The preview is the second step: a one-pixel shift or an extra line break is not always
visible in a snapshot.

### An email without its own shared styles

Every email has effective styles; not every email has its own. One that has not configured them takes the
platform default styles and stays that way: reading and saving do not turn them into its own,
frozen at today's values. The tool says this directly — do not present
platform default values to the user as the styling of their email.

## 9. URL

`Button.url` accepts `https://`, `tel:` and `mailto:`; the backend validates the format and rejects
survey links. Write a simple link as a string `url="https://shop.example"`; a URL with
special characters (`&`, `=`, quotes) — as a JSON string in an expression:
`url={"https://shop.example/?from=email&utm=sale"}`.

## 10. What the serializer adds on its own

All service JSON fields (UUID, envelope, subject, targeting, sampling, materialization, etc.) the serializer adds automatically. All you need is what is listed in §4–§6: tags, hierarchy, column `size` and element attributes.

The email settings — width, background, “Global (CSS) styles” («Глобальные (CSS) стили»), the Gmail annotation — and the email's shared styles **are carried over from the previous version of the email** on save. Those the document does not name stay as they were; width and background can be named with root attributes (§4), the shared styles — with the `<Theme>` tag (§8). The other two settings are not available from markup and are changed only in the editor.

## 11. Runtime validation and self-check

The backend checks the structure, the tag/attribute allowlist, literal-only values, the grid,
group composition, the Button URL and the allowed content of leaf elements. Report an error as
stage + status/code + a short message; the full response body is needed only for separate diagnostics.

The Generator itself additionally checks policy that the backend does not guarantee:

- a static Image URL is confirmed by the user or Ops and uses HTTPS; a personal image has no invented domain prefix;
- no bare Image and no fake unsubscribe; every tenant-specific `systemName` in `<Var>` came from a lookup or from the user;
- colors are written as `#RRGGBB`, numeric values are reasonable;
- if a desktop `style.fontSize`/`style.align` or `simpleTextStyles.fontSize` is set, the mobile branch is set deliberately;
- device variants and mobile ordering match the request;
- the JSX contains no comments, imports or executable expressions.

## 12. Product rows — `<CollectionRow>`

A block has a second kind of row: it holds product cards rather than columns, and renders the card once per
product from the mechanic — recommendations, order items, cart, the project's product list.

```
Template  →  Block  →  CollectionRow  →  CardTemplate | CollectionCard  →  Column  →  element
```

**The full rules are in [product-rows.md](./product-rows.md), and it must be opened before generating an email with
products.** The row cannot be built from memory: which chips are allowed is decided by the mechanic; the mechanic has
required settings, and a successful preview/save does not prove they are filled in; the grid requires an orientation;
manually selected products are named by identifiers, which are never invented.

## 13. A block rendered by an editor template — `<QuokkaBlock>`

Not every block is laid out in the document. The editor has block templates whose layout lives in the
template itself, while the email holds only a reference to it and the values the author changed. Such a block comes as
the `<QuokkaBlock>` tag and sits directly in `<Template>`, next to ordinary `<Block>` elements:

```jsx
<Template>
  <Block><FlexRow><Column size={12}><Text>Header</Text></Column></FlexRow></Block>
  <QuokkaBlock templateId="0f8fad5b-d9cb-469f-a165-70867728950e" accent="#FF0000">
    <Collection name="items" dataSource={{ "key": "ORDER" }} count={3} />
  </QuokkaBlock>
</Template>
```

**Such a block is carried over, not edited.** Having read the email, return the block in the same form, with all its
attributes and collections, as it came. If asked to change the text, image, number of cards or
product source inside it — say that the content of such blocks is edited in the editor, not through the email.
The other blocks of the email are edited as usual meanwhile, and the block can be deleted as a whole or moved between
blocks.

**`templateId` is never invented.** It is the template's identifier, not its name, and there is nowhere to get it
except from the email that was read. A new `<QuokkaBlock>` cannot be built from scratch: you do not have the template it would
refer to. An identifier that does not exist in the project is rejected with that identifier listed —
by the editor or by the converter: the text differs, the cause is the same.

**Only `<Collection>` fits between the tags, and inside it only `<Item>`** — the layout lives in the template, not
in the email.

The rules of `product-rows.md` do not apply to this collection: it brings its mechanic and settings from the email,
and they need not be completed, even if it is built on a product mechanic.

**The block's other attributes are its template's settings**, not the general DSL set. Which ones a block has is visible from
the block that was read: what it printed is what it accepts; a name the template does not know is rejected. Values
equal to the template's are not printed at all — their absence does not mean the setting is empty.

**A collection always states how many positions it has**, in one of three ways:

- `count={N}` with no children — that many positions, as the template renders them;
- `count={N}` and exactly one `<Item>` — that many positions, all from this card;
- a list of `<Item>` without `count` — one position per element, each its own.

`count={0}` is a collection the author emptied; a missing `<Collection>` means "as in the template".
Do not rewrite one form into another: they are different emails.

**A manual product selection** (`dataSource={{ "key": "ProductShowcase" }}`) holds one `<Item>` per product and
has no `count` — the products are the count. A product is named by the triple `internalId`, `externalId`,
`externalSystemName` from one row of the product lookup results, as in `<CollectionCard>` (see
[product-rows.md](./product-rows.md)). For a position of any other mechanic, `product` is rejected: the mechanic
finds the products itself.

**Layout attributes** (`columns`, `isAdaptive`, `columnsSpacing`, `rowsSpacing`, `columnsDivider`,
`rowsDivider`, `containerWidth`) exist only on collections whose templates handed the grid over to the editor. The converter
prints them; preserve them as is and do not add them — a template that renders the grid itself rejects such attributes.

**What the DSL does not write at all:** positions selected by topics (except an empty collection), and a mechanic
that the collection's template does not provide. The converter rejects an email with such a block — this is not a reason to fix it
by hand; pass the rejection on verbatim.
