# Installation

```bash
git clone https://github.com/MariaDB/mcp
cd mcp
cp .env.example .env
# Edit .env with your database credentials
```

**Minimum `.env` configuration:**
```
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=myuser
DB_PASSWORD=mypassword
DB_NAME=mydatabase
MCP_READ_ONLY=true
```

**Run the server:**
```bash
uv run server.py
```
