# Changelog — skill-review

## [1.1.0] — 2026-09-17

### Changed

- Planning no longer depends on a client-specific tool. The preflight used to stop the
  review when `TodoWrite` or `Task` was missing — newer models ship without a built-in
  task-tracking tool, so the skill could not start at all. Now only file read is a hard
  requirement; task tracking and sub-agent delegation are optional capabilities with
  declared fallbacks (plan as a markdown checklist in chat; single-pass regardless of
  volume when there is no sub-agent tool).
- Review plan gained an explicit completion gate: no final report while plan items are open.
- WF12 extended with a portability check of the planning mechanism, WF13 reworded in terms
  of open plan items instead of one tool's status names.

### Added

- Antipattern Bingo #21 "Client tool lock-in" (stage 2, WF12, owner Workflow) — a skill tied
  to a concrete client tool with no fallback.

## [1.0.0] — 2026-05-26

### Added

- Initial public release
- `skill-review`: Standard Review for Agent Skills
  - Single-pass mode (< 500 lines) and sub-agent mode (>= 500 lines)
  - 5 check groups: Structure (ST01–ST16), Workflow (WF01–WF29),
    References (RF01–RF15), Links (LK01–LK07), Lifecycle (LC01–LC05)
  - 20-antipattern Bingo table
  - Summary from an exhausted data scientist
  - 4 review scopes: Personal, Team, Repository, Full
