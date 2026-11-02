# Where to look next

Read when the other references don't cover the case, or you need to confirm exact syntax, lock levels or version behaviour. Prefer the official docs. Treat blogs and tools as secondary, and check their date and Postgres version.

## Contents
- Official documentation
- Skill collections and vendor docs
- Community guidance
- Plans and indexing
- Migrations
- Pooling and drivers

## Official documentation

Use the docs for the target's major version: replace `current` in the URL with the number, such as `/docs/16/`.

| Topic | Link |
|---|---|
| Docs home | https://www.postgresql.org/docs/current/ |
| `ALTER TABLE` (which lock each form takes) | https://www.postgresql.org/docs/current/sql-altertable.html |
| Lock modes and conflicts | https://www.postgresql.org/docs/current/explicit-locking.html |
| Isolation levels | https://www.postgresql.org/docs/current/transaction-iso.html |
| `INSERT ... ON CONFLICT` | https://www.postgresql.org/docs/current/sql-insert.html |
| `MERGE` | https://www.postgresql.org/docs/current/sql-merge.html |
| Indexes, `CREATE INDEX` | https://www.postgresql.org/docs/current/indexes.html, https://www.postgresql.org/docs/current/sql-createindex.html |
| Data types, JSON types | https://www.postgresql.org/docs/current/datatype.html, https://www.postgresql.org/docs/current/datatype-json.html |
| Full text search | https://www.postgresql.org/docs/current/textsearch.html |
| Row security policies | https://www.postgresql.org/docs/current/ddl-rowsecurity.html |
| Using `EXPLAIN` | https://www.postgresql.org/docs/current/using-explain.html |
| Statistics views (`pg_stat_*`) | https://www.postgresql.org/docs/current/monitoring-stats.html |
| `pg_stat_statements` | https://www.postgresql.org/docs/current/pgstatstatements.html |
| Timeouts and session settings | https://www.postgresql.org/docs/current/runtime-config-client.html |
| SQLSTATE error codes | https://www.postgresql.org/docs/current/errcodes-appendix.html |
| Release notes | https://www.postgresql.org/docs/release/ |
| Supported versions and end-of-life dates | https://www.postgresql.org/support/versioning/ |

## Skill collections and vendor docs

Other agent skills for Postgres. Read them for a second opinion, and keep this skill's rules where they disagree.

- Neon postgres-skills: https://github.com/neondatabase/postgres-skills
- Microsoft postgres-skills: https://github.com/microsoft/postgres-skills
- PlanetScale database-skills (Postgres and MySQL): https://github.com/planetscale/database-skills
- wshobson/agents, database plugins (good for spotting anti-patterns, not as rules): https://github.com/wshobson/agents

Vendor docs and learning material. Platform pages apply to that platform's pooler and limits only:

- Neon connection pooling: https://neon.com/docs/connect/connection-pooling
- Neon Postgres tutorial: https://neon.com/postgresql/tutorial
- PlanetScale Postgres docs: https://planetscale.com/docs/postgres
- PlanetScale blog and learning material: https://planetscale.com/blog, https://planetscale.com/learn
- Azure Database for PostgreSQL docs: https://learn.microsoft.com/en-us/azure/postgresql/

## Community guidance

- PostgreSQL wiki, "Don't Do This" (types and patterns to avoid): https://wiki.postgresql.org/wiki/Don%27t_Do_This
- PostgreSQL wiki, lock monitoring queries: https://wiki.postgresql.org/wiki/Lock_Monitoring
- Changes between two versions, gathered from the release notes: https://why-upgrade.depesz.com/

## Plans and indexing

- Use The Index, Luke (how indexes serve queries, on any engine): https://use-the-index-luke.com/
- Keyset pagination explained: https://use-the-index-luke.com/no-offset
- Plan visualizer, offline: [pev2.html](../../assets/pev2.html) is not in the repo. Download it once from https://github.com/dalibo/pev2/releases/download/v1.25.0/pev2.html into `database/assets/` and check `shasum -a 256` gives `a9c992a85aa111b4442ece08b1f1abcd92f5a651cb41fc9510a74ad4b3bba99f`. Without it, use https://explain.dalibo.com. Open it in a browser and paste the plan. Its only network call is a GitHub update check, which doesn't send the plan. Project: https://github.com/dalibo/pev2
- Plan visualizers, online: https://explain.dalibo.com/, https://explain.depesz.com/

Plans contain table names, and with `VERBOSE` or `ANALYZE` they include literal values. Use the offline copy for any non-throwaway database, and ask before pasting such a plan into a public site.

## Migrations

- Squawk, a linter that flags lock-unsafe migration SQL: https://squawkhq.com/docs/
- strong_migrations, a catalogue of unsafe operations and their safe forms (written for Rails, but it applies to any Postgres migration): https://github.com/ankane/strong_migrations

## Pooling and drivers

- PgBouncer: what each pool mode supports: https://www.pgbouncer.org/features.html
- PgBouncer: configuration: https://www.pgbouncer.org/config.html
- pgx (Go): https://pkg.go.dev/github.com/jackc/pgx/v5
- sqlc: https://docs.sqlc.dev/
- node-postgres: https://node-postgres.com/
