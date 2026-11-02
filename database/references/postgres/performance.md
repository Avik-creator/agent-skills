# Performance

Read when a query or endpoint is slow, before adding an index or cache, or before quoting any latency, throughput or size number.

## Contents
- Measure first
- Reading a plan
- Common fixes
- Finding the hot queries
- Measurement basis

## Measure first

- Reproduce on realistic data, such as a restored production dump or a production-sized local copy, at the same Postgres major version. Plans on a 100-row dev table say nothing about 10M rows.
- Get the real plan:

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

- `ANALYZE` executes the statement. For writes, wrap it so nothing persists:

```sql
BEGIN;
EXPLAIN (ANALYZE, BUFFERS) UPDATE ...;
ROLLBACK;
```

- For a parameterized statement you can't run safely, use `EXPLAIN (GENERIC_PLAN)` (PG16+).
- Check waits before tuning SQL. A query blocked on a lock (`pg_stat_activity.wait_event_type = 'Lock'`) needs the blocker fixed, not an index.
- Change one thing, re-measure the same way, and report before and after.

## Reading a plan

- For a big plan, open [pev2.html](../../assets/pev2.html) (PEV2, the offline build of explain.dalibo.com; download steps in links.md) in a browser and paste `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` output. It runs locally, so the plan never leaves the machine.
- Read bottom-up. A node's actual time includes its children, so the slowest node is the one with the largest time after subtracting its children's times.
- Node times are per loop, so multiply by `loops`.
- If estimated rows and actual rows differ by about 10× or more, the statistics are stale or the columns are correlated. Fix stats before adding an index:
  - run `ANALYZE` on the table
  - raise `SET STATISTICS` on skewed columns
  - add `CREATE STATISTICS ... (dependencies, ndistinct, mcv)` for correlated columns
  - `pg_stat_user_tables.n_mod_since_analyze` shows how stale the stats are
- `Rows Removed by Filter` in the thousands next to a small output means the predicate isn't in the index condition. Reshape the query (for example, use a row comparison) or adjust the index.
- `Sort Method: external merge` and hash `Batches > 1` mean spills to disk. First try an index that already delivers the order, then a `work_mem` raise for that transaction only (`SET LOCAL`).
- A Seq Scan is correct when the table is small or most rows match. Don't index reflexively.
- A `SubPlan` with a high loop count is a correlated subquery. Rewrite it as a JOIN or `LATERAL`.

## Common fixes

| Symptom | Usual fix |
|---|---|
| deep pages slow | keyset pagination (queries.md) |
| `col::date = $1` or a function on the column | half-open range, or an expression index |
| `(arg IS NULL OR col = arg)` | build predicates conditionally |
| many small identical queries | batch with `= ANY($1)` (N+1) |
| `SELECT *` on wide or TOASTed rows | select only the needed columns; large text and jsonb come back de-TOASTed in full |
| top-N per group | `LATERAL (... ORDER BY ... LIMIT n)` with a matching index |
| prepared statement fast for one tenant, slow for another | generic plan on skewed data; test with `SET LOCAL plan_cache_mode = force_custom_plan` |
| slow `ILIKE '%x%'` | `pg_trgm` GIN index once measured hot |
| JIT time dominating short queries | raise `jit_above_cost` or disable JIT for OLTP |
| request makes 30–50 queries | fix the call pattern (fan-out, refetch storms) before the SQL |

Caches, materialized views, denormalized tables and partitioning each need a measured bottleneck that simpler fixes didn't solve. Partitioning mainly helps retention and maintenance, not raw query speed.

## Finding the hot queries

With `pg_stat_statements`, sort by total time, not mean. Total is where the server's time goes.

```sql
SELECT left(query, 80) AS query, calls,
       round(total_exec_time::numeric) AS total_ms,
       round(mean_exec_time::numeric, 1) AS mean_ms, rows
FROM pg_stat_statements
WHERE dbid = (SELECT oid FROM pg_database WHERE datname = current_database())
ORDER BY total_exec_time DESC
LIMIT 20;
```

- These columns are named `total_exec_time` and `mean_exec_time` since PG13. Older posts say `total_time` and `mean_time`.
- Many seq scans on a big table (a high `seq_tup_read / seq_scan` in `pg_stat_user_tables`), or `tup_returned` far above `tup_fetched` in `pg_stat_database`, points at a missing index or a poorly shaped predicate.
- Tracing (OpenTelemetry spans per query) shows how many queries one request makes. That is often the real problem.

## Measurement basis

State the unit, the basis and the conditions with every number: cold or warm cache, how many runs, which dataset, and whether the client network is included.

- `pg_column_size(col)` is the compressed on-disk size. Wire or serialized size is `octet_length(col::text)`. The two can differ by several times.
- Measure the real workload. A number from a synthetic proxy query set is labelled as a proxy, not quoted as the real figure.
- Before quoting a throughput ceiling, rule out the client. Results that all cluster at one bandwidth measure your link, not the server.
- `effective_cache_size` is a planner hint, not allocated memory. Cache hit rates under 99% justify more memory only when the cause isn't table scans.
- When a figure turns out wrong, correct it plainly once.
