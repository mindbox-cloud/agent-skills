# skill-review

AI reviewer for Agent Skills. Runs a Standard Review of an isolated skill folder (`SKILL.md` + optional `references/`, `scripts/`, `assets/`) and produces a structured report with a PASS/WARNING/FAIL/N/A breakdown, an **Antipattern Bingo** section, and a summary from Exhausted Vitaly.

## Review Scopes

The skill asks for the usage context at the start and adapts the scope accordingly:

| Context | Stage | Scope |
|---------|-------|-------|
| Personal | 2 | Structure + Workflow (basic) + References RF04–RF12 + Links LK01, LK05 |
| Team | 3 | + portability, negative triggers, navigation, stable cross-refs |
| Repository | 4 | + routing, Lifecycle |
| I don't know | full | everything across all stages without declared target |

## Check Groups

### 1. Structure — form and validity

Answers the question: is this a correctly structured Agent Skill, using Anthropic best practices for skill layout and markdown files?

- **ST01–ST04** — folder structure, exact `SKILL.md` name, folder hygiene, correct use of `references/`, `scripts/`, `assets/`
- **ST05–ST08** — YAML frontmatter: `name`, `description`, trigger phrases, frontmatter safety
- **ST09** — `SKILL.md` size and monolith risk
- **ST10–ST13** — gross form antipatterns: skill-prompt, self-generated artifact, vibe-coded core, monolith
- **ST14–ST16** — justification of the skill as an artifact

Almost all of Structure lives at **stage 2** — covers basic personal usability of the skill.

### 2. Workflow — can you actually act on this skill

The main practical layer. Checks whether the text becomes an executable procedure.

- **WF01–WF04** — imperative instructions, numbered steps, examples, troubleshooting
- **WF05–WF10** — six mandatory workflow elements: trigger, inputs, steps, checks, stop, recovery
- **WF11–WF14** — safeguards: checkpoints, planning discipline, critical rules in an explicit place
- **WF15–WF18** — preconditions, postconditions, boundary conditions, no hardcoded paths
- **WF19–WF22** — correct description of MCP steps
- **WF23–WF25** — correct handling of sub-agents
- **WF26–WF29** — handoff, external planning artifact, context management, extracting independent phases as sub-agents

`WF12` and `WF13` are conditional: significant only for long skills with an explicit planning artifact.

### 3. References — context correctness, progressive disclosure, routing hygiene

- **RF04–RF12** *(stage 2)* — is the knowledge layer in `references/`, is there a file map, explicit triggers, healthy topology, has the reference layer not become a dump
- **RF01, RF03, RF13–RF15** *(stage 3)* — negative triggers, protection from overtriggering, freshness risks, gotchas, extraction of deterministic operations into `scripts/`
- **RF02** *(stage 4)* — routing to neighboring skills in the ecosystem

### 4. Links — mechanical link integrity

- **LK01, LK05** *(stage 2)* — do file refs exist, no orphan files
- **LK02–LK04** *(stage 2)* — are anchors and TOC alive
- **LK06** *(stage 2)* — are external URLs alive, if any
- **LK07** *(stage 3)* — are stable cross-refs used via headings, not line numbers

### 5. Lifecycle — repository maturity

Launched only in a full review (stage 4 or "I don't know").

- **LC01–LC03** — mandatory stage-4 markers: owner, version, changelog/version header
- **LC04–LC05** — hygiene checks: secrets hygiene and README boundary (appear in the report but are not hard gate markers)

## Installation

```shell
/plugin marketplace add https://github.com/mindbox-cloud/agent-skills
/plugin install skill-review@agent-skills
```

## Usage

```
review this skill
quick skill review
check my skill
```

Skill: `skill-review:skill-review-novice`

## References (source materials)

1. **[Agent Skills Specification](https://agentskills.io/specification)**
   Core spec for the skill package: folder structure, `SKILL.md`, progressive disclosure, frontmatter, directory hygiene.

2. **[Anthropic - Agent Skills Overview](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)**
   Official description of how skills are loaded and used as reusable filesystem-based resources.

3. **[Anthropic - Effective Context Engineering for AI Agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents)**
   Foundation for ideas around checkpoints, TODO/planning artifacts, long-context failure modes, and structured note-taking.

4. **[SkillsBench](https://arxiv.org/abs/2602.12670)**
   Research showing that curated skills and human review genuinely improve outcomes.

5. **[Promptware Engineering](https://arxiv.org/abs/2503.02400)**
   Theoretical framework: prompts and skills as first-class software artifacts.

6. **[Anthropic - Building Effective Agents](https://anthropic.com/research/building-effective-agents)**
   Practical principles of agentic workflows: simplicity, explicit contracts, transparency.

7. **[Manus - Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)**
   Practical text on managing long tasks and maintaining the thread of work.

8. **[Bob Renze - AI Agent Subagent Orchestration](https://dev.to/bobrenze/ai-agent-subagent-orchestration-when-to-spawn-vs-when-to-do-it-yourself-4opg)**
   External explanation of when to extract a phase as a sub-agent and why it reduces context overload.
