# Publishing Sparks MCP

This file shows where Sparks is listed, how each listing was submitted, and how to repeat it.
It's written so that a person, or an agent with the right logins, can follow it from top to bottom.

- **Server:** `https://sparks.6x7.gr/api/mcp` (Streamable HTTP, remote).
- **Auth:** OAuth 2.1 with PKCE S256, dynamic client registration, refresh tokens, and scope `mcp`.
- **Support:** help@6x7.gr. **Privacy:** https://sparks.6x7.gr/privacy. **Terms:** https://sparks.6x7.gr/terms.
- **Docs:** https://sparks.6x7.gr/help/connect-agent.
- **Icons:** `assets/logo.png` (512×512) and `assets/logo.svg`, also served at https://sparks.6x7.gr/logo.png and https://sparks.6x7.gr/logo.svg.
- **Source of truth for tools:** `src/lib/mcp/server.ts` in `philipposk/sparks`. The canonical setup skill is
  `docs/agent-skill/connect-to-sparks/SKILL.md` in the same repo.

## Status

| Place | Status | Last action |
| --- | --- | --- |
| Official MCP Registry (`gr.6x7/sparks`) | **Listed**, v1.0.0 | Published 2026-10-22. v1.1.0 is ready to publish. |
| Claude Connectors Directory | **Submitted** 2026-11-25, in review | Owner ticked the policy boxes and submitted. New listings start as Community. |
| Claude Code plugin (this repo as a marketplace) | Works | Merged 2026-11-25. `/plugin marketplace add 6x7gr/sparks-mcp` |
| Cursor Marketplace | Not submitted | Needs the owner signed in at cursor.com first. |
| OpenAI plugin directory (Codex / ChatGPT) | Not submitted | Needs owner sign-in, org identity verification, a demo video and a domain-check route. |
| Grok (xAI) | No public catalog submission | Users add Sparks as a custom connector. |
| Glama connectors | **Listed** | https://glama.ai/mcp/connectors/gr.6x7/sparks (imported from the registry). Not claimed yet. |
| awesome-remote-mcp-servers (punkpeye) | PR open | [punkpeye/awesome-remote-mcp-servers#5852](https://github.com/punkpeye/awesome-remote-mcp-servers/pull/5852), 2026-11-25. |
| awesome-mcp-servers (appcypher) | Not possible | The repo is archived (read-only). |
| mcpservers.org (wong2 list) | **Submitted** 2026-11-25, in review | Free plan, review within 2 weeks. |
| PulseMCP | Not possible right now | Submissions paused since 2026-09-03; it will import from the official registry. |
| mcp.so | Not submitted | Has a Remote Server option. The visible path is a $39 paid listing; a free path isn't confirmed. |
| Smithery | Not possible | Stopped taking servers on 2026-10-31 (now a curated, read-only directory). |

Update this table every time something changes.

## Before any submission: the release checklist

1. Compare the tools in `philipposk/sparks` `src/lib/mcp/server.ts` with the tool table in `README.md`, and fix any differences.
2. Copy `docs/agent-skill/connect-to-sparks/SKILL.md` from `philipposk/sparks` to `skills/connect-to-sparks/SKILL.md`.
3. Bump `version` in all of these files, keeping them the same: `server.json`, `.cursor-plugin/plugin.json`,
   `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `.codex-plugin/plugin.json`.
4. Run `claude plugin validate .`. It must pass with nothing to report.
5. Check every manifest with `python3 -m json.tool <file>`. Keep the `server.json` description at 100 characters or fewer.
6. Check the live server. `curl -i https://sparks.6x7.gr/api/mcp` must return `401` with
   `WWW-Authenticate: Bearer resource_metadata=…`. Then add Sparks as a custom connector in Claude and call every tool once.
7. Merge to `main`. Every listing reads from `main` or from the live server.

## 1. Official MCP Registry

- **What it is:** https://registry.modelcontextprotocol.io. Many directories import from it, including Glama, PulseMCP and the GitHub MCP registry, so update it first.
- **Name:** `gr.6x7/sparks`. A `gr.6x7/*` name must be proven through DNS on `6x7.gr`. The TXT record `v=MCPv1; k=ed25519; p=<public key>` is already in place.
  The matching private key lives only with the owner. Never commit it.
- **Check the current listing:** `curl "https://registry.modelcontextprotocol.io/v0/servers?search=gr.6x7/sparks"`
- **Publish an update.** The `version` in `server.json` must be new; a version can't be published twice.
  ```bash
  # install: https://github.com/modelcontextprotocol/registry/releases (mcp-publisher), or `brew install mcp-publisher`
  mcp-publisher login dns --domain 6x7.gr --private-key <hex private key>
  mcp-publisher publish        # run from the repo root; it reads server.json
  ```
- **If the key is lost:** run `openssl genpkey -algorithm Ed25519 -out key.pem`. Then replace the TXT record with the new public key and log in again.
- **CI option (not set up):** this needs the private key stored as the GitHub secret `MCP_REGISTRY_KEY`.
  A workflow can then run the two commands above whenever a `v*` tag is pushed.

## 2. Claude Connectors Directory

- **Where:** https://claude.ai/directory/manage → **Submit new** → **MCP connector**. You need to be signed in on a paid Claude plan.
  The docs are https://claude.com/docs/connectors/building/submission and https://claude.com/docs/connectors/building/review-criteria.
- **Who:** an agent can fill it; a person ticks the seven policy acknowledgments and presses Submit.
- **How the form works (2026-11-25):** 7 steps: Source, Content, Listing, Test & launch, Data handling, Compliance, Review and submit.
  Step 1 has its own **Add custom connector** button, which adds the connector and runs the Sparks sign-in. Use it when an agent
  drives the browser, because Claude in Chrome isn't allowed on claude.ai Settings pages.
  Step 2 reads the tools live and flags any tool without `annotations.title` (a top-level `title` isn't enough), so the server
  mirrors titles into annotations (philipposk/sparks#50). One acknowledgment says tool descriptions hold no instructions for the model:
  keep descriptions factual and put agent rules in the server `instructions`. After a server change the form keeps showing the old
  tool list; it re-reads the tools at submission time.
- **Answers used:** slug `sparks`; author `6x7`, https://sparks.6x7.gr; use cases = the first three prompts below;
  prerequisite "A free Sparks account"; Read and write; "We own the API"; no health data; no sponsored content.
  Leave "Self-tested" unticked until every tool has really been called.
- **Requirements Sparks meets:** a public HTTPS URL; OAuth; every tool has a `title` plus `readOnlyHint` or `destructiveHint`;
  read and write tools are separate; tool names are 64 characters or fewer; no catch-all request tool.
- **You also need:** a reviewer test account full of realistic data, and proof that every tool was run in MCP Inspector or as a custom connector.
- **Review:** no fixed time. New listings start as "Community". Escalations go to mcp-review@anthropic.com.
- **Note:** the URL slug is permanent, so use `sparks`.

Field values to paste:

| Field | Value |
| --- | --- |
| Name (≤100) | Sparks |
| One-liner (≤200) | Capture and organize ideas in your Sparks notebook. |
| Description (≤2000) | Sparks is an idea notebook. This connector lets Claude find, read, create and organize your notes, projects and subtasks, mark things done, and pick up notes you send to your agent. It signs in with OAuth and never deletes anything. |
| Categories (1–5) | Productivity (the form has no Notes category) |
| Server URL | https://sparks.6x7.gr/api/mcp |
| Auth | OAuth 2.1 (PKCE S256, DCR, refresh tokens), scope `mcp` |
| Docs / Privacy / Terms / Support | see the top of this file |
| Icon | https://sparks.6x7.gr/logo.png |

Example prompts:
1. "What's in my Sparks Inbox?"
2. "Save this idea to my Sparks project 'Product ideas': …"
3. "Show the subtasks of my note about the launch and mark the first one done."
4. "Which of my Sparks notes are due this week?"

**Reviewer account:** create it only after the owner says yes. Use review@6x7.gr, and enter its password only in the portal.
Seed it with 3 projects (Product ideas, Travel, Reading list) and about 12 notes. Include tags, due dates, one pinned note,
2–3 levels of subtasks, some done items and 2 Inbox notes.

## 3. Claude Code plugin

- This repo is its own plugin marketplace (`.claude-plugin/marketplace.json`, `.claude-plugin/plugin.json`).
  Users run `/plugin marketplace add 6x7gr/sparks-mcp`, then `/plugin install sparks@sparks`.
- Validate it with `claude plugin validate .`.
- **Optional:** submit the plugin to Anthropic's plugin directory. See https://claude.com/docs/plugins/submit for the current form.

## 4. Cursor Marketplace

- **Where:** https://cursor.com/marketplace/publish. Paste `https://github.com/6x7gr/sparks-mcp`. Docs: https://cursor.com/docs/plugins.
- **Who:** a person, using the web form. Cursor reviews every plugin by hand, and the repo must be open source (it's MIT).
- **Files Cursor reads:** `.cursor-plugin/plugin.json` (only `name` is required), `mcp.json`, `skills/`, and the README.
- **Test locally first:** copy the repo to `~/.cursor/plugins/local/sparks` and reload Cursor.
- **Updates:** a listed plugin is pinned to one commit, so pushing doesn't update it. Don't resubmit the form. Bump `version`, push,
  then email the Cursor team (the contact is on the publish page) and ask for a re-sync. Reviews can take weeks.

## 5. OpenAI plugin directory (Codex and ChatGPT)

- **Where:** https://platform.openai.com/plugins → **Upload new or existing plugin** → **With MCP**.
  Docs: https://developers.openai.com/codex/submit-plugins and https://developers.openai.com/plugins/deploy/submission.
- **Who:** an owner of an OpenAI platform organization, or a member with "Apps Management Write". The organization must first pass
  individual or business identity verification at https://platform.openai.com/settings/organization/general.
- **Domain verification:** the portal gives you a token. Serve it as plain text at
  `https://sparks.6x7.gr/.well-known/openai-apps-challenge`, which means a change in `philipposk/sparks` (a route under `src/app/.well-known/`).
- **Upload:** zip this repo (the manifest is `.codex-plugin/plugin.json`; `shortDescription` must be 30 characters or fewer). The portal also asks for:
  five positive and three negative test cases, a demo video URL, release notes, and reviewer credentials (in the portal, not in the zip).
- **Rules:** one MCP server per plugin; OAuth (no API-key flow); no lifecycle hooks.
- **Updates:** changes to the MCP server are rescanned daily, or click **Rescan**. Changes to the manifest or skill need a new zip.
- **Codex users without a listing:** add `[mcp_servers.sparks]` with `url = "https://sparks.6x7.gr/api/mcp"` to `~/.codex/config.toml`, then run `codex mcp login sparks`.

## 6. Grok (xAI)

- xAI doesn't document a way for third parties to join its connector catalog (https://docs.x.ai/grok/connectors/catalog).
- Users add Sparks themselves: https://grok.com/connectors → **New Connector** → **Custom** → paste `https://sparks.6x7.gr/api/mcp` → sign in.
  On Business and Enterprise plans, an admin has to provision it first (https://docs.x.ai/grok/connector-management).
- To push for a catalog listing, contact xAI developer relations. Check this page again later.

## 7. Glama

- Glama lists remote servers at https://glama.ai/mcp/connectors, using registry-style names. Sparks is probably at
  https://glama.ai/mcp/connectors/gr.6x7/sparks after the registry import. Open that page. If Sparks is there, claim it by signing in with GitHub as the `6x7gr` owner.
  If it isn't, add it on Glama.
- The awesome-remote list needs this badge:
  `[![Sparks MCP connector](https://glama.ai/mcp/connectors/gr.6x7/sparks/badges/score.svg)](https://glama.ai/mcp/connectors/gr.6x7/sparks)`.
  Add the same badge to this README; the awesome lists check for this backlink.

## 8. awesome-remote-mcp-servers (punkpeye)

- **Repo:** https://github.com/punkpeye/awesome-remote-mcp-servers. Sparks is remote-only, so it goes in this list, not `punkpeye/awesome-mcp-servers`.
- **How:** fork the repo, add the entry in alphabetical order under **🧠 Knowledge & Memory**, and open a PR. Mention the Glama backlink in the PR description.
  The list's CI sends an MCP `initialize` request to the URL. OAuth servers (🔐) are accepted.
- **Entry:**
  ```markdown
  - [Sparks](https://sparks.6x7.gr) `https://sparks.6x7.gr/api/mcp`
    [![Sparks MCP connector](https://glama.ai/mcp/connectors/gr.6x7/sparks/badges/score.svg)](https://glama.ai/mcp/connectors/gr.6x7/sparks)
    🔐 - Idea notebook: find, capture and organize notes, projects and nested subtasks, with no delete tool.
  ```

## 9. awesome-mcp-servers (appcypher)

- The repo is archived, so it takes no PRs. Skip it.

## 10. Web directories

These are plain web forms. Use the same name, one-liner, URL and icon as the Claude listing above.

| Directory | Submit at | Notes |
| --- | --- | --- |
| mcpservers.org (also the wong2 awesome list) | https://mcpservers.org/submit | No sign-in. Fields: name, category (Productivity), short description, docs URL, registry name `gr.6x7/sparks`, remote ✓ + URL, auth OAuth, contact help@6x7.gr, plan Free. |
| PulseMCP | https://pulsemcp.com/submit | Paused since 2026-09-03. It imports the official registry. |
| mcp.so | https://mcp.so/submit | Choose **Remote Server**. Shows a $39 paid option; don't pay without the owner's OK. |
| Smithery | — | Stopped taking servers on 2026-10-31. |

## Notes for agents repeating this

- From a cloud sandbox you can usually reach only GitHub and the MCP Registry. Most directory sites, and `sparks.6x7.gr` itself, are blocked there,
  and access may be limited to the Sparks repos, so you can't open PRs on other people's repos. In that case,
  prepare the entries and the release PR, then give the owner the list of forms and PRs that need them.
- The Claude, OpenAI and Cursor forms need a human: a paid or verified account, policy acknowledgments, and identity verification.
- Never put private keys, passwords or reviewer credentials in this repo.
- On the owner's Mac (2026-11-25), an agent could fill every form, sign in to Sparks with the email code, and open the awesome-list PR.
  Ticking policy acknowledgments and pressing a public form's final Submit were blocked for the agent, so leave those clicks to the owner.
