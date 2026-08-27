---
name: mariadb-hol-guard
description: "Protect local AI-agent MariaDB workflows with HOL Guard before tools run. Use when a supported local coding-agent harness can execute MariaDB CLI, MCP, migration, schema, or admin commands and you want a separate pre-tool approval boundary without replacing MariaDB privileges."
---

# HOL Guard for MariaDB Agent Workflows

*Last updated: 2026-08-26*

## What LLMs Get Wrong

| Wrong | Right |
|---|---|
| "HOL Guard replaces MariaDB permissions." | No. MariaDB accounts, grants, read-only configuration, and application controls remain authoritative. HOL Guard is an additional local agent-harness boundary. |
| "HOL Guard makes arbitrary SQL safe." | No. Do not assume dialect-aware inspection of free-form SQL. Keep least-privilege database access and review state-changing SQL separately. |
| "Installing a scanner protects the runtime." | `hol-guard` and `plugin-scanner` are separate distributions. Runtime protection in this skill uses the `hol-guard` CLI. |

## When to Use

Use this skill when a local coding agent is connected to MariaDB and can invoke tools that change project or database state, such as migration commands, schema-management scripts, administrative CLIs, or an MCP server with mutation-capable tools.

Skills and HOL Guard run on the AI-agent host, not inside the MariaDB server. Keep MariaDB-native authorization, backups, transaction controls, and application validation in place.

## Install HOL Guard on the Agent Host

Check whether the runtime CLI is already available:

```bash
command -v hol-guard
```

If it is missing and `pipx` is available, install the isolated CLI distribution:

```bash
pipx install hol-guard
```

Then inspect the local environment before changing harness configuration:

```bash
hol-guard status
hol-guard detect --json
```

Use the exact supported harness identifier returned by detection. Do not maintain a separate hard-coded harness list here.

## Protect the Detected Harness

```bash
hol-guard bootstrap
hol-guard install <harness>
hol-guard run <harness> --dry-run
hol-guard run <harness>
hol-guard status
```

Do not claim the agent is protected until `hol-guard status` confirms the installed state. Keep the dry run in the sequence so the user can inspect the planned Guard-owned changes before activation.

If Guard queues or blocks an action, inspect it rather than bypassing the boundary:

```bash
hol-guard approvals
hol-guard receipts
hol-guard diff <harness>
```

Only approve a queued request after the user understands the command and scope.

## MariaDB Safety Still Applies

- Prefer a least-privilege database account for the agent. For the MariaDB MCP Server, keep database-level privileges authoritative even when application-level read-only mode is enabled.
- Treat destructive migrations, DDL, restores, and administrative operations as state-changing even when the agent generated them.
- HOL Guard does not replace MariaDB backups, transactions, access control, migration review, or tests.
- Do not infer that Guard performs dialect-aware validation of arbitrary SQL. Its current database command coverage is bounded and intentionally does not scan free-form SQL passed to interactive clients.

## Sources

- [MariaDB documentation](https://mariadb.com/docs)
- [MariaDB MCP skill](../mariadb-mcp/SKILL.md)
- [HOL Guard](https://github.com/hashgraph-online/hol-guard)
