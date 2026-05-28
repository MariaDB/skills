# Proposal: Standardization of MariaDB SKILL.md Framework

## 1. Objective
Standardize the eight existing `SKILL.md` files to improve token efficiency, reduce LLM context noise, and ensure consistency across MariaDB versions (defaulting to 11.8 LTS).

## 2. Shared Boilerplate (`skills/_shared/versioning.md`)
To eliminate redundancy, all files will reference a shared snippet for versioning:
> **Baseline Version:** MariaDB 11.8 LTS (GA May 2025). 
> **Activation:** Automatic based on context.

## 3. Standardized Schema Structure
Every `SKILL.md` file will be refactored to this format:

```markdown
---
name: [skill-name]
description: "[One sentence summary]"
---
# [Title]

## 💡 Quick Context
- **Baseline Version:** [Reference shared/versioning.md]
- **Core Stance:** [E.g., "Vector support is native."]

## 🚫 Common LLM Hallucinations
| Prompt/Context | Correction |
|---|---|
| ... | ... |

## 🚀 Core Syntax & Best Practices
[Compact examples, bullet points over prose.]

## ⚠️ Gotchas
- [Actionable "Do / Don't" list]

## 📚 References
- [Minimal links]
```

## 4. Delivery Strategy (Chained PRs)
- **Constraint**: Each refactored skill will be a separate PR.
- **Budget**: If a single skill refactor exceeds 400 lines (unlikely given compression), it will be split.
- **Order**:
  1. `mariadb-features`
  2. `mariadb-vector`
  3. `mysql-to-mariadb`
  4. `oracle-to-mariadb`
  5. `mariadb-query-optimization`
  6. `mariadb-replication-and-ha`
  7. `mariadb-system-versioned-tables`
  8. `mariadb-mcp`
