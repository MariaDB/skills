# Standard Async Replication

The foundation: one primary, one or more replicas. The primary writes to the binary log; replicas apply changes asynchronously.

## MariaDB GTID

GTID-based replication is the default since MariaDB 10.10 (MDEV-19801) and remains so on 10.11 LTS, 11.4 LTS, and 11.8 LTS. On a fresh replica start, a `RESET SLAVE`, or a `CHANGE MASTER TO` that omits `MASTER_USE_GTID`, the replica defaults to `slave_pos` instead of legacy file/position. If you have configs that rely on the old behavior, set `MASTER_USE_GTID=no` explicitly.

```sql
-- On replica (10.10+ — MASTER_USE_GTID is optional, slave_pos is the default):
CHANGE MASTER TO
  MASTER_HOST='primary.host',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_USE_GTID = slave_pos;

START SLAVE;
```

**Promoting a replica to primary** — historically `MASTER_USE_GTID=current_pos` was used to include locally-written GTIDs. **`current_pos` is deprecated since 10.10** (MDEV-20122). Use `MASTER_DEMOTE_TO_SLAVE=1` instead: it converts the old primary's `gtid_binlog_pos` into `gtid_slave_pos` so the demoted server can attach to the new primary cleanly without race conditions.

```sql
-- On the former primary, being demoted to a replica (10.10+):
CHANGE MASTER TO
  MASTER_HOST='new_primary.host',
  ...,
  MASTER_DEMOTE_TO_SLAVE=1;
START SLAVE;
```

Since MariaDB 13.0, `CHANGE MASTER` also resets `Master_Server_Id` in `SHOW SLAVES STATUS`. On older versions this field could carry stale values across primary changes — check it explicitly when reconfiguring replication on pre-13.0 servers.

### GTID Format and Multi-Source

MariaDB GTIDs have three components: `domain_id-server_id-sequence` (e.g., `0-1-247`).

This is **different from MySQL's** `server_uuid:sequence` format. They are not compatible — a MariaDB primary cannot replicate to a MySQL replica using GTIDs, and vice versa.

**Domain IDs** enable multi-source replication: assign each primary a distinct `gtid_domain_id` so replicas can track multiple sources independently:
```sql
-- On primary A:
SET GLOBAL gtid_domain_id = 1;

-- On primary B:
SET GLOBAL gtid_domain_id = 2;
```

Since MariaDB 13.0, `default_master_connection` can be set at the global level — convenient for replicas that connect to one logical "primary" source across multiple servers without specifying the connection name in every replication command.

## Parallel Replication

By default, replicas apply events serially. Parallel replication (up to 10× faster on write-heavy workloads) uses a pool of worker threads:

```ini
# my.cnf on replica:
slave_parallel_threads = 4
slave_parallel_mode = optimistic   # default since 10.5.1 — tries parallel, retries on conflict
```

`optimistic` mode applies transactions in parallel and retries on conflict. Use `conservative` for stricter workloads where conflict retries are unacceptable.

Since MariaDB 12.1, parallel replication also works when **asynchronously replicating between two Galera clusters** (MDEV-20065) — useful for cross-datacenter or DR setups where one Galera cluster is an async replica of another.

## Replication & Binlog Improvements (10.7–11.4)

- **Optimistic two-phase `ALTER TABLE` replication** (10.8+, MDEV-11675, `binlog_alter_two_phase`) — opt-in: when enabled, a large `ALTER TABLE` is started on the replica in parallel with the primary's execution rather than after, drastically reducing replication lag during schema changes.
- **`mariadb-binlog` improvements** (10.8+) — `--gtid-strict-mode` and GTID range filtering via `--start-position` / `--stop-position` lets point-in-time replay tools target GTIDs directly.
- **`slave_max_statement_time`** (10.10+, MDEV-27161) — caps the execution time of a single replicated query on the SQL thread.
- **`mariadb-binlog` filtering** (10.9+, MDEV-20119) — `--do-domain-ids` / `--ignore-domain-ids` / `--ignore-server-ids` for binlog event extraction.
- **Multi-source replication CHANNEL syntax** (10.7+, MDEV-26307) — MySQL-style `FOR CHANNEL 'name'` clauses work in `CHANGE MASTER TO`, `START SLAVE`, etc.
- **Global limit on binary log disk space** (11.4+, MDEV-31404) — `max_binlog_total_size` triggers binlog purging when total size exceeds threshold.
- **GTID index for binary log** (11.4+, MDEV-4991) — new GTID-to-position index lets reconnecting replicas seek straight to their start position.
- **Detailed replication-lag fields** (11.4+, MDEV-29639) — `SHOW REPLICA STATUS` adds fields for clearer lag interpretation than `Seconds_Behind_Master` alone.

## Performance Improvements (11.7+)

- **Large-transaction commit no longer freezes other transactions** (11.7+, MDEV-32014) — committing large transactions while `log_bin` is on no longer stalls other transactions.
- **Async rollback of prepared transactions during binlog crash recovery** (11.7+, MDEV-33853).
- **`slave_abort_blocking_timeout`** (11.7+, MDEV-34857) — kill long-running queries on a replica when they block replication progress past a threshold.

## Monitoring Replication Lag

```sql
SHOW SLAVE STATUS\G
-- Key fields:
-- Seconds_Behind_Master: estimated lag in seconds
-- Last_SQL_Error: last error stopping the SQL thread
-- Relay_Log_Pos vs Read_Master_Log_Pos: how far behind the relay log is
```

Alert when `Seconds_Behind_Master > 5` for latency-sensitive applications. A value of `NULL` means replication is not running. Note: `Seconds_Behind_Master` can be misleading on idle primaries — use heartbeat tools (e.g., `pt-heartbeat`) for accurate measurement.

Since MariaDB 11.6 (MDEV-33856), the definition of `Seconds_Behind_Master` was refined and new columns were added to `SHOW ALL REPLICAS STATUS` plus a new Information Schema `SLAVE_STATUS` table, providing more nuanced lag visibility (e.g., separate measurements for IO vs SQL thread lag).
