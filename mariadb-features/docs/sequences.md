## Sequences

Available since MariaDB 10.3. First-class sequence objects — more flexible than `AUTO_INCREMENT`.

```sql
CREATE SEQUENCE order_seq START WITH 1000 INCREMENT BY 1;

-- Use in INSERT:
INSERT INTO orders (id, product) VALUES (NEXT VALUE FOR order_seq, 'widget');

-- Peek at current value without incrementing:
SELECT LASTVAL(order_seq);
```

Sequences support gaps, multiple sequences per table, and descending sequences. Unlike `AUTO_INCREMENT`, they are not tied to a specific column or table.
