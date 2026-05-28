# What Requires Manual Rewriting

These Oracle features have no direct equivalent and require code changes:

- **`SYNONYM`** — replace with views (`CREATE VIEW`) or update object references directly
- **`INSERT ALL` / `INSERT FIRST`** — rewrite as multiple `INSERT` statements or application logic
- **`(+)` outer join syntax** — supported natively since MariaDB 12.1 in Oracle mode. On older versions rewrite as `LEFT JOIN` / `RIGHT JOIN`:
  ```sql
  -- Oracle:
  SELECT * FROM a, b WHERE a.id = b.id(+);
  -- MariaDB:
  SELECT * FROM a LEFT JOIN b ON a.id = b.id;
  ```
- **`START WITH ... CONNECT BY`** — rewrite as recursive CTE:
  ```sql
  -- MariaDB equivalent:
  WITH RECURSIVE tree AS (
      SELECT id, parent_id, name FROM categories WHERE parent_id IS NULL
      UNION ALL
      SELECT c.id, c.parent_id, c.name FROM categories c
      JOIN tree t ON c.parent_id = t.id
  )
  SELECT * FROM tree;
  ```
- **`TIMESTAMP WITH TIME ZONE`** — store timezone offset in a separate column or handle in application
- **Object types and inheritance** — no equivalent; restructure as normalized tables
