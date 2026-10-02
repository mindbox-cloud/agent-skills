# Personalization — chips in the email

**When to read:** the first time a `<Var>`, a dynamic image/link or a tenant-specific personalization setting appears in the email.

**Return:** to Workflow / Self-check in `SKILL.md`; check the exact `param`, the allowed placement, the required settings and the lookup of every `systemName`.

A personalized value (promo code, recipient name, price) is written with the `<Var>` tag. The converter builds
a **chip** from it — the same thing the editor inserts as a pill (chip) — so a chip from the email opens and is edited
in the editor as usual. Quokka expressions (`${Customer...}`, `${Order...}`) are not needed for this and
must not be written: the only exception is `${Message.UnsubscribeLink}` in `href` under the rules of SKILL.md.

## Form

`param` is the parameter name; the other attributes are its settings, formatters and affixes:

```jsx
<Text><p style="margin: 0;">Your promo code: <Var param="PromoCodeValue" promoCodePool={{ "systemName": "<PROMO_CODE_POOL_SYSTEM_NAME_FROM_LOOKUP>", "name": "<PROMO_CODE_POOL_NAME_FROM_LOOKUP>", "internalId": "<PROMO_CODE_POOL_INTERNAL_ID_FROM_LOOKUP>" }} caseFormatter={{ "format": "uppercase" }} /></p></Text>
```

Do not copy tenant-specific values before the lookup: take the `systemName`, `name` and `internalId` fields from one row
of `entities_list(entityType: "PromoCodePool")`.

Write only what you change: every part has a default, extra fields are not needed.

An attribute this parameter does not have is a converter error naming the parameter. The list of parameters and their
attributes is at the end of the file.

## Where it can go

**In text** — as above, inside the markup of `<Text>` or `<BulletItem>`.

**In a value string** — as a list of segments: the surrounding text and a `<Var>` in place of the chip.

```jsx
<Button url={["https://shop.ru/sale?promo=", <Var param="PromoCodeValue" promoCodePool={{ "systemName": "<PROMO_CODE_POOL_SYSTEM_NAME_FROM_LOOKUP>", "name": "<PROMO_CODE_POOL_NAME_FROM_LOOKUP>", "internalId": "<PROMO_CODE_POOL_INTERNAL_ID_FROM_LOOKUP>" }} />]}>Claim</Button>
```

**In a label** — the same way, between `<Label>` tags, as a list of segments. A label with an empty variable disappears
entirely instead of leaving an empty space: this is how the “−30%” badge is made that a product without a discount does not have.
Labels live in a product row card: the converter will let them through in an ordinary row too, but the editor canvas does not have them there. Details —
`references/dsl-surface.md` §6 “Labels and Label”.

```jsx
<Labels><Label>{["−", <Var param="ProductDiscount" />]}</Label></Labels>
```

**In button words** — in the same form between the tags. Tenant-specific settings here are also taken from the lookup:

```jsx
<Button>{["Redeem ", <Var param="RecipientBonusBalance" balance={{ "systemName": "<BALANCE_SYSTEM_NAME_FROM_LOOKUP>" }} />, " points"]}</Button>
```

**Inside a structure** — at any depth. This is how a personal image is set: `mode: "dynamic"`, and the address
comes entirely from the parameter. Fields missing from the value are merged onto the default, so `static` does not need
to be written:

```jsx
<Image image={{ "mode": "dynamic", "dynamic": [<Var param="RecipientCustomFieldString" customFieldType={{ "systemName": "<CUSTOM_FIELD_SYSTEM_NAME_FROM_LOOKUP>" }} />] }} />
```

Write a domain prefix before `<Var>` only when the user named it
(`"dynamic": ["https://cdn.shop.ru/", <Var param="ProductPictureUrl" />]`). By default the field value is
the whole address: an invented prefix breaks the image for all recipients.

A string without a chip stays a string — a list is needed only where there is a chip. And the other way round: the converter reads an array without `<Var>`
as an array of values, so joining two texts is not expressed as a list.

## The entity matters more than the name

Every parameter has its own entity. Only one is available from an ordinary column — the email itself (`root`), and that is 49
parameters, including all order data: in an automated email `OrderTotalAmount`, `OrderExternalId`,
`OrderDateTime`, order custom fields and the other `Order*` go anywhere — in a heading, in text, in a
button. The five product entities (`product`, `productView`, `productListItem`, `singleProductListItem`,
`orderItem`) are the remaining 127, and they come alive only inside a block that iterates over products — that is,
a product row `<CollectionRow>` with its cards (`references/product-rows.md`). A chip of an entity other than the one
the row's mechanic iterates over is rejected by the converter, which itself names the counterpart from the right family. A product
chip is not placed in an ordinary column: for products, build a `<CollectionRow>`. The list of email parameters is in
`personalization-parameters.md`,
product parameters by entity are in `product-parameters.md`.

The campaign kind is also a constraint: 26 parameters marked *(automated only)* read the order, and in a bulk
campaign they have nothing to read.

## `type` inside `customFieldType` is not written

It is derived from `param` — a chip of a string field is never a date — and it is the only chip field that the
converter will **not** reject: the value will be merged onto the default and go into storage wrong.

The trap here is in the lookup response: a custom field has a value type, and it should not be carried into the chip. The value
type selects the `param`, and it is not copied into the chip.

The rest of what the converter derives on its own (`versionNumber`, `parameterString`) need not be memorized: the parameter has no attribute with
such a name, and an attempt will come back as an error with that text.

## Tenant-specific names — only from the lookup

A setting that refers to a project reference list requires a real name. **Do not invent a
`systemName`.** It goes into the email expression as is, and the converter does not query the reference list. What
happens depends on what the name turns out to be:

| what is in the setting | what is in the email | how it shows up |
|---|---|---|
| empty | `Order.IDs.` | the expression does not parse: `Invalid symbol`, saving and link enrichment fail |
| a name that does not exist in the project | `Order.IDs.Backend` | «В рассылке содержатся неизвестные параметры» (“The campaign contains unknown parameters”) — a template error on the campaign card |
| a real name, but the customer has no value | `Order.IDs.website` | a silent empty spot in the sent email |

Only the third case stays silent, and it is expected. The first two are caught by the platform and shown to the
user, and the second also comes in the save response as a validation remark: if it is there,
show it to the user rather than reporting a clean save.

**And all the more, do not leave such a setting empty.** The name is substituted into the expression as a path segment
(`Order.IDs.<systemName>`, `PromoCode.WithType<systemName>.Value`), so an empty value gives
`Order.IDs.` — an expression without a name, and the whole email stops being valid.

The converter rejects such a chip, naming the unfilled attribute:

```
<Var param="OrderExternalId"> needs a value in `orderExternalIdSelection`
```

If an empty setting did get into the email, later checks may answer `Line N, symbol M: Invalid
symbol`, one error per place with a broken expression. The rule is the same: first fill the setting from the
lookup or from the user.

Values are required by: `customFieldType`, `externalSystem`, `orderExternalIdSelection`, `promoCodePool`,
`balance`, `hostname` (on `*AddToCartUrl`) — a non-empty name; `discountQuery` — when enabled; `amount` —
greater than zero; `affixVariableText` — the replacement text when `emptyBehavior.mode` is `fallback`.

Hence the rule: **no name — do not write the parameter.** A parameter with an unfilled setting is not “a chip that
the user will finish filling in the editor”, but a broken email. Ask for the name or suggest a parameter that
does not need a reference list.

Which setting to look up with which entity type (the call rules are in the tool's own description):

| setting | how to look it up | parameters |
|---|---|---|
| `promoCodePool` | `entityType: PromoCodePool` | `PromoCodeValue`, `PromoCodeExpirationDateTime`, the `discountQuery` setting |
| `customFieldType` | `entityType: CustomField` + `ownerEntityType` by the parameter's entity | all `*CustomField*` |
| `balance` | `entityType: Balance` | `RecipientBonusBalance`, `RecipientNearestExpirationBonuses*` |
| `externalSystem` | `entityType: ExternalSystem` | `ProductExternalId` and related, `*AddToCartUrl` |
| `orderExternalIdSelection` | `entityType: CustomField`, `ownerEntityType: OrderExternalId` | `OrderExternalId` |
| `productListSystemName` + `productListInternalId` | `entityType: ProductList` | settings of product row mechanics with a list; take both values from one lookup row |
| `recoMechanicSystemName` | `entityType: RecommendationMechanic` | the setting of product row recommendation mechanics |
| `product` | `entityType: Product` | the manual card `ProductShowcase` |

`ownerEntityType` for custom fields is taken from the field's owner, not from the parameter name: `Customer` for
`Recipient*`, `Order` for `Order*CustomField*`, `OrderLine` for `OrderItem*`, `Product` for `Product*`,
`ProductListItem` for `ProductListItem*`.

**External systems and external order identifiers are different reference lists.** `ExternalSystem` is the list
of the project's integrations (“1C”, “Shopify”); product parameters need it. The order number has a different reference list: it is
a custom field type owned by `OrderExternalId`, and the editor asks for exactly that in its popover.
An integration name put in `orderExternalIdSelection` will not be a converter error — the platform
will answer «В рассылке содержатся неизвестные параметры» (“The campaign contains unknown parameters”) on the campaign card.

Carry into the JSX all fields from the lookup response that exist in the setting's model. For ordinary chips, `systemName`
takes part in the email expression, while `name` and `internalId` are needed by the editor: `name` is used to render the chip preview on
the canvas, `internalId` preselects the select in the popover. Do not remove them as “extra” if the lookup returned them.

A separate rule applies to product row mechanics with a list: `productListSystemName` and
`productListInternalId` are both required and are taken from one row of `entities_list(entityType: ProductList)`.
Details are in `product-rows.md`.

Two limitations on pools that the lookup will not warn about. A promo code is read only from pools of single-use
codes and referral pools — a pool of another type does not work in a chip. And a pool's system name is optional: if the
response lacks it, the chip cannot be built — show such pools to the user and ask them to choose another one or to create
a system name.

`hostname` on `*AddToCartUrl` does not live in the lookup — it is the store's domain, and it is asked from the user;
`amount` and `destinationRoute` there have working defaults.

Custom fields are the only setting where the lookup is also needed to choose the `param` itself: exactly one parameter out of seven per entity
fits a given field, and it is chosen by the value type and the type of the field's owner.
Multi-value fields fit no parameter.

## Long text: `formatString`

Text parameters (`ProductName`, `ProductDescription`, `ProductVendorName` and their counterparts in the other
families) have truncation enabled, and the default limit is **150 characters**. In the preview the chip does not
cut but pads: it repeats the value up to the limit. A 72-character name is shown twice with a
tail, and the card spreads to five or six lines, although exactly 72 will arrive in the email.

What to write depends on whether the products are known.

**Products are selected manually** — turn truncation off: the preview will show the real name, exactly what will be sent.

```jsx
<Var param="ProductName" formatString={{ "isTruncateEnabled": false }} />
```

**Products come from a mechanic** (recommendations, list, order) — a limit is needed: the names are not known in advance
and will arrive at any length. The editor presets are `40`, `80`, `120`, `200`; a value outside them will show in
the editor as “other”; `40` is a name in a narrow card, `80`–`120` a description and full width.

```jsx
<Var param="ProductName" formatString={{ "isTruncateEnabled": true, "truncateValue": 40 }} />
```

## Empty value

The `affixVariableText` affix decides what happens if the value is not found: `hide` removes it silently, `fallback`
substitutes a text. It also carries the “text around the variable” — `aroundText` is printed only when the value exists.

```jsx
<Text><p style="margin: 0;">Points: <Var param="RecipientBonusBalance" balance={{ "systemName": "<BALANCE_SYSTEM_NAME_FROM_LOOKUP>" }} affixVariableText={{ "emptyBehavior": { "mode": "fallback", "fallbackText": "0" } }} /></p></Text>
```

An empty value can also be hidden with the whole row: `emptyVariableBehavior="hide"` on `<FlexRow>` removes
the row from the email when a variable in it has no value. This is a node setting, not a chip setting, and it works only
when sending — in the preview the row is always visible. The same is written on an element too.

```jsx
<FlexRow emptyVariableBehavior="hide"><Column size={12}><Text><p style="margin: 0">Delivery: <Var param="OrderDeliveryCost" /></p></Text></Column></FlexRow>
```

## A chip that cannot be read

A parameter missing from this campaign's registry, or an unparseable model, arrives in the stored form:

```jsx
<Text><p style="margin: 0;">Code: <personalization-parameter model="bm90LWEtbW9kZWw="></personalization-parameter></p></Text>
```

Leave such fragments as they are — the converter did not understand them, and rewriting loses the chip. In the conversation you can
say that the email contains personalization that this project does not recognize.

## The greeting carries the whole phrase

`RecipientGreeting` is not a name but «Здравствуйте, » + name + «!», and for a customer without a name —
«Здравствуйте!» (the platform writes these words in the project's language; in an English project: “Hello, ” / “Hello!”). No words are written around it: `Hello, <Var param="RecipientGreeting" />!` will be sent as
“Hello, Hello, Anya!!”.

Your own words are set by `affixVariableText`: `aroundText.before` before the name, `aroundText.after` after it,
`emptyBehavior.fallbackText` — instead of the whole phrase when there is no name.

```jsx
<Var param="RecipientGreeting" affixVariableText={{ "aroundText": { "before": "Hi, ", "after": "!" }, "emptyBehavior": { "mode": "fallback", "fallbackText": "Hi!" } }} />
```

If you set it, set both blocks: the attribute is laid over the default part by part, and `aroundText` alone
will leave the customer without a name with the default phrase. This is visible neither on the canvas nor in the preview.

## Bonuses: the word is already inside the chip

`ProductBonus` and `ProductListItemProductBonus` have pluralization enabled and the word is substituted automatically — do not write it next to
the chip. Your own word is set by forms:

```jsx
<Var param="ProductBonus" pluralizationAffix={{ "pluralForms": { "one": "балл", "few": "балла", "many": "баллов" } }} />
```

The other numeric ones (`RecipientBonusBalance`, `RecipientNearestExpirationBonuses`, `OrderChargedBonuses`,
`RecipientCustomFieldInteger`) have pluralization disabled — the word will appear only if you enable it:

```jsx
<Var param="RecipientBonusBalance" balance={{ "systemName": "<BALANCE_SYSTEM_NAME_FROM_LOOKUP>" }} pluralizationAffix={{ "isEnabled": true, "pluralForms": { "one": "балл", "few": "балла", "many": "баллов" } }} />
```

All three forms are required: with only some of them the converter answers with the error `needs a value in pluralizationAffix`.

## Common parameters

Enough for most emails. The first seven are email parameters (`root`) and go anywhere in it; rows
marked *(automated only)* require an order, that is, an automated campaign. The rest are product parameters and
go only in a product row card: the entity of `Product*` is `product`, of `ProductListItem*` —
`productListItem`, and the choice between them is not free, it is decided by the row's mechanic. The “mechanic →
family” table is in `references/product-rows.md`.

| `param` | what it is | its attributes |
|---|---|---|
| `RecipientGreeting` | the full greeting by name | `affixVariableText` — the greeting words |
| `PromoCodeValue` | promo code | `promoCodePool`, `caseFormatter`, `affixVariableText` |
| `PromoCodeExpirationDateTime` | the date until which the promo code is valid | `promoCodePool`, `formatDate`, `affixVariableText` |
| `RecipientBonusBalance` | bonus points on the balance | `balance`, `formatInteger`, `pluralizationAffix`, `affixVariableText` |
| `OrderTotalAmount` *(automated only)* | order total | `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `OrderExternalId` *(automated only)* | order number | `orderExternalIdSelection`, `affixVariableText` |
| `OrderDateTime` *(automated only)* | order date | `formatDateTime`, `affixVariableText` |
| `ProductName` | product name | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductPrice` | product price | `productSegment`, `formatDecimalPrice`, `affixVariableText` |
| `ProductOldPrice` | price before discount | `productSegment`, `formatDecimalPrice`, `affixVariableText` |
| `ProductPictureUrl` | product image address | `productSegment` |
| `ProductUrl` | product link | `productSegment`, `affixVariableText` |
| `ProductListItemProductName` | product name in a list row | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductPrice` | product price in a list row | `productSegment`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemProductPictureUrl` | product image in a list row | `productSegment` |

**The parameter you need is not here — open the full list:** email parameters in
`references/personalization-parameters.md`, product parameters by entity in `references/product-parameters.md`.
Do not pick a similarly named one from this list: the converter will reject a name that is not in the registry, and a similar
parameter of another entity is not a correct substitute.
