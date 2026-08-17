---
name: find-contact
description: >-
  Looks up one person in Breakthrough by name, email address, or LinkedIn URL and
  returns their stored record — role, employer, linked company. Answers "is this person
  already in the system" and pins down which person the user means before acting.
when_to_use: >-
  Use when the user says "who is this", "do we have Jane", "look up jane@acme.com",
  "what's her title", "is he in the system", "which Jane do we know", "pull up her
  record". Do NOT use to add someone new; that is import-contact. Do NOT use to update
  a record that has gone stale; that is refresh-contact. Do NOT use for what was said
  on calls with them; that is research-transcripts. Do NOT use for questions needing
  judgment about the person rather than their record; that is answer.
---

# Find a contact

Wraps the `contact_profile` MCP tool.

## Usage

Pass the user's phrasing as `query`. The tool extracts embedded email addresses and
LinkedIn URLs itself, so `"do we have anything on jane@acme.com"` works as well as
`"jane@acme.com"` — no need to strip the sentence down first.

`limit_candidates` defaults to 5. Raise it when a common name returns an ambiguous set.

When the request is vague about who is meant, call `resolve_prompt_context` first — it
cross-references contacts against businesses and returns concrete recommendations, which
disambiguates "the CTO at that fintech" better than a name search can.

## Reporting

Report what resolved: full name, contact ID, role, employer, LinkedIn URL.

**If several candidates come back, present them and ask.** Do not pick the top hit and
proceed — the downstream action is usually an email to a real person, and picking wrong
means writing to the wrong one.

**If nothing resolves, say so plainly.** "No contact matching that in Breakthrough" is
the useful answer. Do not fall back to a web search and present what you find as if it
came from the CRM; the user asked what *they* have on file. Offer the import path
(`/breakthrough:import-contact`) if a LinkedIn URL or email is available.
