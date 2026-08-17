---
name: answer
description: >-
  Answers questions about a prospect, account, deal, or call that need reasoning across
  the user's Breakthrough sales data (contacts, companies, recorded call transcripts,
  and a sales playbook) rather than a single lookup. The fallback for any Breakthrough
  request that isn't a record lookup or a written artifact.
when_to_use: >-
  Use for open-ended questions about an account, person, or deal — "what do we know
  about them", "should we chase this", "how do I handle the pricing objection", "how do
  we position against X", "what should I do next here", "help me prep for this call".
  Do NOT use to pull up one person or company record; that is find-contact or
  find-business. Do NOT use for "what did they say on the call"; that is
  research-transcripts. Do NOT use when the user wants something written to send or
  save; that is draft-email, draft-linkedin-message, or draft-note.
---

# Answer a question

The default for questions, general chat, brainstorming, and explanations.

## Steps

**1. Gather context** when the question touches a person, company, or call the user
knows: follow `${CLAUDE_PLUGIN_ROOT}/skills/gather-context/SKILL.md`.

Skip retrieval for questions that don't reference the user's data — "what's a good
discovery question for a security buyer?" needs the playbook at most, not entity
resolution.

For questions about the user themselves — their background, experience, or how to
position who they are — call `whoami(detail="full")` rather than the default
`whoami()`. The default returns bare scope; `detail="full"` adds their LinkedIn
headline, summary, experience, and education. It reads cached data only, so it is
cheap and never prompts.

**2. Answer to the contract** in
`${CLAUDE_PLUGIN_ROOT}/references/output-contracts.md`:

- No hard length cap, but the conciseness rules bind. Conversational questions get
  conversational answers — typically 1–5 sentences.
- Answer, then stop. No recap of the question, no "additional considerations" section,
  no "let me know if you need anything else."
- State each point once.
- Markdown.

## Web search

Use Claude Code's `WebSearch` / `WebFetch` under the rules in
`${CLAUDE_PLUGIN_ROOT}/references/context-assembly.md`. The one that gets violated most:
**do not web-search a question about a call the user had.** Pull the transcript. If the
transcript isn't in scope, say so and offer to find it — substituting a web search for
the user's own call data is the specific failure the system prompt calls out.

## Grounding

Distinguish what the data shows from what you're inferring. "They raised a Series B in
March" and "they're probably budget-constrained this quarter" are different claims and
the user is making decisions on both.

If the question can't be answered from what's in scope, say what's missing and what would
answer it.
