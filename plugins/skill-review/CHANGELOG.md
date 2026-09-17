# Changelog — skill-review

## [1.1.0] — 2026-09-17

### Changed

- The review no longer depends on a client-specific tool. The preflight used to stop the
  review when `TodoWrite` or `Task` was missing, so the skill could not start at all in a
  client that does not offer them. File read is the only hard requirement now.
- The review plan is always a markdown checklist in the chat. Both execution modes now ship a
  ready-made skeleton to copy into the response and check off as the review proceeds — the
  pattern Anthropic's skill authoring guide recommends. It works in any client and names no tool.
- The review plan gained an explicit completion gate: no final report while plan items are open.
- Single-pass mode can now also be activated by the absence of a sub-agent tool, not only by
  volume: it gets a reading strategy for a large skill in one context and an obligation to
  record the missing delegation in `Review Limitations`.
- WF12 now asks for a planning mechanism that survives a client without a given tool.
  WF13 is worded in terms of open plan items instead of one tool's status names.

### Added

- Antipattern Bingo #21 "Client tool lock-in" (stage 3, WF12 + WF15, owner Workflow) — the skill
  names a tool of one client and describes no way to work without it. WF15 is the second anchor
  so the row still fires on a skill too short for WF12 to apply.

### Fixed

- With logging on, the results folder is now created right after the user confirms its path,
  so single-pass mode has somewhere to write `report.md`. Previously the folder was created
  only on the sub-agent branch.

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
