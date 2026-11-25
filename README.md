<img src="assets/logo.svg" width="72" alt="Sparks logo">

# Sparks MCP

Connect any MCP client (Cursor, Claude, Claude Code, Codex, OpenClaw, Grok Bot, ...) to your
[Sparks](https://sparks.6x7.gr) idea notebook. Your agent can find, capture and organize notes,
projects and subtasks. There is deliberately **no delete tool**.

- **Server:** `https://sparks.6x7.gr/api/mcp` (Streamable HTTP, remote, nothing to install)
- **Sign-in:** OAuth 2.1 with PKCE and dynamic client registration. Add the URL, approve once in Sparks. Agents without OAuth use the one-tap connect link in [`skills/connect-to-sparks`](skills/connect-to-sparks/SKILL.md).

## Install

**Cursor:** install the `sparks` plugin from the Marketplace, or add to `mcp.json`:

```json
{ "mcpServers": { "sparks": { "url": "https://sparks.6x7.gr/api/mcp" } } }
```

**Claude Code:** `claude mcp add --transport http sparks https://sparks.6x7.gr/api/mcp`, then `/mcp` → Authenticate.

**Claude Code plugin:** `/plugin marketplace add 6x7gr/sparks-mcp`, then `/plugin install sparks@sparks`.

**Codex:** add `[mcp_servers.sparks]` with `url = "https://sparks.6x7.gr/api/mcp"`, then `codex mcp login sparks`.

More: <https://sparks.6x7.gr/help/connect-agent>

## Tools

| Tool | What it does | Annotations |
| --- | --- | --- |
| `list_projects` | Your projects with note counts | read-only |
| `list_notes` | Search and filter notes (project, tag, text, done, on hold, sent to agent, updated since) | read-only |
| `get_note` | One note with its projects, full subtask tree, `on_hold` and the latest Send to agent job | read-only |
| `create_note` | Create a note (Inbox or a project) | write, non-destructive |
| `update_note` | Change title, body, tags, done, pinned, due date, urgency, project | write, idempotent |
| `add_subtask` | Add a subtask (up to 4 levels) | write, non-destructive |
| `mark_done` | Mark a note or subtask done / not done | write, idempotent |
| `add_note_to_project` | File a note under a project too | write, idempotent |
| `set_agent_run` | Report a Send to agent job as done or failed | write, idempotent |
| `get_agent_connection_status` | Which Send to agent targets are set up | read-only |
| `set_agent_webhook` | Register a webhook, or a poll inbox for local agents, that Sparks uses on Send to agent | write, idempotent |

All tools have `openWorldHint: false`. Notes tagged `#hold` or `#dont-build` come back with `on_hold: true`: agents read them but don't build them.

### Send to agent

In Sparks, a note's menu can send it to your agent. Agents that opt in (`accepts_intent: true` on `set_agent_webhook`) get labelled jobs: **Plan** and **Research** add subtasks only, **Build** may implement. Webhooks are signed with `X-Sparks-Signature: sha256=<hmac>`. Agents with no public URL use `kind: "inbox"` and poll instead. Details are in the [setup skill](skills/connect-to-sparks/SKILL.md).

## Contents

- `.cursor-plugin/plugin.json`: Cursor plugin manifest
- `.claude-plugin/`: Claude Code plugin and marketplace manifests
- `.codex-plugin/plugin.json`: Codex plugin manifest
- `PUBLISHING.md`: where Sparks is listed and how to submit or update each listing
- `mcp.json`: remote MCP server config
- `skills/connect-to-sparks/`: setup skill for agents
- `server.json`: [MCP Registry](https://registry.modelcontextprotocol.io) entry (`gr.6x7/sparks`)

## Privacy, terms and support

- Privacy: <https://sparks.6x7.gr/privacy>
- Terms: <https://sparks.6x7.gr/terms>
- Support: [help@6x7.gr](mailto:help@6x7.gr)

License: MIT
