# Checklist: Link Integrity

> Sub-agent Links. Checks the mechanical integrity of links within a single skill: file refs, markdown anchors, tables of contents, reachability, external URLs, stable cross-refs.
>
> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

---

## LK01, LK05. Basic File Link Integrity

> **Stage: 2** | Axis: personal use

**LK01. File references.** Do all file mentions (`references/foo.md`, `scripts/bar.py`) point to files that actually exist in the skill folder?

*What we look for:* Broken file refs. Check every file path mentioned in `SKILL.md` and reference files — does the file exist at the stated path.

**LK05. Reachability and orphan files.** Are all required files actually reachable from `SKILL.md` via actual links? No orphan files or broken transitions?

*What we look for:* Mechanical integrity of the link graph. Build the graph from actual links and verify that all mentioned files resolve, and that expected files are not dangling without incoming links. This is a mechanical reachability check, not a policy assessment of topology; policy lives in RF08/RF09 in checklist-references.md.

---

## LK02–LK04, LK06–LK07. Navigation, Anchors, and Stable Cross-refs

> **Stage: 3** | Axis: team use

**LK02. Markdown anchors.** Do all internal links in the format `[text](#section-name)` point to a real heading in the same file?

*What we look for:* Broken internal anchors. Collect all `#anchor` links and verify that each has a corresponding heading in the same file.

**LK03. Cross-file anchors.** Do all links in the format `[text](file.md#section)` point to a real heading in a real file?

*What we look for:* Broken cross-file anchors. Check both the file's existence and the heading's existence within it.

**LK04. Table of contents (TOC).** If a file contains a TOC, do all its entries correspond to actual headings?

*What we look for:* Broken TOC. Compare the TOC entries with the actual headings in the file.

**LK06. External URLs.** If the skill contains http/https links — are they accessible? (Optional, if network access is available.)

*What we look for:* Dead external links. Try to check the availability of each external URL. If network access is unavailable, mark `N/A`.

**LK07. Cross-reference stability.** Do cross-references between skill files use headings/anchors (`file.md#section-name`) rather than line numbers (`file.md:42`)?

*Context:* Line numbers break on any edit. Headings and anchors are tied to content. Line numbers are only acceptable in one-off artifacts (review logs, reports). In internal cross-refs within a skill — headings only.
