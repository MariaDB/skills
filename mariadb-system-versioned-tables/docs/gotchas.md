# Key Gotchas

- **`TRUNCATE` is prohibited** on versioned tables (error 4137). Use `DELETE HISTORY` or partition management instead.
- **Replication**: `ROW_END` is implicitly added to the Primary Key. On replicas, this can cause duplicate key errors during log replay. Fix: set `secure_timestamp = YES` on the replica.
- **Backups**: `mysqldump` / `mariadb-dump` skips historical rows by default. Use `--dump-history` (10.11+) to include them. On restore, set `system_versioning_insert_history=ON` (10.11+, MDEV-16546) so the loader is allowed to write directly into `ROW_START`/`ROW_END`; combine with `secure_timestamp` set to allow session timestamp changes.
- **Table growth**: without partitioning, history rows accumulate indefinitely. Plan partitioning from the start for high-update tables.
- **DELETE HISTORY with future timestamps**: using `BEFORE SYSTEM_TIME` with a timestamp beyond `ROW_END` max can accidentally delete active rows (MDEV-25468). The `ROW_END` max is `2038-01-19` on 32-bit platforms and on MariaDB before 11.5; on MariaDB 11.5+ on 64-bit it's extended to `2106-02-07 06:28:15 UTC` (MDEV-32188). Stay well below your platform's max.
- **`SYSTEM` as a column name**: causes parser errors in `ALTER TABLE` statements. Use backticks: `` `SYSTEM` ``.
