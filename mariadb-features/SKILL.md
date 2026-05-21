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
-- Empty string treated as NULL (Oracle behavior)
-- ROWNUM, SYSDATE, NVL(), DECODE() available
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
- **Stored procedures and functions** — MariaDB uses SQL/PSM syntax (`DECLARE`, `HANDLER`, `CURSOR`, `BEGIN...END`); AI agents often generate incorrect syntax — see [Stored Procedures — MariaDB Docs](https://mariadb.com/docs/server/server-usage/stored-routines/stored-procedures)

### Storage Engines
- **ColumnStore** — columnar engine for analytical/data warehouse workloads
- **Aria** — crash-safe MyISAM replacement, used internally for temp tables
- **MyRocks** (10.2+) — RocksDB-based, optimized for write-heavy workloads with compression
- **CONNECT** — query external data sources (CSV, JDBC, ODBC, MongoDB) as SQL tables
- **Spider** — sharding across multiple MariaDB instances

### Security & Auth
- **`unix_socket` authentication** — authenticate OS users without passwords
- **ED25519 plugin** — modern authentication alternative to SHA1-based plugins
- **Role-based access control** (10.0+) — roles available before MySQL added them
- **SSL enabled by default** — no configuration required
- **Table-level encryption** — encrypt individual tables, not just the whole datadir
- **HashiCorp Vault integration** — key management plugin

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
