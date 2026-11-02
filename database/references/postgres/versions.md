# Version-gated features

Read before using a feature listed here: check `SHOW server_version;` on the target, then pick the fallback if the server is older. These facts are version-specific and get reviewed on each skill refresh (last checked 2026-11-02, PG 14–18 supported, PG14 end of life Nov 2026).

| Feature | Since | Fallback |
|---|---|---|
| `websearch_to_tsquery` | 11 | `plainto_tsquery` |
| Non-volatile `ADD COLUMN ... DEFAULT` without a table rewrite | 11 | add nullable, backfill, set default |
| `INCLUDE` (covering) indexes | 11 | wider composite index |
| CTEs inlined by default (`MATERIALIZED` to force a fence) | 12 | CTEs always materialize |
| `REINDEX CONCURRENTLY`, generated `STORED` columns | 12 | rebuild by hand, triggers |
| `gen_random_uuid()` in core | 13 | `pgcrypto` |
| `pg_stat_statements.total_exec_time` (renamed from `total_time`) | 13 | `total_time` |
| `DETACH PARTITION ... CONCURRENTLY`, Memoize, JSONB subscripting, `idle_session_timeout` | 14 | plain detach (ACCESS EXCLUSIVE lock) |
| `MERGE`, `UNIQUE NULLS NOT DISTINCT`, publication column lists and row filters | 15 | `INSERT ... ON CONFLICT`, partial unique index |
| `EXPLAIN (GENERIC_PLAN)`, `pg_stat_io` | 16 | run with sample parameters |
| `MERGE ... RETURNING`, `JSON_TABLE`, `transaction_timeout`, COPY `ON_ERROR` | 17 | separate statements, `jsonb_to_recordset` |
| `uuidv7()`, `RETURNING old/new`, B-tree skip scan, virtual generated columns, `WITHOUT OVERLAPS`, `NOT ENFORCED` constraints | 18 | app-generated UUIDv7, separate SELECT, a leading-column index |

Also on PG18: `EXPLAIN ANALYZE` includes `BUFFERS` by default, async I/O arrives, and md5 password auth is deprecated.

Behaviour differences worth a re-test after an upgrade:
- the planner can change plans for the top queries
- statistics may need a fresh `ANALYZE` (PG18's `pg_upgrade` keeps most of them)
- since PG15, `CREATE` on schema `public` is no longer granted to `PUBLIC`
