# Drivers and query layers

Read when writing data-access code in Go (pgx, sqlc, goqu) or JavaScript/TypeScript (node-postgres, ORMs).

## Contents
- Choosing the layer (Go)
- pgx
- sqlc
- Query builders for dynamic filters
- node-postgres
- ORMs
- Testing data-access code

## Choosing the layer (Go)

Split by the query's character:

| Query character | Tool |
|---|---|
| fixed shape: CRUD, by-id or by-key lookups, reports, transactional writes | sqlc (compile-time types) |
| list or search with many optional filters and keyset navigation | a query builder (goqu) that appends only the present predicates |
| columnar batch writes with `unnest` | raw SQL through pgx |

Before adding a query, look for an existing one or a helper that does the same job.

## pgx

- Create one `pgxpool.Pool` per database at startup and pass it in. Don't use a lazily created global, and don't create a pool per request.
- Pass the request `context.Context` into every query, so timeouts and cancellations reach Postgres.
- Batch reads with `= ANY($1)` and a Go slice. pgx encodes slices as arrays.
- Collect rows with `pgx.CollectRows` / `pgx.CollectOneRow` and `pgx.RowToStructByName`. Name the scanned fields, so a schema change fails loudly.
- Map `pgx.ErrNoRows` to not-found in exactly one helper. Don't scatter the check.
- Map unique violations (`23505`, via `*pgconn.PgError`) to a conflict error.
- Run transactions with `pgx.BeginFunc` or Begin, defer Rollback, then Commit. Rollback after Commit is a no-op.
- Behind a transaction pooler that can't handle prepared statements, configure the statement cache or exec mode for it (for example `QueryExecModeExec` or simple protocol). Verify against the pooler you have.
- Keep pgx-only URL parameters (such as `pool_max_conns`) out of URLs that other tools read.

## sqlc

- `sqlc.narg` combined with `(narg IS NULL OR col = narg)` defeats indexes. Keep sqlc for fixed shapes, and move filter-heavy searches to a builder.
- Several queries returning the same columns can share one row type, plus one mapper. A struct conversion fails to compile when the rows drift, which is how you remove duplication without SQL views.
- Regenerate code (`sqlc generate`) in the same commit as the SQL change.

## Query builders for dynamic filters

- Start from the tenant predicate. Then add each filter only when it is present. Then the keyset seek as a row comparison. Then `ORDER BY` matching the index, then a capped `LIMIT`.
- Values go through placeholders, so turn on the builder's prepared or placeholder mode.
- Identifiers (sort columns) come from an allowlist map only.
- Log the generated SQL in tests, and check it with `EXPLAIN` once.

## node-postgres

- Create one `Pool` per process.
- Use `pool.query(text, values)` for single statements. It acquires and releases for you.
- For transactions, acquire the client, then `BEGIN` ... `COMMIT`, with `ROLLBACK` on error and `client.release()` in `finally`. A missing release on an error path leaks connections until "too many clients".
- Always pass parameters (`$1`), never template strings.
- `bigint` and `numeric` come back as strings by default. Parse them deliberately. Don't use floats for money.
- Give the pool sensible `max`, `idleTimeoutMillis` and `connectionTimeoutMillis`, and keep the sum across instances under the server limit.

## ORMs

- Check the SQL the ORM really emits (query logging) for N+1, `SELECT *` and missing tenant filters.
- Use eager loading or batch loaders (DataLoader in GraphQL), created once per request so caches never cross users or tenants.
- Drop to plain SQL for hot paths, upserts, keyset pagination and batch writes.

## Testing data-access code

- Test repository and SQL code against a real Postgres (a container or a local instance) at the production major version. In-memory fakes can't show constraint violations, races, type coercion or plain SQL mistakes.
- Isolate tests in a transaction that rolls back, or in a fresh schema per test.
- Prove a test can fail: break the query (drop the tenant filter, flip the sort) and watch the test go red.
- For performance-sensitive reads, add a test that asserts the plan uses the intended index.
