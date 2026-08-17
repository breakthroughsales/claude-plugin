---
name: draft-linkedin-message
description: >-
  Writes a LinkedIn connection request, InMail, or DM for the user to send, or revises
  one. Grounds it in the recipient's Breakthrough record and past calls, and can draw
  on the user's own LinkedIn background to find shared history worth opening on. Holds
  to 60-80 words.
when_to_use: >-
  Use when the user says "LinkedIn message", "connection request", "InMail", "DM him",
  "message her on LinkedIn", "send a connection note", or asks to shorten or rework
  one. Do NOT use for email; that is draft-email. Do NOT use for internal call
  write-ups; that is draft-note.
---

# Draft a LinkedIn message

Covers InMail, connection requests, and DMs.

## Steps

**1. Gather context.** Follow `${CLAUDE_PLUGIN_ROOT}/skills/gather-context/SKILL.md`.

**2. Apply the voice rules** in `${CLAUDE_PLUGIN_ROOT}/references/context-assembly.md`.
The banned-phrase list applies with full force — at 60–80 words, a single "I noticed
your recent…" opener is a tenth of the message.

**3. Write to the contract** in `${CLAUDE_PLUGIN_ROOT}/references/output-contracts.md`:

- **60–80 words.** LinkedIn messages stay short to hold attention.
- Markdown, not HTML — see the deviation note in the contracts file.

## Grounding

Same rule as email, tighter budget. One specific, real detail beats three generic ones.
If retrieval came back thin, ask rather than inventing a hook.

Shared background is the strongest hook LinkedIn offers — same employer, same school,
overlapping years. Call `whoami(detail="full")` when you intend to use one, so the connection is
real rather than asserted. It reads cached data only, so it is cheap.

## Humanize before delivering

**Required final step.** Run `${CLAUDE_PLUGIN_ROOT}/skills/humanize/SKILL.md` over the
draft. At 60 to 80 words there is nowhere for a stock phrase to hide, so this matters
more here than in email, not less.

Watch the word budget: the pass usually shortens, but if it pushes you over 80, cut
content rather than restoring the padding.

Show the user one message: the finished draft.

## Delivering

Output the draft. Do not send it. If they want it saved, write it to a local file.
