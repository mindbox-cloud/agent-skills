# Checklist: Workflow

> Sub-agent Workflow. Checks workflow quality: steps, checkpoints, safeguards, preconditions/postconditions, MCP, sub-agents.
>
> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

---

## WF01–WF04. `SKILL.md` Body — Basic Quality

> **Stage: 2** | Axis: personal use

**WF01** Are instructions written in **imperative** (`Run`, `Validate`, `Stop`, `Check`), not descriptively?

*Context:* Agents follow concrete commands better than abstract descriptions. `Validate the data before proceeding` — bad. `Run python scripts/validate.py --input {filename}` — good. Imperative reduces NL-code confusion.

**WF02** Is the workflow formatted as **numbered steps**, not narrative paragraphs?

*Context:* Numbered steps reduce NL-code confusion — the situation where the agent confuses descriptive text and executable instructions. Rule: body is a procedure, not a wiki.

**WF03** Are there at least 1–2 examples (input/output)?

*Context:* Without examples, output is unstable. Examples give the agent a pattern for matching and set expectations for format, volume, and style.

**WF04** Is there a Troubleshooting section or error handling?

*Context:* Without error handling, the agent either silently continues on failure (silent chain failure) or stops without explanation. Minimum: top 3 typical errors with cause and resolution.

---

## WF05–WF10. Six Mandatory Elements

> **Stage: 2** | Axis: personal use

**WF05 Trigger** — when to apply
**WF06 Inputs** — what is needed as input
**WF07 Steps** — how to execute
**WF08 Checks** — how to validate
**WF09 Stop conditions** — when to stop
**WF10 Recovery** — what to do on failure

*Context:* If even one of these elements is implicit, the skill behaves like a long prompt, not a workflow. This is one of the main criteria for the transition from stage 1 to stage 2.

---

## WF11–WF14. Execution Safeguards

> **Stage: 2** | Axis: personal use

**WF11** Does each workflow step have a **checkpoint** or explicit transition condition to the next step?

*Context:* Without checkpoints, the agent continues the workflow on a silent step failure — this is "silent chain failures". A good skill does not just list steps — it sets conditions: what must be true before proceeding.

**WF12** If the workflow is longer than **4 steps** — do the instructions explicitly declare a **planning tool**, task list, or external planning artifact?

*Context:* In long sessions, the agent easily loses its plan and starts jumping between tasks. Anthropic recommends structured note-taking / agentic memory: an explicit task list that maintains state between tool calls.

**WF13** If planning is used, is there an **enforcement gate**: completion is not allowed while there are `pending` or `in_progress` tasks?

*Context:* The most effective way to make planning mandatory is to prohibit completion with unclosed tasks. Otherwise the TODO list remains decorative and does not prevent context loss.

**WF14** Are critical rules, prohibitions, and stop conditions at the beginning of the skill or under explicit `CRITICAL` headers, not buried in the middle of a long text?

*Context:* "Lost in the middle" is a well-known long-context problem: the model pays less attention to instructions in the middle of a file. The most important rules must be visible early and explicitly.

---

## WF15–WF18. Preconditions / Postconditions / Boundaries

> **Stage: 3** | Axis: team use

**WF15** Are **preconditions** explicitly described? (OS, permissions, access, files, limits of the environment, required packages)

*Context:* Without explicit preconditions, the skill is tied to the author's machine and environment. When a colleague runs the skill on a different OS, with different permissions, or a different Python version — the skill silently breaks.

**Additional checks:**
- Is it stated where the input files, tokens, directories, access, and environment variables come from?
- Are there no hidden requirements for shell, package manager, alias, or runtime version?
- Are there no dependencies on "magic" files, local state, or previous chat history?
- If the skill is claimed to be portable — is this confirmed by instructions or artifacts, not by silent assumption?

*Verdict hint:* if without this information a new person cannot start or continue a step — at minimum `WARNING`. If the step requires hidden author knowledge to continue — `FAIL`.

**WF16** Are **postconditions** explicitly described? (What artifacts are created, what checks are passed, what "done" looks like)

*Context:* Without postconditions, there is no completion criterion. The agent does not know whether it has reached its goal, and may stop too early or continue indefinitely.

**WF17** Are **boundary conditions** explicitly described? (Where the skill should not be applied, where to hand off to a human or another skill)

*Context:* Without boundary conditions, the skill tries to do what it is not designed for — for example, processing files > 1 GB when a bulk-processing workflow is needed.

**WF18** Are there no **hardcoded paths**, user-specific directories, local drive names, or other author-specific paths in `SKILL.md`, `scripts/`, or references?

*Context:* Hardcoded paths are a classic portability antipattern. A skill may work perfectly on the author's machine and silently break for colleagues due to `C:\\Users\\...`, `/home/alice/...`, or hardwired absolute paths. Better to use input parameters, relative paths, and explicitly described preconditions.

---

## WF19–WF22. MCP Steps and Capability-Bound References

> **Stage: 2** | Axis: personal use (if the skill uses MCP)

**WF19** For MCP steps, are **capability, inputs, and expected outputs** described, not bare function names?

*Context:* By default, a skill should encode the workflow contract, not mirror the tool contract. A capability-bound formulation (`call the semantic search function`) is more stable than a signature-bound one (`call search_documents(query, top_k=5)`), because it does not break on MCP rename/refactor.

**WF20** Are **negative selectors** specified — which similar but incorrect tools not to choose?

*Context:* When there are several similar MCP functions, the agent may choose the wrong one. `Do not choose the exact ID search function or URL lookup` — an explicit negative selector helps the router.

**WF21** Are exact MCP names/parameters specified **only** for critical steps (safety, compliance, high cost of error)?

*Context:* Capability-bound by default; signature-bound is a conscious exception. If there are several very similar functions with different error costs, or exact enums/flags matter — switch to signature-bound for those specific steps. If exact MCP function names are needed for critical steps, prefer a single reference file with the signatures (`references/mcp-contracts.md`, `api-reference.md`). Inline signatures in `SKILL.md` are only acceptable as a localized exception.

**WF22** Does the skill duplicate full JSON schemas or lists of parameters that already live in the MCP contract?

*Context:* Mirroring MCP schema inside `SKILL.md` is an antipattern. The skill becomes brittle to any MCP server refactor. Better to capture the required capability in the skill, not copy the entire tool contract. If the skill does pin specific signatures per rule WF21, they must not be scattered across multiple files. Only one location is acceptable: either one block in `SKILL.md` or one dedicated reference file. Signatures spread across multiple files are an amplified FAIL signal.

---

## WF23–WF25. Sub-agent Delegation

> **Stage: 2** | Axis: personal use (if the skill uses sub-agents)

**WF23** Are the main agent's skills passed to the sub-agent **explicitly**, rather than assuming automatic inheritance?

*Context:* Skills are not inherited by sub-agents automatically. Required skills must be passed explicitly in the invocation.

**WF24** Does the sub-agent brief contain: **scope, files, expected output, constraints**?

*Context:* Vague invocations (`Implement the feature`) are an antipattern. Without a clear brief, the sub-agent cannot see the overall context and may break code or data dependencies.

**WF25** Does the sub-agent return a **condensed summary**, not a full transcript?

*Context:* The sub-agent architecture is useful precisely because the sub-agent can spend tens of thousands of tokens on exploration, but return only a 1000–2000 token summary. Copying the full transcript into the main thread defeats the entire purpose of delegation.

---

## WF26. Handoff Checkpoints

> **Stage: 2** | Axis: personal use (if there is a handoff between tools / co-skills)

**WF26** Do checkpoints cover **handoffs between tools / co-skills**? After each cross-system step, is it clear what artifact was received, in what format, and what constitutes a valid handoff?

*Context:* Checkpoints matter not only within a single skill (covered by WF11), but also at handoffs between tools and co-skills. Task state is most often lost at transitions between steps and tools. If MCP is described brittlely or the handoff is lost — the skill breaks even for the author.

---

## WF27. External Planning Artifact

> **Stage: 2** | Axis: personal use (if workflow is multi-step with multiple tools / co-skills)

**WF27** If the workflow passes through several tools / co-skills, is there an external planning artifact or TODO list that survives the handoff between steps?

*Context:* In a multi-skill environment, task state is most often lost precisely at transitions between steps and tools. An external planning artifact maintains dependent steps, execution status, and blockers that would otherwise dissolve into the thread history.

---

## WF28–WF29. Context and Architecture

> **Stage: 2** | Axis: personal use (if the workflow is long)

**WF28** Are there explicit instructions for **context management** for long tasks?

**Applicability:** workflow >= 5 steps, OR processing potentially large volumes of data, OR using sub-agents, OR multi-session tasks. If the workflow is short and simple — **N/A**.

*Context:* Long tasks suffer from "lost-in-the-middle" — the agent loses focus on early decisions. Manus (Jul 2025) addresses this with the filesystem as unlimited persistent memory: `todo.md` is rewritten after each step, "reciting" goals to the end of context. Anthropic (Sep 2025) describes three techniques: compaction, structured note-taking, sub-agent architectures. StackOne (Jan 2026) documents 6 context overflow failure patterns.

**Signals to check:**

| Signal | Verdict if absent |
|---|---|
| Saving intermediate results to files (plan.md, progress.md, findings.md) | WARNING (if workflow >= 5 steps) |
| Context-refresh: re-reading the plan before each new phase | WARNING |
| Strategy for context overflow (compaction, sub-agent, respawn) | INFO (if < 10 steps), WARNING (if >= 10) |
| Checkpoints with state capture | WARNING (for critical phases) |
| TODO list as attention management (updated during the work) | INFO |

**Scale:**
- **PASS:** there is an explicit context management strategy (files, checkpoints, refresh)
- **WARNING:** workflow is long (>= 5 steps) but none of the above signals are present
- **N/A:** workflow is short and simple (< 5 steps, no large data)

**Recommendation on WARNING:**
> Add context management instructions to the workflow:
> 1. Save intermediate results to a file after each major phase
> 2. Context-refresh: before each new phase, re-read plan.md and the latest progress.md entries
> 3. For tasks > 10 steps: consider respawning a sub-agent with a clean context
>
> Sources: [Manus blog](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus), [Anthropic guide](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

---

**WF29** Are there workflow phases that should be **extracted as sub-agents** but are executed inline?

**Applicability:** workflow with >= 3 logically distinct phases. If the workflow is simple (1–2 steps) — **N/A**.

*Context:* Anthropic (Sep 2025): "Specialized sub-agents can handle focused tasks with clean context windows. Each subagent might explore extensively, using tens of thousands of tokens, but returns only a condensed summary." Bob Renze (Mar 2026): three criteria — isolation benefit, model specialization, restart tolerance. nibzard/awesome-agentic-patterns: limit to 2–4 subagents; more adds coordination overhead.

**Three criteria (all simultaneously):**

| # | Criterion | Explanation |
|---|---|---|
| 1 | **Independence** | The phase does not depend on intermediate results from other phases |
| 2 | **Result-oriented** | Only the final result matters, not intermediate reasoning |
| 3 | **Verifiability** | The result is checkable by simple criteria (format, completeness, required fields) |

**Algorithm:** for each workflow phase, check all 3 criteria. If all are met AND the phase is NOT structured as a sub-agent → signal.

**Scale:**
- **PASS:** all isolatable phases are already sub-agents, OR there are no such phases
- **WARNING:** 1–2 isolatable phases running inline
- **FAIL:** 3+ isolatable phases + signs of context overload (> 10 steps, large files)
- **N/A:** workflow < 3 phases

**Recommendation on WARNING/FAIL:**
> Consider extracting phases as sub-agents:
> - Context isolation: sub-agent explores 50K+ tokens, returns 1–2K summary
> - Parallel execution of independent phases
> - Resilience: a sub-agent crash does not kill the orchestrator
>
> Constraints: 2–4 sub-agents is optimal. Each spawn ~2–3K tokens overhead. For phases < 500 tokens, inline is cheaper.
>
> Sources: [Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), [Bob Renze](https://dev.to/bobrenze/ai-agent-subagent-orchestration-when-to-spawn-vs-when-to-do-it-yourself-4opg)
