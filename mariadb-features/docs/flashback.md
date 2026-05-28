## FLASHBACK

Available since MariaDB 10.2. Roll back tables to a previous state using the binary log — without restoring a full backup. Flashback is implemented via the `mariadb-binlog` utility, not a SQL statement:

```bash
# Generate reverse SQL from the binary log and pipe it back to MariaDB:
mariadb-binlog --flashback --start-datetime="2026-05-18 10:00:00" /var/log/mysql/mysql-bin.000001 | mariadb
```

Requires binary logging enabled (`log_bin`). Useful for recovering from accidental deletes or bad migrations.
