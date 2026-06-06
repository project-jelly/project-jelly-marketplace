---
name: opsgate-admin
description: >-
  Use ONLY when the user explicitly asks to register, update, or delete an
  opsgate credential (credential lifecycle) via the opsgate admin surface.
  For normal use (querying/calling targets), use opsgate-basic instead.
---

# opsgate admin (credential lifecycle)

opsgate exposes two MCP surfaces. They are **not** a privilege gate — same
authenticated user, same auth. The only difference is **which tools are mounted**:

- `opsgate` (runtime) → use credentials: `api_call`, `sql_query`, ...
- `opsgate-admin` (admin) → **manage** credentials: register / update / delete

Reach for admin tools **only when the user explicitly wants to register or
maintain a credential**. Day-to-day usage never needs them.

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

- These tools **mutate the credential catalog** — only on explicit user request.
- `credential_update_*` **cannot change the secret or the endpoint**. To rotate a
  secret or change a target, **delete and re-register** — there is no in-place edit.
- Target execution (`api_call`, `sql_query`) is **not** on this surface — that's
  the runtime surface (`opsgate-basic`).
- `credential_list` returns metadata/policy only — never secrets or endpoints.
  Never echo secrets back.
