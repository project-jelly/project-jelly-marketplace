---
name: opsgate-admin
description: >-
  Use ONLY when the user explicitly asks to register, update, or delete an
  opsgate credential (credential lifecycle) on the opsgate admin surface. For
  normal use — calling APIs or querying SQL — use opsgate-basic instead.
---

# opsgate admin (credential lifecycle)

opsgate exposes two MCP surfaces. They are **not** a privilege gate — same
authenticated user, same auth. The only difference is **which tools are mounted**:

- `opsgate` (runtime, `/mcp`) → *use* credentials: `api_call`, `sql_query`, …
- `opsgate-admin` (admin, `/mcp/admin`) → *manage* credentials: register / update / delete

Reach for these tools **only when the user explicitly wants to register or
maintain a credential**. Day-to-day usage never needs them.

## Tools

| tool | use |
|---|---|
| `me` | confirm the authenticated user + admin capabilities |
| `credential_list` | list aliases + metadata + policy before update/delete (metadata only) |
| `credential_register_http` | register an HTTP target for `api_call` |
| `credential_register_sql` | register a Postgres target for `sql_schema`/`sql_query` |
| `credential_update_http` | edit metadata/policy of an HTTP alias |
| `credential_update_sql` | edit metadata/policy of a SQL alias |
| `credential_delete` | remove an alias and destroy its sealed secret |

## Register HTTP (`credential_register_http`)

Required: `provider` (grouping label, e.g. `k8s`, `github`, `internal-api`),
`alias`, `origin` (**scheme + host only**, e.g. `https://k8s.example.com` — no paths).
Optional: `base_path` (fixed prefix hidden from `api_call`), `secret_headers`
(`{name, value}` — sealed, never returned, e.g. `Authorization`/`X-API-Key`),
`description`, `env`, `tags`, `policy` (HTTP fields below), `allow_private_network`
(only for trusted internal targets), `tls_server_ca` (PEM for private HTTPS).

## Register SQL (`credential_register_sql`)

Required: `alias`, `database_url` (**without** user/pass, e.g.
`postgres://db.example.com/app?sslmode=require`), `username`, `password`
(both sealed, never returned). Optional: `provider` (defaults to `postgres`),
`description`, `env`, `tags`, `policy` (SQL fields below), `allow_private_network`,
`allow_insecure_transport` (only when non-TLS transport is intentionally required).

## Policy is category-specific

A policy field belonging to the other category is **rejected**. Empty policy =
safe defaults (HTTP → GET on `/`; SQL → metadata/explain off).

| HTTP-only | SQL-only |
|---|---|
| `allowed_methods` (default `["GET"]`) | `allow_metadata` (enables `sql_schema`) |
| `allowed_request_path_prefixes` (default `["/"]`) | `allow_explain`, `allow_explain_analyze` (the latter requires the former; can execute the query) |
| `denied_query_keys` | `denied_functions` |
| `allowed_request_headers` (blocked/secret headers rejected) | `max_rows`, `max_bytes`, `timeout_ms` (0 = service default) |

## Update (`credential_update_http` / `credential_update_sql`)

Required: `alias`, `reason` (stored in audit/history). Optional, replaced only when
present: `description`, `env`, `tags`, `policy`.

- **Cannot** change the target or secret: `origin`/`base_path` + secret headers
  (HTTP) and `database_url` + username/password (SQL) are **immutable**.
- To rotate a secret or change a target → `credential_delete` then re-register.

## Delete (`credential_delete`)

Required: `alias`, `reason`. Destroys the sealed secret material. Use only when
the credential should no longer be callable.

## Rules

- These tools **mutate the credential catalog** — only on explicit user request.
- `credential_list` returns metadata/policy only — never secrets or endpoints. Never echo secrets back.
- Target execution (`api_call`, `sql_query`) is **not** here — that's the runtime surface (`opsgate-basic`).
- There is no role/admin gate; any registered, active user can manage their own credentials.
