# agent-skills — Developer Guide for Claude Code

## Repository Purpose

This is a public marketplace of Claude Code plugins by mindbox.cloud. Each plugin extends Claude Code with skills (slash commands), agents, hooks, or MCP servers.

Plugins are designed to be shared with the community. All content must be in English.

---

## Repository Structure

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json    # marketplace registry — update when adding a plugin
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json  # plugin manifest
│       ├── skills/
│       │   └── <skill-name>/
│       │       ├── SKILL.md
│       │       └── references/
│       ├── CHANGELOG.md
│       └── README.md
├── CLAUDE.md
├── CONTRIBUTING.md
└── README.md
```

---

## How to Add a New Plugin

### Step 1 — Create the directory structure

```bash
mkdir -p plugins/<plugin-name>/.claude-plugin
mkdir -p plugins/<plugin-name>/skills/<skill-name>/references
```

Plugin names: kebab-case, no spaces.

### Step 2 — Write `plugin.json`

```json
{
  "name": "plugin-name",
  "description": "One-line description in English",
  "version": "1.0.0",
  "author": {
    "name": "Author or org name"
  }
}
```

Required fields: `name`. Recommended: `description`, `version`, `author`.

### Step 3 — Write `SKILL.md`

```yaml
---
name: skill-name          # kebab-case, matches folder name
description: >
  What it does — one sentence.
  Use when user says "...", "...", "...".
  Don't use when: user asks for X (use y-skill instead), user asks for Z.
metadata:
  version: 1.0.0          # semver — instruction contract version
---

Skill body: orchestration logic, steps, critical rules, troubleshooting.
Put domain knowledge and checklists in references/, not inline.
```

Frontmatter requirements:
- `name` — kebab-case, 1–64 chars, matches folder name
- `description` — WHAT it does + WHEN to use (trigger phrases) + "Don't use when" (negative triggers)
- `metadata.version` — semver

### Step 4 — Register in `marketplace.json`

Add an entry to `.claude-plugin/marketplace.json`:

```json
{
  "name": "mindbox-cloud-plugins",
  "owner": { "name": "mindbox.cloud" },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/plugin-name",
      "description": "Same one-line description as plugin.json"
    }
  ]
}
```

### Step 5 — Write `README.md` for the plugin

Human-facing documentation at `plugins/<plugin-name>/README.md`. Describe: what it does, review scopes (if applicable), installation command, usage trigger phrases.

### Step 6 — Write `CHANGELOG.md`

At `plugins/<plugin-name>/CHANGELOG.md`. Required for stage 4 lifecycle hygiene (LC03).

### Step 7 — Update root `README.md`

Add the plugin to the plugin table in the root README.

---

## Version Policy

> Any meaningful change to a plugin requires a `plugin.json` version bump. Without it, users with the plugin already installed will not receive the update — their client caches the old version.

There are two independent semvers per plugin:

| File | Version type | When to increment |
|---|---|---|
| `plugin.json` | Package release version | **Any meaningful change** — skill logic, new files, bug fixes, prompt edits |
| `SKILL.md` `metadata.version` | Instruction contract version | Tracks internal iteration of the skill logic |

Both must be updated when making changes. They are independent — do not conflate them.

---

## Conventional Commit Style

Use conventional commits for all changes to this repository:

```
feat(plugin-name): add initial public release
fix(skill-name): correct trigger phrase to avoid overtriggering
docs(skill-review): update README with new scope table
chore: add .gitignore
```

Scopes: use the plugin or skill name when the change is scoped to one plugin; omit scope for repo-wide changes.

---

## Pre-merge Checklist

Before merging a plugin PR:

- [ ] All content in English
- [ ] `plugin.json` has `name`, `description` (English), `version`, `author`
- [ ] `SKILL.md` frontmatter complete: `name`, `description` with trigger phrases and "Don't use when", `metadata.version`
- [ ] `marketplace.json` updated with the new plugin entry
- [ ] Root `README.md` plugin table updated
- [ ] `CHANGELOG.md` present in the plugin root
- [ ] CI (`validate.yml`) passes

---

## References

- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Skills](https://code.claude.com/docs/en/skills)
