# Checklist: Ownership and Lifecycle

> Sub-agent Lifecycle. Checks ownership, metadata hygiene, lifecycle artifacts, and hygiene of a mature skill.
>
> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

---

## LC01–LC03. Stage 4 Gate Markers

> **Stage: 4** | Axis: repository and ecosystem
>
> LC01–LC03 are **marker-level stage-4 checks**: mandatory markers for transitioning to stage 4. Without them, a skill is not considered a repository skill.

**LC01** Is `metadata.author` (skill owner) specified?

*Context:* Orphaned skills without a maintainer are one of the most common problems. When the author leaves, the skill becomes a black box. `metadata.author` is the minimum ownership hygiene.

**LC02** Is `metadata.version` in **semver format**?

*Context:* `metadata.version` (semver) fixes the environment contract and artifact version. Without semver it is impossible to determine whether different versions of the skill are compatible with each other and with the environment.

**LC03** Is there a **CHANGELOG** or **version header with dates** that shows which version of the instruction is current?

*Context:* Update dates let you understand which version of the instruction is current. Acceptable forms: a version header in `SKILL.md` (`> Version: 4.0 | Date: ...`) or a `CHANGELOG.md` next to `SKILL.md` or at the plugin root — but not inside `references/` (that is L3 content for the agent, see RF07, RF08, LK05). If the skill files are not versioned and there is no changelog, lifecycle maturity cannot be confirmed by artifacts.

---

## LC04–LC05. Stage 4 Hygiene Checks

> **Stage: 4** | Axis: repository and ecosystem
>
> LC04–LC05 are **stage-4 hygiene checks**: they must appear in the review and report, but are **not hard gate markers** for the transition to stage 4.

**LC04** Are there no **tokens, keys, passwords, or other secrets** in the skill files?

*Context:* Secrets in skill files are a direct security risk, especially when publishing to a shared repository. Check for: API keys, access tokens, passwords, connection strings, private keys. Even in `references/` and `scripts/` — secrets are not acceptable. If the skill needs to use a secret, a delivery mechanism via environment variable or vault must be described, not a hardcode.

**LC05** Is the human-facing `README.md` at **repository level**, not inside the skill folder? Does `SKILL.md` remain the agent instruction?

*Context:* Skills are designed for consumption by an AI agent, not a human. A `README.md` inside the skill folder is an antipattern: extra markdown files can disrupt routing and discovery. The human-facing README lives at the repository level.
