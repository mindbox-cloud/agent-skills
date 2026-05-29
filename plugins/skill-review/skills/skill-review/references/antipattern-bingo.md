# Antipattern Bingo

> After the main review, the orchestrator **must** fill in a separate report section **"Antipattern Bingo"**.

---

## Verdict Rules

For **each** antipattern, choose one of four verdicts:

- **NONE** — the antipattern was **NOT found** in the skill (= good, clean)
- **MINOR** — **partial signs** of the antipattern (= there is a problem, but not complete); **mandatory** short note of **3–4 words**
- **CRITICAL** — the antipattern was **FOUND** in full (= bad, requires fixing)
- **NOT_CHECKED** — the antipattern belongs to a **stage above the selected scope**

> **Note:** CRITICAL = we **found** the antipattern. This is not a severity grade for impact — it is a fact of detection.

---

## Scope Fill Rule

- If scope = up to 2 → antipatterns of stages 3 and 4 get status `NOT_CHECKED`
- If scope = up to 3 → antipatterns of stage 4 get status `NOT_CHECKED`
- If scope = full → Bingo is filled out completely

`NOT_CHECKED` is needed so that the report does not create a false impression of cleanliness. Higher-stage antipatterns must not silently disappear and must not be counted as `NONE`.

---

## Evidence Rule

- Evaluate Bingo **only by observable artifacts** in the files of the skill under review.
- For `MINOR`, the note must be short, for example: `triggers too generic`, `path hardcoded`, `reference without map`.

---

## Antipattern Reference

> Principle: the checklist is the source of truth for checks. Bingo is a **reference table** pointing to checks, without duplicating descriptions.

| # | Antipattern | Stage | Check Reference | Owner |
|:--|:---|:---|:---|:---|
| 1 | **Huge MD** | 2 | ST09, ST13 | **Structure** |
| 2 | **Vague triggers / Triggering lottery** | 3 | ST06, ST07 | **Structure** |
| 3 | **No negative triggers** | 3 | RF01 | **References** |
| 4 | **No examples** | 2 | WF03 | **Workflow** |
| 5 | **Abstract instructions** | 2 | WF01 | **Workflow** |
| 6 | **Buried critical rules** | 2 | WF14 | **Workflow** |
| 7 | **No error handling** | 2 | WF04 | **Workflow** |
| 8 | **Content duplication** | 3 | RF05 | **References** |
| 9 | **AI generated skill** | 2 | ST11 | **Structure** |
| 10 | **Vibe-coded knowledge core** | 2 | ST12 | **Structure** |
| 11 | **Skill-prompt** | 2 | ST10 | **Structure** |
| 12 | **Hardcoded paths** | 3 | WF18 | **Workflow** |
| 13 | **Mirroring MCP schema** | 2 | WF22 | **Workflow** |
| 14 | **NL-code confusion** | 2 | WF02 | **Workflow** |
| 15 | **Schema drift risk** | 4 | WF22, LC02 | **Lifecycle** |
| 16 | **Context overfitting** | 3 | WF15 | **Workflow** |
| 17 | **Overengineering** | 2 | ST16 | **Structure** |
| 18 | **Lifecycle hygiene gap (rot risk)** | 4 | LC03 | **Lifecycle** |
| 19 | **Silent chain failures** | 2 | WF11 | **Workflow** |
| 20 | **Monolithic reference dump** | 3 | RF06, RF12 | **References** |

---

## Notes on Specific Antipatterns

- **#15 Schema drift risk** vs **#13 Mirroring MCP schema**: both use WF22, but differently. #13 is a structural fact (a schema copy lives in SKILL.md). #15 is a lifecycle risk (the schema version is not pinned, drift is not tracked).
- **#16 Context overfitting** vs **#12 Hardcoded paths**: #12 is a concrete mechanical signal (absolute paths). #16 is a broader pattern (coupling to OS, permissions, environment, implicit requirements). If the only signal is hardcoded paths, do not duplicate the verdict.
- **#16 in novice mode:** checked **partially** — only by mechanical portability signals (WF15: preconditions, OS, permissions, packages). Deep overfitting analysis (IN09–IN14: hidden assumptions, self-sufficiency, implicit knowledge) is only available in skill-review-nightmare with a full Intern walkthrough.

---

## Report Table Template

| # | Antipattern | Stage | Verdict | Note |
|:--|:---|:---|:---|:---|
| 1 | Huge MD | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 2 | Vague triggers / Triggering lottery | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 3 | No negative triggers | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 4 | No examples | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 5 | Abstract instructions | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 6 | Buried critical rules | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 7 | No error handling | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 8 | Content duplication | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 9 | AI generated skill | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 10 | Vibe-coded knowledge core | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 11 | Skill-prompt | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 12 | Hardcoded paths | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 13 | Mirroring MCP schema | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 14 | NL-code confusion | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 15 | Schema drift risk | 4 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 16 | Context overfitting | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 17 | Overengineering | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 18 | Lifecycle hygiene gap (rot risk) | 4 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 19 | Silent chain failures | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 20 | Monolithic reference dump | 3 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
