# Migrations

Read when writing, numbering, reviewing or running a schema migration or a data backfill.

## Contents
- Numbering and ledgers
- Where a migration runs
- Lock safety
- Change recipes
- Backfills
- Before you call it done

## Numbering and ledgers

- Number a new migration as the default branch's highest number + 1. Check with `git ls-tree --name-only origin/main <migrations-dir>/ | tail -1`. The trailing slash lists files, not the directory entry, and the sort is bytewise, so it needs zero-padded or timestamped names.
  - Don't skip ahead to dodge numbers that other open branches hold. Collisions get resolved at merge or rebase, so mention the known ones in the PR.
- Migration tools that track versions in a table (goose, Flyway, golang-migrate and similar) record each version as a row, and a recorded row means "applied" whatever DDL actually ran. So an out-of-band apply from a feature branch burns that number on a shared database, and the real file with that number is skipped forever.
  - Never run branch migrations against shared environments.
  - Before trusting the ledger, check that the objects exist: `SELECT to_regclass('public.some_table');`.
- Duplicate version numbers in one tree can make the tool refuse to start. Resolve duplicates before merging.
- Write DDL to be re-runnable where the tool allows: `CREATE TABLE IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`. Re-runs happen after partial failures and ledger repairs.
  - Exception: `CREATE INDEX CONCURRENTLY IF NOT EXISTS` succeeds silently over an INVALID index. Check `pg_index.indisvalid` before retrying.

## Where a migration runs

- Resolve the target first. Make targets and scripts often load `.env`, whose URL may be a remote database. Pass the connection string explicitly for local runs.
- Don't print or log connection strings. They carry passwords.
- Production migrations run through the normal deploy path, never by hand from a branch.
- Test the migration on a copy with production-scale data: a restored dump or a local clone. A change that takes 2 ms on 100 rows can lock a 100M-row table for minutes.

## Lock safety

DDL waits for its lock, and every later query waits behind the DDL. One long transaction can turn a fast `ALTER TABLE` into an outage.

```sql
-- fail fast instead of queueing behind a long transaction
BEGIN;
SET LOCAL lock_timeout = '3s';
SET LOCAL statement_timeout = '15min';
ALTER TABLE document ADD COLUMN IF NOT EXISTS label text;
COMMIT;
```

`SET LOCAL` only works inside a transaction; outside one it just warns and does nothing. Drop the `BEGIN`/`COMMIT` if your migration tool already wraps each file in a transaction, but check that it does.

| Change | Safe online? | How |
|---|---|---|
| Add nullable column, no default | yes | metadata only |
| Add column with constant or `now()` default | yes (PG11+) | non-volatile defaults are stored in the catalog, no rewrite |
| Add column with volatile default (`clock_timestamp()`, `gen_random_uuid()`, `random()`) | no | rewrites the table; add nullable, backfill, then set the default |
| Add index | yes | `CREATE INDEX CONCURRENTLY`, outside a transaction block |
| Add FK or CHECK | yes | `ADD CONSTRAINT ... NOT VALID`, then `VALIDATE CONSTRAINT` in a separate step |
| Set NOT NULL on a big table | yes | CHECK NOT VALID, VALIDATE, SET NOT NULL, drop the CHECK |
| Change column type | usually no | full rewrite; use add-new-column and switch |
| Rename column or table | breaks running code | expand/contract |
| Drop column | yes for the DB | only after no deployed code reads it |

`CREATE INDEX CONCURRENTLY` can't run inside a transaction or a `DO` block, and not on a partitioned parent. Put it in its own migration file, marked as non-transactional for your tool.

## Change recipes

Safe NOT NULL:

```sql
ALTER TABLE task ADD CONSTRAINT task_title_nn CHECK (title IS NOT NULL) NOT VALID;
ALTER TABLE task VALIDATE CONSTRAINT task_title_nn;
ALTER TABLE task ALTER COLUMN title SET NOT NULL;
ALTER TABLE task DROP CONSTRAINT task_title_nn;
```

Rename by expand, migrate, contract:
1. Add the new column (or table).
2. Deploy code that writes both and reads the old one.
3. Backfill in batches.
4. Deploy code that reads the new one.
5. Drop the old one in a later migration, after a soak.

Each step must work with the code version before it and the one after it.

Changing a function: `CREATE OR REPLACE FUNCTION` with different argument types does not replace it; it creates a second function with the same name. It also can't change the return type. For either, `DROP FUNCTION` and `CREATE FUNCTION` in one migration, and recreate the views, triggers and rules that depend on it.

## Backfills

- Batch them, and commit between batches from the client or the migration runner. A `DO $$ LOOP ... $$` block is one transaction: locks, WAL and dead rows pile up until it ends. `COMMIT` inside `DO` fails when the runner already wrapped the file in a transaction.
- Use keyset batches over the primary key, not OFFSET:

```sql
UPDATE task SET title = trim(title)
WHERE (tenant_id, id) IN (
  SELECT tenant_id, id FROM task
  WHERE (tenant_id, id) > ($1, $2) AND title <> trim(title)
  ORDER BY tenant_id, id
  LIMIT 5000
)
RETURNING tenant_id, id;
```

  The caller feeds the largest returned key back in as the next `$1, $2`.
- Make the batch predicate select only rows that still need work, so a re-run is safe. Watch for a predicate that can never match: add a column `DEFAULT false`, then backfill `WHERE col IS NULL`, and the backfill does nothing, because the default already filled every row.
- Never fabricate values for missing required data during a copy. Fail and report the rows instead.
- For a big one-shot copy: create the target, load it, then build indexes, then `ANALYZE`.

## Before you call it done

- Run up on a fresh database and on a copy of the current production schema. Run down (or the documented rollback) if the project keeps down migrations.
- Re-run "up" to prove it's idempotent where the tool allows.
- For destructive steps (drop, truncate, irreversible data rewrite), write the rollback path: a backup, a restore point, or a compensating script. Ask before running anything destructive on a shared database.
