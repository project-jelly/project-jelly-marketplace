# opsgate plugin

opsgate runtime MCP (read-only operational diagnostics) + the `opsgate-basic` skill.

opsgate is a remote, OAuth-authenticated credential broker. This plugin only
wires the MCP server and ships the usage skill — **no secrets are bundled**.

## Install

```text
/plugin marketplace add project-jelly/project-jelly-marketplace
/plugin install opsgate@project-jelly
/plugin enable opsgate@project-jelly
```

## Authenticate (once, per machine)

The MCP server is registered on enable, but each user authenticates themselves:

```text
/mcp        # complete the opsgate (authgate) OAuth login in the browser
```

## Verify

```text
/help                  # /opsgate:opsgate-basic should appear
/mcp                   # opsgate server should be listed
```

## Notes

- Endpoint: `https://opsgate.project-jelly.io/mcp` (runtime surface, read-only).
- The admin surface (`/mcp/admin`, credential lifecycle) is **not** included here.
