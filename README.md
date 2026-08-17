# Breakthrough for Claude Code

Draft outreach, research calls, and answer questions using your Breakthrough sales
data — contacts, accounts, call transcripts, and your organization's sales playbook.

Everything is scoped to your own Breakthrough organization. The plugin reads your data;
it never posts, sends, or publishes anything on your behalf.

## Install

Install it from your organization's plugin list in `/plugin`. It connects to
Breakthrough automatically once enabled; the first request opens a browser window to
sign in.

## Skills

| Command | What it does | Backing tools |
| --- | --- | --- |
| `/breakthrough:gather-context` | shared prelude: resolve entities, pull evidence and playbook | `whoami`, `resolve_prompt_context`, `contact_profile`, `business_profile`, transcript tools, `sales_playbook` |
| `/breakthrough:draft-email` | writes a sales email | via gather-context |
| `/breakthrough:draft-linkedin-message` | writes a LinkedIn connection request, InMail, or DM | via gather-context |
| `/breakthrough:draft-note` | writes up a call — notes, debrief, MEDDPICC, BANT | via gather-context |
| `/breakthrough:answer` | answers questions needing judgment across the data | via gather-context |
| `/breakthrough:find-contact` | looks up one person | `contact_profile` |
| `/breakthrough:find-business` | looks up one company | `business_profile` |
| `/breakthrough:research-transcripts` | searches and reads call transcripts | `search_transcripts`, `contact_transcripts_list`, `call_transcript_conversation` |
| `/breakthrough:import-contact` | adds a new person | `import_contact` |
| `/breakthrough:refresh-contact` | re-pulls an existing person's details | `refresh_contact` |

Claude selects these from the request, so each description carries literal trigger
phrasings and explicit
hand-offs to its siblings — selection is a comparative judgment, so a description that
only describes itself competes with every neighbour.

`gather-context` is the exception: it sets `disable-model-invocation: true`. It's a
subroutine the drafting and answering skills run first, not something a user asks for,
and leaving it auto-selectable made it compete with all nine others on every
prospect-related request. It still runs directly as `/breakthrough:gather-context` to
preload an account before a working session.

## What it does not do

This brings Breakthrough's **reasoning** to Claude Code against the same production
data. It is not a second Breakthrough client. It cannot write into Breakthrough chat
threads, or create and save Breakthrough Documents. Drafts go to local files.

Web search uses Claude Code's built-in `WebSearch` / `WebFetch`.

**Output is Markdown for every document type**, including emails, LinkedIn messages, and
notes, which the app renders as HTML. See the deviation note in
[`references/output-contracts.md`](references/output-contracts.md).

## Authentication

**Normally** you sign in through the browser on first use. Nothing to configure.

**Without a browser** — for automated or headless use, create an API key on the MCP API
keys page in Breakthrough and register the server yourself:

```bash
claude mcp add --transport http --scope user breakthrough-api https://mcp.breakthroughsales.io/mcp --header "Authorization: Bearer $BREAKTHROUGH_MCP_API_KEY"
```

Keys are scoped to a single Breakthrough seat and can be revoked from the same page.

## Troubleshooting

**`421` from `/mcp` while `/health` returns `200`.** Not an auth failure — the request
is being rejected before it reaches authentication. Report it to the Breakthrough team
rather than regenerating tokens.

**Every tool returns `{"status": "skipped", "reason": "missing_or_invalid_license_context"}`.**
The session is not authenticated against Breakthrough. Confirm with `whoami` — the
returned `license_id` will be missing.

**Changed a skill and nothing happened.** `SKILL.md` edits apply immediately. Changes to
`.mcp.json` or `plugin.json` need `/reload-plugins` or a restart.

## Reporting problems

Send the failing request and what you expected to the Breakthrough team. Skill and tool
behaviour is versioned with the plugin, so include the version from `/plugin`.
