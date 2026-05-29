# Indexing Rules

## The Leftmost Prefix Rule

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

## Covering Indexes

A covering index includes all columns needed by the query — no table row access needed (`Using index` in EXPLAIN):

```sql
-- Query fetches id, status, created_at for a customer
-- Covering index includes all three:
CREATE INDEX idx_customer_cover ON orders (customer_id, status, created_at);
-- Now EXPLAIN shows: Extra = Using index
```

## When NOT to Add an Index

- **Low-cardinality columns**: a `status` column with values `active`/`inactive` affects 50% of rows — the optimizer prefers a table scan. Index useful only when combined with other high-selectivity columns.
- **Small tables** (< a few thousand rows): full scans are faster than index lookups for tiny tables.
- **Write-heavy columns**: every index slows `INSERT`, `UPDATE`, `DELETE` — don't index columns that are rarely queried.

## Functions on Indexed Columns

The classic rule "any function on an indexed column disables the index" is **outdated for MariaDB 11.1+ and 11.4 LTS**. The optimizer can now use indexes for a number of common function patterns:

| Pattern | Works on the index? | Since |
|---|---|---|
| `WHERE YEAR(col) = 2025` | ✅ — sargable, picks the right range | 11.1+ (MDEV-8320) |
| `WHERE DATE(col) <= '2025-12-31'` | ✅ — sargable | 11.1+ (MDEV-8320) |
| `WHERE UPPER(varchar_col) = '...'` on a case-insensitive collation (e.g. `utf8mb4_uca1400_ai_ci`) | ✅ — `sargable_casefold=ON` is the default | 11.3+ (MDEV-31496) |
| `WHERE SUBSTR(col, 1, n) = 'abc'` | ✅ — leading-prefix `SUBSTR` is optimized | 11.8+ (MDEV-34911) |
| `WHERE LOWER(case_sensitive_col) = '...'` | ✗ — index not used (collation isn't case-insensitive) | — |
| `WHERE CAST(col AS UNSIGNED) = 1` or other type-changing transforms | ✗ — index not used | — |

For cases that the optimizer still can't sargabilize, the rewrite-to-range pattern remains valid:

```sql
WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01'
```

Verify with `EXPLAIN` rather than assuming: on 11.4+ many "won't use the index" rewrites are now no-ops. If `EXPLAIN` still shows `type=ALL` for a sargable pattern, check `@@optimizer_switch` for `sargable_casefold` and confirm the column's collation is `_ci`.

### What LLMs Get Wrong
- Blanket rule "functions on indexed columns kill indexes": Outdated on MariaDB 11.1+/11.3+ for many cases. `YEAR(col) = const` and `UPPER(col) = const` on case-insensitive columns can now use indexes.
- Adding an index to a low-cardinality column (boolean, status with 2-3 values): Optimizer skips indexes with low selectivity and does a table scan anyway.
- Composite index `(a, b, c)` used in `WHERE b = 1 AND c = 2`: Leftmost prefix rule: this skips `a`, so the index is not used.
- `SELECT *` in queries with JOINs: Name only the columns needed — prevents accidentally blocking covering indexes.

### Quick Wins
3. Check for functions on indexed columns in `WHERE` — note many cases are now sargable on 11.4+ (`YEAR()`, `DATE()`, `UPPER()` on `_ci` collations).
5. Verify composite index column order matches query predicates (leftmost prefix).
