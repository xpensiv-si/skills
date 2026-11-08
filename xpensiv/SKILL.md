---
name: xpensiv
description: Run an Xpensiv design check on a web app the user is building and apply the fix list. Use when the user asks to check, review, polish or "make it look expensive" for a deployed URL, or after shipping UI changes. Connects the Xpensiv MCP server (check_app, get_check, list_checks) with the user's API key.
---

# Xpensiv design check

Xpensiv loads a public URL on phone and desktop, takes screenshots and returns a fix list
(accessibility, layout overflow, small text, Lighthouse scores). It does not see your source code.

## 1. Connect once

The user needs an API key from https://xpensiv.si/console/keys (keys start with `xpk_`).
Never paste the key into chat logs or commit it. Prefer an environment variable `XPENSIV_API_KEY`.

Check whether a server named `xpensiv` is already connected. If not, add it for the agent you are running in:

- Claude Code:
  `claude mcp add --transport http xpensiv https://xpensiv.si/mcp --header "Authorization: Bearer $XPENSIV_API_KEY"`
- Cursor: add to `~/.cursor/mcp.json`
  `{"mcpServers":{"xpensiv":{"url":"https://xpensiv.si/mcp","headers":{"Authorization":"Bearer xpk_..."}}}}`
- Codex: add to `~/.codex/config.toml`, then export `XPENSIV_API_KEY`
  ```toml
  [mcp_servers.xpensiv]
  url = "https://xpensiv.si/mcp"
  bearer_token_env_var = "XPENSIV_API_KEY"
  ```

The MCP server may need a restart of the agent before its tools appear.
No MCP? Use the REST API: `POST https://xpensiv.si/api/v1/checks` with `{"url": "..."}`,
then `GET /api/v1/checks/{id}` until `status` is `completed`.

## 2. Run a check

1. Call `check_app` with the public `https://` URL. Localhost and private addresses are refused; deploy a preview first.
2. Save the returned `id`. Call `get_check` with it about every 30 seconds until `status` is `completed` or `failed`.
   A check usually takes one to three minutes. `list_checks` lists recent checks for the same key.
   Findings are automated browser measurements. The default provider is mock and does not call a model. Say so. Do not describe the result as an AI judge or a trained model.
3. On `failed` with `timed_out`, run `check_app` once more. On a 402 or "plan limit", tell the user and link the `upgrade` URL. Do not retry.

## 3. Use the fix list

- Work through `result.findings` from high to low severity. Each has `title`, `severity`, `region`, `evidence`, `fix` and a ready `prompt`.
- Map each finding to the user's code, make the change, then run a new check to confirm it is gone.
- Report what you fixed and what you left, with the check id. Point the user to `console` in the result to see screenshots.
- Findings come from automated browser checks. Say so; do not claim a human or a design expert reviewed the page.
