# Queries

Read when writing or reviewing any SELECT, INSERT, UPDATE or DELETE.

## Contents
- Tenant scoping
- Keyset pagination
- Optional filters
- Batching and N+1
- Upsert
- Identifiers you cannot bind
- Full-text and pattern search
- Predicates that silently go wrong
- JSONB traps

## Tenant scoping

Every statement that touches tenant data filters by the tenant key (`tenant_id` below), in every CTE and every joined table, not only the outer query.

```sql
-- members of one tenant with the documents they own
SELECT m.id, m.email, d.id AS document_id
FROM member m
JOIN document d ON d.tenant_id = m.tenant_id AND d.owner_id = m.id
WHERE m.tenant_id = $1
  AND m.deleted_at IS NULL;
```

- Join on the composite key `(tenant_id, id)`, not `id` alone. A join on `id` alone lets one bad row link two tenants, and stops the planner from using the `(tenant_id, ...)` indexes.
- A lookup by primary key still carries the tenant filter: `WHERE tenant_id = $1 AND id = $2`. Reading by `id` alone is an IDOR waiting for a guessed id.
- Keep the filter visible in the query text. A view, SQL function or RLS policy that applies it somewhere else makes reviews unable to see it (see schema-and-indexes.md).

## Keyset pagination

Use a row-value comparison on the same columns, in the same order, as an index. Always end with a unique tiebreaker such as `id`.

```sql
-- index: (tenant_id, created_at DESC, id DESC)
SELECT id, action, created_at
FROM audit_log
WHERE tenant_id = $1
  AND (created_at, id) < ($2, $3)
ORDER BY created_at DESC, id DESC
LIMIT $4;
```

- The row comparison becomes an Index Cond. The expanded OR form (`created_at < $2 OR (created_at = $2 AND id < $3)`) often becomes a Filter that discards every row before the cursor. Reproducible on PG16 with 200k rows and a cursor 150k deep: the OR form removed 149,999 rows by Filter and read 1,846 buffers, while the row comparison read 6.
- The cursor encodes the last row's sort values `(created_at, id)`. It never encodes an offset. A "cursor" that base64-encodes an OFFSET pays the same deep-page cost, and skips or repeats rows when new ones arrive.
- Mixed directions (`created_at DESC, id ASC`) break a single row comparison. Keep every key in the same direction.
- Don't add `COUNT(*)` totals or OFFSET pages unless the user asks. A count runs a second full scan on every request.
- Cap `LIMIT` on the server, including GraphQL `first` arguments.

## Optional filters

`WHERE ($2::text IS NULL OR status = $2)` can't use the index on `status` once a generic plan is chosen, which happens after 5 executions of a prepared statement (pgx caches them) or under `plan_cache_mode = force_generic_plan`. A custom plan folds the NULL check and does use the index. For list or search endpoints with many optional filters, build the SQL and append only the predicates that are present. Use a query builder (goqu in Go), or a small hand-written builder with bound parameters. Keep fixed-shape queries in sqlc.

## Batching and N+1

One query per parent row is N+1. Batch by key array:

```sql
SELECT id, tenant_id, name FROM document WHERE tenant_id = $1 AND id = ANY($2::uuid[]);
```

- For bulk writes, send columnar arrays with `unnest`:

```sql
-- column-wise batch upsert, one round trip
INSERT INTO product (tenant_id, sku, name, updated_at)
SELECT $1, s, n, t FROM unnest($2::text[], $3::text[], $4::timestamptz[]) AS u(s, n, t)
ON CONFLICT (tenant_id, sku) DO UPDATE
SET name = EXCLUDED.name, updated_at = EXCLUDED.updated_at;
```

- N+1 often hides one level down: a per-page loader that queries per row, or a resolver without a batch loader.

## Upsert

Write current state with one `INSERT ... ON CONFLICT ... DO UPDATE`. Never delete then reinsert. Outside a transaction it loses rows on a crash between the two statements. Inside one it still churns indexes, fires cascades and delete triggers, and lets two writers race without a unique key.

```sql
INSERT INTO user_setting (tenant_id, user_id, key, value)
VALUES ($1, $2, $3, $4)
ON CONFLICT (tenant_id, user_id, key) DO UPDATE
SET value = EXCLUDED.value, updated_at = now()
WHERE user_setting.value IS DISTINCT FROM EXCLUDED.value;
```

- The conflict target needs a matching unique index or constraint. A partial unique index works if the statement repeats its predicate: `ON CONFLICT (col) WHERE pred`.
- The trailing `WHERE ... IS DISTINCT FROM` skips no-op updates (less WAL, fewer dead rows).
- Don't use `xmax = 0` to tell inserts from updates. It relies on internals. On PG18+ use `RETURNING old.*, new.*`.

## Identifiers you cannot bind

Values bind as `$n`. Column names, sort direction and table names cannot be bound. Map user input through a fixed allowlist and fall back to a default.

```go
var sortCols = map[string]string{"created": "created_at", "name": "name"} // allowlist only
```

A GraphQL `orderBy` enum that maps to fixed columns is the same idea. Never splice `?sort=` text into SQL.

## Full-text and pattern search

- Bind the user's text as a parameter (that stops injection) and parse it with `websearch_to_tsquery` (that stops syntax errors). `to_tsquery` on raw input throws on punctuation.
- Always pass the text-search config explicitly: `websearch_to_tsquery('english', $2)`. Use `'simple'` for ids, codes and emails, which stemming would mangle.
- Query a stored generated `tsvector` column that has a GIN index. Recomputing `to_tsvector(...)` in the WHERE clause skips the index.
- `ILIKE '%x%'` scans the whole table. When it is measured as hot, a `pg_trgm` GIN index makes it indexable with no query change.
- Rank and limit first, then run `ts_headline` on the survivors.

## Predicates that silently go wrong

- `NOT IN (subquery)` returns no rows if the subquery yields any NULL. Use `NOT EXISTS`.
- `DATE(created_at) = $1` or `created_at::date = $1` can't use the index on `created_at`. Use a half-open range: `created_at >= $1 AND created_at < $1 + interval '1 day'`.
- `BETWEEN '2024-02-01' AND '2024-02-28'` on a timestamp drops everything after midnight on the 28th. Use half-open ranges for time.
- `now()` is the start of the transaction, so it is the same for every statement and row inside it. To measure time passing inside a long transaction, use `clock_timestamp()`.
- Check the direction of every interval comparison. `last_seen <= now() - interval '7 days'` means "older than 7 days", not "within 7 days".
- Check that a "new since" query really has its time predicate. A query reviewed as "returns new messages" without a `WHERE ... > $since` returns everything.
- A parameter typed wider than the column (`int_col = $1::numeric`) casts the column and can block index use. Bind values with the column's type. (`text_col = 5` is an error in Postgres, not a silent cast.)

## JSONB traps

- `->>` returns text, so cast it before comparing numbers or dates: `(data->>'amount')::numeric`.
- A GIN index serves `@>`, `?` and `?|`. It does not serve `->>` equality. For a hot scalar path, use a B-tree expression index or a stored generated column.
- `jsonb_path_ops` is smaller but supports only `@>` and the jsonpath operators `@?` and `@@`, not the key-existence operators `?`, `?|` and `?&`.
- A missing key and a JSON null both give SQL NULL through `->>`. Use `data ? 'k'` for existence, and `data->'k' = 'null'::jsonb` for an explicit null.
- Every `jsonb_set` writes a whole new row version. Frequent writes to big documents bloat the table.
- Prefer real columns for anything you filter, join or sort on.
