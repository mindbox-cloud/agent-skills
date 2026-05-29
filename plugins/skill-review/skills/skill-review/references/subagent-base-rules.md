# Sub-agent Base Rules

> This file is mandatory for any sub-agent review. The sub-agent reads it after its own checklist and before starting checks.

- Evaluate **only by observable artifacts** — do not infer what is not present in the files.
- Do not count as PASS any runs, stability, portability, or lifecycle maturity unless confirmed by files, scripts, examples, tests, or explicit instructions.
- If a section is not applicable (no `references/`, `scripts/`, MCP, sub-agents) — mark **N/A**, not FAIL.
- Do not inflate severity: team/workspace-level patterns should not automatically become FAIL for an isolated skill without evidence.
- Each issue must include a concrete recommendation: **what to fix and where**.
- **Justify before verdict:** for each check, state the file and section you rely on, and give a 1–2 sentence rationale. A full step-by-step breakdown with quotes is not required.
- **Language:** write logs, summaries, and all conclusions in the same language the user started the conversation in.
- If an instruction relies on hidden author knowledge (`"this is obvious"`), explicitly mark it as a **hidden assumption** and do not consider the step self-sufficient.
- If a step depends on OS, shell, package manager, runtime version, permissions, working directory, or project structure — separately flag as a **portability risk**.
- **Review scope:** check only IDs that belong to the current scope. Do not evaluate or mark as N/A any IDs above the scope. Do not soften severity within the checked scope.
- **Evidence format:** primary anchor is `file.md § Section Name`. Line numbers are only acceptable as a temporary auxiliary hint (`line ~N`).
- If a file cannot be read, appears as garbled text, or causes a decode/encoding problem — do not infer its content. Explicitly mark the artifact as unreadable and continue the review with available files.
- If you sense context overflow during the review — first narrow reading to relevant files and sections, then return only a condensed summary. Do not pull long excerpts into the main thread.
- If logging is enabled: first try to write `log-{subagent}.md` directly to the output folder, without requesting additional permissions from the user.
- If direct write fails — save the log to a temporary file and return `temp_log_path` to the orchestrator.
- If the temporary log also cannot be written — notify the orchestrator.
