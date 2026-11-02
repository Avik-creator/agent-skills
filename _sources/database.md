# Sources

Ideas were adapted, not copied wholesale, and a few diagnostic queries were adapted from upstream. To refresh, diff each upstream against the date below and take only what fits the rules in SKILL.md.

Pulled 2026-11-02 from zip downloads. No commit SHA was recorded, so record one on the next refresh.

| Upstream | License | Taken |
|---|---|---|
| neondatabase/postgres-skills (`skills/postgres-best-practices`) | Apache-2.0 | migration safety table, NOT NULL via NOT VALID, isolation and retry rules, pooler incompatibility list, missing-FK-index idea, EXPLAIN reading, version features |
| microsoft/postgres-skills (`plugin/skills/postgresql-best-practices`, `tests/evals`) | MIT | "only what models get wrong" reference style, JSONB, FTS, RLS and connection gotchas, safety model (least privilege, untrusted data, confirm writes), eval prompt ideas |
| planetscale/database-skills (`skills/postgres`, `skills/neki`, `skills/mysql`) | MIT | session-SET-on-pooler rule, unused and duplicate index queries, CIC IF NOT EXISTS trap, over-indexing evidence, composite tenant FK and keyset examples, workflow shape |
| wshobson/agents (database-*, backend-development, framework-migration, data-engineering) | MIT | mostly counter-examples (dual writes, sequence polling gap, check-then-act, offset "cursors", DO-block batching); transactional outbox pattern |
| Author's working rules | n/a | plain queries over views, index-aware keyset reads, composite-key joins, direct upserts, migration numbering and ledger lessons, pooler session leaks, verifying after interrupted writes, measurement basis, read-only production |
| dalibo/pev2 v1.25.0, `pev2.html` release asset, copied unchanged to `assets/pev2.html` (sha256 a9c992a85aa111b4442ece08b1f1abcd92f5a651cb41fc9510a74ad4b3bba99f) | PostgreSQL License | offline EXPLAIN visualizer. To refresh, replace it with the latest release's `pev2.html` |

Every SQL example was executed against PostgreSQL 16. Features gated to 17+ are listed in `references/postgres/versions.md` and were not executed.

## Added 2026-11-02 (source sweep)
Ideas from wshobson/agents `database-design`, `database-migrations`, `database-cloud-optimization` (read by a Cursor subagent), each checked against the PostgreSQL docs on 2026-11-02 before writing:
- `timetz` discouraged ("questionable usefulness") — https://www.postgresql.org/docs/current/datatype-datetime.html
- over-long strings error unless the excess is spaces; an explicit cast truncates; `numeric` rounds scale and errors on integer overflow — datatype-character.html, datatype-numeric.html. The source's "errors, never truncates" was half wrong and was corrected.
- `UNLOGGED` truncated after a crash, not replicated — sql-createtable.html
- default `ON DELETE NO ACTION` — ddl-constraints.html
- `now()` is the transaction start, `clock_timestamp()` changes within a statement — functions-datetime.html
- `CREATE OR REPLACE FUNCTION` with new argument types creates a new function and can't change the return type — sql-createfunction.html
Dropped as wrong in the source: `now()` as volatile, `CREATE INDEX CONCURRENTLY` inside `DO`, `CHECK ... NOT VALID` as PG12+, most-selective column first, optimizer hints, `pgcrypto crypt()` for passwords, `COMMIT` inside a runner's `DO`.
