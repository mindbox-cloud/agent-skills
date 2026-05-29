# Checklist: References and Progressive Disclosure

> Sub-agent References. Checks progressive disclosure, description quality, routing hygiene, and the organization policy of references/.
>
> Result of each check: `PASS` / `FAIL` / `WARNING` / `N/A`.

---

## RF04–RF12. Progressive Disclosure and `references/`

> **Stage: 2** | Axis: personal use (if references/ exists)

**RF04** Does the workflow (how to execute) live in `SKILL.md`, and the methodology/references (what to know) in `references/`?

*Context:* Mixing workflow and knowledge in one file increases NL-code confusion (see WF02 in checklist-workflow.md). `SKILL.md` owns "how to execute" (workflow, checkpoints, stop conditions). `references/` owns "what to know to execute" (methodologies, domain rules, reference material).

**RF05** No duplication between `SKILL.md` and `references/`?

*Context:* Single Source of Truth. When they fall out of sync, the agent receives contradictory instructions. Information must live in only one place.

**RF06** Is each reference file focused on **one topic / scenario / class of problems**?

*Context:* A monolithic reference of thousands of lines is a "monolithic reference dump" antipattern. The agent is forced to read everything, filling its context with irrelevant information. `Keep individual reference files focused. Agents load these on demand, so smaller files mean less use of context.` — Agent Skills Specification.

**RF07** Does `SKILL.md` state **when** to read each reference file (explicit trigger), rather than just listing that it exists?

*Context:* `Read references/api-errors.md if API returns non-200` — correct. `See references/ for more details` — antipattern. Without an explicit trigger, the agent either reads everything (context bloat) or reads nothing (loss of knowledge).

**RF08** Is the primary navigation to reference files **directly from `SKILL.md`**, not only through reference→reference chains?

*Context:* `SKILL.md` must be the orchestration layer. The agent must see the key reference files and their read triggers directly from the main workflow, not discover them through multi-step transitions. Limited cross-refs are acceptable as secondary navigation, but not as the sole entry point.

**RF09** If a reference file links to another reference file, is the link explicitly scoped and explained? (explicit trigger, no cycles, no deep chains)

*Context:* Cross-references are only acceptable as a local clarification: no deeper than 1 additional hop from `SKILL.md`, no cycles, with an explicit statement of when to read the next file. This is a policy check on topology, not a mechanical reachability check.

**RF10** Do files in `references/` **> 100 lines** start with a **table of contents**?

*Context:* A TOC lets the agent understand the file structure and read only the needed section rather than the whole document. The TOC must link to headings/anchors, not line numbers. Section headings must be unique and searchable.

*Assessment rule:* the absence of a TOC in a reference file > 100 lines should by default be treated as a **WARNING** for navigation and selective loading. Use **FAIL** only if as a result the file effectively becomes an unnavigable monolith and the agent cannot identify which section to read.

**RF11** Is content distributed across three loading levels approximately as **L1 (~10%) : L2 (~30%) : L3 (~60%)**?

*Context:* L1 (frontmatter) loads always. L2 (body) loads on activation. L3 (references, scripts, assets) loads on demand. If 90% of content is in the body — progressive disclosure is not working.

**RF12** If there are many reference files, is there a **file map** in `SKILL.md` or `references/INDEX.md` with the purpose of each file?

*Context:* As references grow, the agent needs not just decomposition but a navigation map. The right pattern: `SKILL.md` as the orchestration layer + references as the selective-loading layer. Without a map, even good small files become an unnavigable collection.

---

## RF01, RF03. Description Quality and Routing Hygiene

> **Stage: 3** | Axis: team use

**RF01** Does the description contain "**When NOT to use**" (negative triggers)?

*Context:* Adding negative triggers reduces false activations (overtriggering) by 40–60%. This is one of the most effective techniques. Format: `Don't use when: user asks for code review (use code-review skill)`.

**RF03** Is the description not too broad, not provoking overtriggering?

*Context:* An overly generic description intercepts neighboring requests. If the description covers 5+ different domains or contains no concrete trigger phrases, the router will activate the skill on irrelevant requests.

---

## RF13–RF15. Additional Reference Checks

> **Stage: 3** | Axis: team use (if applicable)

**RF13** Are there no **dynamic knowledge items** in `references/` that become outdated quickly: pricing, benchmarks, regulatory values, API versions?

*Context:* If data changes frequently, it should not be stored as a static reference. Better to use retrieval / RAG / MCP, or at least explicitly mark a freshness risk. Otherwise the skill starts giving plausible-sounding but outdated answers.

**RF14** Are important **gotchas** that apply almost always in `SKILL.md`, not buried deep in `references/`?

*Context:* Not all knowledge is equally suited to selective loading. If the agent will almost certainly encounter a problem, it is better to surface it early in the main body. Otherwise it may not recognize the trigger for loading the needed reference file in time.

**RF15** Are deterministic operations, validations, and transformations extracted into `scripts/` rather than described only in natural language?

*Context:* Code is deterministic; natural language interpretation is not. If a step can be reliably validated or executed by a script, it is better to do it in a script and leave only orchestration and decision logic in `SKILL.md`.

---

## RF02. Routing in the Ecosystem

> **Stage: 4** | Axis: repository and ecosystem

**RF02** Are specific **neighboring skills** to redirect to named?

*Context:* Negative triggers without an alternative (`Don't use for X`) are less effective than those with one (`Don't use for X — use Y-skill instead`). A concrete alternative helps the router make the right decision.
