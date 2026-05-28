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
