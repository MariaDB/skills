## Security: Read-Only Access

The `MCP_READ_ONLY=true` setting enforces read-only at the application level by filtering queries. For production or shared environments, also restrict at the database level:

```sql
CREATE USER 'mcp_agent'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, SHOW DATABASES ON *.* TO 'mcp_agent'@'localhost';
-- Do NOT grant INSERT, UPDATE, DELETE, DROP
```

Application-level read-only alone is not sufficient — database-level privileges are the reliable guarantee.
