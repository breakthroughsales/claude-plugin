---
name: gather-context
description: >-
  Shared subroutine that loads the Breakthrough context behind a request — resolves
  which contacts, companies, and recorded calls are in scope, then retrieves the
  relevant sales playbook sections. The drafting and answering skills each run this
  first, so it is not selected on its own; invoke it directly only to preload
  everything known about an account before a working session.
disable-model-invocation: true
---

# Gather Breakthrough context

The prelude every drafting and answering skill runs before it produces anything:
resolve the entities in scope, then pull the evidence and playbook behind them.

Read `${CLAUDE_PLUGIN_ROOT}/references/context-assembly.md` for the voice, length, and
web-search rules that come with this context. Read
`${CLAUDE_PLUGIN_ROOT}/references/mcp-tools.md` for argument details.

## Sequence

**1. Confirm scope.** Call `whoami`. If `license_id` is missing, stop — the session is
not authenticated and every other tool will return `skipped`. Tell the user to
authenticate rather than reporting an empty result.

**2. Resolve.** Call `resolve_prompt_context` with the user's message **verbatim**. Do
not summarize or extract keywords first; the tool does its own signal extraction and
pre-digesting destroys what it keys on.

The response carries a `recommendations` array generated from real counts against this
org's data. **Follow it unless you have a specific reason not to.** It knows
what exists in scope; the prompt text does not.

**3. Detail.** For entities the map surfaced, call `contact_profile` and
`business_profile`.

**4. Evidence.** Pull transcripts with `search_transcripts`,
`contact_transcripts_list`, or `call_transcript_conversation`. Use `format="summary"`
when scanning several calls — pulling multiple transcripts at `format="full"` buries the
question in raw text. Use `format="full"` for a single deep dive.

**5. Playbook.** Call `sales_playbook` with the user's current message and the
conversation so far.

## When to stop early

Steps 3–5 are conditional on step 2. A request with no entity signal — "what's a good
subject line pattern?" — should skip retrieval entirely rather than firing calls that
come back empty. Retrieving nothing and retrieving nothing *useful* look the same in the
output but cost the user latency.

## Reporting

State what you actually found: which contacts and businesses resolved, which calls you
read, whether the playbook returned relevant sections. If a name in the request resolved
to nothing, say so explicitly — a draft built on an unresolved contact is a draft about
someone the user may not have meant.

Never assert a fact about a contact or business that no retrieved source supports.
