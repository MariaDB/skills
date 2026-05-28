## System-Versioned Tables

Available since MariaDB 10.3. Track the full history of every row automatically, without triggers or audit tables.

```sql
CREATE TABLE prices (
    product VARCHAR(100),
    price DECIMAL(10,2)
) WITH SYSTEM VERSIONING;

-- Query data as it was at a point in time:
SELECT * FROM prices FOR SYSTEM_TIME AS OF '2025-01-01 00:00:00';

-- See all historical versions of a row:
SELECT * FROM prices FOR SYSTEM_TIME ALL WHERE product = 'widget';
```

Use this instead of manually maintained `valid_from` / `valid_to` columns or separate audit tables.
