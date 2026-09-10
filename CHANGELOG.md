# Changelog

All notable changes to this plugin are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project uses
[Semantic Versioning](https://semver.org/).

The plugin is pre-release: versions stay `0.X.Y` — `### Added` / `### Changed` /
`### Breaking` bump X, `### Fixed` / `### Security` bump Y — until the 1.0.0 release is
cut by hand. (1.0.0–1.3.0 were published under those numbers by mistake on
2026-09-03/08 and renumbered to 0.6.0–0.9.0 on 2026-09-08.)

## [Unreleased]

## [0.12.1] - 2026-09-10

### Fixed

- `import_contact` given only a name that matches a company on record now returns that
  company in `businesses` alongside `invalid_input`, so "add Glean as a company" is
  answered from the record instead of asking for a domain.

## [0.12.0] - 2026-09-10

### Changed

- `refresh_contact` takes `user_requested`. Claude states whether the user asked for the
  refresh outright (or just confirmed one); the tool no longer reads that out of the
  wording of `query`, which MCP clients never sent — every "refresh X" over the plugin
  stalled on a confirmation question. The refresh-contact skill and the server
  instructions say when to set it.

## [0.11.1] - 2026-09-09

### Fixed

- `import_contact` given only a name now checks the org's existing contacts first and
  returns `already_on_record` (or `ambiguous_name` with candidates) instead of asking for
  a URL for someone who is already in Breakthrough. An email-only import is held to the
  name the address carries, so a vendor answer for a different person is rejected rather
  than imported.
- find-contact and refresh-contact now claim "where does X work", "is X still at Y",
  "did X leave" and similar currency questions, and say Breakthrough's record comes
  before a web search. On claude.ai such questions were answered from the web with the
  plugin never consulted.

## [0.11.0] - 2026-09-09

### Added

- `contact_profile` and `business_profile` return `recent_calls`: that person's or
  company's calls, newest first, with each call's `tags`. Resolving an entity now puts its
  calls in front of the model the way the app shows the call list when an entity is pulled
  in. The skills say when to read them (a question about past calls reads the thematic
  summaries; up to 3 calls in full) and to re-run `resolve_prompt_context` on the name the
  user gives after a clarifying question.
- Call `tags` ("Sales", "Internal", "Onboarding", …) on every transcript in
  `search_transcripts` and `contact_transcripts_list` results, and as a `filter_by` field
  (`tags:Sales`, `tags:!=Internal`). Calls recorded before this release show their tags
  once the search index has been refreshed. The research-transcripts skill
  now handles "a topic across a window" by bounding the set (date + tags) and reading each
  call's thematic summary, because the search index holds names and imported summaries,
  not what was said.

### Fixed

- `resolve_prompt_context` probes a multi-word person's surname on its own as well as
  the whole name, so a nickname or a misspelt first name ("Rob Jackson" for Robert
  Jackson) still finds the person. A surname that came from a named person is not an
  incidental word hit, and a named first name that starts the candidate's first name
  ("Rob" for Robert) counts as evidence, so one Robert among eight Jacksons resolves
  while two Mark Jacksons still get a question.
- When a resolved contact has calls on record, the contact recommendation no longer
  calls the entity notes "the primary source" — the notes are context; the call is the
  source for what was said, and the **CALLS ON RECORD** step carries the open-the-call
  instruction. In 3 of 82 harness runs the old wording stopped the model at the notes.
- Recommendations and skills call the `<PastCallNotes>` block what it is: the entity notes
  (what we extracted about a person or company from every past call), not a summary of any
  call. The transcript ceiling is stated as a rule: full transcripts up to 3 calls, thematic
  summaries above that. The app's ceiling is 5; the plugin's is lower because everything it
  opens stays in Claude's context.
- The research-transcripts skill resolves the request through `resolve_prompt_context`
  before searching when a person or company is named, and asks when the name is
  ambiguous. Searching transcripts for "Ben demo" and reading the top hit picked one of
  two Bens with a demo on record silently.

## [0.10.2] - 2026-09-08

### Changed

- Marketplace listing and manifest description rewritten: the plugin connects Claude to
  a living playbook distilled from your sales calls, so it has the right context and
  knows what works across your organization. Listing copy only; no behavior change,
  hence a patch version rather than the minor bump a `### Changed` entry normally takes.

## [0.10.1] - 2026-09-08

### Fixed

- A shared first name blocks with a clarifying question only when it is the request's
  only handle. "Email Gianluca Peretti … tie it to Alex's directive" proceeds with
  Gianluca; the map still lists the Alexes and says not to pick one. A call the request
  names is searched by title (`matched_transcripts_by_title`) and recommended directly.
- `resolve_prompt_context` now has a small model read the request for the people and
  companies it names, and searches only those (`mentions`, `mention_source`). Before, every
  word was probed against the index, so "write a short email thanking Ioanna" surfaced a
  Tim Short on the word "short". The per-word probe remains as a fallback.
- `resolve_prompt_context` no longer guesses from wording whether a prompt refers to a
  past call; it reports the resolved person's calls on record (**CALLS ON RECORD**) and
  leaves that judgment to the model. A company or surname qualifier ("Ben at
  Splashtop") resolves a shared first name through an org-scoped lookup, and the
  calls-on-record counts include calls imported by colleagues.

## [0.10.0] - 2026-09-08

### Changed

- `resolve_prompt_context` now flags a first name that matches several people
  (`clarification_needed`, `ambiguous_names` with each candidate's calls on record)
  and its first recommendation says ask, do not pick. The gather-context skill stops
  and asks on that flag instead of guessing the person with the most calls.
- `call_transcript_conversation` returns long transcripts in parts of 60,000
  characters (`part`, `parts`, `next_part`, `total_chars`) and no longer duplicates the
  raw text beside the tag-wrapped copy unless `include_structured` is set. Single
  results of 94-163KB overflowed the client's tool-result cap and the call went unread.
- `resolve_prompt_context` sets `call_reference` when the prompt talks about a call
  ("our call", "my last call", "the demo") and recommends the exact transcript(s) to
  open; a full name in the prompt ("Ben Beal") is not flagged as ambiguous just because
  the first name alone matches several people.
- gather-context, draft-email, and draft-linkedin-message open the call when the
  request is about it (a thank-you, a follow-up to the demo) and read every part
  before writing; the stored call notes are a digest, not the transcript.

## [0.9.2] - 2026-09-08

### Fixed

- The marketplace manifest now declares the plugin version, so claude.ai and Claude
  Code can see that a new release exists. Without it, a marketplace added on claude.ai
  stayed on the version from the day it was added.

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
