# Changelog

All notable changes to this plugin are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project uses
[Semantic Versioning](https://semver.org/).

The plugin is pre-release: versions stay `0.X.Y` — `### Added` / `### Changed` /
`### Breaking` bump X, `### Fixed` / `### Security` bump Y — until the 1.0.0 release is
cut by hand. (1.0.0–1.3.0 were published under those numbers by mistake on
2026-09-03/08 and renumbered to 0.6.0–0.9.0 on 2026-09-08.)

## [Unreleased]

## [0.9.1] - 2026-09-08

### Fixed

- The publisher shown in claude.ai and Claude Code is now "Breakthrough" rather than
  "Breakthrough Engineering" (marketplace owner and plugin author).

## [0.9.0] - 2026-09-08

### Changed

- `answer` now fires on any question about how this company sells — pricing,
  discovery questions, positioning, objections, competitors — even when no person,
  company, or call is named. Before, its description only claimed questions "about a
  prospect, account, deal, or call", so "what's our pricing?" or "give me 3 discovery
  questions for a CFO" fired no skill at all and were answered from general sales
  knowledge instead of the org's playbook (measured: 5 of 27 turns on org 7, all of
  this shape). The skill now resolves people/companies/calls first when the question
  names one — so the resolved call's sales/partnership tag reaches `sales_playbook`,
  which matters on orgs with two playbooks — and otherwise calls `sales_playbook`
  directly on every question, the way the native chat does. Exactly one playbook
  call per question either way.


## [0.8.0] - 2026-09-06

### Added

- `sales_playbook` takes three optional arguments for saying what the conversation is
  about: `context_hint`, `call_transcript_ids` and `business_ids`. They matter on orgs
  that run more than one playbook — a sales playbook and a partnership playbook — where
  a question like "what should I cover on this call?" does not say which motion applies.
  Without them such a question could match nothing and come back
  `skipped: no_rendered_content`, leaving the answer with no playbook behind it and no
  sign anything was missing. Orgs with a single playbook ignore all three.
- The result can now carry a `clarification` field: the motion was not determined, the
  sections returned are the ones that hold under either motion, so use them and then ask
  the user that question rather than picking a motion silently.

## [0.7.0] - 2026-09-03

### Changed

- Expanded the guidance for `sales_playbook` so it is reached in the cases where it
  helps. It is now described as covering the org's playbook family, including a
  separate partnership playbook where one exists, and skills are told to call it
  before drafting anything a prospect or partner will read rather than only when a
  question is explicitly about methodology.

### Fixed

- `gather-context` told Claude to skip retrieval entirely for a request with no
  contact, company, or call in it. That skipped the playbook too, which is exactly
  what such a request needs — the playbook is org-level guidance and needs no
  entity. Entity lookups are still skipped; the playbook step no longer is.

## [0.6.0] - 2026-09-03

### Breaking

- Renamed the bundled MCP server from `api` to `breakthrough`. It is now identifiable in
  the connector list and in tool names. Any saved tool-permission rules referring to the
  old name need to be re-approved once.

### Changed

- Rewrote the install instructions. They now cover the two paths that actually exist —
  adding the marketplace in Claude Desktop / claude.ai, and the Claude Code CLI — and
  state that a marketplace is added per account rather than shared across a company.

### Fixed

- Added the `humanize` skill to the skill table. It shipped in 0.5.0 but was never listed.
- Documented that skill command names are unprefixed on Claude Desktop and claude.ai, and
  namespaced (`/breakthrough:<skill>`) only in Claude Code.
- Corrected the 0.2.0 entry below, which named a tool that was never released under that
  name.
- Pointed the manifest's repository link at this repository instead of a private one, and
  declared the license.

## [0.5.0] - 2026-08-12

### Added

- Humanize skill, run automatically on outbound drafts.

## [0.4.1] - 2026-08-12

### Changed

- Removed internal implementation details from plugin-facing docs.

## [0.4.0] - 2026-08-12

### Changed

- Applied agent-tool guidance to the MCP surface.

## [0.3.1] - 2026-08-12

### Fixed

- Split skill triggers into `when_to_use` for clearer selection.

## [0.3.0] - 2026-08-12

### Changed

- Rewrote skill descriptions for clearer selection.

## [0.2.0] - 2026-08-10

### Added

- The caller's own LinkedIn background, as `whoami(detail="full")`. (This entry
  originally announced a separate `my_profile` tool; no such tool shipped.)

## [0.1.0] - 2026-08-12

### Added

- Initial release of the Breakthrough Claude plugin.
