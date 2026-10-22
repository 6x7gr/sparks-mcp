# Directory submissions — Sparks

Status: prepared, **not submitted**. Support: help@6x7.gr

## Claude Connectors Directory (https://claude.ai/directory/manage → Submit new → MCP connector)

| Field | Value |
| --- | --- |
| Name | Sparks |
| Short description | Capture and organize ideas in your Sparks notebook. |
| Long description | Sparks is an idea notebook. This connector lets Claude find, read, create and organize your notes, projects and subtasks, and mark things done. It signs in with OAuth and never deletes anything. |
| Server URL | https://sparks.6x7.gr/api/mcp (Streamable HTTP) |
| Authentication | OAuth 2.1 (PKCE S256, dynamic client registration, refresh tokens), scope `mcp` |
| Documentation | https://sparks.6x7.gr/help/connect-agent |
| Privacy policy | https://sparks.6x7.gr/privacy |
| Terms | https://sparks.6x7.gr/terms |
| Support | help@6x7.gr |
| Icon | https://sparks.6x7.gr/logo.png (512×512), https://sparks.6x7.gr/logo.svg |

### Tools

| Tool | Title | Annotations |
| --- | --- | --- |
| list_projects | List projects | readOnly, idempotent |
| list_notes | List notes | readOnly, idempotent |
| get_note | Get note | readOnly, idempotent |
| get_agent_connection_status | Agent connection status | readOnly, idempotent |
| create_note | Create note | write, destructive: false |
| add_subtask | Add subtask | write, destructive: false |
| update_note | Update note | write, destructive: false, idempotent |
| mark_done | Mark done | write, destructive: false, idempotent |
| add_note_to_project | Add note to project | write, destructive: false, idempotent |
| set_agent_webhook | Set agent webhook | write, destructive: false, idempotent |

All tools: openWorldHint false. No delete tool exists.

### Example prompts
1. "What's in my Sparks Inbox?"
2. "Save this idea to my Sparks project 'Product ideas': …"
3. "Show the subtasks of my note about the launch and mark the first one done."
4. "Which of my Sparks notes are due this week?"

### Reviewer test account (needs approval before creating)
- Account: review@6x7.gr, strong password entered only in the portal.
- Seed: 3 projects (Product ideas, Travel, Reading list), ~12 notes with tags, due dates, one pinned,
  2–3 levels of subtasks, some done, 2 Inbox notes.

### Compatibility checks (2026-10-22)
- `/api/mcp` → 401 with `WWW-Authenticate: Bearer resource_metadata=…, scope="mcp"` ✓
- Protected resource + authorization server metadata ✓ (`refresh_token` listed)
- DCR with redirect `https://claude.ai/api/mcp/auth_callback` ✓
- Still to do: add as custom connector in Claude.ai and call each tool.

## Cursor Marketplace
Paste https://github.com/6x7gr/sparks-mcp at https://cursor.com/marketplace/publish.
Optional local test: copy this repo to `~/.cursor/plugins/local/sparks`, reload Cursor.

## MCP Registry (`gr.6x7/sparks`)
1. Add TXT record on 6x7.gr: `v=MCPv1; k=ed25519; p=<public key>`.
2. `mcp-publisher login dns --domain 6x7.gr --private-key <hex>`
3. `mcp-publisher publish` (from this repo).
