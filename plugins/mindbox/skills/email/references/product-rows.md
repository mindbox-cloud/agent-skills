# Product rows — `<CollectionRow>`

**When to read:** the email contains products: recommendations, a product list, order products, viewed products, products from an event, or a manual collection. Do not generate `<CollectionRow>` before reading this file.

**Return:** to Workflow / Self-check in `SKILL.md`; check the mechanic, all required settings, lookup values, the placement of product `<Var>` and that the family matches.

A product row is the second kind of block row: instead of columns it holds product cards. Everything about
it is here — how to build it, where it gets its products, which chips are allowed in its card and what an
email with it breaks on. An ordinary email does not need this file; it is opened when the email contains
products; the rest of the DSL surface is in [dsl-surface.md](./dsl-surface.md).

```
Template  →  Block  →  CollectionRow  →  CardTemplate | CollectionCard  →  Column  →  element
```

## The card the editor provides

A product row is added to the canvas as a ready-made preset: image, name, price next to the crossed-out
old price, a button — and all links already lead to the product. The most common preset is three cards in
a row, image above the words; here it is in JSX:

```jsx
<Template>
  <Block>
    <CollectionRow dataSource={{ "key": "ProductShowcase" }} orientation="vertical" columnCount={{ "value": 3 }} rowCount={{ "value": 1 }}>
      <CardTemplate url={[<Var param="ProductUrl" />]}>
        <Column size={12}>
          <Image image={{ mode: "dynamic", dynamic: [<Var param="ProductPictureUrl" />] }} url={[<Var param="ProductUrl" />]} alt={[<Var param="ProductName" formatString={{ "isTruncateEnabled": true, "truncateValue": 40 }} />]} height={{ "height": 180 }} />
          <Text themeVariant="h3"><p style="margin: 0"><Var param="ProductName" formatString={{ "isTruncateEnabled": true, "truncateValue": 40 }} /></p></Text>
          <Split columnsGap={{ "size": 0 }} innerSpacing={{ "top": 10, "bottom": 10 }} height={{ "height": 40 }}>
            <Column size={6}><Text innerSpacing={{ "top": 0, "bottom": 0 }}><p style="margin: 0"><Var param="ProductPrice" /></p></Text></Column>
            <Column size={6}><Text innerSpacing={{ "top": 0, "bottom": 0 }} style={{ "fontSize": 16, "inscription": ["crossed"], "color": "#666666" }}><p style="margin: 0"><Var param="ProductOldPrice" /></p></Text></Column>
          </Split>
          <Button url={[<Var param="ProductUrl" />]} themeVariant="secondary" align={{ "align": "left" }} innerSpacing={{ "top": 12, "right": 16, "bottom": 12 }}>Buy</Button>
        </Column>
      </CardTemplate>
    </CollectionRow>
  </Block>
</Template>
```

This is a copy-safe example: no products are selected yet. Its mechanic is manual selection; `ProductShowcase`
without `<CollectionCard>` is legal. If you know the user's mechanic, set it, and after generation say that
the products need to be selected in the editor or named to you for a lookup.

**This is the base example, not the only correct card.** No mockup and no special requests — build this one.
The user brought a mockup or described the card in words — lay it out accordingly, but build the links, chips
and the connection to the styles the same way as here. Trimming the card on your own initiative (dropping
the old price, the button) is not allowed: that is the user's or the mockup's decision.

**The editor offers three layouts,** and all three are a legal card: three in a row (the example above), two
in a row (`columnCount` 2, image height 260) and a list — `orientation="horizontal"`, a card of two
`<Column size={6}>`: image on the left, name with price and button on the right. Which one to take is said
by the mockup or the user; if nobody said, take the example above.

**All three card links are required** — `CardTemplate.url` (a click on the whole card), `Image.url`,
`Button.url`, and each is the same product chip. Without it the card passes preview, but in the sent email
the click leads nowhere. The SKILL.md rule "`Image.url` — only from an explicit user URL" does not apply
to the card: the address comes from the mechanic, not from a person, and it is never a placeholder.

**The card writes no styles, except for the crossed-out price.** The look of the name and the button comes
from the email's shared styles via `themeVariant`. Own styling is not forbidden, but a node that names a
field those styles set (font, size, color, weight for text; background, border, radius, typography for a
button) silently leaves them; `align` and `innerSpacing` are not set by the styles — those may be written.
The self-check requirement "the CTA has a brand `background`" does not apply to the card button.

**No products selected yet — do not write cards at all.** `<CollectionCard>` means "this exact product stands
here", and without `product` it is not an empty slot but a refusal `A <CollectionCard> names the product it stands for`.
Placeholders are drawn by `<CardTemplate>` itself, and how many is decided by the mechanic. A row **without
`dataSource`** gives a `columnCount × rowCount` grid of empty cards. `ProductShowcase` gives one: it is the
"I'll fill it by hand" mechanic, the number of cards in it is set by the selected products, and the grid only
lays them out. So while the source is not chosen, do not invent a mechanic: a row without one is exactly the
"added, not configured" state.

`formatString` on the name is not for looks: without it the preview pads the value with repeats up to 150
characters and the card falls apart. For products from a mechanic set a limit, for manually selected ones
`isTruncateEnabled: false`, so that the preview shows the real name. Details — `references/personalization.md`,
“Long text: `formatString`”.

## One card renders every product

`<CardTemplate>` is the card that renders every product; `columnCount` × `rowCount` say how many of them to
show, but **only on a vertical row**. By default the row is horizontal and draws one card per line:
`columnCount` is not read there at all — the converter silently accepts it, and the row still draws one card
per line. A grid is
`orientation="vertical"` together with the column count, as in the example above.

Its `<Column>` is the inner structure of the card, and it follows the orientation: a vertical card has one
`<Column size={12}>` — image above the words, a horizontal one has two of `size={6}` — image next to the
words. `size` sums to 12, as with row columns. **A row holds one `<CardTemplate>`**; a second one is the error
`A <CollectionRow> may hold only one <CardTemplate>`.

Product values are **chips**. A product chip is authored only inside such a row; do not put it in an ordinary
column, build a product row. `<Var>` forms are in `references/personalization.md`.

**The product image is personal, not a file.** The address comes as a chip, like a personal image from
customer data: `image={{ mode: "dynamic", dynamic: [<Var param="ProductPictureUrl" />] }}`. A static file in
the card would mean the same image for all products. The link and `alt` are chips too; the card's
composition is above. In a row that iterates over a product list, all card chips are taken from the
`ProductListItem*` family — see the table below.

**The chip family is set by the mechanic, not by the card.** The mechanic determines what exactly the row
iterates over, and the chips must be from the same family:

| Mechanic | Chip family |
|---|---|
| `RECIPIENT_RECOMMENDATIONS`, `PRODUCT_RECOMMENDATIONS`, `FROM_SEGMENT`, `ORDER_PRODUCTS`, `ViewedProducts`, `ViewedProductsInSession`, `SessionProductCategoryViews`, `SessionGetAddedToListProducts`, `RecentlyBoughtProducts`, `CategoryProductsComputedField`, `FromCustomerComputedField`, `ProductShowcase` | `Product*` |
| `FROM_PRODUCT_LIST` | `ProductListItem*` |
| `ProductListItem` | `SingleProductListItem*` |
| `ORDER` | `OrderItem*` |
| `ProductView` | `ProductView*` |

**Names by product value.** The same value is named differently in different families, and you cannot just
glue a prefix onto a name: the price of a product in an order is `OrderItemAmount`, not
`OrderItemProductPrice`. Here is the full table of common values:

| What to output | `Product*` | `ProductListItem*` | `OrderItem*` |
|---|---|---|---|
| name | `ProductName` | `ProductListItemProductName` | `OrderItemProductName` |
| image | `ProductPictureUrl` | `ProductListItemProductPictureUrl` | `OrderItemProductPictureUrl` |
| link | `ProductUrl` | `ProductListItemProductUrl` | `OrderItemProductUrl` |
| price | `ProductPrice` | `ProductListItemProductPrice` | `OrderItemAmount` |
| price before discount | `ProductOldPrice` | `ProductListItemProductOldPrice` | `OrderItemBaseAmount` |
| description | `ProductDescription` | `ProductListItemProductDescription` | `OrderItemProductDescription` |
| discount | `ProductDiscount` | `ProductListItemProductDiscount` | `OrderItemDiscount` |
| vendor code (SKU) | `ProductVendorCode` | `ProductListItemProductVendorCode` | `OrderItemProductVendorCode` |
| manufacturer | `ProductVendorName` | `ProductListItemProductVendorName` | `OrderItemProductVendorName` |

For `SingleProductListItem*` and `ProductView*` names are built the same way as for `ProductListItem*` — with
their own prefix instead of `ProductListItem`. A name that does not exist will not be accepted by the
converter: `Unknown personalization parameter` — so guessing is pointless, and the full list of product
parameters by entity is in `references/product-parameters.md`.

The family is chosen by the entity the mechanic iterates over. The converter rejects a chip from another family and suggests the counterpart parameter from the right family. A product chip outside `<CollectionRow>` is a different case: do not put it in an ordinary column, build a product row.

## Where the products come from

`dataSource` names the mechanic in `key`, and its settings sit under `model.settings.model`:

```jsx
<CollectionRow dataSource={{ "key": "RECIPIENT_RECOMMENDATIONS", "model": { "settings": { "model": { "recoMechanicSystemName": "<SYSTEM_NAME_FROM_LOOKUP>" } } } }}>
```

Do not copy this fragment before the lookup. The value `<SYSTEM_NAME_FROM_LOOKUP>` must be replaced with the exact `systemName` returned by `entities_list(entityType: "RecommendationMechanic")`; preview/save do not check an invented name.

- **Write only what the mechanic declared.** A mechanic on its defaults is written with a single `key`.
- **Mechanic names and their settings are not guessed.** A mechanic that cannot fill a product row is
  rejected by the converter, and the message lists the ones that exist. Content and topic showcases are among
  the rejected: they iterate over the root object, not over products.
- **A row may have no mechanic at all** — that is the state right after adding it, before a source is chosen.
  Then `dataSource` is simply not written.
- **Do not write a mechanic without its required setting at all.** The setting is required by the package
  policy. The current preview rejects an incomplete `RECIPIENT_RECOMMENDATIONS` without `recoMechanicSystemName`
  (`RECIPIENT_RECOMMENDATIONS needs a value in its settings`, hint `recoMechanicSystemName`); save behavior on
  an incomplete mechanic is not separately confirmed, so do not count a successful save as a completeness check.
  The value is found by lookup — the product list, products and the recommendation mechanic are in
  `entities_list` — and what is not there is asked of the user. There is no such thing as an empty setting or a placeholder.

## Mechanic settings

Required ones are marked **★** — without them the row does not work. The rest need not be written unless you
change them: as everywhere in the DSL, what is not declared keeps its value.

| Mechanic | Its settings |
|---|---|
| `RECIPIENT_RECOMMENDATIONS`, `PRODUCT_RECOMMENDATIONS` | **★ `recoMechanicSystemName`** — system name of the recommendation mechanic (see below for where to get it) |
| `FROM_PRODUCT_LIST` | **★ `productListSystemName`** and **★ `productListInternalId`** — both; these are not two ways of naming one thing (see below for why); `availableForRecipient`, `enabledFilterBySegment`, `filterBySegmentDetails`, `singlePerGroup` |
| `FROM_SEGMENT` | **★ `segmentSystemName`** and **★ `segmentationInternalId`** — also both; `random`, `availableForRecipient`, `singlePerGroup`, `filterByRecipientComputedFieldSettings` |
| `SessionGetAddedToListProducts` | **★ `productListSystemName`** and **★ `productListInternalId`** — as for `FROM_PRODUCT_LIST`; `enabledFilterBySegment`, `filterBySegmentDetails`, `availableForRecipient`, `singlePerGroup` |
| `ORDER_PRODUCTS`, `ViewedProducts`, `ViewedProductsInSession`, `SessionProductCategoryViews`, `RecentlyBoughtProducts` | none required; `enabledFilterBySegment`, `filterBySegmentDetails`, `availableForRecipient`, `singlePerGroup` |
| `CategoryProductsComputedField` | **★ `recoMechanicSystemName`** and **★ `computedField`** — an object `{ "systemName": …, "internalId": … }`, both fields |
| `FromCustomerComputedField` | **★ `systemName`** and **★ `internalId`** — here the setting is the computed field object itself, i.e. `"settings": { "model": { "systemName": …, "internalId": … } }`; without the system name the expression stays `Recipient.CustomField` and the row fills nothing |
| `ORDER`, `ProductShowcase`, `ProductListItem`, `ProductView`, `Empty` | no settings — a single `key` is written |

### Where the recommendation mechanic comes from

The system name comes from `entities_list`, `entityType: RecommendationMechanic`. A result row has `systemName`,
`name` and three suitability fields, because **not every mechanic will do**:

| column | what to do with it |
|---|---|
| `target` | must match what the row's mechanic accepts: `Customer` for `RECIPIENT_RECOMMENDATIONS`, `Offer` for `PRODUCT_RECOMMENDATIONS`, `Category` for `CategoryProductsComputedField` |
| `readyToUse` | only `true` is taken |
| `status` | only `RecalculationRequested` or `RecalculationFinished` are taken — two statuses out of eight |

The lookup returns the suitability fields, and the agent must apply them before generation: preview/save do not
prove that the mechanic fits the chosen row or is ready for calculation. So the choice is made by the three
columns, not by the first matching row. If no suitable one is found, tell the user rather than substituting an
unsuitable one: there is nothing to build a recommendations row with, and it is better to suggest another
filling mechanic.

Two pairs are easy to confuse, and mixing them up means building a row with the wrong iteration entity:

- **`ORDER` vs `ORDER_PRODUCTS`.** The first iterates over order lines and wants `OrderItem*` chips; the second
  iterates over the products of those lines and wants `Product*`. If you need the name and price of a product
  from an order, that is `ORDER_PRODUCTS`; if the amount and quantity per order line, that is `ORDER`.
- **`ProductListItem` vs `FROM_PRODUCT_LIST`.** The first is one list line, and it draws one card
  (`columnCount` and `rowCount` are not needed); the second iterates over the whole list.

`filterBySegmentDetails` is an object `{ "systemName": …, "internalId": … }`, and it is needed only when
`enabledFilterBySegment` is on.

**The system name and internalId are not two ways of saying one thing, but two different readers.** The system
name goes into the sent email (`Recipient.GetProductList("...")`) — without it the campaign cannot be
built. `internalId` does not get into the email, and does two other things: by it the editor shows which list
is selected (the select looks it up by `internalId` and looks at nothing else), and by it the email's dependency
on the list is declared — what keeps the platform from deleting a list in use.
Write only the system name and the list expression will be built, but the user, opening the row in the
editor, will see an **empty select**, as if no list were selected, and the platform will consider the list
unused. The same goes for a segment (`segmentSystemName` + `segmentationInternalId`) and for a computed field.
The canvas writes `productListId` as a third field but never reads it — do not invent it yourself, and if you
see it in an email you are editing, leave it as is.

**Where to get the values.** A product list — `entities_list`, entity `ProductList`: **`systemName` and
`internalId` are taken from the same result row**, like the product identifier triple. Recommendation mechanics —
`entities_list`, entity `RecommendationMechanic`; the lookup returns `systemName`, `name`, `target`,
`readyToUse` and `status`, and the agent applies the suitability rules above. This file does not describe a
lookup for segments: a segment is taken from the user or from the email you are editing. An example of a mechanic with a list:

```jsx
<CollectionRow dataSource={{ "key": "FROM_PRODUCT_LIST", "model": { "settings": { "model": { "productListSystemName": "<PRODUCT_LIST_SYSTEM_NAME_FROM_LOOKUP>", "productListInternalId": "<PRODUCT_LIST_INTERNAL_ID_FROM_LOOKUP>" } } } }}>
```

Do not copy this fragment before the lookup. The values `<PRODUCT_LIST_SYSTEM_NAME_FROM_LOOKUP>` and
`<PRODUCT_LIST_INTERNAL_ID_FROM_LOOKUP>` must be replaced with the exact `systemName` and `internalId` from the same row of
`entities_list(entityType: "ProductList")`; preview/save do not check that the list exists.

- **Some mechanics work only in an automated campaign.** They read a customer action, and a bulk
  campaign has no action — the row will go out empty. These are `ORDER`, `ORDER_PRODUCTS` (order products),
  `ViewedProducts`, `ViewedProductsInSession`, `ProductView`, `SessionProductCategoryViews`,
  `SessionGetAddedToListProducts`, `RecentlyBoughtProducts`. Independent of the campaign kind are
  `RECIPIENT_RECOMMENDATIONS`, `PRODUCT_RECOMMENDATIONS`, `FROM_PRODUCT_LIST`, `FROM_SEGMENT`,
  `ProductShowcase`. The converter itself lists everything it accepts in the error message; the campaign kind
  is set at creation and does not change afterwards — see the `email-ops` skill.

## Manually selected list

`ProductShowcase` — the mechanic set by the “Fill manually” toggle («Заполнить вручную») — holds one
`<CollectionCard>` per selected product, and each card names its product with the `product`
attribute: the triple `internalId`, `externalId`, `externalSystemName`, as the lookup prints it in a row.
A card without content is drawn by the template; a card with its own columns — by them. The template in the
example below is cut down to one element so that the cards themselves are visible; the card styling is taken
from the beginning of the file, not from here:

```jsx
<CollectionRow dataSource={{ "key": "ProductShowcase" }} orientation="vertical" columnCount={{ "value": 3 }}>
  <CardTemplate url={[<Var param="ProductUrl" />]}><Column size={12}><Text><p style="margin: 0"><Var param="ProductName" /></p></Text></Column></CardTemplate>
  <CollectionCard product={{ "internalId": "<PRODUCT_INTERNAL_ID_FROM_LOOKUP>", "externalId": "<PRODUCT_EXTERNAL_ID_FROM_LOOKUP>", "externalSystemName": "<PRODUCT_EXTERNAL_SYSTEM_NAME_FROM_LOOKUP>" }} />
  <CollectionCard product={{ "internalId": "<SECOND_PRODUCT_INTERNAL_ID_FROM_LOOKUP>", "externalId": "<SECOND_PRODUCT_EXTERNAL_ID_FROM_LOOKUP>", "externalSystemName": "<SECOND_PRODUCT_EXTERNAL_SYSTEM_NAME_FROM_LOOKUP>" }}><Column size={12}><Text>This exact one</Text></Column></CollectionCard>
</CollectionRow>
```

Do not copy this fragment before the lookup. Each `product` triple must be replaced with the exact
`internalId`/`externalId`/`externalSystemName` from `entities_list(entityType: "Product")` or from the email
you are editing.

The rules of such a row, and breaking any of them is an error:

- **A card always names a product.** A card that names none is not a second template but an error;
  if no products are selected yet, do not write cards at all — see the beginning of the file.
- **The triple is written in full and is not invented.** Two values out of three are not derived from the third, so
  a product named incompletely or with an extra field is rejected. The triple is taken from the lookup or from the
  email you are editing. Do not write an invented triple: preview/save do not prove that such a product exists and
  is usable.
- **Do not write keys or the `selections` map.** The row knows the product under its own name, but that name is
  made by the converter, like all other ids of the document. An email read from the editor sometimes comes in the
  old form — `selectionId="…"` instead of `product` — it is readable and does not need rewriting; but both forms
  on one card are rejected: a product is named once.
- **Products are looked up with `entities_list`, `entityType: Product`.** The user names a product in words
  (“Home Alone quest”), by vendor code or by external id — any of the three is searched with a single `query`.
  The result row has exactly the triple the card asks for: `internalId`, `externalId`, `externalSystemName` — and
  next to it `name`, `price`, `vendorCode`, `isAvailable`, `url`, `pictureUrl`, so the person can recognize the product.

  **More than one found — show them and ask, do not choose yourself.** The catalog holds many products with the
  same name; price and vendor code are printed for exactly this. And do not silently offer a product with `isAvailable: false`.

  Nothing found — do not pick something similar and do not invent: ask the user for a more precise name,
  vendor code or external id, or build the row without cards (below).
- **Product data is filled in on its own — it does not need to be passed.** The email stores only the identifier
  triple per card, and `visual_template_preview` itself reads the name, price and image from the project catalog
  by these triples. It is enough for you to name the product correctly in `product` — the rest will appear in the
  preview without your involvement, and there is no need to restate product data in the call. A product that is
  no longer in the catalog will be drawn by the card as a placeholder; all cards will be drawn the same way if the
  catalog is unavailable — the layout will render in any case. A personal price and bonuses will never be in the
  preview: they are individual for each recipient, and nobody knows them before sending.
- **No products selected yet — write the row without cards.** `ProductShowcase` with one `<CardTemplate>` and not a
  single `<CollectionCard>` is a legal row: the card styling is ready, products are selected into it
  separately. Do this when there is nowhere to take products from, and **tell the user that they need to be
  selected** — in the editor or by naming them to you. Add cards when the products appear.
- **Cards exist only under `ProductShowcase`.** Under a mechanic that finds products itself, the row would
  draw the cards and leave the mechanic unread.
- **A card without its own content needs a `<CardTemplate>`** in the row — otherwise there is nothing to draw it
  with. And such a card carries nothing but the named product: a setting on it would be lost, so it is
  rejected.

## Card heights and `autosizeKey`

`height` on a card element carries, next to the height, an **autosize key**: elements with the same key are
measured together and take the height of the tallest one. The template is drawn once per product, so its
elements are one group by themselves; the key matters where several cards carry their own content.

**Measured together — only with `settingType: "manual"`.** It has two values, and they decide different things.
`manual` — height as a number in px, and the element joins the autosize group: this is exactly how the row's
cards keep the same shape whatever the products. `original` — height by content, and the element **drops out**
of alignment: it will have no group in the markup, and an `autosizeKey` next to it does nothing. Do not align
text with a key while leaving it `original` — the key will be set, but the heights will not match.

**The default differs between elements, and for a reason** (values — §6 `dsl-surface.md`): text length cannot be
guessed in advance, so text has height by content, while products come in different proportions, so the image
has a fixed one. When writing it yourself, stick to the same: `original` for text and the splitter, a number for the image.

**Image height in the editor's layouts is 180** for three in a row and for the list, **260** for two in a row;
if you take a layout from there, take this too. For your own layout, compute from the card width: email width
minus the block's side `innerSpacing` minus `cardGap` between cards, divided by their number. Otherwise the
default container will leave empty space under a square photo.

- **Do not invent a uuid for the key.** It reads as carried over from a saved email, but it groups
  emptiness — the card will silently drop out of the height it was supposed to share.
- **A key you did not write will appear on its own.** The converter issues a fresh one to every card element
  that had no key — so after reading the email you will see a uuid where you set nothing. This is
  not the same as an invented one: it is real, the element is in its group. When editing such an email, carry it
  over as is.
- **The key is either not written** (the card is measured by itself), **or carried over** from the email you are
  editing, **or it is a name in words** for a group you have in mind: `autosizeKey: "the-picked-two"` on two
  cards will put them at the same height.
- A card with its own content stands in its own group by default — the same as a detached card on the
  canvas: its height becomes the author's business, which is what detaching is for.

## Attributes of elements inside a card

An element in a card carries settings it does not have in an ordinary column — outside a card the same
attributes will be `Unknown attribute`:

| Attribute | On | What it does |
|---|---|---|
| `height` | `Text`, `Image`, `Split` | Element height and autosize key. JSON: `{ "settingType": "original" \| "manual", "height": 180, "mobile": { … }, "autosizeKey": "…" }`. How the values differ and what to write by default — above, “Card heights and `autosizeKey`” |
| `url` | `Text`, `Divider` | Link to the product — so that not only the button leads to it. Outside a card the same attribute on them is an error: there is nowhere to lead |
| `alt` | `Image` | Image caption for when it did not load. In a card it is the product name chip; a string is accepted too, but one caption for all products is not what is needed |

## Attributes of row containers

| Container | Attributes |
|---|---|
| `CollectionRow` | `dataSource`, `columnCount`, `rowCount`, `orientation`, `cardGap`, `rowGap`, `isCheckerboardEnabled`, `background`, `border`, `borderRadius`, `innerSpacing`, `isColumnsMobileAdaptive`, `columnVerticalDirection`, `visibilityOnDevices` |
| `CardTemplate` | `url`, `columnGap`, `verticalAlign`, `background`, `border`, `borderRadius`, `innerSpacing` |
| `CollectionCard` | the same as `CardTemplate`, plus the required `product` — the product the card stands for |

Row value forms: `columnCount` and `rowCount` — `{ "value": N }`; `cardGap` and `rowGap` —
`{ "size": N, "mobile": { "size": N } }`; `orientation` — a string: `"horizontal"` (default) draws one
card per line and does not read the column count, `"vertical"` gives a `columnCount` × `rowCount` grid. The
written form is checked against the default, and the response to a wrong form shows the whole default — you do not
need to remember the form, reading the refusal is enough.

`url` on a card makes the whole card lead to the product, not only its button. Inside a card `url` also appears
on `<Text>` and `<Divider>` — outside a card the same attribute on them is an error, because there is nowhere to lead.
