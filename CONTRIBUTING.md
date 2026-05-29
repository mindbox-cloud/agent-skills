# Contributing to agent-skills

Thank you for your interest in contributing. This repository hosts reviewed, documented Claude Code plugins. We aim to keep the quality bar high so that every plugin is genuinely useful and well-structured.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and working
- Familiarity with the [Agent Skills Specification](https://agentskills.io/specification)
- Understanding of `SKILL.md` frontmatter and the progressive disclosure pattern

---

## How to Propose a New Plugin

1. **Open an issue first** — describe what the plugin does, who it is for, and what skills it will include. This avoids building something that duplicates an existing plugin or does not fit the repo's scope.
2. **Fork and branch** — create a feature branch named `feat/<plugin-name>`.
3. **Implement the plugin** — follow the structure and requirements in [CLAUDE.md](./CLAUDE.md).
4. **Open a PR** — fill in the PR template checklist completely.

---

## Plugin Quality Bar

Every plugin in this repository must meet these criteria before merging:

- **English only** — all content in all files must be in English.
- **`plugin.json` complete** — `name`, `description`, `version`, `author` are all present.
- **`SKILL.md` frontmatter complete** — `name` (kebab-case, matches folder), `description` with concrete trigger phrases and "Don't use when" negative triggers, `metadata.version` in semver.
- **Progressive disclosure** — workflow logic in `SKILL.md`, domain knowledge in `references/`. No monolithic skill files.
- **`marketplace.json` updated** — the new plugin is registered.
- **Root `README.md` updated** — the plugin table includes the new entry.
- **`CHANGELOG.md` present** at the plugin root.
- **CI passes** — the `validate.yml` workflow runs green.

See [CLAUDE.md](./CLAUDE.md) for the full pre-merge checklist and structural requirements.

---

## Language Requirement

All content — skill instructions, checklists, report templates, READMEs — must be written in English. This is a hard requirement for inclusion in the public marketplace.

If you are translating a skill from another language, preserve the logic exactly. Do not paraphrase in a way that changes meaning. Technical IDs (ST01, WF01, etc.) are not translated.

---

## Pull Request Process

1. Fill in the PR template completely — skipped checklist items will delay the review.
2. One PR per plugin or per meaningful change — do not bundle unrelated plugins in one PR.
3. A maintainer will review within a reasonable timeframe. Expect at least one round of feedback.
4. Squash or rebase before merging — keep the commit history clean.

---

## Commit Style

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(plugin-name): description of what was added
fix(skill-name): description of what was fixed
docs: update CONTRIBUTING.md
chore: bump CI action versions
```

---

## Code of Conduct

Be respectful and constructive. Feedback on skill content should focus on correctness, clarity, and adherence to the quality bar — not personal preference.
