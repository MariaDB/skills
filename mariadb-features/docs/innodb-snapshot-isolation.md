## Behavior Change: innodb_snapshot_isolation (11.8+)

From MariaDB 11.8 LTS, `innodb_snapshot_isolation` defaults to **ON** (previously OFF, MDEV-35124). This tightens REPEATABLE READ behavior to match true snapshot isolation — transactions see a consistent snapshot from their start and writes detect conflicts more strictly.

**What can change for existing code:**
- Read-modify-write patterns that previously worked silently may now hit conflicts and error out — fail-fast is the intended behavior
- Long-running `REPEATABLE READ` transactions are more likely to see write conflicts at commit time

If existing code depends on the older permissive behavior, opt back in explicitly:
```sql
SET GLOBAL innodb_snapshot_isolation = OFF;  -- restore pre-11.8 behavior
```

The new default is the correct semantics — review code that relies on the looser behavior rather than disabling it long-term.
