# Data Type Mapping

`sql_mode=ORACLE` handles these automatically:

| Oracle | MariaDB | Notes |
|---|---|---|
| `VARCHAR2(n)` | `VARCHAR(n)` | Automatic |
| `NUMBER(p,s)` | `DECIMAL(p,s)` | Automatic |
| `NUMBER` | `DOUBLE` | Automatic |
| `DATE` | `DATETIME` | **Automatic, but note:** Oracle DATE includes time; MariaDB DATE does not |
| `CLOB` | `LONGTEXT` | Automatic |
| `BLOB` | `LONGBLOB` | Automatic |
| `RAW(n)` | `VARBINARY(n)` | Automatic |
| `CHAR(n > 255)` | `VARCHAR(n)` | Manual |
| `TIMESTAMP WITH TIME ZONE` | `DATETIME` | Manual — timezone info lost |
| `BFILE` | `LONGBLOB` | Manual — file path storage differs |
| `ROWID` | `CHAR(10)` | Manual |
