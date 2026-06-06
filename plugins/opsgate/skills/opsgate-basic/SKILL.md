---
name: opsgate-basic
description: >-
  Use when diagnosing infrastructure, Kubernetes, SQL, or API state through the
  opsgate MCP tools. opsgate is a credential broker — you call resources by alias
  and never see keys or endpoints. Read-only by default.
---

# opsgate basic

opsgate is a **credential broker**: it holds secrets, you call by **alias** and
never see keys/URLs. This skill is the *general* operating model for the opsgate
runtime surface (`/mcp`). Deployment-specific topology lives elsewhere (private).

## Tools (runtime surface)

| tool | use |
|---|---|
| `me` | confirm who you're authenticated as |
| `credential_list` | list aliases + policy (metadata only — never secrets/endpoints) |
| `api_call` | call an HTTP target by alias (needs `category=http`) |
| `sql_schema` | inspect structure before querying (needs `category=sql`) |
| `sql_query` | read data by alias (needs `category=sql`) |

`api_call`, `sql_schema`, `sql_query` all require a `purpose`.

## Default flow

```
me
credential_list(category=http|sql)     # pick the alias
api_call            (http)             # or
sql_schema -> sql_query (sql)
```
Always `credential_list` first to discover which aliases exist — don't guess names.

## Rules (read-only by default)

- Start with read-only inspection; widen only when the task needs it.
- **Never** expose credentials, tokens, or raw secret values.
- No write/delete/restart/scale/mutation unless the user **explicitly** approves.
- Credential lifecycle (register/update/delete) is **not** on this surface — it
  lives on the admin surface (`opsgate-admin` tools). Use those only when the user
  explicitly asks to manage a credential; otherwise stay here.

## SQL

- `SELECT` / `WITH` only; add `LIMIT` while exploring.
- Run `sql_schema` before querying unknown tables.
- Select explicit columns; avoid `SELECT *` and dumping sensitive columns.

## Output discipline

- Output is structured/column-oriented — aggregate before drilling down.
- On large results, narrow with the tool's jsonpath/pagination hints rather than
  pulling everything.
