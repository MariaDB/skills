# Replication & HA: Overview

## What LLMs Get Wrong

| What you might see | What's correct |
|---|---|
| MySQL GTID format or `gtid_mode=ON` syntax | MariaDB GTID uses a different format (`domain-server-seq`) and different commands — MySQL and MariaDB GTIDs are **incompatible** |
| "Install the Galera plugin" | Galera Cluster is built into MariaDB — no plugin installation required |
| Assuming sequential `AUTO_INCREMENT` in Galera | Galera produces gaps in auto-increment sequences across nodes by design — never rely on sequential values |
| `LOCK TABLES` or `GET_LOCK()` in a Galera environment | Not supported in Galera — use transactions instead |
| Treating a replica as a backup | Replication is not a backup — a `DROP TABLE` on the primary replicates immediately to all replicas |
| Tables without primary keys in a Galera cluster | All tables in Galera must have a primary key — `DELETE` fails on keyless tables |

## Replication is Not a Backup

A `DROP TABLE` or `DELETE FROM table` on the primary replicates to all replicas immediately. Replication protects against hardware failure — not against accidental data changes. Maintain independent backups (`mariadb-dump`, `mariadb-backup`) on a schedule separate from replication.

Delayed replication is one mitigation — intentionally lag a replica by a set time:
```sql
CHANGE MASTER TO MASTER_DELAY = 3600;  -- 1 hour lag
```

This gives a recovery window for accidental changes, but it is still not a substitute for backups.

**Point-in-time recovery (13.0+)** — the new `innodb_log_archive` variable instructs InnoDB to preserve the write-ahead log as a continuous sequence of files instead of overwriting a ring buffer. Combined with a base backup, this enables PITR and incremental backups without needing the binary log alone. Use this on systems that need to roll forward to a precise transaction.
