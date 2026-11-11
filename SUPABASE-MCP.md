# Supabase MCP — this environment's instructions

Applies **only to this GitHub Codespace** (no GUI browser here). Other machines should use the standard OpenCode `/mcps` sign-in flow.

---

## How auth is configured here

OpenCode's interactive OAuth **cannot** finish in this environment:

- the Supabase hosted-MCP OAuth flow redirects to `http://127.0.0.1:<port>/callback`, which no device in this workflow can serve, and
- `api.supabase.com` rejects `resource=...&project_ref=...&features=...` in the authorize request (`resource must be a valid MCP endpoint` — RFC 8707 forbids query strings in `resource`), so `opencode mcp auth` fails while the project pin is in the URL.

Therefore this environment uses **header-based auth** with an OAuth-issued Bearer token (issued via a one-time manual flow documented below):

- **Server URL:** `https://mcp.supabase.com/mcp?project_ref=qfngbhrlqyojfwoadher&features=docs,account,database,debugging,development,functions,branching,feedback,notebooks`
  - project: `qfngbhrlqyojfwoadher` · org: `haffar app` (`yavjaxftbnroldcdvwlr`)
- **Config:** `./opencode.json` — contains `headers: { Authorization: "Bearer <access_token>" }`
- **Tool override:** all Supabase tools are pre-scoped to the pinned project (`execute_sql`, migrations, functions, branches take no `project_id`).

## Files involved (chmod 600 — never commit, never paste anywhere)

| Path | Contains |
|---|---|
| `./opencode.json` (repo root, gitignored) | MCP server config incl. the Bearer access token |
| `~/.config/opencode/supabase-token.json` | access + refresh token, OAuth `client_id` + `client_secret` |
| `~/.config/opencode/refresh-supabase-token.py` | rotates the token and patches `opencode.json` in place |
| `~/.config/opencode/pkce-init.py` | generates a PKCE pair + full authorize URL (`pkce.json`) |
| `~/.config/opencode/callback-server.py` | local HTTP catcher on port `40877` (append-safe, ignores `/favicon.ico`) |
| `~/.config/opencode/exchange.py` | swaps the authorization `code` for tokens (accepts a pasted callback URL as arg) |
| `~/.config/opencode/apply-token.py` | writes the fresh token into `opencode.json` headers |

`opencode.json` is **gitignored** here — keep it that way. If `.gitignore` ever loses that line, re-add it before committing anything.

## Routine: token refresh

Access tokens expire. When the MCP shows `needs auth` or tools 401:

```sh
python3 ~/.config/opencode/refresh-supabase-token.py
```

OpenCode hot-reloads config — no restart needed. Verify with `opencode mcp list` → expect `✓ supabase connected`.

## Full re-auth (only if refresh is rejected / refresh token revoked)

The hosted OAuth callback can be made reachable from a phone through Codespaces port forwarding:

```sh
# 1. expose the callback port for the duration of the flow
gh codespace ports visibility 40877:public -c "$CODESPACE_NAME"

# 2. mint a fresh PKCE pair + authorize URL
python3 ~/.config/opencode/pkce-init.py          # prints the URL, writes pkce.json

# 3. if a NEW OAuth app is needed (registration endpoint of api.supabase.com):
curl -sS -X POST https://api.supabase.com/platform/oauth/apps/register \
  -H 'Content-Type: application/json' \
  -d '{"client_name":"haffar-codespace-mcp","redirect_uris":["https://'"$CODESPACE_NAME"'-40877.app.github.dev/callback"]}'
#    → keep returned client_id+client_secret ONLY in ~/.config/opencode/supabase-token.json

# 4. run the catcher in the background, then open the authorize URL on a phone
python3 ~/.config/opencode/callback-server.py &   # waits on 0.0.0.0:40877

# 5. after browser shows the green ✅ page:
python3 ~/.config/opencode/exchange.py            # or: exchange.py '<pasted callback url>'
python3 ~/.config/opencode/apply-token.py

# 6. close the exposure again
gh codespace ports visibility 40877:private -c "$CODESPACE_NAME"

opencode mcp list                                 # verify ✓ connected
```

If a long authorize URL is opened on the phone and it errors with `resource: Resource must be a valid MCP endpoint`, the URL got mangled in transit — mint the redirect first here, then open the **short** consent link (the `https://supabase.com/dashboard/authorize?auth_id=…` target) instead.

## Revoke / rotate

Revoke the OAuth app in **Supabase Dashboard → Account → OAuth apps** → `haffar-codespace-mcp`
(`client_id: bd124654-9738-4b0b-a857-54b1e353ee2d`; the secret lives only in `supabase-token.json`). After revoking, do a full re-auth with a **new** registration.

## Health checks

- `opencode mcp list` → `✓ supabase connected`
- via the agent: `list_organizations()` should return `haffar app`
