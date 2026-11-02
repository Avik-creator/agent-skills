# Connections and pooling

Read when configuring a pool or connection URL, running anything through a pooler (PgBouncer, Neon `-pooler`, managed poolers), setting session parameters, using LISTEN/NOTIFY or advisory locks, or debugging "too many connections" and slow acquires.

## Contents
- Transaction-mode poolers
- Setting parameters safely
- Pool sizing
- Timeouts
- Connection strings
- Diagnosing

## Transaction-mode poolers

A transaction-mode pooler hands your server connection to another client after each transaction. Session state either leaks to that client or disappears.

| Feature | Behind a transaction pooler |
|---|---|
| `SET x = y` / `SET SESSION` | leaks to the next client on that connection |
| `SET LOCAL x = y` inside a transaction | safe |
| SQL `PREPARE` / `EXECUTE` | breaks ("prepared statement does not exist") |
| Protocol-level prepared statements | need pooler support (PgBouncer 1.21+ `max_prepared_statements`), or turn driver statement caching off |
| `LISTEN` | receives nothing, silently; use the direct host (`NOTIFY` is fine, it is sent at commit) |
| `pg_advisory_lock` | breaks; use `pg_advisory_xact_lock` |
| Temp tables across transactions | gone |

Example: someone runs `SET default_transaction_read_only = on` through the pooler to make an inspection session "safe". The next application client handed that server connection fails with `cannot execute SELECT FOR UPDATE in a read-only transaction` (SQLSTATE 25006). The fix is `RESET default_transaction_read_only` through the pooler until every pooled backend reports `off`.

## Setting parameters safely

For read-only inspection:

```sql
BEGIN READ ONLY;
SELECT count(*) FROM document WHERE tenant_id = $1;
COMMIT;
```

Or connect to the direct (non-pooler) host for maintenance sessions.

Per-transaction settings, such as a tenant id for RLS or a timeout:

```sql
BEGIN;
SELECT set_config('app.tenant_id', $1, true);
SET LOCAL statement_timeout = '5s';
SELECT ... ;
COMMIT;
```

- `SET LOCAL` and `set_config(..., true)` only last until the end of the current transaction.
- Run as a standalone statement outside `BEGIN`, `SET LOCAL` does nothing. Postgres prints a warning, and the next query runs without the setting.
- Verify with `SELECT current_setting('app.tenant_id', true)` inside the same transaction.

## Pool sizing

- The real load is pool size × instances × pools per instance. That includes auth, config and LISTEN connections, which are easy to forget. Keep the total under the server's limit, minus its reserved slots.
- Request fan-out multiplies demand. One request that runs 15 parallel queries against a 10-connection pool starves everything else. Cap concurrency per request, and don't run blocks a page doesn't need.
- Prefer a pooler and modest pools over raising `max_connections`. Each server connection is a process with its own memory, and `work_mem` applies per operation.
- Create the pool once at startup and share it. Creating a pool per request or per handler exhausts the server.
- Release connections on every path. In node-postgres, `client.release()` goes in `finally`. Prefer `pool.query` for single statements.

## Timeouts

Set them deliberately. Defaults are "wait forever".

| Setting | Purpose |
|---|---|
| `statement_timeout` | cap a runaway query (set per role, per session on a direct connection, or `SET LOCAL`) |
| `idle_in_transaction_session_timeout` | kill sessions that hold a transaction open (an app that forgot to commit) |
| `lock_timeout` | fail fast instead of queueing behind a lock (migrations especially) |
| connect timeout | `connect_timeout` in libpq, or driver config; without it startup can hang on TCP |
| pool acquire timeout | bound how long a request waits for a free connection |

- HTTP server timeouts don't cancel database work unless the request context reaches the query. Pass the context through.
- Set `application_name` per service, so `pg_stat_activity` shows who holds the connections.

## Connection strings

- Driver-only URL parameters (pgx's `pool_max_conns`, for example) break other clients. libpq rejects them. A plain `pgx.Connect` sends them as startup parameters and fails with `unrecognized configuration parameter`. Strip them for psql and single connections.
- Use `sslmode=verify-full` with the provider's CA for production. `require` encrypts but doesn't verify the server.
- Never paste, echo or commit connection strings with passwords. Use env vars or a secret store. Some make targets echo the full URL, so don't run them where output gets shared.

## Diagnosing

```sql
-- who holds connections, in what state
SELECT application_name, usename, state, count(*)
FROM pg_stat_activity
WHERE backend_type = 'client backend'
GROUP BY 1, 2, 3
ORDER BY 4 DESC;

-- sessions stuck inside a transaction
SELECT pid, now() - xact_start AS age, left(query, 80)
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY age DESC;

-- who blocks whom
SELECT pid, pg_blocking_pids(pid) AS blocked_by, left(query, 80)
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;
```

- `FATAL: remaining connection slots are reserved` means the cluster is out of slots. Count the clients before blaming one service.
- Cancel (`pg_cancel_backend`) before you terminate (`pg_terminate_backend`). Ask before doing either on a shared database.
