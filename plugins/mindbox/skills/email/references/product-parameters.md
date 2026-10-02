# Product parameters — only inside a product row

**When to read:** the product value you need is not in the short table in `product-rows.md`, or you need to pick a product `<Var>` for a card.

**Return:** to `product-rows.md`, then to Self-check in `SKILL.md`; the parameter must match the entity that the chosen mechanic iterates over.

Five product entities — product (`product`), viewed product (`productView`), product list row
(`productListItem`), single list row (`singleProductListItem`) and order line (`orderItem`) —
127 parameters out of 176, with the attributes of each. Open it when the product value you need is not in the
“value → parameter name” table in [product-rows.md](./product-rows.md). Parameters of the email itself are in
[personalization-parameters.md](./personalization-parameters.md).

Such a parameter is authored only inside a block that iterates over products — that is, a product row
`<CollectionRow>` with its cards (`references/product-rows.md`). Do not use product chips outside a product
row card: for products, build a `<CollectionRow>`.

**The family is chosen not by the author but by the row's mechanic.** A chip of an entity other than the one the mechanic iterates over is
rejected by the converter, which itself names the counterpart parameter from the right family (“Use `ProductListItemProductName` — the
same value, read from `productListItem`”). The “mechanic → family” table is in `references/product-rows.md`;
here it is repeated in the table headings.

**When asked to display order products, a cart, recommendations or viewed products, do not pick a similar
parameter from the email list above** — build a product row. A row with recommendations or another
automatic mechanic can be built entirely from JSX. So can a row with manually selected products: the
products themselves are found with a lookup (`entities_list`, `entityType: Product`) or taken from the email you are editing, and
are never invented. If there is nowhere to take them from, build the design and say that the products need to be selected.

## `product` — product (19)

Mechanics `Recipient.Recommendations`, `Products.GetBySegment`, `SessionGetViewProducts`, `SessionGetAddedToListProducts` and the other twelve — everything except the three below.

| `param` | its attributes |
|---|---|
| `ProductAddToCartUrl` | `hostname`, `externalSystem`, `amount`, `discountQuery`, `destinationRoute`, `productSegment`, `affixVariableText` |
| `ProductBonus` | `inputLimitSetting`, `productSegment`, `numericFilter`, `formatDecimal`, `pluralizationAffix`, `affixVariableText` |
| `ProductCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `ProductCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `ProductCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `ProductCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductDescription` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductDiscount` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductExternalId` | `externalSystem`, `productSegment`, `affixVariableText` |
| `ProductName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductOldPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductPersonalPrice` | `personalPriceTypeSettings`, `inputLimitSetting`, `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixPersonalPriceEditor`, `affixVariableText` |
| `ProductPictureUrl` | `productSegment` |
| `ProductPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductUrl` | `productSegment`, `affixVariableText` |
| `ProductVendorCode` | `productSegment`, `caseFormatter`, `affixVariableText` |
| `ProductVendorName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |

## `productListItem` — product list row (33)

Mechanic `FROM_PRODUCT_LIST` — the row iterates over a project product list.

| `param` | its attributes |
|---|---|
| `ProductListItemCount` | `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemCurrentPriceOfLine` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `ProductListItemCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `ProductListItemCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `ProductListItemCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemDiscountByOldProductPrice` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemDiscountByPriceOfLine` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemOldPriceOfLine` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemPriceOfLine` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemProductAddToCartUrl` | `hostname`, `externalSystem`, `amount`, `discountQuery`, `destinationRoute`, `productSegment`, `affixVariableText` |
| `ProductListItemProductBonus` | `inputLimitSetting`, `productSegment`, `numericFilter`, `formatDecimal`, `pluralizationAffix`, `affixVariableText` |
| `ProductListItemProductCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `ProductListItemProductCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `ProductListItemProductCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemProductCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `ProductListItemProductCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductDescription` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductDiscount` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemProductDiscountByPriceInList` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductListItemProductExternalId` | `externalSystem`, `productSegment`, `affixVariableText` |
| `ProductListItemProductName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductOldPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemProductPersonalPrice` | `personalPriceTypeSettings`, `inputLimitSetting`, `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixPersonalPriceEditor`, `affixVariableText` |
| `ProductListItemProductPictureUrl` | `productSegment` |
| `ProductListItemProductPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductListItemProductUrl` | `productSegment`, `affixVariableText` |
| `ProductListItemProductVendorCode` | `productSegment`, `caseFormatter`, `affixVariableText` |
| `ProductListItemProductVendorName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |

## `singleProductListItem` — single list row (26)

Mechanic `ProductListItem`.

| `param` | its attributes |
|---|---|
| `SingleProductListItemCount` | `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `SingleProductListItemCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `SingleProductListItemCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `SingleProductListItemCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `SingleProductListItemCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `SingleProductListItemCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `SingleProductListItemProductAddToCartUrl` | `hostname`, `externalSystem`, `amount`, `discountQuery`, `destinationRoute`, `productSegment`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `SingleProductListItemProductCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemProductDescription` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemProductDiscount` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `SingleProductListItemProductDiscountByPriceInList` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `SingleProductListItemProductExternalId` | `externalSystem`, `productSegment`, `affixVariableText` |
| `SingleProductListItemProductName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemProductOldPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `SingleProductListItemProductPictureUrl` | `productSegment` |
| `SingleProductListItemProductPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `SingleProductListItemProductUrl` | `productSegment`, `affixVariableText` |
| `SingleProductListItemProductVendorCode` | `productSegment`, `caseFormatter`, `affixVariableText` |
| `SingleProductListItemProductVendorName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |

## `orderItem` — order line (30)

Mechanic `ORDER` — the row iterates over the order lines.

| `param` | its attributes |
|---|---|
| `OrderItemAmount` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `OrderItemBaseAmount` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `OrderItemBasePricePerUnit` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `OrderItemCount` | `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `OrderItemCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `OrderItemCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `OrderItemCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `OrderItemCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `OrderItemCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemDiscount` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `OrderItemDiscountPerUnit` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `OrderItemPresentmentLineTotal` | `productSegment`, `numericFilter`, `formatMoney`, `affixVariableText` |
| `OrderItemPresentmentUnitPrice` | `productSegment`, `numericFilter`, `formatMoney`, `affixVariableText` |
| `OrderItemPricePerUnit` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `OrderItemProductAddToCartUrl` | `hostname`, `externalSystem`, `amount`, `discountQuery`, `destinationRoute`, `productSegment`, `affixVariableText` |
| `OrderItemProductCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `OrderItemProductCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `OrderItemProductCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `OrderItemProductCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemProductCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `OrderItemProductCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemProductDescription` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemProductExternalId` | `externalSystem`, `productSegment`, `affixVariableText` |
| `OrderItemProductName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemProductPictureUrl` | `productSegment` |
| `OrderItemProductUrl` | `productSegment`, `affixVariableText` |
| `OrderItemProductVendorCode` | `productSegment`, `caseFormatter`, `affixVariableText` |
| `OrderItemProductVendorName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `OrderItemStatus` | `productSegment`, `caseFormatter`, `affixVariableText` |

## `productView` — viewed product (19)

Mechanic `ProductView`.

| `param` | its attributes |
|---|---|
| `ProductViewPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductViewProductAddToCartUrl` | `hostname`, `externalSystem`, `amount`, `discountQuery`, `destinationRoute`, `productSegment`, `affixVariableText` |
| `ProductViewProductCustomFieldDate` | `customFieldType`, `productSegment`, `formatDate`, `affixVariableText` |
| `ProductViewProductCustomFieldDateTime` | `customFieldType`, `productSegment`, `formatDateTime`, `affixVariableText` |
| `ProductViewProductCustomFieldDecimal` | `customFieldType`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductViewProductCustomFieldEnum` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductViewProductCustomFieldInteger` | `customFieldType`, `productSegment`, `numericFilter`, `formatInteger`, `affixVariableText` |
| `ProductViewProductCustomFieldString` | `customFieldType`, `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductViewProductDescription` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductViewProductDiscount` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductViewProductDiscountByViewedPrice` | `discountSettings`, `productSegment`, `numericFilter`, `formatDecimal`, `affixVariableText` |
| `ProductViewProductExternalId` | `externalSystem`, `productSegment`, `affixVariableText` |
| `ProductViewProductName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
| `ProductViewProductOldPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductViewProductPictureUrl` | `productSegment` |
| `ProductViewProductPrice` | `productSegment`, `numericFilter`, `formatDecimalPrice`, `affixVariableText` |
| `ProductViewProductUrl` | `productSegment`, `affixVariableText` |
| `ProductViewProductVendorCode` | `productSegment`, `caseFormatter`, `affixVariableText` |
| `ProductViewProductVendorName` | `productSegment`, `formatString`, `caseFormatter`, `affixVariableText` |
