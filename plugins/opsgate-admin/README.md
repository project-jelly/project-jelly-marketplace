# opsgate-admin plugin

opsgate **admin** MCP surface (`/mcp/admin`) — credential lifecycle
(register / update / delete) + the `opsgate-admin` skill.

Privileged and write-capable, so it's a **separate** install from the read-only
`opsgate` runtime plugin. No secrets are bundled — users authenticate via `/mcp`.

## Install

```text
/plugin marketplace add project-jelly/project-jelly-marketplace
/plugin install opsgate-admin@project-jelly
/plugin enable opsgate-admin@project-jelly
```

## Authenticate (once, per machine)

```text
/mcp        # complete the opsgate admin (authgate) OAuth login in the browser
```

## Notes

- Endpoint: `https://opsgate.project-jelly.io/mcp/admin`.
- Using credentials (`api_call`, `sql_query`) is **not** here — that's the
  `opsgate` runtime plugin.
- `credential_update_*` can't change a secret or endpoint; rotate by delete + re-register.
