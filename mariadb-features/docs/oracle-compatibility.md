## Oracle Compatibility Mode

Available since MariaDB 10.3. `sql_mode=ORACLE` enables PL/SQL syntax, Oracle-compatible NULL handling, packages, and Oracle-style functions — useful when migrating from Oracle or supporting Oracle-experienced developers.

```sql
SET sql_mode=ORACLE;

-- Oracle-style stored procedures, packages, and NULL semantics work here
-- ROWNUM, SYSDATE, NVL(), DECODE() available
-- Note: EMPTY_STRING_IS_NULL is NOT included — add it separately if needed: SET sql_mode='ORACLE,EMPTY_STRING_IS_NULL'
```

Not a complete Oracle replacement, but significantly reduces migration friction.
