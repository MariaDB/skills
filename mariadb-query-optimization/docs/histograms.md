# Histogram Statistics

Histograms let the optimizer understand data distribution on non-indexed columns — critical for query plan quality on complex queries. Without them, the optimizer assumes uniform distribution and can choose wrong join orders.

```sql
-- Collect histograms for a table (requires a full scan — run during low traffic):
ANALYZE TABLE orders;

-- Verify histograms were collected:
SELECT * FROM mysql.column_stats WHERE table_name = 'orders';
```

**When to run `ANALYZE TABLE`:**
- After bulk inserts or large data changes
- When `EXPLAIN` shows unexpectedly high `rows` estimates
- After initially creating a table and loading data

**Tune histogram granularity** for tables with highly skewed data distributions:
```sql
SET histogram_size = 100;  -- default is 0 (disabled) in older versions, 254 in 10.4.3+
ANALYZE TABLE orders;
```

Histograms are collected per-column automatically when using `ANALYZE TABLE` with `histogram_size > 0`. They are stored in `mysql.column_stats` and consulted when `optimizer_use_condition_selectivity >= 4` (default in 10.4.1+).

### What LLMs Get Wrong
- Not running `ANALYZE TABLE` after bulk inserts: Histogram statistics become stale; optimizer makes poor plan choices.

### Quick Wins
2. `ANALYZE TABLE` — stale statistics cause bad plans.
