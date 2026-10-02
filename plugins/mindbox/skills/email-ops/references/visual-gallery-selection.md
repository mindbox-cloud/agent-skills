# Visual image selection from the gallery

**When to read:** the user wants to look at images visually, choose from
several similar options, or the text list of candidates is not enough.
**Return:** after a number is chosen — to the "Images: confirmation procedure" section of
`SKILL.md`; pass Generator `{url, fileName}` (formula at the bottom of the file).

The text list remains the quick default answer; make a visual contact sheet
at the user's request or when there are ambiguous similar options.

1. First get the candidates via MCP `gallery_images_list`:

   ```json
   {
     "nameSubstring": "<the user's query>",
     "includeSystemImages": true,
     "limit": 12
   }
   ```

   `fileExtensions` is not needed: the tool itself returns only formats for emails. If the user explicitly asks only for project assets, pass
   `includeSystemImages: false`. If there are too many results — ask to refine
   the query or use cursor pagination; do not build a contact sheet of hundreds of images.
   8–20 cards are usually enough for a visual sheet.

2. Selection invariants:

   - the email always gets the original `url` from `gallery_images_list`, exactly as
     MCP printed it; for an image stored in WebP, this is an imgproxy link to its
     PNG copy, not to the original S3 object — this is normal;
   - rely on the actual columns of the `gallery_images_list` response: `name`,
     `fileExtension`, `isSystem`, `url`; size/date may be missing;
   - `fileName` is built as `name + fileExtension`, if `name` does not already
     end with that extension;
   - the user chooses the card number; do not choose among similar options silently;
   - a `data:` URL is forbidden in email JSX.

## Codex / local agent

Codex always uses a simple local HTML contact sheet with remote links to
the images.

1. Create `<projectRoot>/gallery-preview.html`.
2. Insert cards with `<img src="<url from MCP>" loading="lazy">`.
3. Do not download the images and do not convert them to base64.
4. Give the user the path to the HTML and ask them to open the file in an external browser.
5. After they look, ask: "Which image number should I use?"

The images are loaded by the user's browser from the gallery links. The agent does not read the image
bytes and does not pass them to the model.

## Cowork

The Cowork side panel does not render remote images: `create_artifact`, `show_widget`,
markdown images and remote `<img src="https://...">` do not work there — the sandbox/CSP
blocks external image domains. So the contact sheet opens in an external browser.

1. Create `outputs/gallery-preview.html` with remote `<img src="<url from MCP>">`.
2. Show the file via `mcp__cowork__present_files`.
3. Say: "Open the file in an external browser — the images will load from the gallery."
4. Do not expect the images to render in the Cowork side panel.
5. After they look, ask: "Which image number should I use?"

## Remote contact sheet card template

```html
<article class="card">
  <a class="thumb" href="<url>" target="_blank" rel="noreferrer">
    <img src="<url>" alt="<fileName>" loading="lazy" decoding="async" referrerpolicy="no-referrer">
  </a>
  <div class="name"><span class="num">#N</span> <fileName></div>
  <div class="source">system icon / project asset</div>
  <a class="open" href="<url>" target="_blank" rel="noreferrer">open original</a>
</article>
```

## After the choice

When the user has chosen a number, return to Generator:

```json
{
  "url": "<the original url from gallery_images_list>",
  "fileName": "<name + fileExtension>"
}
```

If `name` already ends with `fileExtension`, do not duplicate the extension.
