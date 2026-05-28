# PL/SQL Syntax (10.3+)

- Packages: `CREATE PACKAGE`, `CREATE PACKAGE BODY` — since 11.4 also work outside `sql_mode=ORACLE` (MDEV-10075), letting you use packages in mixed-mode codebases
- Cursors: explicit, implicit, parameterized, `%ISOPEN`, `%ROWCOUNT`, `%FOUND`, `%NOTFOUND`
- Cursors on prepared statements (12.3+)
- Cursor variables: `TYPE ... IS REF CURSOR` (13.0+) — pass cursors as procedure parameters and return values
- Pre-defined weak `SYS_REFCURSOR` (12.0+) — built-in cursor type, no `TYPE` declaration needed; `max_open_cursors` system variable caps concurrent open cursors
- Variable types: `:=` assignment, `%TYPE`, `%ROWTYPE`
- `ROW` data type as stored function return value (11.7+) — function returns a structured row, similar to Oracle row types
- `RECORD` types in routine parameters and function `RETURN` clauses (13.0+)
- Associative arrays: `DECLARE TYPE ... TABLE OF ... INDEX BY` (12.1+)
- Stored routine parameters can have default values (11.8+) — call procedures with fewer arguments, like Oracle's `DEFAULT` clause
- Control flow: `FOR i IN 1..10 LOOP`, `GOTO`, `EXIT WHEN`, `ELSIF`, `CONTINUE`
- Exception handling: `EXCEPTION WHEN TOO_MANY_ROWS / NO_DATA_FOUND / DUP_VAL_ON_INDEX`
- Anonymous blocks: `BEGIN ... END`
- Dynamic SQL: `EXECUTE IMMEDIATE ... USING`
- Trigger variables: `:NEW`, `:OLD`

## Other Automatic Behaviors
- `||` as string concatenation (NULL-ignoring)
- `MINUS` as synonym for `EXCEPT` (10.6+)
- Named placeholders (`:1`, `:2`) in prepared statements
- `SELECT UNIQUE` as synonym for `SELECT DISTINCT`
- `DUAL` table support
