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
   `{ "note_id", "title", "project_id", "url", "send_id", "intent?" }`.
2. Call MCP tool `set_agent_webhook` with `url` (public https, port 443), `secret` (if your webhook needs one)
   and `auth_header` (default `Authorization`, sent as `Bearer <secret>`; e.g. `X-Webhook-Secret` otherwise).
   Pass `accepts_intent: true` only if you follow section 3. That unlocks Plan, Build, and Research.
   Without it, Sparks keeps one Send to agent button and does not label the job.
3. Call `get_agent_connection_status` and confirm `configured: true`. Each target includes `accepts_intent`.

If you run locally (no public URL — DeepSeek Harness desktop, local Claude Code, …):
1. Call `set_agent_webhook` with `kind: "inbox"` (and optional `name`, and `accepts_intent: true` if you follow section 3). It returns `poll_url` and a
   one-time `token` — store the token as a secret.
2. Poll `GET <poll_url>` with `Authorization: Bearer <token>` on a schedule (e.g. every 10-15 min);
   queued sends come back as `{ items: [{ note_id, title, project_id, url, send_id, intent, received_at }] }` and are
   claimed (delivered once). `intent` is omitted on an old unlabeled send.

Skip this if you can't receive webhooks; MCP still works.

## 3. When woken by Sparks

The JSON is `{ note_id, title, project_id, url, send_id, intent? }`.
`intent` is `plan`, `build`, or `research`. If it is missing, this is an unlabeled send and means work.

Webhook calls include `X-Sparks-Signature: sha256=<hex>`, the HMAC-SHA256 of the raw body using the webhook secret.
Check it when you have that secret. Inbox polls are authenticated by the poll token instead, so those items are not signed.

1. If you already handled this `send_id`, stop.
2. Call `get_note`. Trust `on_hold` and `agent_job` there over the webhook. The note may have changed since the send.
3. If `on_hold` is true, do not build. Plan and Research are still allowed.
4. Switch on `agent_job.intent` (same value as the payload `intent` when you are the latest send):
   - **plan** or **research**: add subtasks with `add_subtask`. Do not edit a repository. Do not replace the note body.
   - **build**: implement, unless `on_hold` is true — then stop.
   - missing: treat it as build, and stop if `on_hold` is true.
5. Call `set_agent_run` with that `send_id` and `done` or `failed`. A stale `send_id` is rejected.

When you mention another Sparks note anywhere (note text, a subtask, your message to the user), include its `url` from `list_notes` / `get_note` next to the title, e.g. `[Snack: speech check](https://sparks.6x7.gr/notes/<id>)`. Never name a note by title alone.

Skip notes with `on_hold: true` when looking for work (`list_notes` with `on_hold: false`). You may still read them.

## 4. Confirm to the user

Tell them it's connected (they can disconnect in Settings → Agent). After `accepts_intent`, a note's ⋯ menu shows Plan, Build, and Research. Until then it stays Send to agent.
