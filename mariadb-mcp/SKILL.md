---
name: mariadb-mcp
description: "How to connect AI agents to MariaDB using the Model Context Protocol (MCP). Use when setting up MCP with MariaDB, connecting Claude Code, Cursor, or other AI tools to a MariaDB database, enabling natural language database queries, or building AI applications that need live database access. The MariaDB MCP Server lets agents query schemas, run read-only SQL, and perform semantic search against MariaDB."
---

# MariaDB MCP Server
## 💡 Quick Context
- **Baseline Version:** See [MariaDB Versioning Context](../_shared/versioning.md).
- **Requirements:** Python 3.11+, `uv` package manager, MariaDB server.

*Last updated: 2026-05-25*

The Model Context Protocol (MCP) lets AI agents connect to external tools and data sources. The MariaDB MCP Server gives agents direct, controlled access to a MariaDB database — reading schemas, running queries, and optionally performing vector/semantic search — without requiring the developer to write integration code.

## Documentation

- [Deep Dive Index]
  - [Overview](docs/overview.md)
  - [Installation](docs/installation.md)
  - [Connecting](docs/connecting.md)
  - [Vector & Semantic Search](docs/vector-search.md)
  - [Security](docs/security.md)
  - [Key Gotchas](docs/gotchas.md)
- [Sources](docs/sources.md)
