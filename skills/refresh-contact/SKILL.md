---
name: refresh-contact
description: >-
  Re-pulls an EXISTING Breakthrough contact's details from their LinkedIn profile when
  the stored record has gone stale, wrong, or out of date. Confirms with the user before
  re-pulling when the mention was incidental rather than a direct instruction.
when_to_use: >-
  Use for direct requests ("refresh Jane", "update his title"); complaints that a record
  is wrong ("her info is out of date", "this looks stale", "you have him at his old
  company"); reports of a job or title change in ANY phrasing, including offhand and
  declarative ones ("Sarah left Acme", "he's at Globex now", "she's the CEO now", "FYI
  they moved on"); and currency questions ("is this current?", "do we have anything
  newer on her?"). Prefer this over answer whenever a contact's employer or title is
  reported changed, stale, or questioned — the tool confirms before changing anything,
  so routing here is safe. Do NOT use for someone not yet in the system; that is
  import-contact.
---

# Refresh a contact

Wraps the `refresh_contact` MCP tool.

## When this applies

The backend routes broadly to refresh on purpose, because the tool confirms before
changing anything. All of these belong here:

- **Explicit commands** — "refresh Jane's contact info", "update this contact's title"
- **Stale-data complaints** — "John's data is wrong", "you have her at her old employer"
- **Employer or title changes, however phrased** — including passive and note-style
  mentions with no refresh verb at all: "Sarah no longer works at Acme", "he's at Globex
  now", "FYI Sarah left Acme"
- **Currency questions** — "is his info up to date?", "do you have more current data?"

## Calling

`refresh_contact(linkedin_url=None, email=None, query=None)`. Pass the user's message as
`query` when you have no identifier — resolution is the tool's job.

Annotated **destructive**; Claude Code prompts before it runs. Don't ask the user to
pre-approve it.

## Handling the result — read this before reporting anything

Five statuses. **Only `started` changed any state.** Every other status is a safe no-op
that enqueued nothing.

| Status | What happened | What you must do |
| --- | --- | --- |
| `started` | Refresh was **enqueued** | Report it's queued. Enrichment is *not* complete — don't present current field values as refreshed. |
| `needs_confirmation` | One contact resolved, but from an **implicit** signal | **Stop and ask the user before refreshing.** Show which contact you matched. Do not call again until they confirm. |
| `ambiguous` | Several candidates matched | Present the candidates and ask which. Do not pick one. |
| `no_linkedin_url` | Contact found, but no stored URL to re-enrich from | Report it. Retrying will not help. |
| `no_match` | Nothing resolved | Report it. Offer `/breakthrough:import-contact` if they have a URL or email. |

### Why `needs_confirmation` exists

Breakthrough deliberately separates an explicit refresh *command* from an implicit
*signal*. A passing mention — a note, a complaint, a question about currency — resolves
the contact but stops short of refreshing, so a stray mention never silently triggers
a re-pull.

There is a second deliberate case. Phrasing that asserts a **new value** — "update
Marco's employer, he's now at Kleecks" — returns `needs_confirmation` rather than
queueing. Refresh only re-reads LinkedIn; it does not write the value the user just
stated, and LinkedIn may not reflect it yet. So confirm rather than implying the change
was applied.

Collapsing these statuses into "done" defeats the guardrail. Report the one you got.

## Reporting

Name the contact you matched — full name, ID, current role and employer — so the user can
catch a wrong match before anything is re-enriched.

For `started`, say the refresh is queued and data will update, not that it is updated.
