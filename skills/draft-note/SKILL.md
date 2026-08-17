---
name: draft-note
description: >-
  Writes up a call as a document — meeting notes, a debrief, action items, or a
  structured MEDDPICC or BANT analysis — from the user's recorded Breakthrough call
  transcripts. Organized around what was decided and what happens next, so it stays
  useful to someone reading it weeks later.
when_to_use: >-
  Use when the user says "write up the call", "call notes", "meeting summary", "recap",
  "debrief", "action items", "MEDDPICC", "BANT", "notes for the team", or is prepping a
  deal review. Do NOT use to answer a question about what was said; that is retrieval,
  use research-transcripts. Do NOT use for anything sent to the prospect; that is
  draft-email or draft-linkedin-message.
---

# Draft a note

Covers call notes, meeting summaries, MEDDPICC and BANT analyses, action items, and
debriefs.

## Steps

**1. Gather context.** Follow `${CLAUDE_PLUGIN_ROOT}/skills/gather-context/SKILL.md`.

Notes are the one document type where the transcript *is* the source material, not
background. Read the actual calls — `call_transcript_conversation` with `format="full"`
for a single call, `format="summary"` when synthesizing across several. Working from
search snippets alone produces a note that sounds right and misses what was decided.

**2. Write to the contract** in `${CLAUDE_PLUGIN_ROOT}/references/output-contracts.md`:

- **500 words maximum** unless the user explicitly asked for more
- Markdown, not HTML — see the deviation note in the contracts file

## Structure

Notes get read later by someone deciding what to do next — often the user, weeks on.
Lead with what was **decided** and what happens **next**. Narrative recap goes below
that, or not at all.

When the user names a framework, use that framework's real sections rather than generic
headings. MEDDPICC means Metrics, Economic buyer, Decision criteria, Decision process,
Paper process, Identify pain, Champion, Competition — not "Key Points" and "Next Steps"
with the framework mentioned in passing.

## Grounding

Every claim traces to the transcript. Attribute who said what when it matters —
"the economic buyer is the VP Eng" and "Dana said she thinks the VP Eng signs off" are
different findings, and a deal review acts on them differently.

Where the call left a framework slot empty, say it's unestablished. A MEDDPICC with an
invented Champion is worse than one with a blank Champion, because the blank prompts the
question and the invention forecloses it.

## Delivering

Output the note. If they want it saved, write it to a local file — the plugin cannot
create Breakthrough Documents.
