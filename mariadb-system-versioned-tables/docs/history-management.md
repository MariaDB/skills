# Managing History Growth

Every update adds a row. For high-update tables, history grows fast. Use partitioning to keep current data performant:

```sql
-- Separate current and historical partitions:
CREATE TABLE events (
    id INT,
    payload JSON
) WITH SYSTEM VERSIONING
  PARTITION BY SYSTEM_TIME (
    PARTITION p_hist HISTORY,
    PARTITION p_cur  CURRENT
  );

-- Auto-rotate history by time interval (10.9+):
CREATE TABLE prices (
    symbol VARCHAR(10),
    price DECIMAL(10,4)
) WITH SYSTEM VERSIONING
  PARTITION BY SYSTEM_TIME INTERVAL 1 MONTH AUTO (
    PARTITION p_cur CURRENT
  );
```

**Delete old history:**
```sql
-- Delete all history before a date:
DELETE HISTORY FROM employees BEFORE SYSTEM_TIME '2024-01-01';

-- Delete all history (requires DROP SYSTEM VERSIONING + re-add, or partition drop):
ALTER TABLE employees DROP PARTITION p_hist;
```

Requires `DELETE HISTORY` privilege. `TRUNCATE` is not allowed on versioned tables.
