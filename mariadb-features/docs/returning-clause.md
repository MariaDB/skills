## RETURNING Clause

Get inserted, updated, or deleted rows back without a second query.

### INSERT and DELETE (10.5+)

Available on 11.8 LTS and earlier supported releases:

```sql
-- Get the generated ID after insert:
INSERT INTO orders (product, qty) VALUES ('widget', 5)
    RETURNING id, created_at;

-- Get deleted rows for logging:
DELETE FROM queue WHERE processed = 1
    RETURNING id, payload;
```

### UPDATE (13.0+ only)

Not available on 11.8 LTS — confirm server version before suggesting. On older releases use a follow-up `SELECT` or redesign:

```sql
UPDATE orders SET qty = qty + 1 WHERE id = 42
    RETURNING id, qty;
```
