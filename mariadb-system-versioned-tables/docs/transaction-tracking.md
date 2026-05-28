# Transaction-Precise History (InnoDB Only)

For exact transaction-boundary tracking instead of timestamps:

```sql
CREATE TABLE ledger (
    id INT,
    amount DECIMAL(10,2),
    trx_start BIGINT UNSIGNED GENERATED ALWAYS AS ROW START,
    trx_end   BIGINT UNSIGNED GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME(trx_start, trx_end)
) WITH SYSTEM VERSIONING;

SELECT * FROM ledger FOR SYSTEM_TIME AS OF TRANSACTION 12345;
```

Uses `mysql.transaction_registry` internally. Not compatible with `PARTITION BY SYSTEM_TIME`.
