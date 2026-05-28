# Querying Historical Data

The `FOR SYSTEM_TIME` clause goes directly after the table name:

```sql
-- Data as it was at a specific point in time:
SELECT * FROM employees FOR SYSTEM_TIME AS OF '2026-01-01 00:00:00';

-- All rows that existed during a period (both boundaries included):
SELECT * FROM employees FOR SYSTEM_TIME BETWEEN '2025-01-01' AND '2026-01-01';

-- Half-open range (includes start, excludes end):
SELECT * FROM employees FOR SYSTEM_TIME FROM '2025-01-01' TO '2026-01-01';

-- Every version of every row, ever:
SELECT * FROM employees FOR SYSTEM_TIME ALL;

-- See the full history of one employee:
SELECT *, ROW_START, ROW_END FROM employees FOR SYSTEM_TIME ALL WHERE id = 42;
```

**Apply an implicit AS OF to all queries in a session:**
```sql
SET system_versioning_asof = '2026-01-01 00:00:00';
-- Now all queries against versioned tables see that point in time
SELECT * FROM employees; -- returns 2026-01-01 snapshot
SET system_versioning_asof = DEFAULT; -- reset
```
