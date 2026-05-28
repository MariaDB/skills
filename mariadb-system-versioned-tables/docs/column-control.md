# Column-Level Control

Exclude specific columns from versioning (useful for frequently-updated columns like counters):

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    view_count INT WITHOUT SYSTEM VERSIONING  -- not tracked
) WITH SYSTEM VERSIONING;
```

Or version only specific columns in a non-versioned table:
```sql
CREATE TABLE config (
    key VARCHAR(50),
    value TEXT WITH SYSTEM VERSIONING  -- only this column tracked
);
```
