# Reading EXPLAIN

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

### Optimizer Improvements in the 10.7–10.11 LTS Window

The 10.11 LTS line bundles features that arrived in the 10.7–10.10 short-term releases:

- **JSON-format histograms** (10.8+, MDEV-21130, MDEV-26519) — histogram statistics are stored in JSON and are more precise than the older binary format. Just running `ANALYZE TABLE` on 10.8+ gives the optimizer better cardinality estimates.
- **Descending indexes** (10.8+, MDEV-13756) — `CREATE INDEX idx ON t (a ASC, b DESC)` is supported; useful for composite `ORDER BY a, b DESC` patterns and for `MIN()`/`MAX()` on descending indexes.
- **`SHOW ANALYZE [FORMAT=JSON]`** (10.9+, MDEV-27021) — get the optimizer plan and runtime stats for a query running in another connection without intrusion. `EXPLAIN FOR CONNECTION` syntax also supported (MDEV-10000).
- **Improved optimization for joins with many `eq_ref` tables** (10.10+, MDEV-28852, MDEV-26278) — large star-schema-style joins plan dramatically better.
- **`ANALYZE FORMAT=JSON` reports time spent in the optimizer itself** (10.11+, MDEV-28926) — separates planning time from execution time.

### Optimizer Improvements in 11.4 LTS

The 11.4 LTS line continues the overhaul:

- **New cost-based cost model** (11.0+) — replaces the older rule-based heuristics with a tuned model aware of SSDs and per-engine characteristics. `EXPLAIN` and join-order choices in 10.6 vs. 11.4 can differ noticeably on the same query. If you have manual `optimizer_adjust_secondary_key_costs` settings from 10.x, remove them — they're no-ops on 11.4+.
- **Semi-join optimization for single-table `UPDATE`/`DELETE`** (11.1+, MDEV-7487) — subqueries inside `UPDATE`/`DELETE` can now use the same subquery rewrites that `SELECT` uses (materialization, semi-join, etc.). Often a large speedup, no rewrite needed.
- **Sargable `DATE`/`YEAR` comparisons against constants** (11.1+, MDEV-8320) — see [Functions on Indexed Columns](#functions-on-indexed-columns) above.
- **Sargable case-folding** (11.3+, MDEV-31496, `sargable_casefold` on by default) — `UCASE`/`LCASE`/`UPPER`/`LOWER` on a column with a case-insensitive collation can use the index.

### Optimizer Improvements in 11.5–11.8 LTS

These are part of the current LTS baseline — useful for understanding what the optimizer can do today:

- **Index Condition Pushdown on partitioned tables** (11.5+, MDEV-12404) — previously partitioned tables couldn't use ICP; now they do, often a large speedup on partitioned schemas
- **`ANALYZE` shows selectivity of pushed index condition** (11.5+, MDEV-18478) — useful when diagnosing whether ICP is helping
- **Charset Narrowing Optimization on by default** (11.8+, MDEV-34380) — eliminates unnecessary character set conversions in WHERE clauses
- **`SUBSTR(col, 1, n) = const_str` optimization** (11.8+, MDEV-34911) — the optimizer can now use a column index even when the condition is a leading-prefix `SUBSTR`
- **Virtual column support in the optimizer** (11.8+, MDEV-35616) — see [Virtual Column Support in the Optimizer](https://mariadb.com/docs/server/ha-and-performance/optimization-and-tuning/query-optimizations/virtual-column-support-in-the-optimizer); previously, virtual columns were largely invisible to the optimizer
- **Cost-based subquery strategy for single-table `UPDATE`/`DELETE`** (11.8+, MDEV-25008) — the optimizer now picks between subquery strategies by cost

### Optimizer Improvements in MariaDB 12.x

Several further limitations were lifted in the 12.x rolling releases:

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

### Quick Wins
1. `EXPLAIN` the slow query — confirm where the time actually is.
6. Check `EXPLAIN` Extra column for `Using filesort` or `Using temporary` — these often point to a missing or misordered index.
