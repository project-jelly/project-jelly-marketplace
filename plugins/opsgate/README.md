# opsgate plugin

Wires the opsgate MCP (a remote, OAuth-authenticated credential broker) and ships
the usage skills. **No secrets are bundled** — each user authenticates via `/mcp`.

opsgate exposes two MCP surfaces, both connected by this one plugin. They are
**not** a privilege gate (same authenticated user) — they just mount different tools:

| MCP server | surface | tools | skill |
|---|---|---|---|
| `opsgate` | `/mcp` (runtime) | `api_call`, `sql_query`, `sql_schema`, `credential_list`, `me` | `opsgate-basic` |
| `opsgate-admin` | `/mcp/admin` (admin) | `credential_register/update/delete_*`, `credential_list`, `me` | `opsgate-admin` |

Day-to-day you use the runtime tools; the admin tools are for registering/maintaining
credentials and are only reached on explicit request.

## Supported clients

This one plugin installs in **both Claude Code and Codex CLI** from the same repo.
The `skills/` are shared verbatim; only the manifest and MCP wiring differ per client
(HTTP MCP is declared differently by each tool):

| file | client | MCP shape |
|---|---|---|
| `.claude-plugin/plugin.json` → `.mcp.json` | Claude Code | `{ "type": "http", "url": … }` |
| `.codex-plugin/plugin.json` → `mcp.codex.json` | Codex CLI | `{ "url": … }` (OAuth via `codex mcp login`) |

**No secrets are bundled** — each user authenticates themselves.

### Claude Code

```text
/plugin marketplace add project-jelly/project-jelly-marketplace
/plugin install opsgate@project-jelly
/plugin enable opsgate@project-jelly
/mcp        # complete the opsgate (authgate) OAuth login in the browser
```

Verify: `/help` shows `/opsgate:opsgate-basic` + `/opsgate:opsgate-admin`; `/mcp` lists both servers.

### Codex CLI

```sh
codex plugin marketplace add project-jelly/project-jelly-marketplace
codex plugin add opsgate@project-jelly
```

Verify install: `codex plugin list` shows opsgate; `codex mcp list` shows the opsgate server(s).

**Authentication caveat.** opsgate authenticates via authgate using the **Client ID
Metadata Document (CIMD)** flow; Dynamic Client Registration (RFC 7591) is off by
default. Codex's `codex mcp login` currently requires DCR and does not support CIMD
or a pre-registered client ([openai/codex#19154](https://github.com/openai/codex/issues/19154)),
so `codex mcp login opsgate` will fail against a default opsgate deployment. Two ways
to authenticate Codex:

1. **Bearer token (works today)** — obtain an authgate-issued token and point Codex at
   it via `bearer_token_env_var`. Add to `~/.codex/config.toml` (or project `.codex/config.toml`):
   ```toml
   [mcp_servers.opsgate]
   url = "https://opsgate.project-jelly.io/mcp"
   bearer_token_env_var = "OPSGATE_TOKEN"
   ```
   then `export OPSGATE_TOKEN=<token>` before launching Codex.
2. **Enable DCR on authgate** — set `ENABLE_DYNAMIC_CLIENT_REGISTRATION=true` on the
   authgate deployment so its authorization-server metadata advertises a
   `registration_endpoint`; then `codex mcp login opsgate` can register and log in.

Claude Code is unaffected — it supports CIMD natively, so `/mcp` login works as-is.
