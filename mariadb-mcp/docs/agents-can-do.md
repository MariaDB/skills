## What Agents Can Do

With the MariaDB MCP Server connected, an agent can:

- List databases and tables
- Read table schemas including foreign key relationships
- Run read-only SQL queries (`SELECT`, `SHOW`, `DESCRIBE`)
- Create databases
- Perform semantic/vector search on stored documents (when embedding provider is configured)

Write operations (`INSERT`, `UPDATE`, `DELETE`) are disabled by default.
