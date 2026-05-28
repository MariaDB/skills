# Oracle to MariaDB: Overview

## The First Step: Enable Oracle Compatibility Mode

```sql
SET sql_mode = 'ORACLE';
-- or permanently in configuration:
-- sql_mode = ORACLE
```

Without this, PL/SQL syntax, Oracle data type synonyms, and Oracle-style functions will not work. This is the single most important step and the most commonly missed.

## What LLMs Get Wrong

| What you might see | What's correct |
|---|---|
| PL/SQL that fails with syntax errors | Set `sql_mode=ORACLE` first — without it, PL/SQL constructs are not recognized |
| Oracle `DATE` mapped to MariaDB `DATE` | Oracle `DATE` stores date AND time — map to `DATETIME`, not `DATE` |
| Assuming 100% PL/SQL compatibility | ~80% works without changes; `SYNONYM`, `INSERT ALL`, and `CONNECT BY` usually require rewrites — `(+)` joins work in Oracle mode on 12.1+ (see next row) |
| `SYNONYM` usage in schema or code | No equivalent in MariaDB — replace with views or direct object references |
| `(+)` outer join notation | Supported in Oracle mode since MariaDB 12.1 (MDEV-13817) — on older versions rewrite as `LEFT JOIN` / `RIGHT JOIN` |
| `START WITH ... CONNECT BY` | Not supported — rewrite as recursive CTE using `WITH RECURSIVE` |
| `TIMESTAMP WITH TIME ZONE` | Loses timezone on migration — becomes `DATETIME`; handle timezone in application |

## NULL Handling

Oracle treats empty strings as `NULL`. `sql_mode=ORACLE` does **not** activate this automatically — `EMPTY_STRING_IS_NULL` must be added separately:

```sql
SET sql_mode = 'ORACLE,EMPTY_STRING_IS_NULL';
```

Without `EMPTY_STRING_IS_NULL`, `''` is not `NULL` in MariaDB even in Oracle mode.

## Migration Tools

- **[MariaDB Migration Assessment Tool](https://mariadb.com/resources/blog/mariadb-migration-assessment-tool-how-ready-are-you-to-migrate-to-mariadb-from-oracle/)** — analyzes Oracle DDL export and scores compatibility before you start
- **Connect Storage Engine** — create MariaDB tables that read live Oracle data via ODBC; useful for phased migration without full cutover
- **DBeaver** — schema mapping, data comparison, and transfer between Oracle and MariaDB
- **MaxScale query rewriting** — intercept and rewrite unsupported Oracle syntax on-the-fly for cases that can't be changed in the application
