## INet4 and INet6 Data Types

`INET6` is available since MariaDB 10.5 (stores both IPv4 and IPv6, 16 bytes). The dedicated `INET4` type (4-byte IPv4-only) was added later in 10.10 (MDEV-23287). Native storage gives correct comparison, sorting, and indexing — no need for `VARCHAR` plus application-side validation.

```sql
CREATE TABLE connections (
    client_ip INet6 NOT NULL,
    connected_at DATETIME NOT NULL,
    INDEX (client_ip)
);

INSERT INTO connections VALUES (INet6('192.168.1.1'), NOW());
INSERT INTO connections VALUES (INet6('::1'), NOW());

-- Range queries work correctly:
SELECT * FROM connections WHERE client_ip BETWEEN INet6('10.0.0.0') AND INet6('10.255.255.255');
```

Use `INET4` (10.10+) when you know a column is IPv4-only and want the smaller storage; `INET6` is the right default for mixed or IPv6-capable workloads.
