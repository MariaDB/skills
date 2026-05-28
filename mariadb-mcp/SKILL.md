---
name: mariadb-mcp
description: "How to connect AI agents to MariaDB using the Model Context Protocol (MCP). Use when setting up MCP with MariaDB, connecting Claude Code, Cursor, or other AI tools to a MariaDB database, enabling natural language database queries, or building AI applications that need live database access. The MariaDB MCP Server lets agents query schemas, run read-only SQL, and perform semantic search against MariaDB."
---

# MariaDB MCP Server

The Model Context Protocol (MCP) lets AI agents connect to external tools and data sources. The MariaDB MCP Server gives agents direct, controlled access to a MariaDB database — reading schemas, running queries, and optionally performing vector/semantic search — without requiring the developer to write integration code.

The server is open source (MIT), maintained by the MariaDB Foundation at [github.com/MariaDB/mcp](https://github.com/MariaDB/mcp).

> **Requires:** Python 3.11+, `uv` package manager, MariaDB server.

## Deep Dive Index

- [What Agents Can Do](./docs/agents-can-do.md)
- [Installation](./docs/installation.md)
- [Connecting Claude Code, Cursor, or Windsurf](./docs/connecting.md)
- [Available Tools](./docs/available-tools.md)
- [Vector & Semantic Search](./docs/vector-search.md)
- [Security: Read-Only Access](./docs/security.md)
- [Key Gotchas](./docs/key-gotchas.md)
- [Sources](./docs/sources.md)

*For topics not covered here, see the official MariaDB documentation at [mariadb.com/docs](https://mariadb.com/docs).*
