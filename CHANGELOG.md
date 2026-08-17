# Changelog

All notable changes to this plugin are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project uses
[Semantic Versioning](https://semver.org/).

Add your entry under `## [Unreleased]` in the same PR as your change, using one of:
`### Added`, `### Changed`, `### Fixed`, `### Security`, `### Breaking`. Then run
`scripts/bump_plugin_version.sh` to bump `plugin.json`'s version and stamp the release —
`### Added`/`### Changed` bump minor, `### Fixed`/`### Security` bump patch, and
`### Breaking` bumps major. CI fails the PR if `plugins/breakthrough/` changes without a
changelog entry, or if `plugin.json`'s version doesn't match what `[Unreleased]` implies.

## [Unreleased]

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

- `my_profile` MCP tool with LinkedIn background context.

## [0.1.0] - 2026-08-12

### Added

- Initial release of the Breakthrough Claude Code plugin.
