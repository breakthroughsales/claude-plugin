---
name: find-business
description: >-
  Looks up one company in Breakthrough by name, website domain, or LinkedIn company URL
  and returns its stored profile. Answers "is this account already in the system" and
  pins down which company the user means before acting.
when_to_use: >-
  Use when the user says "pull up Acme", "do we have anything on acme.com", "look up
  this company", "are we already working with them", "what do we have on that account".
  Do NOT use to find a person; that is find-contact. Do NOT use for what was said on
  calls with the account; that is research-transcripts. Do NOT use for questions
  needing judgment about the account rather than its record; that is answer.
---

# Find a business

Wraps the `business_profile` MCP tool.

## Usage

Pass the user's phrasing as `query`. The tool extracts domains and LinkedIn company URLs
from the text itself, so a full sentence works.

`limit_candidates` defaults to 5.

For "who do we know at X", call `resolve_prompt_context` instead — it cross-references
businesses to contacts in one pass, which `business_profile` alone does not do.

## Reporting

Report the resolved business and, where the data includes them, the contacts tied to it.

If several candidates come back, present them and ask. If nothing resolves, say so —
don't substitute web results for CRM data without labelling them as such.
