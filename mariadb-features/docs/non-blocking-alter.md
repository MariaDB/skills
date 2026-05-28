## Non-Blocking ALTER TABLE (Instant + Online by Default)

MariaDB's `ALTER TABLE` works on a tiered model:

- **`ALGORITHM=INSTANT`** (10.4+) — metadata-only changes (drop column, modify default, change column order, etc.) complete in microseconds without a table rebuild.
- **`ALGORITHM=COPY, LOCK=NONE` as the default for non-instant operations** (11.2+, MDEV-16329) — even when a rebuild is needed, MariaDB now runs it non-blocking by default: concurrent DML on the table proceeds while the copy is happening, with only a brief lock at the swap. The need for external tools like `pt-online-schema-change` is largely gone for routine `ALTER`s.
- **Optimistic two-phase replication of large `ALTER TABLE`** (11.4+, `binlog_alter_two_phase=1`, off by default) — see the `mariadb-replication-and-ha` skill.

```sql
ALTER TABLE large_table DROP COLUMN old_column, ALGORITHM=INSTANT;
ALTER TABLE large_table MODIFY COLUMN name VARCHAR(200), ALGORITHM=INSTANT;
-- Non-instant change runs non-blocking by default on 11.2+:
ALTER TABLE large_table ADD INDEX (created_at);
```

Use `ALGORITHM=INSTANT` explicitly when you need to guarantee a metadata-only change; the operation will fail rather than silently fall back to a rebuild.
