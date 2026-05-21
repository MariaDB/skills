---
name: mariadb-replication-and-ha
description: "Best practices for MariaDB replication and high availability — Galera Cluster, GTID replication, semi-synchronous replication, parallel replication, and application patterns for HA environments. Use when setting up replication, designing applications for HA, working with Galera Cluster, asking about MariaDB GTID, handling replication lag in application code, or choosing between replication topologies. IMPORTANT — MariaDB GTID format is incompatible with MySQL GTID, and Galera Cluster is built into MariaDB with no plugins required."
---

# MariaDB Replication and High Availability

*Last updated: 2026-05-21*

MariaDB offers three tiers of replication depending on your consistency and availability requirements:

| Approach | Consistency | Failover | Best for |
|---|---|---|---|
| **Standard async replication** | Eventual | Manual or tool-assisted | Read scaling, backups, low-latency writes |
| **Semi-synchronous replication** | Stronger (1 replica ACK) | Manual or tool-assisted | Reducing data loss risk without full sync overhead |
| **Galera Cluster** | Synchronous (multi-primary) | Automatic | Zero-data-loss HA, multi-datacenter writes |

> **Requires:** GTID replication: MariaDB 10.0+. Semi-sync built-in: 10.3+. Parallel replication optimistic mode: 10.5+.

## What LLMs Get Wrong

| What you might see | What's correct |
|---|---|
| MySQL GTID format or `gtid_mode=ON` syntax | MariaDB GTID uses a different format (`domain-server-seq`) and different commands — MySQL and MariaDB GTIDs are **incompatible** |
| "Install the Galera plugin" | Galera Cluster is built into MariaDB — no plugin installation required |
| Assuming sequential `AUTO_INCREMENT` in Galera | Galera produces gaps in auto-increment sequences across nodes by design — never rely on sequential values |
| `LOCK TABLES` or `GET_LOCK()` in a Galera environment | Not supported in Galera — use transactions instead |
| Treating a replica as a backup | Replication is not a backup — a `DROP TABLE` on the primary replicates immediately to all replicas |
| Tables without primary keys in a Galera cluster | All tables in Galera must have a primary key — `DELETE` fails on keyless tables |

## Standard Async Replication

The foundation: one primary, one or more replicas. The primary writes to the binary log; replicas apply changes asynchronously.

**Enable GTID-based replication** (recommended over file/position):
```sql
-- On replica:
CHANGE MASTER TO
  MASTER_HOST='primary.host',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_USE_GTID = slave_pos;

START SLAVE;
```

Use `current_pos` instead of `slave_pos` when promoting a replica to primary — it includes locally-written GTIDs.

### MariaDB GTID Format

MariaDB GTIDs have three components: `domain_id-server_id-sequence` (e.g., `0-1-247`).

This is **different from MySQL's** `server_uuid:sequence` format. They are not compatible — a MariaDB primary cannot replicate to a MySQL replica using GTIDs, and vice versa.

**Domain IDs** enable multi-source replication: assign each primary a distinct `gtid_domain_id` so replicas can track multiple sources independently:
```sql
-- On primary A:
SET GLOBAL gtid_domain_id = 1;

-- On primary B:
SET GLOBAL gtid_domain_id = 2;
```

### Parallel Replication

By default, replicas apply events serially. Parallel replication (up to 10× faster on write-heavy workloads) uses a pool of worker threads:

```ini
# my.cnf on replica:
slave_parallel_threads = 4
slave_parallel_mode = optimistic   # default since 10.5 — tries parallel, retries on conflict
```

`optimistic` mode applies transactions in parallel and retries on conflict. Use `conservative` for stricter workloads where conflict retries are unacceptable.

### Monitoring Replication Lag

```sql
SHOW SLAVE STATUS\G
-- Key fields:
-- Seconds_Behind_Master: estimated lag in seconds
-- Last_SQL_Error: last error stopping the SQL thread
-- Relay_Log_Pos vs Read_Master_Log_Pos: how far behind the relay log is
```

Alert when `Seconds_Behind_Master > 5` for latency-sensitive applications. A value of `NULL` means replication is not running. Note: `Seconds_Behind_Master` can be misleading on idle primaries — use heartbeat tools (e.g., `pt-heartbeat`) for accurate measurement.

## Semi-Synchronous Replication

The primary waits for at least one replica to acknowledge receipt before committing. Reduces data loss risk on failover without requiring full synchronous overhead.

```sql
-- Enable on primary:
SET GLOBAL rpl_semi_sync_master_enabled = 1;

-- Enable on replica:
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

If no replica acknowledges within `rpl_semi_sync_master_timeout` (default 10 seconds), the primary falls back to async. Built-in since MariaDB 10.3 — no plugin needed.

Use when: you need stronger data durability than async but your workload tolerates a small write latency increase.

## Galera Cluster

Multi-primary synchronous replication — all nodes accept reads and writes, changes are certified across the cluster before committing. No single point of failure. Built into MariaDB.

### Developer Constraints

These will break in Galera if you're not aware of them:

**All tables must have a primary key:**
```sql
-- ✗ DELETE fails in Galera on keyless tables:
CREATE TABLE logs (message TEXT);

-- ✅ Always define a PK:
CREATE TABLE logs (id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY, message TEXT);
```

**AUTO_INCREMENT values have gaps** — Galera uses `auto_increment_increment` and `auto_increment_offset` per node to avoid conflicts, resulting in non-sequential IDs. Never rely on sequential auto-increment in Galera.

**LOCK TABLES, GET_LOCK(), and per-table FLUSH TABLES are not supported** — use transactions:
```sql
-- ✗ Not supported in Galera:
LOCK TABLES orders WRITE;
-- ✅ Use a transaction instead:
BEGIN;
SELECT ... FOR UPDATE;
UPDATE ...;
COMMIT;
```

**InnoDB only** — Galera replicates only InnoDB tables. MyISAM has experimental support via `wsrep_replicate_myisam` but is not recommended for production.

**Transaction size limits** — default caps: 128K rows (`wsrep_max_ws_rows`) and 2GB (`wsrep_max_ws_size`). Extremely large transactions degrade cluster performance significantly and may require config tuning.

**Binary log must be ROW format** — do not change `binlog_format` at runtime in a Galera cluster.

**Query cache must be disabled:**
```ini
query_cache_size = 0
```

### Stale Reads and Consistency

Galera is "virtually synchronous" — a committed write on one node may not be immediately visible on another without additional synchronization:

```sql
-- Force a sync point before reading (performance cost):
SET SESSION wsrep_sync_wait = 1;
SELECT * FROM orders WHERE id = 42;
```

Use `wsrep_sync_wait` only where strict read-after-write consistency is required. For most reads, eventual consistency across nodes (milliseconds) is acceptable.

### Lost Updates in Galera

Galera does not prevent lost updates in read-modify-write patterns. Use `SELECT ... FOR UPDATE` explicitly:

```sql
-- ✗ Race condition — another node could modify between SELECT and UPDATE:
SELECT balance FROM accounts WHERE id = 1;
-- ... application logic ...
UPDATE accounts SET balance = new_value WHERE id = 1;

-- ✅ Lock the row at read time:
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = new_value WHERE id = 1;
COMMIT;
```

## Replication is Not a Backup

A `DROP TABLE` or `DELETE FROM table` on the primary replicates to all replicas immediately. Replication protects against hardware failure — not against accidental data changes. Maintain independent backups (`mariadb-dump`, `mariadb-backup`) on a schedule separate from replication.

Delayed replication is one mitigation — intentionally lag a replica by a set time:
```sql
CHANGE MASTER TO MASTER_DELAY = 3600;  -- 1 hour lag
```

This gives a recovery window for accidental changes, but it is still not a substitute for backups.

## Failover Tools

- **MaxScale** — MariaDB's proxy with automatic failover, read-write splitting, and connection routing. Detects primary failure and promotes the most up-to-date replica. Requires GTID replication. [mariadb.com/docs/maxscale](https://mariadb.com/docs/maxscale)
- **ProxySQL** — third-party proxy, widely used for read-write splitting and connection pooling with MariaDB

For Galera, failover is automatic — any surviving node continues accepting writes. No proxy required for basic HA, though MaxScale or ProxySQL add connection routing.

## Sources

- [Replication Overview — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/standard-replication/replication-overview)
- [GTID — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/standard-replication/gtid)
- [Parallel Replication — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/standard-replication/parallel-replication)
- [Semi-Synchronous Replication — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/standard-replication/semisynchronous-replication)
- [Galera Cluster Known Limitations — MariaDB Docs](https://mariadb.com/docs/galera-cluster/reference/mariadb-galera-cluster-known-limitations)
- [Auto-Increments in Galera — mariadb.org](https://mariadb.org/auto-increments-in-galera/)
- [Automatic Failover with MariaDB Monitor — MaxScale Docs](https://mariadb.com/docs/maxscale/mariadb-maxscale-tutorials/automatic-failover-with-mariadb-monitor)

*For topics not covered here, see the official MariaDB documentation at [mariadb.com/docs](https://mariadb.com/docs).*
