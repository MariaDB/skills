# MariaDB Agent Skills

Skills for AI agents working with MariaDB.

> Open for improvements and more skills. Contributions welcome.

## Skills

### mariadb-vector

Best practices for using MariaDB's built-in vector support for AI and semantic search. Covers RAG patterns, vector indexes, distance functions, embedding model selection, and framework integrations (LangChain, LlamaIndex, Spring AI). Unlike MySQL (no vector support) or PostgreSQL (which requires pgvector), MariaDB has native vector support since version 11.7 — no extensions or plugins needed.

```
npx skills add mariadb/skills/mariadb-vector
```

### mysql-to-mariadb

Migration guide for developers moving from MySQL to MariaDB, or where MySQL habits cause unexpected behavior. Covers authentication differences, JSON handling, features MySQL has that MariaDB doesn't, and features MariaDB has that MySQL doesn't — including things LLMs won't suggest because they assume MySQL behavior.

```
npx skills add mariadb/skills/mysql-to-mariadb
```

### mariadb-features

MariaDB-specific features worth knowing about — the things AI agents don't suggest because they default to generic SQL advice. Covers system-versioned tables, RETURNING, sequences, instant DDL, native IP types, Oracle compatibility mode, Galera, ColumnStore, and more.

```
npx skills add mariadb/skills/mariadb-features
```

## Installing without Node.js

To install any skill globally in Claude Code without npx:

```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -o ~/.claude/skills/<skill-name>/SKILL.md \
  https://raw.githubusercontent.com/mariadb/skills/main/<skill-name>/SKILL.md
```

Replace `<skill-name>` with `mariadb-vector`, `mysql-to-mariadb`, or `mariadb-features`.

## About

These skills teach AI agents the MariaDB-specific knowledge they need to give correct advice — particularly where MariaDB differs from MySQL or PostgreSQL defaults.

Contributions and feedback welcome via issues.
