# Skill Review Novice — Report Template

> The orchestrator uses this template to generate the final report after collecting condensed summaries from the launched sub-agents.

---

## Report Language Rules

Write findings in human-readable form. The language must be understandable to a product owner, analyst, or manager:
- **FAIL / WARNING:** `[ID] — [human-readable description]` (ID is needed for traceability, but the description is primary)
- **PASS:** descriptions only without IDs, comma-separated: `Folder in kebab-case, instructions in imperative, numbered steps`
- **N/A:** grouped by reason, without IDs: `No markdown anchors — 3 checks not applicable`

---

## Evidence and Reference Rules

- **Primary anchor:** `file.md § Section Name` — a stable reference to the skill artifact.
- **Temporary marker:** `line ~N` — an auxiliary hint for the current review.
- **Review artifact reference:** `log-subagent.md#ID` — traceability in logs (if logging is enabled).

In findings, rely on `file § section`. Line numbers — only as an optional hint.

---

## Template

```markdown
# Skill Review: [skill-name]

## Overall Assessment

| Parameter | Value |
|---|---|
| **Review level** | Standard (Novice) |
| **Review mode** | [Single-pass / Sub-agent] |
| **Declared target stage** | [2 / 3 / 4 / not specified] |
| **Review scope** | [up to 2 / up to 3 / full] |

---

## Review Limitations

[Optional section. Show only if there were coverage limitations: unreadable files, sub-agent failure, partially restricted scope.]

> - [What went wrong]
> - [How it affected the completeness of the review]
> - [What was done as fallback]

---

## Summary Statistics

| FAIL | WARNING | PASS | N/A |
|---|---|---|---|
| [N] | [N] | [N] | [N] |

**Review scope:** [up to 2 / up to 3 / full]. [If scope < full: "Stage [3, 4 / 4] checks were not performed."]

---

## Stage 2 Issues — Personal Use

[If checked and there are findings:]

### [ID] — [Human-readable title]

**Problem:** [What exactly is wrong — specifically, in plain language]

**Where:** [file § section]

**Why it matters:** [Consequences — understandable for a non-specialist]

**Recommendation:** [Concrete action: what to do, where]

[If logging is enabled:] **Details:** `log-[subagent].md#[ID]`

[If checked and no issues:]
> All stage 2 checks passed.

---

## Stage 3 Issues — Team Use

[If checked and there are findings — same format]

[If checked and no issues:]
> All stage 3 checks passed.

[If not checked:]
> Stage 3 was not checked for the selected scope.

---

## Stage 4 Issues — Repository

[Same as stage 3]

---

## Not Applicable Checks (N/A)

[Group related checks on one line with a shared reason. No IDs.]

Example format:
> - No markdown anchors or TOC — 3 checks not applicable
> - No external URLs in the skill — 1 check not applicable
> - Skill does not use sub-agents — 3 checks not applicable

---

## Antipattern Bingo

[Insert the filled-in table from references/antipattern-bingo.md]

**Total:** [N] CRITICAL, [N] MINOR, [N] NONE, [N] NOT_CHECKED.

## Top 3 Recommendations

[Each recommendation has a bold title, a concrete action, and an expected effect.]

**1. [Action title]**
[What exactly to do. Where. What result.]

**2. [Action title]**
[What exactly to do. Where. What result.]

**3. [Action title]**
[What exactly to do. Where. What result.]

---

## Summary from Exhausted Vitaly

> [3–5 sentences. Tone: sentimental, mildly sarcastic but not rude — like the melancholy robot Marvin from The Hitchhiker's Guide to the Galaxy by Douglas Adams. Exhausted Vitaly comments on findings in the context of the declared goal (Personal / Team / Repository / "I don't know"). No label "stage N". Points to the main pain and predicts what will improve if the top issues are fixed.]
>
> Example (Personal scope, few issues): *"Well, it works. I've seen worse — mostly from optimists with access to a keyboard. Fix the missing examples and at least your future self won't have to guess what 'valid input' means."*
>
> Example (Team scope, several issues): *"Your colleagues will probably manage to run this. Probably. The lack of preconditions means the first person on a different OS gets to discover your hidden assumptions. I envy them the adventure."*

---

## Passed Checks (PASS)

[Compact paragraph. List **without IDs**, descriptions only, comma-separated.]

Example format:
> Folder in kebab-case, SKILL.md correctly named, instructions in imperative, numbered steps, checkpoints on each step, negative triggers present, all file references valid, no orphan files.

---

## Review Complete

[Short summary after the report — 3–4 lines for quick scanning:]

**Skill [strong/average/weak]** — [one sentence: main strength].

**Main pain:** [one sentence].

**Total:** FAIL: [N], WARNING: [N].

**Next steps by priority:**
1. [Action 1 — most important]
2. [Action 2]
3. [Action 3]
```

---

## Report Formation Rules

- Do not invent confirmations of things not present in the files.
- Do not inflate severity: team/workspace-level patterns should not automatically become FAIL for an isolated skill without evidence.
- The external report must be professional.
- Review goal: understand whether the skill works in the **declared or checked context**.
- Each FAIL and WARNING must contain a reference to the detailed sub-agent log (`log-[subagent].md#[ID]`), **if logging is enabled and mode is sub-agent**. In single-pass mode, sub-agent log references are not created. If logging is off — the report is self-contained.
- In the PASS section — a compact paragraph without IDs, descriptions comma-separated.
- In the N/A section — group related checks without IDs.
- If the review had limitations (unreadable files, sub-agent failure) — the `Review Limitations` section is mandatory.
- Top 3 — concrete actions, not abstractions.
- In "Review Complete" — a summary for those who will not read the full report.
- **Grouping findings by stage:** FAIL and WARNING are distributed across stage 2/3/4 sections. Each section has three states: (1) checked, has findings; (2) checked, no issues; (3) not checked by scope.
- **Evidence:** primary anchor — `file § section`. Line numbers — only an auxiliary hint.
