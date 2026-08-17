# Breakthrough MCP tool reference

Every tool is scoped to the caller's own Breakthrough organization, derived from the
authenticated session. There is no way to query across organizations, and no tool takes
an organization argument.

Every non-success payload carries a `message` written for you, not for a log: what
happened, what to do, and whether retrying can change anything. **Read it and act on
it.** When a message says not to retry, retrying wastes turns on a result that cannot
change — the commonest case being `missing_or_invalid_license_context`, which means the
session is unauthenticated, not that the org has no data. Never report that one to the
user as "no data found."

## Read-only tools

### `health()`

Liveness only. Returns `status`, `service`, `environment`. Takes no arguments and
touches no org data. Use it to distinguish "the server is down" from "my auth is wrong."

### `whoami(detail="basic" | "full")`

**`detail="basic"`** (default) — `email`, `license_id`, `organization_id`. Reads the
access token only, no database round-trip, so it is free to call at the start of any
task. A populated `license_id` is what every other tool scopes on; if it is missing,
nothing else will work.

**`detail="full"`** — adds `full_name`, `organization_name`, `linkedin_url`,
`linkedin_profile` (markdown: headline, summary, experience, education, location), and
`linkedin_profile_fetched_at`. Use when the task turns on who the user is — writing in
their voice, drawing on their own experience, "what's my background".

`full` reads only **cached** enrichment. It never performs a live LinkedIn fetch and
never writes, which is what keeps the whole tool read-only and free of permission
prompts on the hot path. Branch on `profile_status`:

| `profile_status` | Meaning |
| --- | --- |
| `ok` | `linkedin_profile` populated |
| `no_linkedin_url` | User hasn't connected LinkedIn. Identity valid. Do not retry |
| `not_cached` | URL exists, enrichment hasn't run yet. Identity valid. Do not retry |

### `resolve_prompt_context(user_prompt)`

The entry point for almost every task. Pass the user's message **verbatim** — do not
summarize, clean up, or extract keywords first. The tool does its own signal-token
extraction, and pre-digesting the prompt destroys the signal it keys on.

Returns a context map:

- `status`
- `signal_tokens` — the tokens it keyed on
- `matched_transcripts_by_name` — `[{"pattern": ..., "count": N}, ...]`
- matched contacts and businesses, cross-referenced against each other
- `recommendations` — concrete next tool calls

**Follow `recommendations` unless you have a specific reason not to.** They are generated
from real counts against this org's data, so they encode what actually exists
in scope — something you cannot infer from the prompt text alone.

### `contact_profile(query, limit_candidates=5)`

Resolves a contact from free text. Understands raw email addresses and LinkedIn URLs
embedded in the query, so passing the user's phrasing directly usually works.

### `business_profile(query, limit_candidates=5)`

Same, for businesses. Understands domains and LinkedIn company URLs.

### `contact_transcripts_list(contact_ids, limit=50)`

Transcript **metadata** for a set of contact IDs. Takes integer IDs, not names — resolve
via `contact_profile` or `resolve_prompt_context` first. Returns no transcript bodies.

### `search_transcripts(query="", limit=20, filter_by="", sort_by="")`

Semantic + keyword search across the org's call transcripts.

- `query` — search text (`"pricing"`, `"SOC2 timeline"`). Leave empty to browse without
  relevance ranking.
- `limit` — 1–50, default 20.
- `filter_by` — filter grammar, applied **before** ranking. Available fields:

  | Field | Type | Matches |
  | --- | --- | --- |
  | `name` | string | the call name |
  | `participant_names` | string[] | any participant |
  | `business_names` | string[] | any business tied to the call |
  | `call_date` | int64 | unix seconds |

- `sort_by` — sort grammar. Defaults depend on `query`: with an empty query it
  sorts `call_date:desc` (newest first); with a query it sorts by relevance.

Only these four fields are filterable. Inventing a field name produces an error, not an
ignored clause.

### `call_transcript_conversation(transcript_id, include_structured=False, format="full")`

Fetches one transcript by integer ID.

- `format="full"` (default) — full raw transcript plus a tag-wrapped `llm_string`. Use
  for a single deep dive.
- `format="summary"` — thematic summary only. Use when scanning several calls; pulling
  several transcripts at `full` will bury the actual question in raw text.
- `include_structured=True` — adds `transcript_sentences` with per-sentence timing and
  speaker structure. Only meaningful with `format="full"`.

### `sales_playbook(latest_user_message, conversation_history=None)`

Returns the playbook sections relevant to the current message, rather than the whole
playbook. `latest_user_message` is the user's current message; `conversation_history` is
the prior context, if any.

## Mutating tools

Both are annotated `destructive`, so Claude Code prompts before calling them. Never
instruct the user to pre-approve these.

### `import_contact(linkedin_url=None, email=None, query=None)`

Imports a contact from a LinkedIn profile URL or an email address.

Side effects: queues background enrichment against external data sources and polls for
the new contact for up to ~90s. A slow return is normal.

### `refresh_contact(linkedin_url=None, email=None, query=None)`

Re-enriches an existing contact from its stored LinkedIn URL. See
`skills/refresh-contact/SKILL.md` — the five return statuses mean materially different
things, and only one of them changed any state.
