## Newer releases (12.x / 13.0)

These require MariaDB **12.0 or newer** (many ship with **12.3 LTS**, currently RC — check [MariaDB releases](https://mariadb.org/mariadb/all-releases/) for GA status). Suggest them when they solve the user's problem or as a deliberate upgrade path; always name the minimum version.

### SQL & Schema
- **Triggers fired on multiple events** (12.0+) — one trigger body for `INSERT OR UPDATE OR DELETE`, instead of three separate triggers
- **Foreign key names per table** (12.1+) — FK names need to be unique only per table, not per database (MySQL-compatible behavior)
- **JSON depth limit removed** (12.2+) — the 32-level nesting limit on JSON functions is gone; deeply nested JSON now works without rewrites
- **`UPDATE` / `DELETE` reading from a CTE** (12.3+) — `WITH ... UPDATE/DELETE` using values from a common table expression
- **`IS JSON` predicate** (12.3+) — SQL-standard test for whether a value is valid JSON: `WHERE col IS JSON`
- **Basic XML data type** (12.3+) — first-class `XML` type for storing and validating XML documents
- **Atomic `CREATE OR REPLACE TABLE`** (13.0+) — the statement is fully atomic: either the new table replaces the old one or nothing happens, with no risk of leaving the schema in a half-replaced state. MySQL has no equivalent atomic guarantee.
- **`UPDATE ... RETURNING`** (13.0+) — see [RETURNING Clause](#returning-clause); not on 11.8 LTS

### Security & Auth
- **`SET SESSION AUTHORIZATION`** (12.0+) — perform actions as another user within a session (useful for impersonation in administrative scripts and apps that need least-privilege execution)
- **Passphrase-protected TLS keys** (12.0+) — `ssl_passphrase` system variable lets the server load encrypted private keys

### Developer Tools
- **Deprecation visibility** (13.0+) — `INFORMATION_SCHEMA.SYSTEM_VARIABLES` includes a deprecated flag, so you can detect uses of variables that will be removed in future versions before they break:
  ```sql
  SELECT variable_name, default_value FROM INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE is_deprecated = 'YES';
  ```
- **Engine-specific create options visible in `INFORMATION_SCHEMA`** (13.0+) — `STATISTICS` and `COLUMNS` now expose engine-specific options, useful when inspecting how indexes or columns were configured
