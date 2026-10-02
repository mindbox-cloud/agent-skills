# Campaign recipients: segment or filter

**When to read:** the user asks to set or change a campaign's recipients, to send it
"to a segment", "to a filter" or "to those who …".
**Return:** to the user with the write result; the live launch stays in the UI.

`campaign_edit` sets the recipients of a bulk campaign as a segment or a filter. A one-off
file and the "exclude inactive/new" toggles are UI only.

## Workflow

1. `campaign_get`. For `Automatic`, the audience is set by the flow — say so and do not call
   `campaign_edit`. Remember the current recipients from the `Recipients` block.
2. Choose the method by the request:
   - **An existing segment by name** → `segments_list`. Several candidates or nothing —
     show what was found by name and let the user choose; do not ask the person for an id.
   - **The audience is described by conditions, or a segment plus conditions** → the filter-building skill (`filter-build`)
     with `root: User`, `filterablePropertySet: Default` and `includeFilterJson: true`. The result
     is usable only with `status: ready` and `platform: accepted`. Pass `filterJson` from the
     `filter_compile_preview` response into `recipientsFilterJson` as is — without edits and without
     re-escaping. If the filter-building skill is not in your skill listing, do not compose
     the filter yourself: say that recipients by conditions are unavailable and offer a segment.
3. Confirmation before the write is always separate, even with an explicit request. Name
   the campaign, the method and the audience in words — the segment name or the filter conditions — and
   say that the previous recipients, if any, will be replaced.
4. `campaign_edit`, then check the `Recipients` block of the response and say that launching the campaign is done in the UI.
