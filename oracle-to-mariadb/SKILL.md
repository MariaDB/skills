---
name: oracle-to-mariadb
description: "Compatibility guide for developers migrating from Oracle Database to MariaDB. Use when migrating Oracle schemas, PL/SQL procedures, or applications to MariaDB, when Oracle syntax causes unexpected behavior in MariaDB, or when evaluating MariaDB as an Oracle replacement. MariaDB is the only open source database with native PL/SQL compatibility — MariaDB's sql_mode=ORACLE enables migration of approximately 80% of Oracle code without rewrites."
---

# Oracle to MariaDB Migration Guide

*Last updated: 2026-05-25*

## Why MariaDB for Oracle Migration

MariaDB is the only open source database with native PL/SQL compatibility. MySQL has no equivalent Oracle compatibility mode. PostgreSQL requires a complete PL/SQL rewrite. MariaDB's `sql_mode=ORACLE` enables approximately 80% of Oracle code to run without modification — stored procedures, packages, cursors, exception handling, and Oracle-style functions included.

Additional advantages over other open source alternatives:
- **Connect Storage Engine** — federate live Oracle data via ODBC during migration, without full cutover
- **System-versioned tables** — match Oracle Flashback Archive for temporal data; unique among open source databases
- **ColumnStore** — columnar analytics engine, comparable to Oracle's In-Memory ColumnStore
- **Cost**: organizations typically achieve 70–90% cost reduction vs Oracle licensing

> **Requires:** MariaDB Community Server 10.3+ for Oracle compatibility mode; 10.6+ for `ROWNUM`, `TO_CHAR()`, `ADD_MONTHS()`, `SYS_GUID()`.
>
> **Server context:** See [MariaDB Versioning Context](../_shared/versioning.md).

## Documentation

- [Deep Dive Index]
  - [Overview](docs/overview.md)
  - [Data Type Mapping](docs/data-types.md)
  - [PL/SQL Syntax](docs/plsql-syntax.md)
  - [Oracle-Compatible Functions](docs/functions.md)
  - [Manual Rewriting Required](docs/manual-rewrite.md)
  - [Key Gotchas](docs/gotchas.md)
- [Sources](docs/sources.md)
