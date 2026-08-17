# Output contracts by document type

Length, style, and format rules for each kind of thing these skills produce. They match
what Breakthrough itself applies, so a draft written here reads like one written in the
app.

## Deliberate deviation: format

**In the app, emails, LinkedIn messages, and notes render as HTML**, because that content
feeds a rich preview pane and gets pushed on to email clients and CRMs.

**This plugin emits Markdown for every document type.** There is no preview pane and no
CRM push in Claude Code; output goes to a terminal and to local files. Emitting
`<html>` wrappers here would produce something the user has to strip by hand before it
is useful.

This is the one place the plugin knowingly departs from the shipped prompts. Everything
below — voice, length, structure — is ported as-is. If you paste plugin output into
Breakthrough and need HTML, convert at that point.

## Email

Length: **120 words maximum** unless the user explicitly asks for more.

Style guidance, injected whenever the request mentions "email" or "e-mail":

> Keep emails concise and professional. Focus on clarity and brevity. If you can detect
> the user's communication style, or they have specified a tone, use it.

Apply the voice rules from `context-assembly.md` with particular force here. Openers like
"I noticed…" or "Given your…" are exactly the phrasings the system prompt calls out, and
cold email is where they surface most.

Ground the specifics. A detail pulled from a real transcript is the entire value of
sending this through Breakthrough rather than writing it cold.

## LinkedIn message

Length: **60–80 words.**

Covers InMail, connection requests, and DMs. Brevity is the point — at this length one
wasted opening line is a tenth of the message.

## Note

Length: **500 words maximum** unless the user explicitly asks for more.

Covers call notes, meeting summaries, MEDDPICC analyses, BANT analyses, action items,
and debriefs.

Notes are read later by someone deciding what to do next — often the user, weeks on.
Lead with what was decided and what happens next; put narrative recap below that, or
leave it out. When the note is a named framework (MEDDPICC, BANT), use that framework's
actual sections rather than generic headings.

Ground every claim in the transcript. A note that asserts something the call did not
establish is worse than a shorter note.

## Answers and general chat

No length cap beyond the general conciseness rules. This is the default: questions,
general chat, brainstorming, explanations.

Conversational questions get conversational answers — typically 1–5 sentences unless
complexity clearly warrants more. Answer, then stop.

## Saved Breakthrough Documents

Not supported here. Proposals, 1-pagers, playbooks, and battle cards are saved inside
Breakthrough, and this plugin cannot write them.

If a user asks for a proposal, 1-pager, playbook, or battle card, say plainly that the
plugin cannot save Breakthrough Documents, then offer to write it to a local file
instead. Do not silently produce one as a `note`.
