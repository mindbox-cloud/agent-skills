# agent-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Plugins](https://img.shields.io/badge/plugins-2-blue)](#available-plugins)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-orange)](https://claude.ai/code)

Documented Claude Code plugins by [mindbox.cloud](https://mindbox.cloud/?locale=en_US) — ready to install.

---

## Available Plugins

| Plugin | Description | Skill | Install |
|--------|-------------|-------|---------|
| [skill-review](./plugins/skill-review/) | Quick AI reviewer for Agent Skills: checks structure, workflow, references, and links. Clear report without jargon, with a summary from an exhausted data scientist. | `skill-review:skill-review` | see below |
| [mindbox](./plugins/mindbox/) | Build marketing scenarios and audience filters in a Mindbox project from plain-language requests: flow-create designs and fills a scenario, filter-build builds a platform-confirmed filter, filter-explain reads one back in business terms. Nothing is launched and no filter is saved as a segment. | `mindbox:filter-build`, `mindbox:filter-explain`, `mindbox:flow-create` | see below |

---

## Quick Install

```shell
/plugin marketplace add https://github.com/mindbox-cloud/agent-skills
/plugin install <plugin-name>@mindbox-cloud-plugins
```

See each plugin's README for available skills and usage.

---

## What's in this Repo

A curated collection of Claude Code plugins built by [mindbox.cloud](https://mindbox.cloud/?locale=en_US). Each plugin is:

- **Documented** — clear README, usage examples, changelog
- **English-only** — all content is in English for broad accessibility
- **MIT licensed** — free to use and adapt

More plugins will be added over time. See [CONTRIBUTING.md](./CONTRIBUTING.md) to propose one.

---

## Contributing

New plugins are welcome. Read [CONTRIBUTING.md](./CONTRIBUTING.md) for the quality bar and process.

---

## License

[MIT](./LICENSE)
