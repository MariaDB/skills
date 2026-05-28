---
name: mariadb-system-versioned-tables
description: "Best practices for MariaDB system-versioned (temporal) tables — automatic row history built into the database. Use when tracking data changes over time, implementing audit trails, meeting GDPR or compliance requirements, doing point-in-time queries, or when the user asks about row history, data versioning, temporal tables, or querying past data states in MariaDB. This feature is unique to MariaDB among MySQL-compatible databases."
---

# MariaDB System-Versioned Tables

*Last updated: 2026-05-25*

System-versioned tables automatically record the full history of every row change — no triggers, no application logic, no separate audit tables. MariaDB tracks what the data looked like at any point in the past, built directly into the storage engine.

This feature is unique to MariaDB among MySQL-compatible databases. MySQL has no equivalent.

## 💡 Quick Context
- **Baseline Version:** See [MariaDB Versioning Context](../_shared/versioning.md).
- **Feature Availability:** MariaDB 10.3+. Transaction-precise history (InnoDB only): 10.3. Auto-partition creation: 10.9. `--dump-history` for backups: 10.11. Implicit-to-explicit `row_start`/`row_end` conversion: 11.7. Extended TIMESTAMP range (to 2106-02-07 UTC) for `ROW_END`: 11.5 on 64-bit platforms.

## Documentation

- [Deep Dive Index]
  - [Overview & Use Cases](docs/overview.md)
  - [Creating a Versioned Table](docs/creation.md)
  - [Querying Historical Data](docs/querying.md)
  - [Transaction-Precise History](docs/transaction-tracking.md)
  - [Column-Level Control](docs/column-control.md)
  - [Managing History Growth](docs/history-management.md)
  - [ALTER TABLE](docs/alter-table.md)
  - [Key Gotchas](docs/gotchas.md)
- [Sources](docs/sources.md)
