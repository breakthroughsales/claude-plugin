---
name: humanize
description: >-
  Strips the patterns that make writing read as machine-generated: em dashes, "not just
  X but Y" constructions, promotional adjectives, participle padding, forced triplets,
  filler, and hedging. Keeps every fact and the author's voice. Runs automatically at
  the end of any email or LinkedIn message these skills produce, and can be pointed at
  text the user pastes in.
when_to_use: >-
  Use when the user says "make this sound less like AI", "this reads like ChatGPT",
  "humanize this", "make it sound like me", "less robotic", "too formal", or pastes a
  draft and asks for a rewrite. Do NOT use to change what a draft says or to shorten it
  to a target length; that is a revision, and draft-email or draft-linkedin-message owns
  it.
---

# Humanize a draft

Read `${CLAUDE_PLUGIN_ROOT}/references/humanizing.md` for the full pattern catalogue.
It also carries the "what not to flag" list, which matters as much as the patterns:
over-editing strips out the specifics that make writing sound like a person.

## When this runs on its own

The drafting skills invoke this as their last step, so you rarely need to call it
directly. Call it when the user hands you text and asks for it to sound less
machine-written.

## Process

1. **Draft the rewrite.** Read it aloud in your head. Sentence lengths should vary.
   Prefer `is` and `has` over `serves as` and `boasts`.

2. **Audit your own draft** against two questions:
   - What in this still reads as AI-generated?
   - Does it state any fact, name, number, date, or quote that was not in the source?

3. **Revise.** Then scan for `—` and `–`. Any hit means it is not finished.

## The rule that matters most here

**Never invent facts.** A humanized sentence that sounds natural and asserts something
untrue about a real prospect is a worse outcome than the stiff version, because it goes
out under the user's name to someone who knows whether it is true.

If a sentence needs a detail you do not have, write the plain version or ask. Vagueness
is recoverable; a confident fabrication in a sent email is not.

## Voice

If the user's own writing is in the conversation, or they supply a sample, match its
habits rather than applying the default rules. A sample outranks everything in the
reference file, including the no-dashes rule.

Otherwise match the register the draft is already in. This pass changes how the writing
sounds, not what it says or how formal it is.

## Output

Return only the final text. No draft, no list of what you changed, no preamble. When
this runs as the last step of a drafting skill, the user should see one message, not a
rewrite log.
