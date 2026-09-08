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

**If the response says `clarification_needed: true`, stop and ask.** A first name that
matches several people ("Ben", "Andre", "Marco") is a question for the user, not a
guess for you — even when only one of them has calls on record. Reply with the
candidates from `ambiguous_names` (name, company, calls on record) and nothing else; do
not resolve, draft, or answer until the user says which person they mean. The native
app never has to guess here because the user attached the record; asking is how this
surface gets the same certainty.

**3. Detail.** For entities the map surfaced, call `contact_profile` and
`business_profile`.

**4. Evidence.** Pull transcripts with `search_transcripts`,
`contact_transcripts_list`, or `call_transcript_conversation`. Use `format="summary"`
when scanning several calls — pulling multiple transcripts at `format="full"` buries the
question in raw text. Use `format="full"` for a single deep dive, and when the request
turns on what was said on one call (a summary of it, notes from it, a follow-up to it,
a thank-you for it), open that call — the native app hands the whole transcript to the
model, and answering from the stored notes instead is the gap. A long transcript comes
back in parts: the response carries `parts` and `next_part`; keep calling with the next
`part` until `next_part` is null before you summarize or extract anything from it.
`resolve_prompt_context` marks this for you: when the map says `call_reference: true`
its recommendations name the exact calls to open (`**OPEN THE CALL**`). Follow that before
you write anything; stored notes are not the transcript.

**5. Playbook.** Call `sales_playbook` with the user's current message and the
conversation so far.

## When to stop early

Steps 3 and 4 are conditional on step 2. A request with no entity signal — "what's a
good subject line pattern?" — has no contact, business, or call to resolve, so skip
those rather than firing lookups that come back empty. Retrieving nothing and retrieving
nothing *useful* look the same in the output but cost the user latency.

**Step 5 is not conditional.** The playbook is org-level guidance, so it is exactly
what a question with no entity signal needs — "what's a good subject line pattern?" is
answered by this org's playbook, not by a generic one. Skip `sales_playbook` only when
the request turns on no guidance at all: a bare record lookup, or reading back what was
said on a call.

## Reporting

State what you actually found: which contacts and businesses resolved, which calls you
read, whether the playbook returned relevant sections. If a name in the request resolved
to nothing, say so explicitly — a draft built on an unresolved contact is a draft about
someone the user may not have meant.

Never assert a fact about a contact or business that no retrieved source supports.
