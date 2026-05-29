---
name: mariadb-replication-and-ha
description: "Best practices for MariaDB replication and high availability — Galera Cluster, GTID replication, semi-synchronous replication, parallel replication, and application patterns for HA environments. Use when setting up replication, designing applications for HA, working with Galera Cluster, asking about MariaDB GTID, handling replication lag in application code, or choosing between replication topologies. IMPORTANT — MariaDB GTID format is incompatible with MySQL GTID, and Galera Cluster is built into MariaDB with no plugins required."
---

# MariaDB Replication and High Availability

*Last updated: 2026-05-25*

MariaDB offers three tiers of replication depending on your consistency and availability requirements:

| Approach | Consistency | Failover | Best for |
|---|---|---|---|
| **Standard async replication** | Eventual | Manual or tool-assisted | Read scaling, backups, low-latency writes |
| **Semi-synchronous replication** | Stronger (1 replica ACK) | Manual or tool-assisted | Reducing data loss risk without full sync overhead |
| **Galera Cluster** | Synchronous (multi-primary) | Automatic | Zero-data-loss HA, multi-datacenter writes |

> **Requires:** GTID replication: MariaDB 10.0+. Semi-sync built-in: 10.3+. Parallel replication optimistic mode: 10.5.1+.
>
> **Server context:** See [MariaDB Versioning Context](../_shared/versioning.md).

## Documentation

- [Deep Dive Index]
  - [Overview](docs/overview.md)
  - [Standard Async Replication](docs/async-replication.md)
  - [Semi-Synchronous Replication](docs/semi-sync-replication.md)
  - [Galera Cluster](docs/galera-cluster.md)
  - [Failover Tools](docs/failover-tools.md)
- [Sources](docs/sources.md)
