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

**Codex:** add `[mcp_servers.sparks]` with `url = "https://sparks.6x7.gr/api/mcp"`, then `codex mcp login sparks`.

More: <https://sparks.6x7.gr/help/connect-agent>

## Tools

| Tool | What it does | Annotations |
| --- | --- | --- |
| `list_projects` | Your projects with note counts | read-only |
| `list_notes` | Search/filter notes | read-only |
| `get_note` | One note with projects and full subtask tree | read-only |
| `create_note` | Create a note (Inbox or a project) | write, non-destructive |
| `update_note` | Change title, body, tags, done, pinned, due date, project | write, idempotent |
| `add_subtask` | Add a subtask (up to 4 levels) | write, non-destructive |
| `mark_done` | Mark a note or subtask done / not done | write, idempotent |
| `add_note_to_project` | File a note under a project too | write, idempotent |
| `get_agent_connection_status` | Whether a Send-to-agent webhook is set | read-only |
| `set_agent_webhook` | Register the webhook Sparks calls on "Send to agent" | write, idempotent |

## Contents

- `.cursor-plugin/plugin.json`: Cursor plugin manifest
- `mcp.json`: remote MCP server config
- `skills/connect-to-sparks/`: setup skill for agents
- `server.json`: [MCP Registry](https://registry.modelcontextprotocol.io) entry (`gr.6x7/sparks`)

## Privacy, terms and support

- Privacy: <https://sparks.6x7.gr/privacy>
- Terms: <https://sparks.6x7.gr/terms>
- Support: [help@6x7.gr](mailto:help@6x7.gr)

License: MIT
