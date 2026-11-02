---
name: database
description: Designs, writes, reviews and changes database-backed application code safely - tenant-scoped, index-aware queries, keyset pagination, schema and constraint design, zero-downtime migrations, race-free transactions, connection pooling, and careful access to real databases - with detailed Postgres guidance. Use when writing or reviewing SQL or a data-access layer (query builders, generated queries, drivers, ORMs), adding a table, column, index or migration, building a list or search endpoint, chasing a slow query or connection exhaustion, sizing a pool, or running anything against a dev, staging, preview or production database, even when the user never names the database. Not for warehouse or ELT pipelines (dbt, Spark, Airflow) or server administration such as replication, backups and major-version upgrades.
---

# Database

Most data bugs come from a handful of mistakes that look harmless in review: a missing tenant filter, a read-then-write race, an index the query can't use, a migration that locks a hot table, or a write to the wrong database. Every rule below carries its reason, so apply the reason to cases the rule doesn't name.

## Workflow

### 1. Know where you are
- Name the target before running anything: engine and version, host, database, role. Config files and build targets often point "local" commands at remote databases.
- Treat production as read-only by default: write scripts for a human to run there, and investigate on a restored copy. Write to it only when the user explicitly asks and confirms the exact statement.
- Before any write to a shared database, show the target, the statement and the expected row count, then ask.

### 2. Read before writing
- Read the table definitions, their indexes, and two or three existing queries on the same tables. Shape the new query to an index that already exists.
- Reuse an existing query, helper or row type before adding a new one.

### 3. Write it
1. **When data is multi-tenant, filter by the tenant key in every statement, subquery and join**, and join on composite `(tenant_id, id)` keys. Why: one missing filter leaks data across tenants.
2. **Prefer plain queries over views, stored functions and triggers for application logic.** Why: they hide where the tenant filter and business rules live, so reviews can't see them.
3. **Paginate by keyset**: a seek on the sort columns plus a unique tiebreaker, `ORDER BY` matching an index, and a server-capped limit. Avoid OFFSET and per-page total counts unless asked. Why: OFFSET reads and discards every earlier row.
4. **Add an index only for a real query that needs it.** Check the existing ones first, and never drop one without asking. Why: every index taxes every write.
5. **Write current state with one atomic upsert**, never delete-then-insert. Keep related writes in one transaction. Why: outside a transaction delete-then-insert loses rows on a crash, and inside one it still fires cascades and delete triggers, churns indexes, and races without a unique key.
6. **Let the database enforce invariants**: NOT NULL, CHECK, UNIQUE (scoped to the tenant when there is one), foreign keys. Use conditional `UPDATE ... WHERE` instead of read-then-write. Why: application checks race, constraints don't.
7. **Bind every value as a parameter.** Map identifiers (sort column, direction) through an allowlist. Why: injection, and identifiers can't be bound.
8. **Keep session state off pooled connections.** Scope settings to one transaction. Why: a transaction-mode pooler hands your session to the next client.
9. **Make migrations ordered, re-runnable and lock-safe**: with numbered files, take the next number after the default branch's latest (timestamped tools such as Rails, Django or Prisma order themselves); set a lock timeout; expand, then migrate, then contract. Never apply branch migrations to shared environments. Why: a stray apply burns a version in the ledger, and one long lock stalls every query.
10. **Call nothing external inside a transaction or before it commits.** Record events in an outbox in the same transaction. Why: retries and rollbacks duplicate or orphan side effects.

Example of rule 6 (any engine):
```sql
-- bad: read in the app, then write; a concurrent withdrawal is silently lost
SELECT balance FROM account WHERE tenant_id = ? AND id = ?;
UPDATE account SET balance = ? WHERE tenant_id = ? AND id = ?;
-- good: one guarded statement; 0 rows affected means insufficient funds or no such row
UPDATE account SET balance = balance - ? WHERE tenant_id = ? AND id = ? AND balance >= ?;
```

### 4. Prove it
- Read the real execution plan on production-sized data before claiming a query is fast.
- Test data-access code against the real engine and version, and watch each test fail once. In-memory fakes hide constraint violations, races and bad SQL.
- After a rejected, cancelled or timed-out write, check the state it would have changed before doing anything else. It may have committed.

### 5. Report
- Give every number its unit and basis: cache state, dataset, runs. Check that a "size" means what you think (on-disk compressed or bytes on the wire).
- Say what you didn't verify.

## Gotchas (any SQL engine)
- `(param IS NULL OR col = param)` usually defeats the index. Build predicates conditionally.
- A function or cast on an indexed column (`DATE(col)`) skips a plain index on that column. Use half-open ranges `>= start AND < end`, or a matching expression index.
- `NOT IN (subquery)` returns nothing if the subquery yields a NULL. Use `NOT EXISTS`.
- On engines with concurrent writers, polling `WHERE id > last_seen` skips rows whose transaction commits late.
- "Check, then insert" races. Rely on a unique constraint and handle the violation.
- An update that affects 0 rows is an outcome to handle, not a success. On MySQL, 0 can also mean matched but unchanged, unless the client sets `CLIENT_FOUND_ROWS`.
- Adding a column with a default fills existing rows on most engines, so a later backfill `WHERE col IS NULL` matches nothing. SQL Server leaves a nullable column NULL unless you add `WITH VALUES`.

## Postgres
Read only the file the task needs.

| File | Read when |
|---|---|
| [queries.md](references/postgres/queries.md) | writing or reviewing queries: tenant scoping, keyset, filters, batching, upsert, search, JSONB |
| [schema-and-indexes.md](references/postgres/schema-and-indexes.md) | tables, types, constraints, multi-tenant keys, index shapes, index audits |
| [migrations.md](references/postgres/migrations.md) | writing, numbering or running a migration or backfill |
| [transactions.md](references/postgres/transactions.md) | read-then-write, locking, queues, retries, webhooks and other at-least-once handlers, idempotency, outbox, polling |
| [connections.md](references/postgres/connections.md) | pools, PgBouncer-style poolers, session settings, timeouts, connection exhaustion |
| [performance.md](references/postgres/performance.md) | slow queries, EXPLAIN (with an offline plan viewer, [assets/pev2.html](assets/pev2.html)), hot-query hunting, measurement basis |
| [safe-access.md](references/postgres/safe-access.md) | touching any non-throwaway database, copying data, credentials |
| [drivers.md](references/postgres/drivers.md) | Go (pgx, sqlc, query builders) or JS/TS (node-postgres, ORMs) data-access code |
| [versions.md](references/postgres/versions.md) | using a feature tied to a Postgres version |
| [links.md](references/postgres/links.md) | the files above don't cover it, or you must confirm syntax, locks or version behaviour at the source |

## Other engines
No engine-specific references exist yet. Apply the rules above, and check syntax, locking and online-DDL behaviour in that engine's own documentation rather than carrying Postgres syntax over. For example, MySQL upserts use `ON DUPLICATE KEY UPDATE`, and online DDL is controlled with `ALGORITHM` and `LOCK`. Add a `references/<engine>/` folder when one is needed.
