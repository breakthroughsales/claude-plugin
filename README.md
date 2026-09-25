# Breakthrough

Connects your AI assistant (like Claude, Codex or Cursor) to a living playbook distilled
from your sales calls, so it always has the right context and knows what works best across
your organization, without you having to manage any of it yourself.

Everything is scoped to your own Breakthrough organization. The plugin reads your data;
it never posts, sends, or publishes anything on your behalf.

## Install

The plugin is distributed through the public marketplace repository
[`breakthroughsales/plugin`](https://github.com/breakthroughsales/plugin). You add that
marketplace to **your own account** — there is no shared or organization-wide install, so
everyone who wants the plugin does this once.

Skills and tools install together wherever the host supports plugins — Claude, ChatGPT/Codex
and Cursor. Claude works either way, desktop or web. ChatGPT has one catch: **chatgpt.com in
a browser cannot add a marketplace**, so use the ChatGPT desktop app, which installs against
your account and so shows up in the browser too. The Codex CLI can also install it, but that
install is local to that machine and does not reach your account or the browser.

### Claude Desktop and claude.ai

1. **Customize** in the left nav, then the **Plugins** tab.
2. **Add** (top right) → **Add marketplace** → **Add from a repository**.
3. In **URL**, enter `breakthroughsales/plugin`, then **Sync**.
4. Breakthrough appears under **Discover**; click **Add**.

Leave **Sync automatically** on to receive updates as they are published.

### Claude Code

```bash
claude plugin marketplace add breakthroughsales/plugin
claude plugin install breakthrough@breakthrough
```

Or interactively: `/plugin marketplace add breakthroughsales/plugin`, then `/plugin`.

### ChatGPT and Codex

Use the **ChatGPT desktop app** — the Codex section of it, which is where plugins live now.
The browser at chatgpt.com can only browse already-installed plugins; it has no marketplace
option.

1. **Plugins** in the left nav.
2. **Add** (top right) → **Add a marketplace**.
3. Enter `breakthroughsales/plugin`, then install Breakthrough from the list.

Or from the Codex CLI, for that machine only — both commands are needed, the first just
registers the marketplace and leaves the plugin uninstalled:

```bash
codex plugin marketplace add breakthroughsales/plugin
codex plugin add breakthrough@breakthrough
```

`codex plugin list` confirms it installed. Interactively, `/plugins` opens the same browser
with marketplace tabs to switch sources.

This route writes to the local `CODEX_HOME` and nothing else. Only OpenAI's own remote
marketplaces install against your ChatGPT account, so a plugin added this way works in that
CLI and will not appear on chatgpt.com — use the desktop app if you want it there.

Codex reads the same marketplace file this repository already publishes, so the skills and
the MCP server install as one unit, the same as on Claude.

**Browser only?** Add the MCP server as a connector instead and you get the tools without the
skills: **Settings** → **Connectors** → **Advanced** → **Developer Mode**, then **Create**
with `https://mcp.breakthroughsales.io/mcp` and **OAuth**. Requires a paid plan (Plus, Pro,
Business, Enterprise or Edu).

### Cursor

**Plugins** → **+ Add** → **From GitHub Repository**, then enter
`breakthroughsales/plugin`.

Cursor bundles rules, skills, subagents, commands, MCP servers and hooks into one
installable package, so the skills and the MCP server arrive together as they do on Claude
and Codex.

### First run

Whichever host you use, it connects to Breakthrough automatically once enabled; the first
request opens a browser window to sign in. If you have already added Breakthrough as a
custom connector, the plugin reuses that existing connection rather than adding a second
one.

## Skills

Your assistant selects these from your request, so you rarely need to type one. Command
names differ by surface: in Claude Desktop and claude.ai they are unprefixed
(`/draft-email`), while Claude Code namespaces them per plugin
(`/breakthrough:draft-email`).

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

Because the assistant selects skills by comparing their descriptions, each description carries
literal trigger phrasings and explicit hand-offs to its siblings — a description that only
describes itself competes with every neighbour.

`gather-context` is the exception: it sets `disable-model-invocation: true`. It's a
subroutine the drafting and answering skills run first, not something a user asks for,
and leaving it auto-selectable made it compete with all nine others on every
prospect-related request. You can still run it directly to preload an account before a
working session.

## What it does not do

This brings Breakthrough's **reasoning** to your assistant against the same production
data. It is not a second Breakthrough client. It cannot write into Breakthrough chat threads, or
create and save Breakthrough Documents. Drafts go to your conversation or to local files.

Web search uses the assistant's own built-in search.

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
`breakthroughsales/plugin`; `breakthrough/plugin` does not exist.

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
