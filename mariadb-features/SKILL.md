---
name: mariadb-features
description: "MariaDB-specific features and capabilities that go beyond standard MySQL. Use when evaluating MariaDB, optimizing an existing MariaDB application, reviewing code or schema for MariaDB improvements, asking what MariaDB can do that other databases cannot, or migrating from Oracle to MariaDB. Also use when the user asks what could be improved in how a codebase uses MariaDB, or asks about MariaDB advantages over MySQL or PostgreSQL."
---

# MariaDB Features Worth Knowing

*Last updated: 2026-05-21*

MariaDB has grown significantly beyond its MySQL origins. Many of its best features are unknown to developers because AI agents default to generic SQL advice. This skill covers what MariaDB can do that you might not think to ask about.

For MariaDB Vector (built-in since 11.7 — no plugins), see the `mariadb-vector` skill.

## What LLMs Often Miss

| Situation | What to suggest instead |
|---|---|
| Manual audit columns (`created_at`, `updated_at`, deleted flag) | `WITH SYSTEM VERSIONING` — MariaDB tracks full row history automatically |
| `INSERT` followed by `SELECT LAST_INSERT_ID()` | `RETURNING` — get the inserted row in one statement (10.5+) |
| `AUTO_INCREMENT` for sequence-like needs | `CREATE SEQUENCE` — first-class sequence objects with full control |
| IP addresses stored as `VARCHAR` | `INet4` / `INet6` — native IP types with comparison and indexing |
| Dropping or reordering columns with full table rebuild | `INSTANT` algorithm for `ALTER TABLE` — no rebuild needed (10.4+) |
| Oracle migration assumed to require full rewrite | `sql_mode=ORACLE` — PL/SQL, packages, Oracle-compatible NULL handling |
| Asking what changed in a row over time | System-versioned tables with `FOR SYSTEM_TIME AS OF` |
| Analytics queries on OLTP tables | ColumnStore engine — columnar storage for analytical workloads |
| Links or references to `mariadb.com/kb/en/` | The Knowledge Base no longer exists — all documentation is now at [mariadb.com/docs](https://mariadb.com/docs) |

## Defaults Changed in 11.5–11.8 LTS

The current LTS (11.8) flipped several long-standing defaults. New installations behave differently from older ones — relevant when migrating or comparing behavior:

- **Default character set: `latin1` → `utf8mb4`** (11.6+, MDEV-19123) — new tables use `utf8mb4` unless overridden. Replication to MariaDB 10.6 or older replicas needs care (older replicas may not understand all `utf8mb4` collations).
- **Default Unicode collation: `uca1400_ai_ci`** (11.5+, MDEV-25829) — modern Unicode collation with proper SMP (supplementary multilingual plane) support including emoji. Replaces the older `utf8mb4_general_ci` default.
- **`alter_algorithm` deprecated and ignored** (11.5+, MDEV-33655) — specify `ALGORITHM=INSTANT|INPLACE|COPY` on the statement itself instead.
- **TIMESTAMP range extended** (11.5+ 64-bit, MDEV-32188) — upper bound raised from `2038-01-19 03:14:07 UTC` to `2106-02-07 06:28:15 UTC`. Storage format unchanged; old servers can still read values within the old range.
- **`innodb_snapshot_isolation` default ON** — see next section.

## Behavior Change: innodb_snapshot_isolation (11.8+)

From MariaDB 11.8 LTS, `innodb_snapshot_isolation` defaults to **ON** (previously OFF, MDEV-35124). This tightens REPEATABLE READ behavior to match true snapshot isolation — transactions see a consistent snapshot from their start and writes detect conflicts more strictly.

**What can change for existing code:**
- Read-modify-write patterns that previously worked silently may now hit conflicts and error out — fail-fast is the intended behavior
- Long-running `REPEATABLE READ` transactions are more likely to see write conflicts at commit time

If existing code depends on the older permissive behavior, opt back in explicitly:
```sql
SET GLOBAL innodb_snapshot_isolation = OFF;  -- restore pre-11.8 behavior
```

The new default is the correct semantics — review code that relies on the looser behavior rather than disabling it long-term.

## System-Versioned Tables

Available since MariaDB 10.3. Track the full history of every row automatically, without triggers or audit tables.

```sql
CREATE TABLE prices (
    product VARCHAR(100),
    price DECIMAL(10,2)
) WITH SYSTEM VERSIONING;

-- Query data as it was at a point in time:
SELECT * FROM prices FOR SYSTEM_TIME AS OF '2025-01-01 00:00:00';

-- See all historical versions of a row:
SELECT * FROM prices FOR SYSTEM_TIME ALL WHERE product = 'widget';
```

Use this instead of manually maintained `valid_from` / `valid_to` columns or separate audit tables.

## RETURNING Clause

Get inserted, updated, or deleted rows back without a second query.

```sql
-- Get the generated ID after insert:
INSERT INTO orders (product, qty) VALUES ('widget', 5)
    RETURNING id, created_at;

-- Get deleted rows for logging:
DELETE FROM queue WHERE processed = 1
    RETURNING id, payload;
```

`INSERT ... RETURNING` and `DELETE ... RETURNING` available since MariaDB 10.5. `UPDATE ... RETURNING` available since MariaDB 13.0.

## Sequences

Available since MariaDB 10.3. First-class sequence objects — more flexible than `AUTO_INCREMENT`.

```sql
CREATE SEQUENCE order_seq START WITH 1000 INCREMENT BY 1;

-- Use in INSERT:
INSERT INTO orders (id, product) VALUES (NEXT VALUE FOR order_seq, 'widget');

-- Peek at current value without incrementing:
SELECT LASTVAL(order_seq);
```

Sequences support gaps, multiple sequences per table, and descending sequences. Unlike `AUTO_INCREMENT`, they are not tied to a specific column or table.

## Instant ALTER TABLE

Drop or modify columns without rebuilding the table — no downtime on large tables.

```sql
ALTER TABLE large_table DROP COLUMN old_column, ALGORITHM=INSTANT;
ALTER TABLE large_table MODIFY COLUMN name VARCHAR(200), ALGORITHM=INSTANT;
```

Available since MariaDB 10.4. Use `ALGORITHM=INSTANT` explicitly; fall back to `INPLACE` or `COPY` if the operation doesn't qualify.

## INet4 and INet6 Data Types

Available since MariaDB 10.5. Native IP address storage with correct comparison and indexing.

```sql
CREATE TABLE connections (
    client_ip INet6 NOT NULL,
    connected_at DATETIME NOT NULL,
    INDEX (client_ip)
);

INSERT INTO connections VALUES (INet6('192.168.1.1'), NOW());
INSERT INTO connections VALUES (INet6('::1'), NOW());

-- Range queries work correctly:
SELECT * FROM connections WHERE client_ip BETWEEN INet6('10.0.0.0') AND INet6('10.255.255.255');
```

`INet4` stores IPv4 (4 bytes), `INet6` stores both IPv4 and IPv6 (16 bytes).

## Oracle Compatibility Mode

Available since MariaDB 10.3. `sql_mode=ORACLE` enables PL/SQL syntax, Oracle-compatible NULL handling, packages, and Oracle-style functions — useful when migrating from Oracle or supporting Oracle-experienced developers.

```sql
SET sql_mode=ORACLE;

-- Oracle-style stored procedures, packages, and NULL semantics work here
-- ROWNUM, SYSDATE, NVL(), DECODE() available
-- Note: EMPTY_STRING_IS_NULL is NOT included — add it separately if needed: SET sql_mode='ORACLE,EMPTY_STRING_IS_NULL'
```

Not a complete Oracle replacement, but significantly reduces migration friction.

## FLASHBACK

Available since MariaDB 10.2. Roll back tables to a previous state using the binary log — without restoring a full backup. Flashback is implemented via the `mariadb-binlog` utility, not a SQL statement:

```bash
# Generate reverse SQL from the binary log and pipe it back to MariaDB:
mariadb-binlog --flashback --start-datetime="2026-05-18 10:00:00" /var/log/mysql/mysql-bin.000001 | mariadb
```

Requires binary logging enabled (`log_bin`). Useful for recovering from accidental deletes or bad migrations.

## More MariaDB Features

### SQL & Schema
- **Invisible columns** (10.3+) — hidden from `SELECT *`, still writable; useful for schema evolution without breaking existing queries
- **`DEFAULT` expressions on BLOB/TEXT** — not supported in MySQL
- **`DECIMAL` precision to 38 digits** — MySQL stops at 30
- **`INTERSECT` and `EXCEPT`** (10.3+) — set operators not available in MySQL
- **`LIMIT` in subqueries** — supported; MySQL restricts this
- **`SELECT ... OFFSET ... FETCH`** — SQL standard syntax for pagination
- **Dynamic columns** (5.3+) — schema-less key/value storage inside a single column
- **`SFORMAT()`** — string formatting function
- **`UUID_v4()` and `UUID_v7()` functions** (11.7+) — generate version-4 random or version-7 time-ordered UUIDs; the v7 form is sortable and ideal for primary keys
- **`FORMAT_BYTES()`** (11.8+) — convert a byte count to a human-readable string (e.g. `1234567` → `1.18 MiB`)
- **Single-table `DELETE` with table aliases** (11.6+) — `DELETE t FROM mytable t WHERE ...` syntax now works without rewriting
- **`REPAIR TABLE ... FORCE`** (11.5+) — force-repair even when the table appears clean
- **Stored routine parameter default values** (11.8+, MDEV-10862) — `PROCEDURE p(a INT DEFAULT 0, b INT DEFAULT 0)` — call with fewer arguments
- **`ROW` data type as stored function return value** (11.7+, MDEV-12252) — return structured rows from stored functions
- **Update triggers with column list** (11.8+, MDEV-34551) — `CREATE TRIGGER ... BEFORE UPDATE OF col1, col2 ON t` — fire only when those columns are updated
- **Atomic `CREATE OR REPLACE TABLE`** (13.0+) — the statement is fully atomic: either the new table replaces the old one or nothing happens, with no risk of leaving the schema in a half-replaced state. MySQL has no equivalent atomic guarantee.
- **`UPDATE` / `DELETE` reading from a CTE** (12.3+) — `WITH ... UPDATE/DELETE` using values from a common table expression
- **`IS JSON` predicate** (12.3+) — SQL-standard test for whether a value is valid JSON: `WHERE col IS JSON`
- **JSON depth limit removed** (12.2+) — the 32-level nesting limit on JSON functions is gone; deeply nested JSON now works without rewrites
- **Basic XML data type** (12.3+) — first-class `XML` type for storing and validating XML documents
- **Triggers fired on multiple events** (12.0+) — one trigger body for `INSERT OR UPDATE OR DELETE`, instead of three separate triggers
- **Foreign key names per table** (12.1+) — FK names need to be unique only per table, not per database (MySQL-compatible behavior)
- **Stored procedures and functions** — MariaDB uses SQL/PSM syntax (`DECLARE`, `HANDLER`, `CURSOR`, `BEGIN...END`); AI agents often generate incorrect syntax — see [Stored Procedures — MariaDB Docs](https://mariadb.com/docs/server/server-usage/stored-routines/stored-procedures)

### Storage Engines
- **ColumnStore** — columnar engine for analytical/data warehouse workloads
- **Aria** — crash-safe MyISAM replacement, used internally for temp tables
- **MyRocks** (10.2+) — RocksDB-based, optimized for write-heavy workloads with compression
- **CONNECT** — query external data sources (CSV, JDBC, ODBC, MongoDB) as SQL tables
- **Spider** — sharding across multiple MariaDB instances

### Security & Auth
- **`unix_socket` authentication** — authenticate OS users without passwords; `authentication_string` support added in 11.6+ for finer-grained mapping
- **ED25519 plugin** — modern authentication alternative to SHA1-based plugins
- **PARSEC plugin** (11.6+, MDEV-32618) — Password Authentication using Response Signed with Elliptic Curve; salt and per-installation key separation make stolen hashes unusable elsewhere
- **Role-based access control** (10.0+) — roles available before MySQL added them
- **SSL enabled by default** — no configuration required
- **Table-level encryption** — encrypt individual tables, not just the whole datadir
- **HashiCorp Vault integration** — key management plugin
- **`SET SESSION AUTHORIZATION`** (12.0+) — perform actions as another user within a session (useful for impersonation in administrative scripts and apps that need least-privilege execution)
- **Passphrase-protected TLS keys** (12.0+) — `ssl_passphrase` system variable lets the server load encrypted private keys

### Replication & HA
- **Galera Cluster** — built-in synchronous multi-master clustering
- **Multi-source replication** — replicate from multiple primaries simultaneously
- **Parallel replication** — faster replica apply
- **Lag-free `ALTER TABLE` replication** — schema changes don't stall replicas

### Connectors
- **LGPL-licensed connectors** for C, C++, Java, Python, Node.js, ODBC, R2DBC — permissive licensing for commercial applications; MySQL connectors are GPL

### Developer Tools
- **`EXPLAIN` in slow query log** — automatic execution plan logging for slow queries
- **Progress reporting** for `ALTER TABLE` and `CHECK TABLE`
- **`mariadb-backup`** — hot backup with backup locks (no `FLUSH TABLES WITH READ LOCK`)
- **Non-blocking client API** — async queries without threads
- **Deprecation visibility** (13.0+) — `INFORMATION_SCHEMA.SYSTEM_VARIABLES` includes a deprecated flag, so you can detect uses of variables that will be removed in future versions before they break:
  ```sql
  SELECT variable_name, default_value FROM INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE is_deprecated = 'YES';
  ```
- **Engine-specific create options visible in `INFORMATION_SCHEMA`** (13.0+) — `STATISTICS` and `COLUMNS` now expose engine-specific options, useful when inspecting how indexes or columns were configured

## Sources

- [Monty Widenius: Celebrating 15 years of MariaDB](http://monty-says.blogspot.com/2024/10/celebrating-15-years-of-mariadb.html)
- [MariaDB vs MySQL Features — MariaDB Docs](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/mariadb-vs-mysql-features)
- [System-Versioned Tables — MariaDB KB](https://mariadb.com/docs/server/reference/sql-structure/temporal-tables/system-versioned-tables)
- [RETURNING — MariaDB KB](https://mariadb.com/docs/server/reference/sql-statements/data-manipulation/inserting-loading-data/insertreturning)
- [CREATE SEQUENCE — MariaDB KB](https://mariadb.com/docs/server/reference/sql-structure/sequences/create-sequence)
- [Instant ALTER TABLE — MariaDB KB](https://mariadb.com/docs/server/server-usage/storage-engines/innodb/innodb-online-ddl/innodb-online-ddl-operations-with-the-instant-alter-algorithm)
- [INet6 Data Type — MariaDB KB](https://mariadb.com/docs/server/reference/data-types/string-data-types/inet6)
- [Oracle Compatibility — MariaDB KB](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/sql_modeoracle)
- [FLASHBACK — MariaDB KB](https://mariadb.com/docs/server/server-management/server-monitoring-logs/binary-log/flashback)

*For topics not covered here, see the official MariaDB documentation at [mariadb.com/docs](https://mariadb.com/docs).*
