# ALTER TABLE on Versioned Tables

By default, `ALTER TABLE` on a versioned table raises an error to protect history integrity. To allow it:

```sql
SET system_versioning_alter_history = KEEP;
ALTER TABLE employees ADD COLUMN manager_id INT;
SET system_versioning_alter_history = ERROR; -- restore default
```

`KEEP` allows the alter but historical rows for the new column will have `NULL` values — the history is technically incomplete. This is usually acceptable for adding columns.

**Convert hidden `ROW_START`/`ROW_END` to explicit columns** (11.7+, MDEV-27293) — once a table is versioned with the hidden form (no `PERIOD FOR SYSTEM_TIME` clause), you can promote them to explicit columns later without dropping versioning. Useful if you initially started simple and later want transaction-precise history or explicit naming for tooling.
