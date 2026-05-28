# Galera Cluster

Multi-primary synchronous replication — all nodes accept reads and writes, changes are certified across the cluster before committing. No single point of failure. Built into MariaDB.

> **Packaging change (12.3+):** The Galera library is no longer included as a server-package dependency or in the MariaDB repositories by default (MDEV-38744). On 12.3+ you must install `galera-4` (or your distro's equivalent) separately when setting up a Galera node. The MariaDB server still understands Galera natively — only the library distribution changed.

## Developer Constraints

These will break in Galera if you're not aware of them:

- **All tables must have a primary key:**
  ```sql
  -- ✗ DELETE fails in Galera on keyless tables:
  CREATE TABLE logs (message TEXT);

  -- ✅ Always define a PK:
  CREATE TABLE logs (id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY, message TEXT);
  ```
- **AUTO_INCREMENT values have gaps** — Galera uses `auto_increment_increment` and `auto_increment_offset` per node to avoid conflicts, resulting in non-sequential IDs.
- **LOCK TABLES, GET_LOCK(), and FLUSH TABLES {table list} WITH READ LOCK are not supported** — use transactions. Note: global `FLUSH TABLES WITH READ LOCK` (no table list) IS supported.
- **InnoDB only** — Galera replicates only InnoDB tables.
- **Transaction size limits** — default caps: 128K rows (`wsrep_max_ws_rows`) and 2GB (`wsrep_max_ws_size`).
- **Binary log must be ROW format**.
- **Write-set retry on conflict** (12.1+) — `wsrep_applier_retry_count` controls how many times an applier retries a write set.
- **Automatic SST user account management** (11.6+, MDEV-31809) — Galera now manages the dedicated SST (State Snapshot Transfer) user account automatically.
- **IP allowlist for nodes joining the cluster** (10.10+, MDEV-27246) — `wsrep_allowlist` restricts which IPs can make SST/IST requests.

## Stale Reads and Consistency

Galera is "virtually synchronous" — a committed write on one node may not be immediately visible on another without additional synchronization:

```sql
-- Force a sync point before reading (performance cost):
SET SESSION wsrep_sync_wait = 1;
SELECT * FROM orders WHERE id = 42;
```

Use `wsrep_sync_wait` only where strict read-after-write consistency is required.

## Lost Updates in Galera

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
