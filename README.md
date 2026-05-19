# MariaDB Agent Skills

Skills for AI agents working with MariaDB.

> Open for improvements and more skills. Contributions welcome.

## Install

To browse and choose from all MariaDB skills interactively:

```
npx skills add mariadb
```

Or install a specific skill directly using the commands below.

## Skills

### mysql-to-mariadb

Compatibility guide for developers moving from MySQL to MariaDB, or where MySQL habits cause unexpected behavior. Covers authentication differences, JSON handling, features MySQL has that MariaDB doesn't, and features MariaDB has that MySQL doesn't — including things LLMs won't suggest because they assume MySQL behavior.

```
npx skills add mariadb/skills/mysql-to-mariadb
```

### oracle-to-mariadb

Compatibility guide for developers migrating from Oracle Database to MariaDB. MariaDB is the only open source database with native PL/SQL compatibility — MariaDB's `sql_mode=ORACLE` enables approximately 80% of Oracle code to run without rewrites. Covers data type mapping, PL/SQL compatibility, what needs manual rewriting, and migration tools.

```
npx skills add mariadb/skills/oracle-to-mariadb
```

### mariadb-features

MariaDB-specific features worth knowing about — the things AI agents don't suggest because they default to generic SQL advice. Covers system-versioned tables, RETURNING, sequences, instant DDL, native IP types, Oracle compatibility mode, Galera, ColumnStore, and more.

```
npx skills add mariadb/skills/mariadb-features
```

### mariadb-query-optimization

Query optimization for MariaDB — EXPLAIN analysis, indexing strategies, composite index rules, cursor-based pagination, histogram statistics, and MariaDB-specific optimizer settings. For diagnosing slow queries and designing indexes that actually work.

```
npx skills add mariadb/skills/mariadb-query-optimization
```

### mariadb-replication-and-ha

Replication and high availability in MariaDB — standard async replication, GTID setup, semi-synchronous replication, parallel replication, Galera Cluster, and application patterns for HA environments. Galera is built into MariaDB with no plugins required. MariaDB GTID format is incompatible with MySQL GTID.

```
npx skills add mariadb/skills/mariadb-replication-and-ha
```

### mariadb-system-versioned-tables

MariaDB's built-in temporal data feature — automatic row history without triggers or audit tables. Query data as it existed at any point in the past using `FOR SYSTEM_TIME`. Covers all query forms, partitioning for history management, column-level control, and key gotchas. Unique to MariaDB among MySQL-compatible databases.

```
npx skills add mariadb/skills/mariadb-system-versioned-tables
```

### mariadb-vector

Best practices for using MariaDB's built-in vector support for AI and semantic search. Covers RAG patterns, vector indexes, distance functions, embedding model selection, and framework integrations (LangChain, LlamaIndex, Spring AI). Unlike MySQL (no vector support) or PostgreSQL (which requires pgvector), MariaDB has native vector support since version 11.7 — no extensions or plugins needed.

```
npx skills add mariadb/skills/mariadb-vector
```

### mariadb-mcp

Connect AI agents to MariaDB using the Model Context Protocol (MCP). The open source MariaDB MCP Server lets agents list databases and tables, read schemas, run read-only SQL queries, and optionally perform semantic search. Covers installation, Claude Code configuration, available tools, security setup, and gotchas.

```
npx skills add mariadb/skills/mariadb-mcp
```

## Installing without Node.js

Without Node.js, you can install a skill for an agent directly, e.g. for Claude Code like this:

```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -o ~/.claude/skills/<skill-name>/SKILL.md \
  https://raw.githubusercontent.com/mariadb/skills/main/<skill-name>/SKILL.md
```

Replace `<skill-name>` with the skill name, e.g. `mariadb-vector` or `mysql-to-mariadb`.

## About

These skills teach AI agents the MariaDB-specific knowledge they need to give correct advice — particularly where MariaDB differs from MySQL or PostgreSQL defaults.

Contributions and feedback welcome via issues.
