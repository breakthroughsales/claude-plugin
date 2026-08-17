---
name: research-transcripts
description: >-
  Searches and reads the user's recorded Breakthrough call transcripts — finds which
  calls touched a topic, lists a contact's calls, or reads one call in full. Answers
  what was actually said, by whom, and when, citing the call it came from.
when_to_use: >-
  Use when the user says "what did they say about pricing", "did we discuss budget",
  "when did we last talk to Acme", "what were their objections", "pull up that call",
  "did they commit to anything", "who was on the call", "did it come up". Do NOT use to
  write the call up as a document; that is draft-note. Do NOT use for questions needing
  judgment beyond what the calls actually state; that is answer.
---

# Research call transcripts

Wraps `search_transcripts`, `contact_transcripts_list`, and
`call_transcript_conversation`. Full argument details in
`${CLAUDE_PLUGIN_ROOT}/references/mcp-tools.md`.

## Choosing the entry point

| The question | Tool |
| --- | --- |
| "when did X come up" / topic across calls | `search_transcripts` |
| "what calls have we had with Jane" | `contact_transcripts_list` (needs contact IDs) |
| "what happened on that call" | `call_transcript_conversation` |

`contact_transcripts_list` takes **integer contact IDs**, not names — resolve through
`contact_profile` or `resolve_prompt_context` first. It returns metadata only, no
transcript bodies.

## Searching

`search_transcripts(query, limit, filter_by, sort_by)`:

- `query` — semantic + keyword. Empty browses without relevance ranking.
- `filter_by` — filter grammar, applied **before** ranking. Only four fields exist:
  `name`, `participant_names`, `business_names`, `call_date` (unix seconds). Inventing a
  field name errors out rather than being ignored.
- `sort_by` — sort grammar. Defaults to `call_date:desc` on an empty query,
  relevance otherwise.

Filter when you know the constraint (a named account, a date range); let relevance rank
when you don't.

## Reading

`call_transcript_conversation(transcript_id, include_structured, format)`:

- **`format="summary"`** when scanning several calls. Pulling three or four at `"full"`
  buries the question in raw dialogue.
- **`format="full"`** for a single deep dive, or when exact wording matters — an
  objection, a commitment, a number.
- `include_structured=True` adds per-sentence timing and speaker structure. Only with
  `format="full"`, and only when you need to attribute lines to speakers.

## Reporting

Cite which call each finding came from — call name and date. "They pushed back on
pricing" is worth little; "on the Feb 12 call, Dana said the $40k figure would need
CFO sign-off" is actionable.

Distinguish what was said from what it implies. And when the search returns nothing, say
the calls don't cover it rather than reasoning from general knowledge about the account —
the entire premise of the question was the user's own call data.
