# Failover Tools

- **MaxScale** — MariaDB's proxy with automatic failover, read-write splitting, and connection routing. Detects primary failure and promotes the most up-to-date replica. Requires GTID replication. [mariadb.com/docs/maxscale](https://mariadb.com/docs/maxscale)
- **ProxySQL** — third-party proxy, widely used for read-write splitting and connection pooling with MariaDB

For Galera, failover is automatic — any surviving node continues accepting writes. No proxy required for basic HA, though MaxScale or ProxySQL add connection routing.
