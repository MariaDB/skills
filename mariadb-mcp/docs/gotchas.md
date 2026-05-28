# Key Gotchas

- **`uv` is required**: the MCP server uses `uv` for dependency management, not `pip` directly. Install: `pip install uv` or `brew install uv`.
- **Read-only is not fully enforced in software**: rely on database-level privileges, not just `MCP_READ_ONLY=true`.
- **Vector tools are disabled** until `EMBEDDING_PROVIDER` is set in `.env`.
- **Connection pooling** defaults to 10 connections — tune `MCP_MAX_POOL_SIZE` for concurrent agent use.
- **SSL/TLS**: configure `DB_SSL_CA`, `DB_SSL_CERT`, `DB_SSL_KEY` in `.env` for remote or production databases.
- **Enterprise version**: MariaDB also offers a MariaDB Enterprise MCP Server with additional access control and multi-agent features. See [mariadb.com/products/mcp-server](https://mariadb.com/products/mcp-server/).
