---
name: opsgate-basic
description: >-
  Use when inspecting or diagnosing services through opsgate — calling HTTP/REST
  APIs (e.g. a Kubernetes API server or any registered REST endpoint) and
  querying Postgres databases by alias. opsgate is a credential broker: you call
  by alias and never see secrets or target URLs. Read-only by default.
---

# opsgate basic (runtime surface)

opsgate is a **policy-gated credential broker**. It holds the secrets and target
URLs; you reference a target by **alias** and opsgate performs the HTTP call or
SQL query on your behalf. You never see keys, tokens, `origin`/`base_path`, or
`database_url`. This skill covers the **runtime** surface (`/mcp`).

## Tools

| tool | required | use |
|---|---|---|
| `me` | — | confirm the authenticated owner + a `credential_summary` (counts by category/provider/tag). No secrets. |
| `credential_list` | — | discover aliases + metadata + policy. Never returns secrets or endpoints. |
| `api_call` | `alias`, `purpose`, `request_path` | call an HTTP alias (`category=http`). |
| `sql_schema` | `alias`, `purpose` | inspect Postgres structure before querying (`category=sql`). |
| `sql_query` | `alias`, `purpose`, `query` | read data from a SQL alias (`category=sql`). |

`api_call`, `sql_schema`, `sql_query` all require a short human `purpose` (stored in audit/history).

## Default flow

```
me                                  # who am I, what credentials exist
credential_list(category=http|sql)  # pick the alias — never guess names
api_call                  (http)    # or
sql_schema -> sql_query   (sql)
```

Always `credential_list` first to discover which aliases exist and read their `policy`.

## api_call (HTTP)

- `request_path` is the path **only** (e.g. `/api/v1/pods`) — appended after the
  hidden `origin`/`base_path`. No scheme/host/base_path, no `..`.
- `method` defaults to `GET`; allowed: GET/POST/PUT/PATCH/DELETE — and must be in
  the credential's `policy.allowed_methods`. GET must not carry a `body`.
- `query` / `headers` are optional; the policy may deny specific query keys.
  `Authorization` and other secret/unsafe headers are **blocked** — opsgate
  injects the credential's sealed headers itself. A custom `Accept` must request JSON.
- `body` is optional JSON; `content_type` (if set with a body) must be JSON.
- `max_bytes` defaults to 4096 (min 256, max 1 MiB).

## SQL (sql_schema → sql_query)

- `sql_query` accepts **read-only `SELECT` / `WITH` only**. A single trailing `;`
  is stripped. Use positional binds via `params` (max 64) instead of inlining values.
- Prefer explicit columns, `WHERE`, `count`/`group`, or keyset pagination — **avoid `SELECT *`**.
- Run `sql_schema` before querying unknown tables: `mode=tables` lists tables;
  `mode=table` with `namespace` (default `public`) + `table` shows columns/indexes.
  Schema returns structure only — never row data (needs `policy.allow_metadata`).
- Defaults: `max_rows` 100 (max 1000), `max_bytes` 64 KiB, `timeout_ms` 3000 (max 30000).
- Optional `database` selects another DB on the **same** registered Postgres server.

## Output discipline (api_call & sql_query)

Both return a JSON envelope with `body_mode`, `body_state`, and (when truncated)
`more.options.next_action`. Don't fight truncation — follow the hint:

- `body_state: "omitted"` → `body` is `null`; read `omit_reason` + `next_action`:
  - `output_body_too_large` → **add_jsonpath**: read `more.preview.paths`, pick 1–3 JSONPath projections.
  - `projection_body_too_large` → **narrow_jsonpath**: fewer/tighter expressions.
  - `source_body_too_large` → **narrow_request**: shrink upstream via path/query/filter/pagination/time range.
  - SQL `row_limit` → **adjust_max_rows**: lower `max_rows`, add `WHERE`/aggregate/keyset paging.
- JSONPath is RFC 9535 (no recursive `..`, max 16 expressions). Regex via
  `search(v, p)` (partial) / `match(v, p)` (full). `.length()` / `.count()` suffixes for counts.
- SQL rows come back column-oriented (`columnar_json`); for row count use `row_count`
  or `$.column.length()` — `.count()` over a column counts 1 array node, not rows.

## Rules

- Read-only by default. No write/POST-mutation/DELETE/restart unless the user **explicitly** approves.
- **Never** echo secrets, tokens, or raw endpoints — opsgate already hides them; keep them hidden.
- Credential lifecycle (register/update/delete) is **not** here — it lives on the
  admin surface (`opsgate-admin`). Switch only when the user explicitly asks to manage a credential.
