# Creating a System-Versioned Table

**Simplified syntax (recommended):**
```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2)
) WITH SYSTEM VERSIONING;
```

MariaDB automatically adds hidden `ROW_START` and `ROW_END` timestamp columns. Every `INSERT`, `UPDATE`, and `DELETE` creates a history row.

**Add versioning to an existing table:**
```sql
ALTER TABLE employees ADD SYSTEM VERSIONING;
```

**Remove versioning (deletes all history):**
```sql
ALTER TABLE employees DROP SYSTEM VERSIONING;
```
