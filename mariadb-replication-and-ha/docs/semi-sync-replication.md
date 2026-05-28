# Semi-Synchronous Replication

The primary waits for at least one replica to acknowledge receipt before committing. Reduces data loss risk on failover without requiring full synchronous overhead.

```sql
-- Enable on primary:
SET GLOBAL rpl_semi_sync_master_enabled = 1;

-- Enable on replica:
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

If no replica acknowledges within `rpl_semi_sync_master_timeout` (default 10 seconds), the primary falls back to async. Built-in since MariaDB 10.3 — no plugin needed.

Use when: you need stronger data durability than async but your workload tolerates a small write latency increase.
