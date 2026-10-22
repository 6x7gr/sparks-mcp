---
name: connect-to-sparks
description: Connect this agent to the user's Sparks notebook (sparks.6x7.gr) - add the Sparks MCP server (OAuth or a one-tap connect link, no token copying) and register this agent's webhook so "Send to agent" wakes it. Use when the user says "connect to Sparks" or similar.
---

# Connect to Sparks

Sparks is the user's idea notebook. After this skill you can read/update their notes over MCP, and
Sparks can wake you when they tap **Send to agent** on a note. Never ask the user to copy a token.

MCP URL: `https://sparks.6x7.gr/api/mcp` (Streamable HTTP).

## 1a. If your MCP client supports OAuth (Claude Code, Cursor, Codex, MCP Inspector, ...)

Add the server by URL only, with no headers. The client discovers OAuth from the 401
(`/.well-known/oauth-protected-resource` → `/.well-known/oauth-authorization-server`), registers itself
(dynamic client registration), and opens a Sparks page where the user taps **Allow** (PKCE, S256).
Examples: `claude mcp add --transport http sparks https://sparks.6x7.gr/api/mcp` then `/mcp` → Authenticate;
Cursor `mcp.json` `{ "mcpServers": { "sparks": { "url": "https://sparks.6x7.gr/api/mcp" } } }`;
Codex `[mcp_servers.sparks] url = "..."` then `codex mcp login sparks`.

## 1b. Otherwise: connect link (device-style flow)

1. `POST https://sparks.6x7.gr/api/agent/connect/start` with JSON `{"name": "<your agent name>"}`.
   Response: `{ code, approve_url, poll_token, poll_url, expires_in: 900, interval: 5 }`.
2. Show the user `approve_url` and `code` ("Tap this link and approve; check the code is <code>").
   Keep `poll_token` private (secret store / 0600 file); never show it.
3. Every `interval` seconds, `POST poll_url` with `{"poll_token": "..."}` until `status` is not `pending`:
   - `approved` → response has `token` (`spk_mcp_…`, shown only once) and `mcp_url`. Store the token as a secret.
   - `denied` / `expired` / `claimed` / `invalid` → stop and tell the user (start again if they want).
4. Add the MCP server with header `Authorization: Bearer <token>`.

Verify either way by calling `list_projects`.

## 2. Register your webhook (for Send to agent)

If your platform can receive webhooks:
1. Create or reuse an inbound webhook/trigger that starts a run on POST. Sparks sends JSON
   `{ "note_id", "title", "project_id", "url" }`.
2. Call MCP tool `set_agent_webhook` with `url` (public https, port 443), `secret` (if your webhook needs one)
   and `auth_header` (default `Authorization`, sent as `Bearer <secret>`; e.g. `X-Webhook-Secret` otherwise).
3. Call `get_agent_connection_status` and confirm `configured: true`.

Skip this if you can't receive webhooks; MCP still works.

## 3. When woken by Sparks

Call `get_note` with the payload's `note_id`, do the work, then update the note (`update_note`,
`add_subtask`, `mark_done`). Never delete anything.

## 4. Confirm to the user

Tell them it's connected (they can disconnect in Settings → Agent) and suggest: open a note → ⋯ → Send to agent.
