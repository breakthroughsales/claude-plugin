---
name: draft-email
description: >-
  Writes a sales email for the user to send — cold outreach, a follow-up after a call,
  re-engagement, or a revision of an earlier draft. Grounds it in the recipient's
  Breakthrough record, past call transcripts, and the org's sales playbook so it cites
  real specifics instead of generic copy.
when_to_use: >-
  Use when the user says "write an email", "draft something to", "follow up with",
  "reach out to", "send them a note", "email her about", or asks to revise a draft
  ("make it shorter", "warmer", "more direct", "try again"). Do NOT use for LinkedIn
  messages; that is draft-linkedin-message. Do NOT use for call notes or MEDDPICC
  write-ups; that is draft-note. Do NOT use when the user is asking a question and
  does not want an email produced; that is answer.
---

# Draft an email



## Steps

**1. Gather context.** Follow `${CLAUDE_PLUGIN_ROOT}/skills/gather-context/SKILL.md`. An
email to a contact you haven't resolved is a guess — resolve them, read the calls, pull
the playbook.

**2. Apply the voice rules** in `${CLAUDE_PLUGIN_ROOT}/references/context-assembly.md`.
These matter more here than anywhere else: cold email is where AI phrasing is most
obvious and most costly. The system prompt names the offenders — "I noticed…", "Given
your…", "I've been following", "caught my eye" — and bans a specific word list. Read it.

**3. Write to the contract** in `${CLAUDE_PLUGIN_ROOT}/references/output-contracts.md`:

- **120 words maximum** unless the user explicitly asked for more
- Concise and professional; clarity over cleverness
- Match the user's communication style or specified tone if you can detect it
- Markdown, not HTML — see the deviation note in the contracts file

**Writing as the user.** When the email leans on who *they* are — shared background, their
own experience, why they specifically are reaching out — call `whoami(detail="full")` for their
LinkedIn background. It reads cached data only, so it is cheap. Skip it when the email doesn't turn on the sender's background.

## Grounding

Every specific claim should trace to something retrieved: a line from a transcript, a
field on the contact record, a section of the playbook. A detail from a real call is the
whole reason this email goes through Breakthrough instead of being written cold.

If retrieval came back thin, say so and ask rather than inventing a hook. A plausible
fabricated detail about a real prospect is the worst possible output here — it reads as
confident and it is wrong in front of a customer.

## Humanize before delivering

**Required final step.** Run `${CLAUDE_PLUGIN_ROOT}/skills/humanize/SKILL.md` over the
draft before you show it to anyone. This email goes to a real prospect under the user's
name, and the patterns that mark writing as machine-generated cost the reply regardless
of what the message says.

The humanize pass changes how it reads, not what it claims. If it wants to sharpen a
vague sentence, the specific has to come from the retrieved data, never from invention.

Show the user one message: the finished email. Not a before-and-after, not a list of
what changed.

## Delivering

Output the draft. Do not send it — no MCP tool sends mail, and sending is the user's
call. If they want it saved, write it to a local file; the plugin cannot create
Breakthrough Documents.
