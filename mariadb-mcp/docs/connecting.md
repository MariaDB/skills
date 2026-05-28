## Connecting Claude Code, Cursor, or Windsurf

Use the official server from a local clone (see Installation). Point your MCP config at `uv run server.py` and the `.env` file with database credentials.

**Claude Code** (project-scoped `.mcp.json` at the repo root):

```json
{
  "mcpServers": {
    "mariadb": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/mcp",
        "run",
        "server.py"
      ],
      "envFile": "/absolute/path/to/mcp/.env"
    }
  }
}
```

Or add via CLI (replace paths):

```bash
claude mcp add mariadb -- uv --directory /absolute/path/to/mcp run server.py
```

**Cursor / VS Code** — same `mcpServers` block in MCP settings; use absolute paths.

**SSE or HTTP** — run `uv run server.py --transport sse` (or `http`) and connect to the URL; see [MariaDB/mcp README — Integration](https://github.com/MariaDB/mcp#integration---claude-desktopcursorwindsurfvscode).

> **Not official:** PyPI package `mcp-server-mariadb` (`uvx mcp-server-mariadb`) is a third-party project, not maintained by the MariaDB Foundation. Prefer `github.com/MariaDB/mcp` unless you have a specific reason to use the PyPI package.
