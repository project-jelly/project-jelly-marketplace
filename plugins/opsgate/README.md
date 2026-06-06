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

## Install

```text
/plugin marketplace add project-jelly/project-jelly-marketplace
/plugin install opsgate@project-jelly
/plugin enable opsgate@project-jelly
```

## Authenticate (once, per machine)

Both servers register on enable; each user authenticates themselves:

```text
/mcp        # complete the opsgate (authgate) OAuth login in the browser
```

## Verify

```text
/help                  # /opsgate:opsgate-basic and /opsgate:opsgate-admin appear
/mcp                   # opsgate and opsgate-admin servers listed
```
