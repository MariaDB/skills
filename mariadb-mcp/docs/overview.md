# MariaDB MCP Server: Overview

## What Agents Can Do

With the MariaDB MCP Server connected, an agent can:

- List databases and tables
- Read table schemas including foreign key relationships
- Run read-only SQL queries (`SELECT`, `SHOW`, `DESCRIBE`)
- Create databases
- Perform semantic/vector search on stored documents (when embedding provider is configured)

Write operations (`INSERT`, `UPDATE`, `DELETE`) are disabled by default.

## Available Tools

| Tool | What it does |
|---|---|
| `list_databases` | Lists all databases the user can access |
| `list_tables` | Lists tables in a database |
| `get_table_schema` | Returns column definitions and types |
| `get_table_schema_with_relations` | Includes foreign key relationships |
| `execute_sql` | Runs a read-only SQL query |
| `create_database` | Creates a new database |
