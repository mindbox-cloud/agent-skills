# Examples (curated)

**When to read:** you need a verified sample of a typical block; §14 is mandatory before
the first numbered-point block.
**Return:** to Workflow / Self-check in `SKILL.md`.

The patterns below cover typical cases. Each example is self-contained, with an explanation of “why this way”.

## 1. Text block (heading + body)

**User description:** “Heading "50% off", body "Only until the end of the week"”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Text style={{ fontSize: 32, inscription: ["bold"], align: "center", mobile: { fontSize: 28, align: "center" } }}>50% off</Text>
        <Text>Only until the end of the week</Text>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `style={{ fontSize: 32, inscription: ["bold"], align: "center", mobile: { fontSize: 28, align: "center" } }}` — the heading has an explicit size, alignment and mobile branch. This detaches it from the email's shared styles: `style` is the entire typography of this text, and there is no halfway (§8 `dsl-surface.md`).
- `size={12}` — one full-width column.
- Plain text — no `<b>`, no `<a>`, no HTML markup.
- `Text` without `style` — ordinary default text from the theme.

## 2. CTA button

**User description:** “A "Buy now" button linking to https://shop.example.com”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Button url="https://shop.example.com">Buy now</Button>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- The address is known — write it in `url`; the converter validates the value: `https://`, `tel:` or `mailto:`. No address — ask for it; if the user does not have it, `<Button>` without `url`, and the button is saved with an empty link.
- The button text is plain text, no markup.
- The default alignment is `center` (no need to specify it).
- The default `background` is black, `color` is white. This is a stand-in for when nothing is known about the style, not a finished design: if a brand or reference is given, set the brand `background` and `simpleTextStyles.color` explicitly.

## 3. Rich text with markup

**User description:** “Text with a link and a bold word”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Text>
          <p style="margin: 0;">Summer <strong>sale</strong>, <a href="https://shop.example">see more</a></p>
        </Text>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- HTML markup inside `<Text>` — `<p>`, `<strong>`, `<a>` — as in the editor.
- `<p style="margin: 0;">` — the paragraph wrapper, preserves the text structure.
- No `style` on `<Text>` — default styles from the theme.
- `href` inside `<a>` — a link in the markup, not in an element attribute.

## 4. Two-column layout (6+6)

**User description:** “Text on the left, button on the right”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={6}>
        <Text>Left column</Text>
      </Column>
      <Column size={6}>
        <Button url="https://example.com">Button</Button>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `6+6=12` — the sum of `size` within one `FlexRow` is strictly 12.
- Order in JSX = left-to-right order in the email.
- `Text` and `Button` in different columns are independent elements.
- No `width` in percent — width is set only through `size`.

## 4a. Transferring a row with two images (6+6)

**User description:** “Transfer from HTML or another email platform a 1200px-wide row with two adjacent 600px + 600px images”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={6}>
        <Image image={{ mode: "static", static: { url: "https://cdn.example.com/left.jpg", fileName: "left.jpg" } }} />
      </Column>
      <Column size={6}>
        <Image image={{ mode: "static", static: { url: "https://cdn.example.com/right.jpg", fileName: "right.jpg" } }} />
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- The two images were neighbours in one row of the source HTML or another email platform, so they stay in one
  `FlexRow` instead of turning into two consecutive rows of `Column size={12}`.
- `600+600` out of a `1200px` container is `6+6` in a 12-column grid; the column sum is strictly 12.

## 5. Block with background and padding

**User description:** “A section with a grey background and 24px padding”

```jsx
<Template>
  <Block background={{ type: "color", value: "#f5f5f5" }} innerSpacing={{ top: 24, bottom: 24, left: 24, right: 24 }}>
    <FlexRow>
      <Column size={12}>
        <Text>Text inside</Text>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `background` is a JSON object `{ type: "color", value: "#f5f5f5" }`, not a string.
- `innerSpacing` is a JSON object with fields `top`, `bottom`, `left`, `right`; partial merge — specify only what changes, the other fields come from the prop's default.
- `Block` is an email section; the background and padding apply to the whole section.
- `Block.innerSpacing` creates space around the row; `Column` keeps its own defaults.
- Both `FlexRow` and `Column` have `background` too: a background on a `Column` together with `borderRadius` makes a card; alternating backgrounds on adjacent `Block`s gives section contrast. Do not settle for a single grey background for the whole email (example §13).
- Ordinary vertical spacing is done through `innerSpacing`, not
  `gapAfterBlock`, so the section color continues under the spacing. `gapAfterBlock`
  would be needed only for a separate gap in the external background color.

## 6. Custom HTML

**User description:** “Insert custom HTML — a table with an unclosed `<br>` and a comment”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Html>{"<table><tr><td>A table, a comment, an unclosed <br> — anything</td></tr></table>"}</Html>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `<Html>` accepts only a quoted string: the live backend rejects direct JSX (`<Html><div>…</div></Html>`) (`<Html> may only contain text`).
- The quoted string is needed for required raw markup that cannot be expressed with standard blocks: for example tables, Outlook conditionals, entities and unclosed tags.
- Emails are stored as JSON; reading JSON→JSX may emit a quoted string — no data is lost.
- Use `<Html>` only when nothing else fits — text, button and image survive manual edits, an html block does not.

## 7. Menu of Text lines

**User description:** “Horizontal menu: Catalog and Sale”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Menu align={{ align: "center", mobile: { align: "center" } }} itemsGap={{ size: 16, mobile: { size: 16 } }}>
          <Text>Catalog</Text>
          <Text>Sale</Text>
        </Menu>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `<Menu>` contains direct lines without `<Column>`.
- All lines are of one type: here only `<Text>`.

## 8. Bulleted list

**User description:** “A list of two short benefits”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <BulletList bulletIcon={{ url: "https://cdn.example.com/dot.png", fileName: "dot.png" }}>
          <BulletItem>Free delivery</BulletItem>
          <BulletItem>Returns within 30 days</BulletItem>
        </BulletList>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `<BulletList>` contains only `<BulletItem>` and at least one line.
- `<BulletItem>` does not need a standalone stored schema here: the group creates the line.
- `bulletIcon` is set **mandatorily**: without it the system marker renders as a broken placeholder icon. The form is a flat object `{ url, fileName }`; `{{ mode: "static", static: {...} }}` is silently ignored, a string crashes the preview. The URL here is illustrative — take a confirmed one (icons are in the gallery's system folder).
- In this flat form the marker width is 4px, so the icon is a dot or a simple circle.
  A large marker uses a separate custom form with `size` — §14.
- This format is for **short one-liners**. A set of features with a heading and a description is built as a grid of cards — §13.

## 9. Social networks

**User description:** “Three social network icons 40px wide”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Socials imageSize={{ type: "fixed", width: 40, mobile: { type: "fixed", width: 40 } }}>
          <Image image={{ mode: "static", static: { url: "https://cdn.example.com/vk.png", fileName: "vk.png" } }} />
          <Image image={{ mode: "static", static: { url: "https://cdn.example.com/tg.png", fileName: "tg.png" } }} />
          <Image image={{ mode: "static", static: { url: "https://cdn.example.com/fb.png", fileName: "fb.png" } }} />
        </Socials>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `<Socials>` contains only `<Image image={{...}} />` lines; each icon has its own URL.
- `imageSize` is set on the group, not on individual Image lines.
- All `cdn.example.com` URLs here are illustrative. In a real answer replace each with only a URL
  that the user gave or that an unambiguously chosen gallery DTO returned.

## 10. Split 5+7

**User description:** “Text on the left, button on the right in a 5+7 layout”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Split>
          <Column size={5}><Text>Promotion terms</Text></Column>
          <Column size={7}><Button url="https://shop.example">Buy</Button></Column>
        </Split>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `5+7=12`: column sizes inside `<Split>` must add up to 12.
- Each Split column has only `size` and exactly one element.

## 11. Image from an external HTTPS URL

**User description:** “Put in a banner from my HTTPS link”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={12}>
        <Image image={{ mode: "static", static: { url: "https://cdn.example.com/banner.png", fileName: "banner.png" } }} />
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- `<Image>` is self-closing and contains the required `image` with `url` and `fileName`.
- The external HTTPS URL is inserted as is; the skill does not convert it into an internal URL.
- `https://cdn.example.com/banner.png` is an illustration. Generate it only when the user gave
  a specific URL; otherwise use a confirmed URL from a gallery DTO.

## 12. Cards with aligned CTAs (synchronized rows)

**User description:** “Two cards side by side, buttons at exactly the same height”

```jsx
<Template>
  <Block>
    <FlexRow>
      <Column size={6}><Image image={{ mode: "static", static: { url: "https://cdn.example.com/a.jpg", fileName: "a.jpg" } }} /></Column>
      <Column size={6}><Image image={{ mode: "static", static: { url: "https://cdn.example.com/b.jpg", fileName: "b.jpg" } }} /></Column>
    </FlexRow>
    <FlexRow>
      <Column size={6}><Text style={{ fontSize: 18, mobile: { fontSize: 16, align: "left" } }}>River cruises</Text></Column>
      <Column size={6}><Text style={{ fontSize: 18, mobile: { fontSize: 16, align: "left" } }}>Parks and estates</Text></Column>
    </FlexRow>
    <FlexRow>
      <Column size={6}><Text>Three evening routes along the river with stops at the best spots.</Text></Column>
      <Column size={6}><Text>Kolomenskoye, Tsaritsyno and the quiet corners of Izmailovo.</Text></Column>
    </FlexRow>
    <FlexRow>
      <Column size={6}><Button url="https://example.com/river">Water routes</Button></Column>
      <Column size={6}><Button url="https://example.com/parks">Green map</Button></Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- **This is a pattern for static content** that you write yourself. Product cards from a mechanic —
  recommendations, viewed products, order items — are built with a product row `<CollectionRow>`
  (`references/product-rows.md`): there the card is rendered once per product and the heights are synchronized on their own,
  without splitting into rows.
- Do not put each card whole into its own `Column`: column heights are computed independently,
  texts wrap differently, and the buttons “float” vertically. Fixed heights via
  `<Html>` tables are fragile: the renderer wraps components in its own tables/cells.
- Synchronized rows (images → headings → descriptions → buttons) guarantee the same top
  coordinate for all CTAs — they are in one `FlexRow`. This generalizes to 4+4+4 and any number of rows.
- **The price of the pattern is the mobile order.** With adaptive column stacking a mobile
  user will see “all images → all headings → all descriptions → all buttons”, not
  the cards one by one. If card-by-card mobile order matters, that is a product choice
  (for example separate mobile rows via `visibilityOnDevices`); discuss it with the user,
  do not decide silently.
- The card headings change the desktop `fontSize`, so each has a mobile branch; without it the backend may substitute `{ fontSize: 18, align: "left" }`.
- `cdn.example.com` URLs are illustrative — in a real answer only URLs from the user or a gallery DTO.

## 13. Card section: background, contrast, accent

**User description:** “A block about the service's features, in the brand style from the website”

```jsx
<Template>
  <Block background={{ type: "color", value: "#1f2933" }} innerSpacing={{ top: 32, bottom: 32, left: 24, right: 24 }}>
    <FlexRow>
      <Column size={12}>
        <Text style={{ fontSize: 28, color: "#ffffff", align: "center", mobile: { fontSize: 22, align: "center" } }}>
          <h1 style="margin: 0;">What the service includes</h1>
        </Text>
      </Column>
    </FlexRow>
    <FlexRow columnsGap={{ size: 16, mobile: { size: 12 } }} rowsGap={{ size: 16 }}>
      <Column size={6} background={{ type: "color", value: "#2c3844" }} borderRadius={{ topLeft: 12, topRight: 12, bottomLeft: 12, bottomRight: 12 }} innerSpacing={{ top: 20, bottom: 20, left: 20, right: 20 }}>
        <Text style={{ fontSize: 18, color: "#f2b705", mobile: { fontSize: 16, align: "left" } }}>Delivery</Text>
        <Text style={{ fontSize: 14, color: "#d5dbe1", mobile: { fontSize: 14, align: "left" } }}>We pick up and deliver at a time that suits you</Text>
      </Column>
      <Column size={6} background={{ type: "color", value: "#2c3844" }} borderRadius={{ topLeft: 12, topRight: 12, bottomLeft: 12, bottomRight: 12 }} innerSpacing={{ top: 20, bottom: 20, left: 20, right: 20 }}>
        <Text style={{ fontSize: 18, color: "#f2b705", mobile: { fontSize: 16, align: "left" } }}>Support</Text>
        <Text style={{ fontSize: 14, color: "#d5dbe1", mobile: { fontSize: 14, align: "left" } }}>We answer in chat seven days a week</Text>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**
- This is what a feature list looks like by default — a grid, not a `BulletList`. It generalizes to 4+4+4; with an odd number of cards the last row is filled out with an empty `<Column>`.
- Background + `borderRadius` + `innerSpacing` on a `Column` make a card; the `Block` background gives contrast with adjacent sections.
- Copy **color roles, not values**: section background → card background (one step lighter or darker than the section background) → heading accent → muted description color. The specific hex values come from the client's reference. The section does not have to be dark: a light brand has a light section background and dark text — contrast matters more than direction.
- Three size levels (28 / 18 / 14) create the hierarchy; the numbers themselves are also adjusted to the mockup.
- Styles are set via `style` on the node itself: the example detaches the texts from the email's shared styles rather than editing them. Editing the shared styles would spread across the whole email.
- Every `<Text>` with `fontSize` has a mobile branch, otherwise the backend substitutes its own values.
- **The font is intentionally not set here:** the description says nothing about it, so the look comes from the email's shared styles. Keep in mind that the theme may assign different typefaces to different components (for example `Menu` and `BulletList` get a different family than `Text`) — if uniform typography matters for the email, set `font.family` explicitly on all nodes, only from the lists in `formats.md` §style.
- Verified live: preview accepts such a section and returns correct HTML.

## 14. Editable numbered point

```jsx
<Template>
  <Block innerSpacing={{ top: 20, bottom: 20, left: 24, right: 24, mobile: { top: 16, bottom: 16, left: 16, right: 16 } }}>
    <FlexRow>
      <Column size={12}>
        <BulletList
          iconTextGap={{ size: 12, mobile: { size: 12 } }}
          bulletIcon={{ type: "custom", url: "https://cdn.example.com/one.png", fileName: "one.png", size: 40 }}
        >
          <BulletItem>Gentle morning movement instead of intense workouts</BulletItem>
        </BulletList>
      </Column>
    </FlexRow>
  </Block>
</Template>
```

**Why this way:**

- `bulletIcon.type="custom"` + `size={40}` creates a separate editor marker;
  the stored template and live HTML confirm the actual width of 40px.
- Do not replace the marker with a styled `<span>` inside `<Text>` or `<Html>`: they render, but
  do not give the user a separate editable element.
- One BulletList uses one icon for all lines. For different digits create
  a separate BulletList per point or use a separate fixed-size `<Image>` and `<Text>`.
- The URL is an example: before generating, get a confirmed asset via Ops.
- A custom marker has one `size` for desktop/mobile, so before saving visually
  check the point on mobile too; if a separate mobile size is needed, use a standalone
  fixed-size Image with `size.mobile.width`.
- The benchmark for a “verified mobile”: the marker stays 40px, the text does not slide under it,
  `iconTextGap.mobile` matches desktop. If the marker overlaps the text, reduce
  `size` or switch to a standalone Image.
