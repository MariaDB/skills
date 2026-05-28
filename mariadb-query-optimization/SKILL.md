---
name: mariadb-query-optimization
description: "Best practices for query optimization in MariaDB — indexing strategies, EXPLAIN analysis, pagination, histogram statistics, and MariaDB-specific optimizer settings. Use when diagnosing slow queries, designing indexes, reviewing schema or query performance, or when queries involve large tables, pagination, GROUP BY, or ORDER BY. Also use when the user asks about MariaDB query performance, EXPLAIN output, or optimizer behavior."
---

# MariaDB Query Optimization

*Last updated: 2026-05-25*

> **Requires:** MariaDB 10.1+ for `ANALYZE` and histograms.
>
> **Server context:** See [MariaDB Versioning Context](../_shared/versioning.md).

## Documentation

- [Deep Dive Index]
  - [Execution Plans](docs/execution-plans.md)
  - [Index Strategies](docs/index-strategies.md)
  - [Pagination](docs/pagination.md)
  - [Histograms](docs/histograms.md)
- [Sources](docs/sources.md)
