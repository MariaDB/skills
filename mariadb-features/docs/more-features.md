## More MariaDB Features (through 11.8 LTS)

Additional capabilities on the current LTS baseline and supported older releases. See [Newer releases (12.x / 13.0)](#newer-releases-12x--130) for features that require a newer server.

### SQL & Schema
- **Invisible columns** (10.3+) — hidden from `SELECT *`, still writable; useful for schema evolution without breaking existing queries
- **`DEFAULT` expressions on BLOB/TEXT** — not supported in MySQL
- **`DECIMAL` precision to 38 digits** — MySQL stops at 30
- **`INTERSECT` and `EXCEPT`** (10.3+) — set operators not available in MySQL
- **`LIMIT` in subqueries** — supported; MySQL restricts this
- **`SELECT ... OFFSET ... FETCH`** (10.6+, MDEV-23908) — SQL-standard pagination syntax
- **Atomic DDL** (10.6+, MDEV-23842) — `CREATE TABLE`, `ALTER TABLE`, `RENAME TABLE`, `DROP TABLE`, `DROP DATABASE` are atomic on supported engines (InnoDB, Aria, MyRocks): a partial server crash mid-DDL leaves the schema in its pre-statement state, no manual cleanup needed. Multi-table `DROP TABLE` is atomic per individual drop, not for the whole list.
- **`SELECT ... SKIP LOCKED`** (10.6+, MDEV-13115, InnoDB only) — work-queue pattern: workers grab the next available row and skip rows other transactions are processing, with no lock waits
- **Ignored Indexes** (10.6+, MDEV-7317) — `ALTER TABLE t ALTER INDEX idx IGNORED` keeps the index updated but makes it invisible to the optimizer. Use this to test whether dropping an index would hurt performance before actually dropping it (zero-downtime rollback by re-enabling).
- **Dynamic columns** (5.3+) — schema-less key/value storage inside a single column
- **`SFORMAT()`** (10.7+, MDEV-25015) — string formatting function with positional placeholders
- **`NATURAL_SORT_KEY()`** (10.7+, MDEV-4742) — produces a sort key that orders strings "naturally" (so `v9` sorts before `v10`); useful in `ORDER BY` for version-like or mixed-alphanumeric data
- **JSON enhancements** — MariaDB has been catching up to MySQL 8 JSON functions over several releases:
  - `JSON_EQUALS(a, b)` / `JSON_NORMALIZE(doc)` (10.7+, MDEV-23143 / MDEV-16375) — semantic equality and canonical form for hashing or unique indexing
  - `JSON_OVERLAPS(a, b)` (10.9+, MDEV-27677) — detect shared key/value or array elements between two documents
  - JSON path syntax supports negative indices (`$.A[-1]`, `$.A[last]`) and ranges (`$.A[1 to 3]`) (10.9+, MDEV-22224 / MDEV-27911)
  - `JSON_SCHEMA_VALID(schema, doc)` (11.4+) — validate JSON against a JSON Schema Draft 2020 schema, usable inside `CHECK` constraints
  - `JSON_KEY_VALUE`, `JSON_ARRAY_INTERSECT`, `JSON_OBJECT_TO_ARRAY`, `JSON_OBJECT_FILTER_KEYS` (11.4+) — structural manipulation primitives that compose well with `JSON_TABLE`
- **`UUID_v4()` and `UUID_v7()` functions** (11.7+) — generate version-4 random or version-7 time-ordered UUIDs; the v7 form is sortable and ideal for primary keys
- **`FORMAT_BYTES()`** (11.8+) — convert a byte count to a human-readable string (e.g. `1234567` → `1.18 MiB`)
- **`CONV()` extended to base 62** (11.4+, MDEV-30190) — `CONV(61,10,62)` returns `z`; useful for short opaque IDs
- **`CRC32C()` function and `CRC32()` with optional initial-value argument** (10.8+, MDEV-27208) — Castagnoli polynomial CRC, and seedable CRC32 for chained checksums
- **Single-table `DELETE` with table aliases** (11.6+) — `DELETE t FROM mytable t WHERE ...` syntax now works without rewriting
- **`REPAIR TABLE ... FORCE`** (11.5+) — force-repair even when the table appears clean
- **Stored routine parameter default values** (11.8+, MDEV-10862) — `PROCEDURE p(a INT DEFAULT 0, b INT DEFAULT 0)` — call with fewer arguments
- **Stored function `IN`/`OUT`/`INOUT` parameter qualifiers** (10.8+, MDEV-10654) — bring stored functions in line with stored procedure parameter modes
- **`ROW` data type as stored function return value** (11.7+, MDEV-12252) — return structured rows from stored functions
- **`CREATE PACKAGE` / `CREATE PACKAGE BODY` outside Oracle mode** (11.4+, MDEV-10075) — package routines work under the default `sql_mode` too, not only with `sql_mode=ORACLE`
- **Update triggers with column list** (11.8+, MDEV-34551) — `CREATE TRIGGER ... BEFORE UPDATE OF col1, col2 ON t` — fire only when those columns are updated
- **Stored procedures and functions** — MariaDB uses SQL/PSM syntax (`DECLARE`, `HANDLER`, `CURSOR`, `BEGIN...END`); AI agents often generate incorrect syntax — see [Stored Procedures — MariaDB Docs](https://mariadb.com/docs/server/server-usage/stored-routines/stored-procedures)

### Storage Engines
- **ColumnStore** — columnar engine for analytical/data warehouse workloads
- **Aria** — crash-safe MyISAM replacement, used internally for temp tables
- **MyRocks** (10.2+) — RocksDB-based, optimized for write-heavy workloads with compression
- **CONNECT** — query external data sources (CSV, JDBC, ODBC, MongoDB) as SQL tables
- **Spider** — sharding across multiple MariaDB instances

### Security & Auth
- **`unix_socket` authentication** — authenticate OS users without passwords; `authentication_string` support added in 11.6+ for finer-grained mapping
- **ED25519 plugin** — modern authentication alternative to SHA1-based plugins
- **PARSEC plugin** (11.6+, MDEV-32618) — Password Authentication using Response Signed with Elliptic Curve; salt and per-installation key separation make stolen hashes unusable elsewhere
- **`password_reuse_check` plugin** (10.7+, MDEV-9245) — prevent password reuse for a configurable number of days via `password_reuse_check_interval`
- **`GRANT ... TO PUBLIC`** (10.11+, MDEV-5215) — grant privileges to all users in one statement; pair with `SHOW GRANTS FOR PUBLIC`
- **`SHOW CREATE ROUTINE` privilege** (11.4+, MDEV-23149) — let users inspect a routine's definition without granting `SELECT` on `mysql.proc`
- **`READ ONLY ADMIN` is now a distinct privilege** (10.11+, MDEV-29596) — split out of `SUPER` so a true read-only replica role can be granted; existing accounts that need to write to a `read_only=1` replica need this privilege granted explicitly
- **Role-based access control** (10.0+) — roles available before MySQL added them
- **SSL by default** — the `mariadb` client opts into SSL by default since 10.10 (MDEV-27105). The server side requires SSL by default since 11.4 LTS, with auto-generated self-signed certificates and automatic client-side verification (`tls_fp` for fingerprint-pinning).
- **`AES_ENCRYPT()` / `AES_DECRYPT()` with `iv` and `mode`** (11.4+, MDEV-30878) — `AES_ENCRYPT(str, key, iv, mode)`; supported modes include CBC, OFB, CFB128, CTR (default mode comes from the new `block_encryption_mode` variable). Brings parity with MySQL's encryption interface.
- **`KDF()` key-derivation function** (11.4+, MDEV-31474) — derive an encryption key from a passphrase using PBKDF2-HMAC or HKDF — `AES_ENCRYPT(data, KDF('passw0rd', 'salt', 'info', 'hkdf'), iv)`. Use this rather than feeding a raw password into `AES_ENCRYPT`.
- **`RANDOM_BYTES(n)`** (10.10+, MDEV-25704) — cryptographically secure random bytes (1–1024) from the SSL library's RNG
- **`DES_ENCRYPT()` / `DES_DECRYPT()` deprecated** (10.10+, MDEV-27104) — old DES cipher; use `AES_ENCRYPT` / `AES_DECRYPT` with `KDF()` instead
- **Table-level encryption** — encrypt individual tables, not just the whole datadir
- **HashiCorp Vault integration** — key management plugin

### Replication & HA
- **Galera Cluster** — built-in synchronous multi-master clustering
- **Multi-source replication** — replicate from multiple primaries simultaneously
- **Parallel replication** — faster replica apply
- **Lag-free `ALTER TABLE` replication** — schema changes don't stall replicas

### Connectors
- **LGPL-licensed connectors** for C, C++, Java, Python, Node.js, ODBC, R2DBC — permissive licensing for commercial applications; MySQL connectors are GPL

### Developer Tools
- **`EXPLAIN` in slow query log** — automatic execution plan logging for slow queries
- **Progress reporting** for `ALTER TABLE` and `CHECK TABLE`
- **`mariadb-backup`** — hot backup with backup locks (no `FLUSH TABLES WITH READ LOCK`)
- **Non-blocking client API** — async queries without threads
