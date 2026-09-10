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
- `mentions` / `mention_source` — the people, companies, and calls the request names, as
  read by a small model (`mention_source: "model"`); only those are searched. If that read
  fails the tool falls back to probing each word (`"fallback"`).
- `matched_transcripts_by_title` — when the request names a call, the transcripts whose
  title matches that mention (id, name, date, participants), with the query shown so a
  generic mention ("our call") can be judged.
- `clarification_needed` / `ambiguous_names` — set when a name token matches more than
  one person; each candidate carries `transcript_count`. Ask the user which one; do not
  pick, even if only one has calls on record.

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
  Set only when the shared name is the request's only handle. When something else in the
  request resolved (a full name, a company, a call found by title), `ambiguous_names`
  still lists the candidates but the recommendation says to proceed with the resolved
  target and not to pick one of them.
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
  | `tags` | string[] | the call's tags: `Sales`, `Internal`, `Onboarding`, `Instructional`, … |

- `sort_by` — sort grammar. Defaults depend on `query`: with an empty query it
  sorts `call_date:desc` (newest first); with a query it sorts by relevance.

Only these five fields are filterable. Inventing a field name produces an error, not an
ignored clause. `tags:Sales` keeps to prospect calls; `tags:!=Internal` leaves out your own
team's calls. Every transcript in a list or search result carries its `tags`.

### `call_transcript_conversation(transcript_id, include_structured=False, format="full", part=1)`

Fetches one transcript by integer ID.

- `format="full"` (default) — the tag-wrapped `llm_string`, in parts of 60,000
  characters. The response carries `part`, `parts` and `next_part`; call again with
  `part=next_part` until it is null. Use for a single deep dive, and whenever the request
  is about what was said on one specific call.
- `format="summary"` — thematic summary only. Use when scanning several calls; pulling
  several transcripts at `full` will bury the actual question in raw text.
- `include_structured=True` — adds `transcript_sentences` with per-sentence timing and
  speaker structure. Only meaningful with `format="full"`.

### `sales_playbook(latest_user_message, conversation_history=None, context_hint=None, call_transcript_ids=None, business_ids=None)`

Returns the sections of this org's playbook relevant to the current message, rather than
the whole playbook. `latest_user_message` is the user's current message, passed
**verbatim** — summarizing loses the signal the selector keys on. `conversation_history`
is the prior context, if any; worth passing on later turns, since it is often what
disambiguates a short follow-up like "make it shorter" or "what about for a partner?".

**This is the highest-value tool here.** It is what makes an answer specific to this
company rather than generic sales advice. Call it liberally: retrieval is cheap, it
returns only matching sections, and an empty result costs nothing.

Call it whenever:

- The user asks how to handle, approach, position, pitch, frame, or sell something.
- **You are about to draft anything a prospect or partner will read** — email, LinkedIn
  message, proposal, follow-up, recap. Call it *before* writing, so the draft carries
  this org's positioning and language. Do this even when the user never mentions the
  playbook.
- The user asks what the company does, what makes it different, what its value or
  elevator pitch is, or how to explain it to a particular audience.
- The user asks for discovery questions, talk tracks, next steps, or how to move a deal
  forward.
- The user asks about a competitor, pricing, packaging, or a business case.
- The user asks about partnerships, channel, or co-selling. Some orgs keep a separate
  partnership playbook and this tool selects across all of them, so ask it for
  partner-motion questions exactly as you would for direct-sales ones.

When in doubt, call it. The common failure is not calling it and producing advice that
could have been written about any company.

**Carry over what you already know.** An org can run more than one playbook — typically a
sales playbook for selling directly to a buyer and a partnership playbook for working
through a partner — and a question like "what should I cover on this call?" does not say
which motion is in play. Three optional arguments settle it:

- `context_hint` — a short phrase describing the conversation when the question alone
  does not ("partner conversation with a CTV platform", "direct sales discovery call").
- `call_transcript_ids` — ids of the calls the question is about, if you already resolved
  them via `resolve_prompt_context` or `search_transcripts`. Their names and
  sales/partnership labels are a stronger signal than a freeform hint, and this is the
  most reliable way to land on the right playbook.
- `business_ids` — ids of the businesses the question is about, used the same way.

Without any of these, a vague question on a multi-playbook org can match nothing and come
back `skipped: no_rendered_content`. If that happens, retry ONCE with whatever of the
three you have, and do not retry again after that. Orgs with a single playbook ignore all
three.

The result may also carry a `clarification` field: the motion was not determined, the
sections returned are the ones that hold either way, so use them and then ask the user
that question rather than picking a motion silently.

Not for facts about a specific person or company (`contact_profile` /
`business_profile`), or for what was said on a call (`search_transcripts`).

## Mutating tools

Both are annotated `destructive`, so Claude Code prompts before calling them. Never
instruct the user to pre-approve these.

### `import_contact(linkedin_url=None, email=None, query=None)`

Imports a contact from a LinkedIn profile URL or an email address.

Given only a name (`query="Ben Beal"`), it first checks the org's existing contacts:
`already_on_record` (with `contact_id`) means the person is already in Breakthrough and
nothing was imported; use `contact_profile` or `refresh_contact` instead. `ambiguous_name`
lists `candidates`; ask which one. `invalid_input` means no URL or email was found and
nobody on record matches: ask the user for one, never guess a profile.

An email-only import is held to the name the address carries: a vendor answer whose
name contradicts `john.smith@…` is rejected rather than imported.

Side effects: queues background enrichment against external data sources and polls for
the new contact for up to ~90s. A slow return is normal.

### `refresh_contact(linkedin_url=None, email=None, query=None)`

Re-enriches an existing contact from its stored LinkedIn URL. See
`skills/refresh-contact/SKILL.md` — the five return statuses mean materially different
things, and only one of them changed any state.
