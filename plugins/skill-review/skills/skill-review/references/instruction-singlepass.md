# Skill Review — Single-pass Mode

> **Activation context:** This mode was activated by the orchestrator because the total volume of the skill under review is < 500 lines. All checks are executed in one context without sub-agents. The set of checks, report format, and Bingo are **identical** to sub-agent mode.

---

## Preconditions

> This mode assumes the orchestrator has already run the preflight of the parent skill.

- `TodoWrite` and file access are expected to be available.
- If the parent orchestrator enabled logging, file write is additionally required.
- If any of these capabilities is unexpectedly unavailable, do not start the review and immediately notify the user that single-pass cannot start in the current client.

---

## Mandatory Rules

- **No questions to the user.** Review parameters (scope, declared target, logging) were already determined by the orchestrator in Step 0.
- Create a **TODO plan** for the review: one TODO item per 3–4 checks.
- Evaluate **only by observable artifacts** — do not infer what is not present in the files.
- Do not count as PASS any runs, stability, or lifecycle maturity without file confirmation.
- If a section is not applicable (no `references/`, `scripts/`, MCP, sub-agents) — mark **N/A**, not FAIL.
- Do not inflate severity: team/workspace-level patterns do not become FAIL for an isolated skill without evidence.
- Each issue — with a concrete recommendation: **what to fix and where**.
- **Justify before verdict:** the file and section you rely on, 1–2 sentences.
- **Evidence:** primary anchor — `file.md § Section Name`. Line numbers — only an auxiliary hint (`line ~N`).
- **Scope:** check only IDs belonging to the current scope. Do not evaluate or mark as N/A any IDs above the scope.
- Do not soften severity within the checked scope.
- If an instruction relies on hidden author knowledge — mark as **hidden assumption**.
- If a step depends on OS, shell, runtime version, permissions, working directory — flag as **portability risk**.
- Any abnormal situation (unreadable files, invalid artifact) must not silently abort the review. See `## Troubleshooting` for details.
- **Language:** run the entire review in the user's language.

---

## Scope and Check Matrix

| Scope | Structure | Workflow | References | Links | Lifecycle |
|:---|:---|:---|:---|:---|:---|
| **up to 2** | ST01–ST16 | WF01–WF14, WF19–WF29 | RF04–RF12 | LK01, LK05 | — |
| **up to 3** | ST01–ST16 | WF01–WF29 | RF01, RF03–RF15 | LK01–LK07 | — |
| **full** | ST01–ST16 | WF01–WF29 | RF01–RF15 | LK01–LK07 | LC01–LC05 |

Checks outside scope are marked **"not checked for selected scope"** (not N/A and not FAIL).

---

## Algorithm

```text
1. Create the TODO plan for the review (compact, no sub-agents)
2. Sequentially run all checks by scope:
   Part A → Part B → Part C → Part D → [Part E if scope is full]
3. Fill Antipattern Bingo (Part F)
4. Generate the final report (Part G)
5. If logging ON — write report.md; if logging OFF — do not write any files
```

---

## Troubleshooting

> **General principle:** an abnormal situation must not silently abort the review. On file read problems — produce the most complete report possible from available artifacts and add a `Review Limitations` section. On file write problems — notify the user and stop the review.

### 1. An invalid skill artifact was passed

**Symptoms:** no `SKILL.md`, folder is empty, or an arbitrary set of files was passed instead of a skill.
**Cause:** the input artifact is not a skill folder.
**Resolution:** flag this as the primary critical issue, perform a residual review from available structure and readable files, and tell the user what is missing.

### 2. Skill files cannot be read or are damaged

**Symptoms:** read error, mojibake, garbage instead of text, truncation in a critical section, unsupported encoding — the meaning of the text cannot be reliably recovered.
**Cause:** corrupted encoding, binary file, access problems, or damaged artifact.
**Resolution:**
1. Retry reading **once**.
2. If the retry fails — consider the file unreadable.
3. If `SKILL.md` cannot be read — this is the primary critical issue; perform a residual review from the folder structure and readable files.
4. If a reference file cannot be read — do not infer its content; continue the review with other artifacts.
5. The unreadable file must appear in `Review Limitations`.

### 3. File write is unavailable

**Symptoms:** `report.md` cannot be written with logging ON.
**Cause:** no write permissions, read-only filesystem, environment restrictions.
**Resolution:**
1. Retry the write **once**.
2. If write is still unavailable — **notify the user and stop the review**.
3. Tell the user: which files could not be written, suggest checking permissions or switching to a different working directory.

---

## Part A — Structure and Form (ST01–ST16)

> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.
> All checks — **stage 2**.

### A1. Structure and Validity (ST01–ST04)

**ST01.** Is the folder in `kebab-case`? (lowercase a–z, 0–9, hyphens; no spaces, underscores, CamelCase)

*Context:* The Agent Skills Specification requires kebab-case for the folder name. Other formats (`Notion Project Setup`, `notion_setup`, `NotionSetup`) are not picked up by many clients or cause validation errors.

**ST02.** Is the file named **exactly** `SKILL.md`? (not `skill.md`, not `SKILL.MD`)

*Context:* The name is case-sensitive. Agents scan directories for exactly `SKILL.md` — other variants will not be discovered.

**ST03.** No `README.md` inside the skill folder?

*Context:* This is a structural folder-hygiene check for an isolated skill: there should be no extra human-facing markdown inside the folder that creates noise during discovery or overrides the role of `SKILL.md`. See also `LC05`: that check covers the repository-level boundary — that a README as a human artifact lives at the repository level, not inside the skill folder.

**ST04.** If `references/`, `scripts/`, `assets/` exist — used correctly? (references = documentation, scripts = code, assets = templates)

*Context:* Correct distribution across directories is the foundation of progressive disclosure. The agent loads files from these directories only when needed, saving up to 85–95% of tokens compared to flat loading.

### A2. YAML Frontmatter (ST05–ST08)

**ST05.** Is there a `name` field in kebab-case that matches the folder name?

*Context:* The specification requires: name = 1–64 characters, lowercase alphanumeric + hyphens, matches the parent directory. A mismatch causes validation errors.

**ST06.** Is there a `description` field with the formula: **WHAT it does** + **WHEN to use** (trigger phrases)?

*Context:* The description is the most critical part of a skill. The router uses it to decide whether to activate the skill. A vague description (`Helps with projects`) means the skill never fires or fires incorrectly.

**ST07.** Does the description contain **concrete trigger phrases**, not abstractions?

*Context:* Good: `Use when user says "plan sprint", "create tasks", "set up project"`. Bad: `Creates sophisticated multi-page documentation systems`. Concrete phrases allow the router to accurately match user requests to the description.

**ST08.** No XML tags (`<`, `>`) anywhere in the YAML?

*Context:* XML tags in YAML frontmatter create a prompt injection risk. The specification explicitly prohibits their use.

### A3. Size (ST09)

**ST09.** Is `SKILL.md` < 5000 words / 500 lines?

*Context:* The specification recommends keeping `SKILL.md` under 500 lines and 5000 tokens. If exceeded — immediate decomposition: details to `references/`, scripts to `scripts/`. A bloated `SKILL.md` causes context overload and quality degradation.

### A4. Basic Antipatterns (ST10–ST13)

**ST10. Skill-prompt.** Is the skill simply a long prompt without a clear (1) role, (2) constraints, (3) output format?

*Context:* Skill = role + constraints + output format. Without these three components — it is just a prompt with a name. Early skill users copied long prompts into `SKILL.md` and got `maybe slightly better than before`.

**ST11. Self-generated skill.** Does the artifact appear to have been generated by a single command without traces of human verification or domain adaptation?

*Context:* Self-generated skills provide ~0 pp improvement per SkillsBench data. The problem is not the writing style — it is the absence of verified human expertise: the model paraphrases what it already "knows".

**ST12. Vibe-coded knowledge core.** Is the methodology written "from model memory" (generic phrases, abstract best practices) rather than a concrete, verifiable procedure?

*Context:* Knowledge in a skill should be a procedure for obtaining and verifying data, not an "encyclopedia from model memory". Even an expertly written skill degrades if the knowledge core remains vague and unverifiable.

**ST13. Monolith.** Is everything dumped into one `SKILL.md`: workflow, references, examples, API schemas?

*Context:* Everything that is not the core workflow should live in `references/` or `scripts/`. Otherwise — context overload: the agent drowns in details and loses focus on the main process.

### A5. Justification (ST14–ST16)

**ST14.** One coherent workflow, not a set of unrelated commands?

*Context:* One skill = one coherent action. If `SKILL.md` describes 3 independent operations — those are 3 skills, not 1. Atomicity simplifies triggering, testing, and composability.

**ST15.** Is the instruction > 300 tokens?

*Context:* Very short instructions (< 300 tokens) that are needed in every conversation are better placed in `CLAUDE.md` or a system prompt — the overhead of creating a skill will not pay off.

**ST16.** Is the artifact justified as a skill? (3+ steps, a domain procedure, a repeatable workflow, non-trivial decision points)

*Context:* Not every artifact should become a skill. If there is no multi-step process, domain methodology, or repeatable routine inside, this may be overengineering. An extra skill increases routing overhead and context noise with no real benefit.

---

## Part B — Workflow (WF01–WF29)

> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

### B1. Basic `SKILL.md` Body Quality (WF01–WF04) — stage 2

**WF01.** Are instructions in **imperative** (`Run`, `Validate`, `Stop`, `Check`), not descriptively?

*Context:* Agents follow concrete commands better than abstract descriptions. `Validate the data before proceeding` — bad. `Run python scripts/validate.py --input {filename}` — good. Imperative reduces NL-code confusion.

**WF02.** Is the workflow formatted as **numbered steps**, not narrative paragraphs?

*Context:* Numbered steps reduce NL-code confusion — the situation where the agent confuses descriptive text and executable instructions. Rule: body is a procedure, not a wiki.

**WF03.** Are there at least 1–2 examples (input/output)?

*Context:* Without examples, output is unstable. Examples give the agent a pattern for matching and set expectations for format, volume, and style.

**WF04.** Is there a Troubleshooting section or error handling?

*Context:* Without error handling, the agent either silently continues on failure (silent chain failure) or stops without explanation. Minimum: top 3 typical errors with cause and resolution.

### B2. Six Mandatory Elements (WF05–WF10) — stage 2

**WF05** Trigger — when to apply
**WF06** Inputs — what is needed as input
**WF07** Steps — how to execute
**WF08** Checks — how to validate
**WF09** Stop conditions — when to stop
**WF10** Recovery — what to do on failure

*Context:* If even one of these elements is implicit, the skill behaves like a long prompt, not a workflow. This is one of the main criteria for the transition from stage 1 to stage 2.

### B3. Execution Safeguards (WF11–WF14) — stage 2

**WF11.** Does each workflow step have a **checkpoint** or explicit transition condition?

*Context:* Without checkpoints, the agent continues the workflow on a silent step failure — "silent chain failures". A good skill does not just list steps — it sets conditions: what must be true before proceeding.

**WF12.** If the workflow has > 4 steps — is there a **planning tool**, task list, or external planning artifact?

*Context:* In long sessions, the agent easily loses its plan. Anthropic recommends structured note-taking / agentic memory: an explicit task list that maintains state between tool calls.

**WF13.** If planning is used — is there an **enforcement gate**: completion is not allowed while there are `pending`/`in_progress` tasks?

*Context:* The most effective way to make planning mandatory is to prohibit completion with unclosed tasks. Otherwise the TODO list remains decorative and does not prevent context loss.

**WF14.** Are critical rules and prohibitions at the beginning of the skill or under `CRITICAL` headers, not buried in the middle?

*Context:* "Lost in the middle" is a well-known long-context problem: the model pays less attention to instructions in the middle of a file. The most important rules must be visible early and explicitly.

### B4. Preconditions / Postconditions / Boundaries (WF15–WF18) — stage 3

> Checked only when scope >= up to 3.

**WF15.** Are **preconditions** explicitly described? (OS, permissions, access, files, packages, environment limits)

*Context:* Without explicit preconditions, the skill is tied to the author's machine. When a colleague runs the skill on a different OS, with different permissions, or a different Python version — the skill silently breaks.

**Additional checks:**
- Is it stated where input files, tokens, directories, access, and environment variables come from?
- Are there no hidden requirements for shell, package manager, alias, or runtime version?
- Are there no dependencies on "magic" files, local state, or chat history?
- If the skill is claimed to be portable — is this confirmed by instructions, not silent assumption?

*Hint:* if without this a new person cannot continue a step — at minimum WARNING. If hidden author knowledge is required — FAIL.

**WF16.** Are **postconditions** explicitly described? (Artifacts created, checks passed, what "done" looks like)

*Context:* Without postconditions, there is no completion criterion. The agent does not know whether it has reached its goal.

**WF17.** Are **boundary conditions** explicitly described? (Where not to apply, where to hand off to a human or another skill)

*Context:* Without boundary conditions, the skill tries to do what it is not designed for.

**WF18.** Are there no **hardcoded paths**, user-specific directories, local drive names?

*Context:* Hardcoded paths are a classic portability antipattern. A skill may work on the author's machine and silently break for colleagues. Better to use input parameters, relative paths, and explicitly described preconditions.

### B5. MCP Steps (WF19–WF22) — stage 2 (if the skill uses MCP)

**WF19.** For MCP steps, are **capability, inputs, and expected outputs** described, not bare function names?

*Context:* By default, a skill should encode the workflow contract, not mirror the tool contract. A capability-bound formulation (`call the semantic search function`) is more stable than signature-bound (`call search_documents(query, top_k=5)`), because it does not break on MCP rename/refactor.

**WF20.** Are **negative selectors** specified — which similar but incorrect tools not to choose?

*Context:* When there are similar MCP functions, the agent may choose the wrong one. An explicit negative selector helps the router.

**WF21.** Are exact MCP names/parameters specified **only** for critical steps (safety, compliance, high cost of error)?

*Context:* Capability-bound by default; signature-bound is a conscious exception. If exact names are needed for critical steps, prefer a single reference file with the signatures. Inline signatures in `SKILL.md` are only acceptable as a localized exception.

**WF22.** Does the skill duplicate full JSON schemas / lists of parameters from the MCP contract?

*Context:* Mirroring MCP schema inside `SKILL.md` is an antipattern. The skill becomes brittle to any MCP server refactor. If signatures are pinned per WF21, they must be localized in one place. Signatures spread across multiple files — amplified FAIL signal.

### B6. Sub-agent Delegation (WF23–WF25) — stage 2 (if the skill uses sub-agents)

**WF23.** Are the main agent's skills passed to the sub-agent **explicitly**, not assuming automatic inheritance?

*Context:* Skills are not inherited by sub-agents automatically. Required skills must be passed explicitly in the invocation.

**WF24.** Does the sub-agent brief contain: **scope, files, expected output, constraints**?

*Context:* Vague invocations (`Implement the feature`) are an antipattern. Without a clear brief, the sub-agent cannot see the overall context.

**WF25.** Does the sub-agent return a **condensed summary**, not a full transcript?

*Context:* The sub-agent architecture is useful precisely because the sub-agent can spend tens of thousands of tokens on exploration, but return only a 1000–2000 token summary. Copying the full transcript defeats the purpose of delegation.

### B7. Handoff and Planning (WF26–WF27) — stage 2 (if applicable)

**WF26.** Do checkpoints cover **handoffs between tools / co-skills**? After each cross-system step, is it clear what artifact was received, in what format, and what constitutes a valid handoff?

*Context:* Checkpoints matter not only within a single skill (WF11), but also at handoffs between tools and co-skills. Task state is most often lost at these transitions.

**WF27.** If the workflow passes through several tools / co-skills — is there an **external planning artifact / TODO** that survives the handoff?

*Context:* An external planning artifact maintains dependent steps, execution status, and blockers that would otherwise dissolve into the thread history.

### B8. Context and Architecture (WF28–WF29) — stage 2 (if the workflow is long)

**WF28.** Are there explicit instructions for **context management** for long tasks?

**Applicability:** workflow >= 5 steps, OR large data, OR sub-agents, OR multi-session. Otherwise N/A.

*Context:* Long tasks suffer from "lost-in-the-middle" — the agent loses focus on early decisions. Manus (Jul 2025) addresses this with the filesystem as unlimited persistent memory: `todo.md` is rewritten after each step. Anthropic (Sep 2025) describes three techniques: compaction, structured note-taking, sub-agent architectures. StackOne (Jan 2026) documents 6 context overflow failure patterns.

| Signal | Verdict if absent |
|---|---|
| Saving intermediate results to files | WARNING (>= 5 steps) |
| Context-refresh: re-reading plan before each phase | WARNING |
| Strategy for context overflow | INFO (< 10 steps), WARNING (>= 10) |
| Checkpoints with state capture | WARNING (for critical phases) |

- **PASS:** explicit context management strategy is present
- **WARNING:** workflow is long (>= 5 steps), none of the signals above are present
- **N/A:** workflow is short and simple (< 5 steps, no large data)

**WF29.** Are there workflow phases that should be **extracted as sub-agents** but run inline?

**Applicability:** workflow with >= 3 distinct phases. Otherwise N/A.

*Context:* Anthropic (Sep 2025): `Specialized sub-agents can handle focused tasks with clean context windows. Each subagent might explore extensively, using tens of thousands of tokens, but returns only a condensed summary.` Bob Renze (Mar 2026) adds three criteria: isolation benefit, model specialization, restart tolerance. Practical conclusion: 2–4 sub-agents is usually optimal; beyond that, coordination overhead starts eating the benefit.

Three criteria (all simultaneously): (1) Independence, (2) Result-oriented, (3) Verifiability. If all are met AND the phase is NOT structured as a sub-agent → signal.

- **PASS:** all isolatable phases are already sub-agents, OR there are none
- **WARNING:** 1–2 isolatable phases running inline
- **FAIL:** 3+ isolatable phases + context overload signs (> 10 steps, large files)
- **N/A:** workflow < 3 phases

---

## Part C — References and Progressive Disclosure (RF01–RF15)

> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

### C1. Progressive Disclosure and `references/` (RF04–RF12) — stage 2 (if references/ exists)

**RF04.** Does the workflow (how to execute) live in `SKILL.md`, and the methodology/references (what to know) in `references/`?

*Context:* Mixing workflow and knowledge in one file increases NL-code confusion. `SKILL.md` owns "how to execute". `references/` owns "what to know to execute".

**RF05.** No duplication between `SKILL.md` and `references/`?

*Context:* Single Source of Truth. When they fall out of sync, the agent receives contradictory instructions.

**RF06.** Is each reference file focused on **one topic / scenario / class of problems**?

*Context:* A monolithic reference on thousands of lines is a "monolithic reference dump" antipattern. The agent is forced to read everything. `Keep individual reference files focused. Agents load these on demand, so smaller files mean less use of context.`

**RF07.** Does `SKILL.md` state **when** to read each reference file (explicit trigger)?

*Context:* `Read references/api-errors.md if API returns non-200` — correct. `See references/ for more details` — antipattern.

**RF08.** Is the primary navigation to reference files **directly from `SKILL.md`**, not only through reference→reference chains?

*Context:* `SKILL.md` must be the orchestration layer. Limited cross-refs are acceptable as secondary navigation, but not as the sole entry point.

**RF09.** If a reference links to another reference — is the link explicitly scoped? (explicit trigger, no cycles, no deeper than 1 hop from SKILL.md)

*Context:* Cross-references are only acceptable as a local clarification.

**RF10.** Do files > 100 lines start with a **table of contents**?

*Context:* A TOC lets the agent read only the needed section. By default treat missing TOC in a reference > 100 lines as **WARNING**. Use **FAIL** only if the file is effectively an unnavigable monolith.

**RF11.** Is content distributed approximately as L1 (~10%) : L2 (~30%) : L3 (~60%)?

*Context:* L1 (frontmatter) loads always. L2 (body) loads on activation. L3 (references, scripts, assets) loads on demand. If 90% of content is in the body — progressive disclosure is not working.

**RF12.** If there are many reference files — is there a **file map** with the purpose of each?

*Context:* Without a map, even good small files become an unnavigable collection.

### C2. Description Quality and Routing (RF01, RF03) — stage 3

> Checked only when scope >= up to 3.

**RF01.** Does the description contain "**When NOT to use**" (negative triggers)?

*Context:* Adding negative triggers reduces false activations by 40–60%. Format: `Don't use when: user asks for code review (use code-review skill)`.

**RF03.** Is the description not too broad, not provoking overtriggering?

*Context:* An overly generic description intercepts neighboring requests.

### C3. Additional Reference Checks (RF13–RF15) — stage 3

> Checked only when scope >= up to 3.

**RF13.** Are there no **dynamic knowledge items** in references/ (pricing, benchmarks, API versions)?

*Context:* Frequently changing data should not be stored as a static reference.

**RF14.** Are important **gotchas** that apply almost always in `SKILL.md`, not buried deep in references/?

*Context:* If the agent will almost certainly encounter a problem, surface it early in the main body.

**RF15.** Are deterministic operations extracted into `scripts/`, not only described in natural language?

*Context:* Code is deterministic; natural language interpretation is not.

### C4. Routing in the Ecosystem (RF02) — stage 4

> Checked only when scope = full.

**RF02.** Are specific **neighboring skills** to redirect to named?

*Context:* Negative triggers without an alternative are less effective than those with one (`Don't use for X — use Y-skill instead`).

---

## Part D — Link Integrity (LK01–LK07)

> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

### D1. Basic Integrity (LK01, LK05) — stage 2

**LK01.** Do all file mentions (`references/foo.md`, `scripts/bar.py`) point to files that actually exist?

*What we look for:* Broken file refs. Check every file path mentioned in `SKILL.md` and reference files.

**LK05.** Are all files reachable from `SKILL.md` via actual links? No orphan files?

*What we look for:* Mechanical integrity of the link graph. This is a mechanical reachability check, not a policy assessment of topology; policy lives in RF08/RF09.

### D2. Navigation, Anchors, and Cross-refs (LK02–LK04, LK06–LK07) — stage 3

> Checked only when scope >= up to 3.

**LK02.** Do all `[text](#section-name)` links point to a real heading in the same file?

*What we look for:* Broken internal anchors.

**LK03.** Do all `[text](file.md#section)` links point to a real heading in a real file?

*What we look for:* Broken cross-file anchors. Check both the file's existence and the heading's existence.

**LK04.** If there is a TOC — do all entries correspond to actual headings?

*What we look for:* Broken TOC.

**LK06.** If there are http/https links — are they accessible? (N/A if network access is unavailable)

*What we look for:* Dead external links.

**LK07.** Do cross-references between skill files use headings/anchors, not line numbers?

*Context:* Line numbers break on any edit. Line numbers are only acceptable in one-off artifacts (review logs, reports).

---

## Part E — Ownership and Lifecycle (LC01–LC05)

> Checked **only** when scope = full. All checks — **stage 4**.

### E1. Stage 4 Gate Markers (LC01–LC03)

> Mandatory markers for transitioning to stage 4. Without them, the skill is not a repository skill.

**LC01.** Is `metadata.author` (skill owner) specified?

*Context:* Orphaned skills without a maintainer are one of the most common problems. When the author leaves, the skill becomes a black box.

**LC02.** Is `metadata.version` in **semver format**?

*Context:* `metadata.version` (semver) fixes the environment contract. Without semver it is impossible to determine compatibility between different versions.

**LC03.** Is there a **CHANGELOG** or **version header with dates**?

*Context:* Update dates let you understand which version of the instruction is current. Acceptable: a version header in `SKILL.md` or a `CHANGELOG.md` next to `SKILL.md` or at the plugin root — but not inside `references/` (that is L3 content for the agent, see RF07, RF08, LK05).

### E2. Stage 4 Hygiene Checks (LC04–LC05)

> Not hard gate markers, but must appear in the report.

**LC04.** Are there no **tokens, keys, passwords, or secrets** in the skill files?

*Context:* Secrets in skill files are a direct security risk, especially when publishing to a shared repository.

**LC05.** Is the human-facing `README.md` at **repository level**, not inside the skill folder?

*Context:* Skills are designed for consumption by an AI agent. A README inside the skill folder is an antipattern.

---

## Part F — Antipattern Bingo

> **Mandatory** to fill in after the main checks.

### Verdicts

- **NONE** — the antipattern was **NOT found** (= clean)
- **MINOR** — **partial signs** (= a problem, but not complete); add a 3–4 word note
- **CRITICAL** — **FOUND** in full (= requires fixing)
- **NOT_CHECKED** — stage is above scope

> **CRITICAL** = we **found** the antipattern. This is not a severity grade for impact — it is a fact of detection.

### Scope Rule

- scope = up to 2 → stage 3 and 4 antipatterns = `NOT_CHECKED`
- scope = up to 3 → stage 4 antipatterns = `NOT_CHECKED`
- scope = full → Bingo filled completely

### Evidence Rule

Evaluate **only by observable artifacts**. For `MINOR`, the note is short: `triggers too generic`, `path hardcoded`, `reference without map`.

### Antipattern Reference

| # | Antipattern | Stage | Check Reference |
|:--|:---|:---|:---|
| 1 | **Huge MD** | 2 | ST09, ST13 |
| 2 | **Vague triggers / Triggering lottery** | 3 | ST06, ST07 |
| 3 | **No negative triggers** | 3 | RF01 |
| 4 | **No examples** | 2 | WF03 |
| 5 | **Abstract instructions** | 2 | WF01 |
| 6 | **Buried critical rules** | 2 | WF14 |
| 7 | **No error handling** | 2 | WF04 |
| 8 | **Content duplication** | 3 | RF05 |
| 9 | **AI generated skill** | 2 | ST11 |
| 10 | **Vibe-coded knowledge core** | 2 | ST12 |
| 11 | **Skill-prompt** | 2 | ST10 |
| 12 | **Hardcoded paths** | 3 | WF18 |
| 13 | **Mirroring MCP schema** | 2 | WF22 |
| 14 | **NL-code confusion** | 2 | WF02 |
| 15 | **Schema drift risk** | 4 | WF22, LC02 |
| 16 | **Context overfitting** | 3 | WF15 |
| 17 | **Overengineering** | 2 | ST16 |
| 18 | **Lifecycle hygiene gap (rot risk)** | 4 | LC03 |
| 19 | **Silent chain failures** | 2 | WF11 |
| 20 | **Monolithic reference dump** | 3 | RF06, RF12 |

### Notes

- **#15 vs #13:** both use WF22, but #13 is a structural fact (schema copy), #15 is a lifecycle risk (drift).
- **#16 vs #12:** #12 is mechanical (absolute paths), #16 is broader (coupling to OS, permissions, environment). Do not duplicate the verdict.
- **#16 in novice mode:** checked partially — only by mechanical portability signals (WF15: preconditions, OS, permissions).

### Bingo Table for Report

| # | Antipattern | Stage | Verdict | Note |
|:--|:---|:---|:---|:---|
| 1 | Huge MD | 2 | [NONE / MINOR / CRITICAL / NOT_CHECKED] | [if MINOR: 3–4 words, otherwise `—`] |
| 2 | Vague triggers / Triggering lottery | 3 | | |
| 3 | No negative triggers | 3 | | |
| 4 | No examples | 2 | | |
| 5 | Abstract instructions | 2 | | |
| 6 | Buried critical rules | 2 | | |
| 7 | No error handling | 2 | | |
| 8 | Content duplication | 3 | | |
| 9 | AI generated skill | 2 | | |
| 10 | Vibe-coded knowledge core | 2 | | |
| 11 | Skill-prompt | 2 | | |
| 12 | Hardcoded paths | 3 | | |
| 13 | Mirroring MCP schema | 2 | | |
| 14 | NL-code confusion | 2 | | |
| 15 | Schema drift risk | 4 | | |
| 16 | Context overfitting | 3 | | |
| 17 | Overengineering | 2 | | |
| 18 | Lifecycle hygiene gap (rot risk) | 4 | | |
| 19 | Silent chain failures | 2 | | |
| 20 | Monolithic reference dump | 3 | | |

---

## Part G — Report Format

### Language Rules

- **FAIL / WARNING:** `[ID] — [human-readable description]` (ID for traceability, description is primary)
- **PASS:** descriptions without IDs, comma-separated
- **N/A:** grouped by reason, without IDs

### Template

```markdown
# Skill Review: [skill-name]

## Overall Assessment

| Parameter | Value |
|---|---|
| **Review level** | Standard (Novice) |
| **Review mode** | Single-pass |
| **Declared target stage** | [2 / 3 / 4 / not specified] |
| **Review scope** | [up to 2 / up to 3 / full] |

---

## Review Limitations

[Optional section. Show only if there were execution degradations: unreadable files, log write failure, context overflow, partially restricted scope.]

> - [What went wrong]
> - [How it affected completeness]
> - [What was done as fallback]

---

## Summary Statistics

| FAIL | WARNING | PASS | N/A |
|---|---|---|---|
| [N] | [N] | [N] | [N] |

**Review scope:** [up to 2 / up to 3 / full]. [If scope < full: "Stage [3, 4 / 4] checks were not performed."]

---

## Stage 2 Issues — Personal Use

### [ID] — [Human-readable title]
**Problem:** [What is wrong — specifically]
**Where:** [file § section]
**Why it matters:** [Consequences — understandable for a non-specialist]
**Recommendation:** [Concrete action]

[If no issues:]
> All stage 2 checks passed.

---

## Stage 3 Issues — Team Use

[Same format. If not checked:]
> Stage 3 was not checked for the selected scope.

---

## Stage 4 Issues — Repository

[Same format.]

---

## Passed Checks (PASS)

[Compact paragraph without IDs, descriptions comma-separated.]

---

## Not Applicable Checks (N/A)

[Group related checks with a shared reason, without IDs.]

---

## Antipattern Bingo

[Filled-in table from Part F]

**Total:** [N] CRITICAL, [N] MINOR, [N] NONE, [N] NOT_CHECKED.

## Top 3 Recommendations

**1. [Action]** [What, where, result.]
**2. [Action]** [What, where, result.]
**3. [Action]** [What, where, result.]

---

## Summary from Exhausted Vitaly

> [3–5 sentences. Tone: Marvin from The Hitchhiker's Guide to the Galaxy — melancholic, precise, to the point.]
> Exhausted Vitaly comments on findings in the context of the declared goal (Personal / Team / Repository / "I don't know").
> No "stage N" label. Points to the main pain and predicts what will improve if the top issues are fixed.
> - Personal: "for solo use — [sufficient / lacking this]"
> - Team: "for the team — [main stage 3 findings are ...]"
> - Repository: "for the repository — [main stage 4 findings are ...]"
> - "I don't know": "if for yourself — [..]; if for a team — [..]; if for a repository — [..]"

---

## Review Complete

**Skill [strong/average/weak]** — [one sentence].
**Main pain:** [one sentence].
**Total:** FAIL: [N], WARNING: [N].
**Next steps:** 1. [...] 2. [...] 3. [...]
```

### Formation Rules

- Do not invent confirmations; do not inflate severity.
- If the review had limitations (unreadable files, invalid artifact) — the `Review Limitations` section is mandatory.
- Group findings by stages 2/3/4. Each section has three states: has findings / no issues / not checked.
- Evidence: `file § section`. Line numbers — only a hint.
- Top 3 — concrete actions, not abstractions.
- In "Review Complete" — a summary for quick scanning.
- If logging ON — write the full report to `report.md` in the folder confirmed by the user, and output a condensed summary to chat: counters, main pain, top 3, path to file.
- If logging OFF — do not write any files; output the full report to chat in its entirety.
