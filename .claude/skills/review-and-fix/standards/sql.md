# SQL / Database Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers `.sql` files, migrations (Flyway, Liquibase, EF, Alembic, Rails, Knex, Prisma), stored procedures, and SQL strings embedded in application code.

## Authorities
- The vendor documentation for the actual engine (PostgreSQL, SQL Server, Oracle, MySQL/MariaDB). Check which engine is used before judging syntax or locking behavior.
- OWASP SQL Injection Prevention and Query Parameterization cheat sheets.

## Tooling to look for and run
- `sqlfluff lint` (if configured)
- The migration tool's validate or dry-run command
- `EXPLAIN` / `EXPLAIN ANALYZE` or the execution plan, when a database is available and the user agrees

## Security
- `[C]` Dynamic SQL inside procedures built from parameters: `EXEC(@sql)`, `EXECUTE IMMEDIATE`, `EXECUTE format(...)` without `%I` / `%L` or `quote_ident`. Use `sp_executesql` with parameters, or bind variables.
- `[C]` Application SQL built by concatenation or interpolation (see the platform file for the API).
- `[H]` The app connects as a superuser or `sa`/`root`; `GRANT ALL`; new tables without the row-level security or permissions their siblings have.
- `[H]` Sensitive columns (passwords, card data, national IDs) stored in plain text where the schema elsewhere uses hashing or encryption.

## Data integrity
- `[C]` `UPDATE` or `DELETE` without a `WHERE` clause, or with a `WHERE` that can be accidentally empty or always true.
- `[H]` Multi-statement changes that must succeed together, with no transaction.
- `[H]` Read-then-write races (`SELECT` a balance, then `UPDATE`). Use an atomic `UPDATE ... WHERE`, `SELECT ... FOR UPDATE`, optimistic version columns, or the right isolation level.
- `[H]` Missing `NOT NULL`, `UNIQUE`, `FOREIGN KEY` or `CHECK` constraints that the application assumes are there.
- `[M]` Money in `FLOAT` or `REAL`. Use `DECIMAL` / `NUMERIC`. Timestamps without time zone, where the system spans time zones.

## Correctness
- `[H]` `= NULL` or `<> NULL`, which is never true. Use `IS NULL` or `IS NOT NULL`.
- `[H]` `NOT IN (subquery)` where the subquery can return NULL. The result is always empty. Use `NOT EXISTS`.
- `[H]` `LEFT JOIN` turned into an inner join by a `WHERE` condition on the right-hand table.
- `[M]` Aggregates mixed with non-grouped columns (MySQL without `ONLY_FULL_GROUP_BY`); `BETWEEN` on timestamps that misses the last day; integer division where a fraction was expected.
- `[M]` `SELECT *` in application queries, which breaks when columns change and over-fetches; `ORDER BY` missing where the result order matters, for example with paging.

## Performance
- `[H]` Non-sargable predicates on indexed columns: functions applied to the column (`WHERE YEAR(created) = 2024`, `LOWER(email) = ...` without a functional index), or implicit type conversions.
- `[H]` Missing indexes on foreign-key or join columns of large tables, or on new frequent filters; N+1 queries driven from application loops.
- `[M]` Leading-wildcard `LIKE '%x'` on large tables; `OFFSET` paging deep into big tables (use keyset paging); cursors or row-by-row loops where a set-based statement would do.

## Migrations
- `[H]` Editing a migration that has already been applied in shared environments. Add a new migration instead.
- `[H]` Locking operations on large tables in a single deploy: adding indexes without `CONCURRENTLY` (PostgreSQL) or `ONLINE = ON` (SQL Server Enterprise); adding a `NOT NULL` column that needs a table rewrite on the engine or version in use; changing a column type.
- `[H]` Dropping or renaming columns or tables that the currently deployed code still reads. Use expand-then-contract across releases.
- `[M]` No down or rollback migration, where the project writes them; data backfills in the same transaction as the schema change on large tables.
