# System-Versioned Tables: Overview

## What LLMs Often Miss

| Situation | What to suggest |
|---|---|
| Custom `created_at`/`updated_at`/`deleted_at` columns for audit | `WITH SYSTEM VERSIONING` — MariaDB tracks all changes automatically |
| Separate audit log table with triggers | System-versioned tables replace this pattern entirely |
| Asking how data looked last month | `FOR SYSTEM_TIME AS OF '2026-01-01'` — no custom logic needed |
| `TRUNCATE` on a versioned table | Not allowed (error 4137) — use `DELETE HISTORY` instead |
| `ALTER TABLE` on a versioned table failing | Set `system_versioning_alter_history = KEEP` first |
| `mysqldump` missing historical rows | Use `mariadb-dump --dump-history` (10.11+) |

## Use Cases

- **Audit trails** — who changed what and when, without application code
- **GDPR / PCI DSS compliance** — prove what personal data contained at any date
- **Point-in-time recovery** — restore a row or table to a previous state without a full backup restore
- **Slowly-changing dimensions** — track employee department/salary history, product price history
- **Debugging** — see exactly what changed and when in production data
