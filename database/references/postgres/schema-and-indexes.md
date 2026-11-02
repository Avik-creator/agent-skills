# Schema and indexes

Read when adding or changing a table, column, constraint or index, or when reviewing a schema.

## Contents
- Types
- Constraints carry the invariants
- Multi-tenant shape
- No views or SQL functions for app logic
- When to add an index
- Index shapes
- Auditing indexes

## Types

| Use | Instead of | Why |
|---|---|---|
| `bigint GENERATED ALWAYS AS IDENTITY` | `serial`, `bigserial` | identity has clear ownership and blocks accidental manual ids |
| `uuid` from `uuidv7()` (PG18) or app-generated v7 | `gen_random_uuid()` as a hot PK | v4 is random, so inserts scatter across the B-tree |
| `timestamptz` | `timestamp` | stores an instant; naive timestamps get read in whatever session TimeZone is set |
| `text` + `CHECK (length(x) <= n)` | `varchar(n)`, `char(n)` | same storage; a CHECK is easier to change |
| `bigint` cents or `numeric(p,s)` | `float`, `money` | floats can't hold money exactly |
| `text` + `CHECK (x IN (...))` or a lookup table | `CREATE TYPE ... AS ENUM` for business states | you can't remove an enum value |
| `boolean NOT NULL` | nullable flags | three-valued logic surprises |

- Use unquoted `snake_case` everywhere. Quoted mixed-case names need quotes forever.
- Don't name tables `user` or `order`. They are reserved words.
- Identity and sequence values have gaps after rollbacks. That is normal, don't "fix" it.
- Never `timetz` (`time with time zone`). The docs call it of questionable usefulness: a time-zone offset means little without a date. Use `timestamptz`.
- Too-long text is an error, except that an explicit cast (`'too long'::varchar(5)`) truncates silently. `numeric(p,s)` rounds extra decimals and errors only when the integer part overflows. Round money on purpose before storing it.
- `UNLOGGED` tables are truncated after a crash or unclean shutdown and are not replicated to standbys. Use them only for data you can rebuild.

## Constraints carry the invariants

Application checks race, and constraints don't.

- `NOT NULL` wherever the domain requires a value. A `CHECK` passes on NULL, so combine the two.
- Enforce uniqueness with a `UNIQUE` constraint, not with "select, then insert if missing". Two requests both see "missing". Catch the unique violation (SQLSTATE 23505) and map it to a conflict.
- State `ON DELETE` on every foreign key. The default, `NO ACTION`, fails the parent delete while child rows still point at it; choose `CASCADE`, `RESTRICT` or `SET NULL` on purpose.
- Use foreign keys, and index the referencing columns yourself. Postgres doesn't do it, and without the index a parent delete scans the child table.
- `UNIQUE NULLS NOT DISTINCT` (PG15+) when a NULL should count as a value.
- `EXCLUDE USING gist` (with `btree_gist`) to prevent overlapping ranges.
- When a bad state can be made unrepresentable with a constraint, prefer that to validating at the edge.

## Multi-tenant shape

- Put the tenant key on every tenant-owned table.
- Make child primary keys or unique keys lead with it.
- Point foreign keys at `(tenant_id, id)`, so a child can never reference another tenant's parent:

```sql
CREATE TABLE project (
  tenant_id bigint NOT NULL,
  id     bigint GENERATED ALWAYS AS IDENTITY,
  name   text   NOT NULL,
  PRIMARY KEY (tenant_id, id)
);

CREATE TABLE task (
  tenant_id     bigint NOT NULL,
  id         bigint GENERATED ALWAYS AS IDENTITY,
  project_id bigint NOT NULL,
  title      text   NOT NULL,
  PRIMARY KEY (tenant_id, id),
  FOREIGN KEY (tenant_id, project_id) REFERENCES project (tenant_id, id)
);
CREATE INDEX task_tenant_project_idx ON task (tenant_id, project_id);
```

- Scope unique constraints to the tenant: `UNIQUE (tenant_id, email)`, not `UNIQUE (email)`. The exception is a value that truly must be unique across tenants.
- RLS is optional defence in depth on top of explicit filters, not a replacement for them. If used:
  - add `FORCE ROW LEVEL SECURITY` (owners bypass RLS otherwise; superusers and `BYPASSRLS` roles always do)
  - give policies both `USING` and `WITH CHECK`
  - set the tenant with `set_config('app.tenant_id', $1, true)` inside the same transaction (see connections.md)
  - remember that `current_setting('app.tenant_id', true)` returns NULL if never set on the connection, but `''` once an earlier transaction set it. Pooled connections usually hit the `''` case, where a cast like `::uuid` raises an error. Use `nullif(current_setting('app.tenant_id', true), '')`

## No views or SQL functions for app logic

Write plain queries with the tenant filter in the query text. Remove duplication in application code instead, for example shared row types plus one mapper.

- Why: a view or function hides where the tenant filter lives, so a reviewer can't see it, and an unfiltered call path is one refactor away.
- This includes calling existing SQL functions from new queries. Reimplement the logic in code. Watch NULL handling when you do: a non-STRICT function may map NULL to a default value that the copy must preserve.
- Triggers, procedures and materialized views fall under the same rule unless the user asks for one.
- If project docs prefer views or SQL functions, flag the conflict and ask, instead of following either rule silently.

## When to add an index

Add an index for a query that exists and has been shown to need it. Never "just in case".

- Every index costs writes, WAL, memory, vacuum time, and HOT-update eligibility. One measured case (Percona, PG 17) dropped from about 1,400 to 600 TPS (58%) going from 7 to 39 indexes on one table.
- First check whether an existing index already serves the query, possibly after reshaping the query (column order, ORDER BY, row comparison). Prefer reshaping the query to adding an index.
- Don't index columns that change often (`status`, `updated_at`) unless a hot query needs it, because those indexes block HOT updates.
- Don't drop an index without asking, even one with zero scans. Stats may be young, and it may back a constraint or a monthly job.
- When the user says there are too many indexes, audit them (queries below) and propose a list, with evidence for each.

## Index shapes

- **Composite:** equality columns first, then the range or sort column, then the tiebreaker: `(tenant_id, status, created_at DESC, id DESC)`. Only a leftmost prefix is usable (PG18 skip scan relaxes this for low-cardinality leading columns).
- **Partial:** `WHERE deleted_at IS NULL` or `WHERE status = 'pending'` for hot subsets. The query must repeat the predicate.
- **Covering:** `INCLUDE (cols)` enables index-only scans. It still needs a well-vacuumed table (visibility map), and it adds write cost.
- **Expression:** `(lower(email))`. The query must use exactly the same expression.
- **GIN:** jsonb containment, arrays, tsvector, trigram. **BRIN:** huge append-only tables whose physical order matches the column (check `pg_stats.correlation`).
- **Partitioned parents:** a parent can't take `CREATE INDEX CONCURRENTLY`. Build `ON ONLY` on the parent, then build each partition concurrently, then `ATTACH`. Unique indexes must include the partition key.

## Auditing indexes

Check stats age before calling anything unused:

```sql
SELECT stats_reset FROM pg_stat_database WHERE datname = current_database();
```

Unused indexes, excluding unique, expression and constraint-backing ones. `idx_scan` counts only the server you query, so an index used only on a read replica looks unused on the primary:

```sql
SELECT s.relname AS table_name, s.indexrelname AS index_name,
       pg_size_pretty(pg_relation_size(s.indexrelid)) AS size, s.idx_scan
FROM pg_stat_user_indexes s
JOIN pg_index i ON i.indexrelid = s.indexrelid
WHERE s.idx_scan = 0
  AND NOT i.indisunique
  AND 0 <> ALL (i.indkey)
  AND NOT EXISTS (SELECT 1 FROM pg_constraint c WHERE c.conindid = s.indexrelid)
ORDER BY pg_relation_size(s.indexrelid) DESC;
```

Duplicate definitions:

```sql
SELECT tablename, array_agg(indexname) AS duplicates
FROM pg_indexes
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
GROUP BY schemaname, tablename, regexp_replace(indexdef, 'INDEX \S+ ON ', 'INDEX ON ')
HAVING count(*) > 1;
```

Invalid indexes, left behind by a failed concurrent build:

```sql
SELECT indexrelid::regclass FROM pg_index WHERE NOT indisvalid;
```

Foreign keys whose columns no index leads with:

```sql
SELECT c.conrelid::regclass AS table_name, c.conname, pg_get_constraintdef(c.oid)
FROM pg_constraint c
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid
      AND (i.indkey::int2[])[0:cardinality(c.conkey) - 1] = c.conkey
  );
```

Also redundant: `(a)` when `(a, b)` exists. But `(a, b)` is not redundant next to `(a, b, c)` if index-only scans on `(a, b)` matter.
