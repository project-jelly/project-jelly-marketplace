---
name: opsgate-admin
description: >-
  Use when registering, updating, or deleting opsgate credentials (credential
  lifecycle) through the opsgate admin MCP surface. Privileged, write-capable.
  Not for using credentials — read/query lives in the opsgate runtime skill.
---

# opsgate admin

The opsgate **admin surface** (`/mcp/admin`) manages the **credential lifecycle**.
It does *not* execute against targets — using a credential (`api_call`, `sql_query`)
lives on the runtime surface (the `opsgate-basic` skill). Keep the two separate.

## Tools (admin surface)

| tool | use |
|---|---|
| `me` | confirm the authenticated user |
| `credential_register_http` | register a new HTTP-target credential |
| `credential_register_sql` | register a new SQL-target credential |
| `credential_update_http` | edit metadata/policy of an HTTP credential |
| `credential_update_sql` | edit metadata/policy of a SQL credential |
| `credential_list` | list aliases + policy (metadata only) |
| `credential_delete` | remove a credential |

## Typical flow

```
me
credential_register_http | credential_register_sql
credential_list
credential_update_http   | credential_update_sql
credential_delete
```

## Rules

- Requires a valid, active authenticated user (no separate role/admin gate).
- **Write-capable** — only register/update/delete when the user **explicitly** asks.
- `credential_update_*` **cannot change the secret or the endpoint**. To rotate a
  secret or change a target, **delete and re-register** — there is no in-place edit.
- Target execution (`api_call`, `sql_query`) is **not** on this surface. Don't try
  it here; use the runtime surface.
- Never echo back secrets/endpoints — `credential_list` returns metadata only.

## Why a separate plugin

This surface can mutate the credential catalog, so it ships apart from the
read-only runtime plugin. Install/enable it only for users who manage credentials.
