# Breakthrough for Claude

Draft outreach, research calls, and answer questions using your Breakthrough sales
data — contacts, accounts, call transcripts, and your organization's sales playbook.

Everything is scoped to your own Breakthrough organization. The plugin reads your data;
it never posts, sends, or publishes anything on your behalf.

## Install

The plugin is distributed through the public marketplace repository
[`breakthroughsales/claude-plugin`](https://github.com/breakthroughsales/claude-plugin).
You add that marketplace to **your own account** — there is no shared or organization-wide
install, so everyone who wants the plugin does this once.

### Claude Desktop and claude.ai

1. **Customize** in the left nav, then the **Plugins** tab.
2. **Add** (top right) → **Add marketplace** → **Add from a repository**.
3. Enter `breakthroughsales/claude-plugin`. The owner is **breakthroughsales**.
4. **Sync**. Breakthrough then appears under **Discover**; click **Add**.

Leave **Sync automatically** on to receive updates as they are published.

### Claude Code

```bash
claude plugin marketplace add breakthroughsales/claude-plugin
claude plugin install breakthrough@breakthrough
```

Or interactively: `/plugin marketplace add breakthroughsales/claude-plugin`, then `/plugin`.

Either way, it connects to Breakthrough automatically once enabled; the first request
opens a browser window to sign in. If you have already added Breakthrough as a custom
connector, the plugin reuses that existing connection rather than adding a second one.

## Skills

Claude selects these from your request, so you rarely need to type one. Command names
differ by surface: in Claude Desktop and claude.ai they are unprefixed (`/draft-email`),
while Claude Code namespaces them per plugin (`/breakthrough:draft-email`).

| Skill | What it does | Backing tools |
| --- | --- | --- |
| `gather-context` | shared prelude: resolve entities, pull evidence and playbook | `whoami`, `resolve_prompt_context`, `contact_profile`, `business_profile`, transcript tools, `sales_playbook` |
| `draft-email` | writes a sales email | via gather-context |
| `draft-linkedin-message` | writes a LinkedIn connection request, InMail, or DM | via gather-context |
| `draft-note` | writes up a call — notes, debrief, MEDDPICC, BANT | via gather-context |
| `answer` | answers questions needing judgment across the data | via gather-context |
| `humanize` | strips the patterns that make writing read as machine-generated; runs automatically on every outbound draft | none |
| `find-contact` | looks up one person | `contact_profile` |
| `find-business` | looks up one company | `business_profile` |
| `research-transcripts` | searches and reads call transcripts | `search_transcripts`, `contact_transcripts_list`, `call_transcript_conversation` |
| `import-contact` | adds a new person | `import_contact` |
| `refresh-contact` | re-pulls an existing person's details | `refresh_contact` |

Because Claude selects skills by comparing their descriptions, each description carries
literal trigger phrasings and explicit hand-offs to its siblings — a description that only
describes itself competes with every neighbour.

`gather-context` is the exception: it sets `disable-model-invocation: true`. It's a
subroutine the drafting and answering skills run first, not something a user asks for,
and leaving it auto-selectable made it compete with all nine others on every
prospect-related request. You can still run it directly to preload an account before a
working session.

## What it does not do

This brings Breakthrough's **reasoning** to Claude against the same production data. It is
not a second Breakthrough client. It cannot write into Breakthrough chat threads, or
create and save Breakthrough Documents. Drafts go to your conversation or to local files.

Web search uses Claude's own built-in search.

**Output is Markdown for every document type**, including emails, LinkedIn messages, and
notes, which the app renders as HTML. See the deviation note in
[`references/output-contracts.md`](references/output-contracts.md).

## Authentication

**Normally** you sign in through the browser on first use. Nothing to configure.

**Without a browser** — for automated or headless use, create an API key on the MCP API
keys page in Breakthrough and register the server yourself:

```bash
claude mcp add --transport http --scope user breakthrough https://mcp.breakthroughsales.io/mcp --header "Authorization: Bearer $BREAKTHROUGH_MCP_API_KEY"
```

Keys are scoped to a single Breakthrough seat and can be revoked from the same page.

## Troubleshooting

**"Repository not found" when adding the marketplace.** Check the owner. It is
`breakthroughsales/claude-plugin`; `breakthrough/claude-plugin` does not exist.

**A teammate has it and you don't see it.** Marketplaces are added per account, not per
company. Add it yourself with the steps above.

**`421` from `/mcp` while `/health` returns `200`.** Not an auth failure — the request
is being rejected before it reaches authentication. Report it to the Breakthrough team
rather than regenerating tokens.

**Every tool returns `{"status": "skipped", "reason": "missing_or_invalid_license_context"}`.**
The session is not authenticated against Breakthrough. Confirm with `whoami` — the
returned `license_id` will be missing.

**Changed a skill and nothing happened.** Skill edits apply immediately. Changes to the
plugin's server or manifest configuration need `/reload-plugins` or a restart.

## Reporting problems

Send the failing request and what you expected to the Breakthrough team. Skill and tool
behaviour is versioned with the plugin, so include the version shown on the plugin's page.
