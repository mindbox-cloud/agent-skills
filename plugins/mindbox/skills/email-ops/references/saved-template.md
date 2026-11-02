# Saved templates

**When to read:** the user asks for an email "by template X", "from my templates", to see their saved templates,
or to change, rename, create or copy a saved template.
**Return:** to the canonical workflow in `SKILL.md` — §3 for edits; for an email by a template also §1–§2 and §4–§5.

**Choosing the template** — `client_visual_template_list`. A template named in words with no exact match by
name, or several — show the candidates and let the user choose. No look-alike.

## An email by a saved template

`visual_template_get` gives a saved template's blocks as JSX. Its width, background, CSS, shared styles and
product deduplication are not in it and go separately — `settingsSource`, the saved template. Without it the
email gets the template's blocks but not its look.

1. **Target.** §1–§2 of the canonical workflow; a new campaign — "New campaign from scratch" first.
2. **Reading.** `visual_template_get` with `template` — the saved template.
3. **Shared styles**, when the generator needs them — `visual_template_theme_get` with `template` — the saved
   template — and `jsx`: after the save the email has the template's styles, not the campaign's.
4. **Confirmation** — §4, with the shared-styles line "the email's width, background, CSS and shared styles are
   taken from template <name>", plus "and changed by this document" if it has `<Theme>`.
5. **Save** — §5 with `settingsSource` — the saved template, in this one save only. The next edits are saved
   without it: the campaign already holds the settings, and passing it again overwrites what the user has
   changed in the editor since.

## Changing, creating, copying a saved template

1. **Changing the content** — `visual_template_get` with `template` — the saved template; edits and the preview —
   §3 and the preview rules, the saved template standing wherever they name the campaign's template; the write —
   `client_visual_template_edit`.
2. **A copy** — "save this email as a template", "a template based on X": `client_visual_template_create` with
   `content` `fromExisting`, `source` — the campaign's template or X. The editor copies what is saved, settings
   included; its JSX is not read. A campaign — `campaign_get` first; several A/B variants, or two versions listed
   (active and draft) — ask which; any campaign state reads. Changes to the content — one confirmation before the
   copy, naming the copy and the change, then item 1 on the copy. The email in this conversation has changes not
   saved to the campaign — not a copy: item 3 with that JSX and `settingsSource` — the campaign's template.
3. **From JSX** — `client_visual_template_create` with `content` `new` and the generator's JSX; `settingsSource` —
   the template whose look the user names, or the campaign's template of item 2. Without one the template gets
   the platform defaults.
4. **The name** of a new template or a copy is the user's: ask for it if the request does not give one.
5. **Confirmation** before writing JSX — `client_visual_template_edit` with `jsx` and
   `client_visual_template_create` with `content` `new`: the template's name, the change, the preview status and
   the shared-styles line — for a change, the line of §4; for a new template, "the template's width, background,
   CSS and shared styles are taken from <template>" or "… are the platform defaults", plus "and changed by this
   document" if it has `<Theme>`. A rename and a copy are done on the request.
