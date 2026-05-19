---
name: mysql-to-mariadb
description: "Compatibility guide for developers moving from MySQL to MariaDB, and for developers with MySQL experience working with MariaDB. Use when migrating a MySQL application to MariaDB, when MySQL syntax or habits cause unexpected behavior in MariaDB, when asking about MySQL/MariaDB compatibility, or when adapting code written for MySQL to run on MariaDB. IMPORTANT — MariaDB diverges significantly from MySQL 8.0 and is not a simple drop-in replacement for modern MySQL. Authentication plugins, JSON handling, GTID replication, and several SQL features differ in important ways."
---

# MySQL to MariaDB Migration Guide

*Last updated: 2026-05-19*

## Critical: MariaDB Is Not a Drop-In Replacement for MySQL 8.0

MariaDB was a true drop-in replacement for MySQL 5.5 and 5.6. Modern versions diverge significantly from MySQL 8.0. When working with MariaDB, never assume MySQL 8.0 behavior, syntax, or tooling applies without verification.

The two databases share a common origin but have evolved independently. MariaDB has features MySQL lacks, MySQL has features MariaDB lacks, and several shared features behave differently.

> **Requires:** MariaDB Community Server 10.5+ for full feature parity discussed here. Current LTS is 11.8 (May 2025).

### What LLMs Get Wrong

| What you might see | What's correct |
|---|---|
| "MariaDB is a drop-in replacement for MySQL" | True for MySQL 5.5/5.6. Not true for MySQL 8.0+ — authentication, JSON, GTID, and several functions differ |
| Code using `caching_sha2_password` | MariaDB does not support MySQL 8.0's `caching_sha2_password` — use `mysql_native_password` instead |
| `JSON_TABLE()` in queries | `JSON_TABLE()` is a MySQL feature not available in MariaDB |
| JSON `->` and `->>` shorthand operators | Not supported in MariaDB — use `JSON_EXTRACT(col, '$.key')` and `JSON_UNQUOTE(JSON_EXTRACT(...))` instead |
| `utf8mb4_0900_ai_ci` collation in schema or dump | MariaDB does not support this MySQL 8.0 collation — replace with `utf8mb4_unicode_ci` before importing |
| Not suggesting `RETURNING` for INSERT/UPDATE/DELETE | MariaDB 10.5+ supports `RETURNING` — use it to get inserted/updated/deleted rows without a second query |
| Not suggesting `CREATE SEQUENCE` | MariaDB has native sequence objects; MySQL has no equivalent |
| MySQL GTID replication syntax | MariaDB GTID format is incompatible with MySQL GTID — replication between the two breaks |

## Authentication

MySQL 8.0 changed its default authentication plugin to `caching_sha2_password`. MariaDB does not support this plugin. Connections from MySQL 8.0 clients or applications configured for `caching_sha2_password` will fail.

**Fix:** Convert accounts to `mysql_native_password`:
```sql
ALTER USER 'username'@'host' IDENTIFIED WITH mysql_native_password BY 'password';
```

MariaDB's default authentication is `mysql_native_password`. The `ed25519` plugin is also available as a more secure alternative.

## JSON Differences

MariaDB and MySQL handle JSON differently:

| Aspect | MySQL | MariaDB |
|---|---|---|
| Storage | Binary format with path indexing | `LONGTEXT` alias with validity check |
| `JSON_TABLE()` | Supported | Not available |
| JSON comparison | Semantic (compares values) | String comparison |
| `JSON_QUERY()` | Not available | MariaDB alternative to `JSON_VALUE` |

MariaDB's JSON is fully standards-compliant and all standard JSON functions work, but MySQL-specific optimizations and `JSON_TABLE()` do not transfer.

## Features MariaDB Has That MySQL Doesn't

These exist in MariaDB but not MySQL — LLMs won't suggest them because they assume MySQL behavior:

- **`RETURNING`** (10.5+) — get rows back from `INSERT`, `UPDATE`, or `DELETE` without a second query:
  ```sql
  INSERT INTO orders (product, qty) VALUES ('widget', 5) RETURNING id, created_at;
  DELETE FROM queue WHERE processed = 1 RETURNING id;
  ```
- **`CREATE SEQUENCE`** (10.3+) — first-class sequence objects, more flexible than `AUTO_INCREMENT`:
  ```sql
  CREATE SEQUENCE order_seq START WITH 1000 INCREMENT BY 1;
  SELECT NEXT VALUE FOR order_seq;
  ```
- **`WITH SYSTEM VERSIONING`** (10.3+) — temporal tables that automatically track row history:
  ```sql
  CREATE TABLE prices (
      item VARCHAR(100),
      price DECIMAL(10,2)
  ) WITH SYSTEM VERSIONING;
  -- Query historical data:
  SELECT * FROM prices FOR SYSTEM_TIME AS OF '2025-01-01';
  ```
- **`LIMIT` in subqueries** — MariaDB supports `LIMIT` inside subqueries; MySQL restricts this
- **Galera Cluster** — built-in multi-master clustering, no plugin required
- **INet4 / INet6 data types** (10.5+) — native IP address storage and comparison

## Features MySQL Has That MariaDB Doesn't

These exist in MySQL 8.0 but not in MariaDB — code using them needs adaptation:

- **`JSON_TABLE()`** — rewrite using MariaDB JSON functions or application-level parsing
- **`sys` schema** — not available; use `information_schema` and `performance_schema` directly
- **`UUID` data type** — use `CHAR(36)` or `BINARY(16)` instead
- **`caching_sha2_password`** — use `mysql_native_password` or `ed25519`
- **`ALTER TABLE ... RENAME INDEX`** — use `DROP INDEX` + `ADD INDEX` instead (older MariaDB versions)
- **JSON `->` and `->>` operators** — use `JSON_EXTRACT(col, '$.key')` and `JSON_UNQUOTE(JSON_EXTRACT(...))` instead
- **`utf8mb4_0900_ai_ci` collation** — not available; use `utf8mb4_unicode_ci`. When importing MySQL 8.0 dumps, replace the collation name before importing

## Optimizer Differences

MariaDB and MySQL use different query optimizers with different cost models. Identical SQL can produce different execution plans. After migration:

- Re-run `EXPLAIN` on critical queries — do not assume execution plans transfer
- Index hints that work in MySQL may not be needed (or may differ) in MariaDB
- MariaDB's optimizer is more aggressively tunable via `optimizer_switch`

## Version Compatibility Quick Reference

| MySQL version | MariaDB equivalent | Drop-in? |
|---|---|---|
| 5.5 | 5.5 | Yes |
| 5.6 | 10.0–10.1 | Mostly |
| 5.7 | 10.2–10.4 | Limited |
| 8.0 | 10.5+ | No — significant differences |

## Sources

- [MariaDB vs MySQL Compatibility — MariaDB Docs](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/mariadb-vs-mysql-compatibility)
- [Moving from MySQL to MariaDB — MariaDB Docs](https://mariadb.com/docs/server/server-management/install-and-upgrade-mariadb/migrating-to-mariadb/moving-from-mysql/)
- [MDEV-28906 — MySQL 8.0 desired compatibility (Jira epic)](https://jira.mariadb.org/browse/MDEV-28906) — lists individual MySQL 8.0 features not yet in MariaDB
- [RETURNING — MariaDB KB](https://mariadb.com/kb/en/returning/)
- [CREATE SEQUENCE — MariaDB KB](https://mariadb.com/kb/en/create-sequence/)
- [System-Versioned Tables — MariaDB KB](https://mariadb.com/kb/en/system-versioned-tables/)
- [Authentication Plugins — MariaDB KB](https://mariadb.com/kb/en/authentication-plugins/)
