---
name: mariadb-query-optimization
description: "Best practices for query optimization in MariaDB — indexing strategies, EXPLAIN analysis, pagination, histogram statistics, and MariaDB-specific optimizer settings. Use when diagnosing slow queries, designing indexes, reviewing schema or query performance, or when queries involve large tables, pagination, GROUP BY, or ORDER BY. Also use when the user asks about MariaDB query performance, EXPLAIN output, or optimizer behavior."
---

# MariaDB Query Optimization

*Last updated: 2026-05-21*

## What LLMs Get Wrong

| Pattern | What to do instead |
|---|---|
| `SELECT * FROM table LIMIT 10 OFFSET 50000` | Use cursor-based pagination — `OFFSET` scans all skipped rows |
| Functions on indexed columns: `WHERE YEAR(created_at) = 2025` | Rewrite to use the index: `WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01'` |
| Adding an index to a low-cardinality column (boolean, status with 2-3 values) | Optimizer skips indexes with low selectivity and does a table scan anyway |
| Not running `ANALYZE TABLE` after bulk inserts | Histogram statistics become stale; optimizer makes poor plan choices |
| Composite index `(a, b, c)` used in `WHERE b = 1 AND c = 2` | Leftmost prefix rule: this skips `a`, so the index is not used |
| `SELECT *` in queries with JOINs | Name only the columns needed — prevents accidentally blocking covering indexes |

## Reading EXPLAIN

Run `EXPLAIN` before tuning anything:

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 10;
```

**Red flags in the output:**

| Field | Red flag | What it means |
|---|---|---|
| `type` | `ALL` | Full table scan — missing index or index not used |
| `key` | `NULL` | No index used despite one existing — check for function on column or type mismatch |
| `rows` | Very high number | Optimizer estimates scanning many rows |
| `Extra` | `Using filesort` | Expensive sort not covered by an index |
| `Extra` | `Using temporary` | Temp table created — often from `GROUP BY` or `DISTINCT` |
| `Extra` | `Using index` | ✅ Good — covering index, no table row access needed |

**`ANALYZE`** statement (MariaDB 10.1+) actually executes the query and shows real row counts vs. estimates — more reliable than `EXPLAIN` alone. Note: MariaDB uses `ANALYZE`, not `EXPLAIN ANALYZE`:

```sql
ANALYZE SELECT * FROM orders WHERE customer_id = 42;
```

**Optimizer Trace** shows the optimizer's full decision process. Since MariaDB 12.1 the trace can include full table and view definitions (`optimizer_record_context` system variable). Since 13.0 it also includes the specific statistics (histograms, index stats) used for cardinality estimates — together they're powerful for diagnosing surprising `rows` estimates:

```sql
SET optimizer_trace = 'enabled=on';
SELECT * FROM orders WHERE customer_id = 42;
SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE\G
SET optimizer_trace = 'enabled=off';
```

## Indexing Rules

### The Leftmost Prefix Rule

For a composite index `(a, b, c)`, MariaDB can use:
- `WHERE a = 1` ✅
- `WHERE a = 1 AND b = 2` ✅
- `WHERE a = 1 AND b = 2 AND c = 3` ✅
- `WHERE b = 2` ✗ — skips `a`, index not used
- `WHERE a = 1 AND c = 3` — only `a` part is used

Put the most selective equality conditions first, then range conditions last:
```sql
-- Query: WHERE status = 'active' AND created_at > '2025-01-01' ORDER BY created_at
INDEX (status, created_at)  -- ✅ equality first, range last
INDEX (created_at, status)  -- ✗ range first breaks the prefix for status
```

### Covering Indexes

A covering index includes all columns needed by the query — no table row access needed (`Using index` in EXPLAIN):

```sql
-- Query fetches id, status, created_at for a customer
-- Covering index includes all three:
CREATE INDEX idx_customer_cover ON orders (customer_id, status, created_at);
-- Now EXPLAIN shows: Extra = Using index
```

### When NOT to Add an Index

- **Low-cardinality columns**: a `status` column with values `active`/`inactive` affects 50% of rows — the optimizer prefers a table scan. Index useful only when combined with other high-selectivity columns.
- **Small tables** (< a few thousand rows): full scans are faster than index lookups for tiny tables.
- **Write-heavy columns**: every index slows `INSERT`, `UPDATE`, `DELETE` — don't index columns that are rarely queried.

### Functions on Indexed Columns Kill Indexes

```sql
-- ✗ Index on created_at is not used:
WHERE YEAR(created_at) = 2025
WHERE DATE(created_at) = '2025-06-01'
WHERE UPPER(email) = 'USER@EXAMPLE.COM'

-- ✅ Rewrite to let the index work:
WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01'
WHERE created_at >= '2025-06-01' AND created_at < '2025-06-02'
WHERE email = 'user@example.com'  -- store normalized, don't transform at query time
```

## Pagination: Cursor-Based Instead of OFFSET

`OFFSET` is a hidden performance trap. `LIMIT 10 OFFSET 50000` scans and discards 50,000 rows on every page load.

```sql
-- ✗ Slow — scans 50,000 rows to skip them:
SELECT id, title FROM posts ORDER BY id DESC LIMIT 10 OFFSET 50000;

-- ✅ Fast — index seek directly to the cursor position:
-- First page:
SELECT id, title FROM posts ORDER BY id DESC LIMIT 10;

-- Next page (pass last id from previous result as $last_id):
SELECT id, title FROM posts WHERE id < $last_id ORDER BY id DESC LIMIT 10;
```

For filtered queries, include the filter column in the index alongside id:

```sql
-- Query: WHERE category = 'news' ORDER BY id DESC
CREATE INDEX idx_cat_id ON posts (category, id);
-- Cursor query:
SELECT id, title FROM posts WHERE category = 'news' AND id < $last_id ORDER BY id DESC LIMIT 10;
```

To detect whether another page exists, fetch `LIMIT 11` and check if the 11th row appears.

## Histogram Statistics

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

## MariaDB Optimizer Switches

MariaDB's optimizer has more tunable flags than MySQL. The most useful for developers:

```sql
-- See current settings:
SELECT @@optimizer_switch\G

-- Disable a specific optimization for a session (useful for debugging):
SET optimizer_switch = 'derived_merge=off';

-- Re-enable:
SET optimizer_switch = 'derived_merge=on';
```

**Most impactful flags:**

| Flag | Default | Effect |
|---|---|---|
| `derived_merge` | on | Merges derived tables into outer query — usually faster |
| `semijoin` | on | Optimizes `IN`/`EXISTS` subqueries — disable to debug unexpected plans |
| `subquery_cache` | on | Caches correlated subquery results — big win for repeated subqueries |
| `rowid_filter` | on | Pre-filters rowids before fetching rows — helps range queries |
| `mrr` | off | Multi-Range Read — enable for large range scans on spinning disks |

Turn flags off one at a time to isolate which optimization is causing a bad plan, then report via JIRA if a default setting produces a worse plan than the alternative.

### Optimizer Improvements in MariaDB 12.x

Several long-standing limitations were lifted in the 12.x rolling releases — useful to know when you see surprisingly bad plans on the 11.8 LTS baseline:

- **Rowid filtering on reverse-ordered scans** (12.0+) — previously `ORDER BY ... DESC` queries couldn't benefit from rowid filtering; now they can
- **Index Condition Pushdown on reverse-ordered scans** (12.0+) — same fix for ICP
- **Loose Index Scan ("Use index for group-by") works with `DESC` key parts** (12.0+) — previously required `ASC` indexes
- **GROUP BY / ORDER BY can use indexes on virtual columns** (12.1+)
- **Reorderable LEFT JOIN optimization** (12.3+) — the optimizer can now reorder more `LEFT JOIN` combinations
- **Distinct GROUP BY column inference** (12.2+) — derived tables with `GROUP BY` are recognized as having distinct group keys, enabling more optimizations downstream

If you target the 11.8 LTS baseline and see a plan that looks needlessly slow on a reverse-ordered or virtual-column query, it may be one of these — verify by running the same query on a 12.x version.

## Optimizer Hints

MariaDB 12.0 introduced a comprehensive MySQL-8-style optimizer hints framework (MDEV-35504), with additional hints added through 12.1 and 12.2. Hints go in a `/*+ ... */` comment right after `SELECT` and override the optimizer for one query without changing session settings:

```sql
SELECT /*+ JOIN_ORDER(o, c) */ *
FROM orders o JOIN customers c ON c.id = o.customer_id;
```

**Available hints:**

| Hint | Since | Purpose |
|---|---|---|
| `QB_NAME(name)` | 12.0 | Name a query block so other hints can target it from outside |
| `JOIN_FIXED_ORDER` / `JOIN_ORDER(t1, t2, ...)` | 12.0 | Force a join order (`JOIN_FIXED_ORDER` is similar to `STRAIGHT_JOIN`) |
| `JOIN_PREFIX(t1, ...)` / `JOIN_SUFFIX(t1, ...)` | 12.0 | Force specific tables to be first or last in the join order |
| `MAX_EXECUTION_TIME(ms)` | 12.0 | Abort the query if it runs longer than the timeout |
| `[NO_]MRR` / `[NO_]BKA` / `[NO_]BNL` | 12.0 | Toggle Multi-Range Read, Batched Key Access, Block Nested Loop |
| `[NO_]ICP` | 12.0 | Toggle Index Condition Pushdown |
| `[NO_]RANGE_OPTIMIZATION` | 12.0 | Toggle range optimizer |
| `SEMIJOIN(strategy, ...)` / `SUBQUERY(strategy)` | 12.0 | Pick subquery rewrite strategy |
| `[NO_]INDEX(t idx, ...)` / `[NO_]JOIN_INDEX` / `[NO_]GROUP_INDEX` / `[NO_]ORDER_INDEX` | 12.1 | Force / forbid specific index usage by purpose |
| `[NO_]SPLIT_MATERIALIZED` / `[NO_]DERIVED_CONDITION_PUSHDOWN` / `[NO_]MERGE` | 12.1 | Control subquery / derived-table optimizations |
| `[NO_]ROWID_FILTER` / `[NO_]INDEX_MERGE` | 12.2 | Toggle rowid filtering and index merge |

**`QB_NAME()` example** — name a subquery so an outer hint can target it:

```sql
SELECT /*+ NO_MERGE(@sub) */ *
FROM (
    SELECT /*+ QB_NAME(sub) */ customer_id, COUNT(*) AS n
    FROM orders
    GROUP BY customer_id
) t
WHERE n > 10;
```

Hints are more targeted than `SET optimizer_switch` because they apply only to the query they're in, not the whole session.

## Quick Wins Checklist

Before adding indexes or rewriting queries, check these first:

1. `EXPLAIN` the slow query — confirm where the time actually is
2. `ANALYZE TABLE` — stale statistics cause bad plans
3. Check for functions on indexed columns in `WHERE`
4. Check for `OFFSET` in pagination queries
5. Verify composite index column order matches query predicates (leftmost prefix)
6. Check `EXPLAIN` Extra column for `Using filesort` or `Using temporary` — these often point to a missing or misordered index

## Sources

- [Query Optimizations — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/query-optimizations)
- [EXPLAIN — MariaDB KB](https://mariadb.com/docs/server/reference/sql-statements/administrative-sql-statements/analyze-and-explain-statements/explain)
- [optimizer_switch — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/query-optimizations/optimizer-switch)
- [Getting Started with Indexes — MariaDB KB](https://mariadb.com/docs/server/mariadb-quickstart-guides/mariadb-indexes-guide)
- [Building the Best Index for a Given SELECT — MariaDB Docs](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/optimization-and-indexes/building-the-best-index-for-a-given-select)
- [Histogram-Based Statistics — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/query-optimizations/statistics-for-optimizing-queries/histogram-based-statistics)
- [Pagination Optimization — MariaDB KB](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/query-optimizations/pagination-optimization)

*For topics not covered here, see the official MariaDB documentation at [mariadb.com/docs](https://mariadb.com/docs).*
