# Key Gotchas

- **Autocommit**: MariaDB has autocommit enabled by default; Oracle does not. Add explicit `COMMIT`/`ROLLBACK` handling or disable autocommit.
- **`SYSDATE`**: In MariaDB Oracle mode, `SYSDATE` works but returns `DATETIME`. Verify date-only comparisons.
- **`DUAL`**: Works in MariaDB but not identically — avoid schema-qualified references like `schema.DUAL`.
- **Sequences**: `CREATE SEQUENCE` works in MariaDB 10.3+ with Oracle-compatible syntax (`MINVALUE`, `MAXVALUE`, `INCREMENT BY`).
- **`DROP USER` with active sessions** (12.1+): Oracle-compatible behavior — in Oracle mode, `DROP USER` fails if the user has active sessions; in other modes, it issues a warning.
- **~20% rewrite**: No tool achieves 100% Oracle-to-MariaDB conversion. Plan for manual review of complex PL/SQL, object types, and unsupported syntax.
