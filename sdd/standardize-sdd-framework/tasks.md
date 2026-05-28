# Tasks: Standardize MariaDB Skills

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | 500-800 |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Suggested split | PR 1 → PR 2 → PR 3 |
| Delivery strategy | ask-on-risk |
| Chain strategy | stacked-to-main |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: stacked-to-main
400-line budget risk: High

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|------|------|-----------|-------|
| 1 | Create shared versioning & Refactor 3 skills | PR 1 | Base branch: main |
| 2 | Refactor 3 skills | PR 2 | Base branch: PR 1 |
| 3 | Refactor remaining 2 skills | PR 3 | Base branch: PR 2 |

## Phase 1: Foundation (Infrastructure)

- [ ] 1.1 Create `skills/_shared/versioning.md` with standardized schema.

## Phase 2: Core Implementation (Refactoring)

- [ ] 2.1 Refactor `mariadb-features/SKILL.md` to standardized format.
- [ ] 2.2 Refactor `mariadb-vector/SKILL.md` to standardized format.
- [ ] 2.3 Refactor `mysql-to-mariadb/SKILL.md` to standardized format.
- [ ] 2.4 Refactor `oracle-to-mariadb/SKILL.md` to standardized format.
- [ ] 2.5 Refactor `mariadb-query-optimization/SKILL.md` to standardized format.
- [ ] 2.6 Refactor `mariadb-replication-and-ha/SKILL.md` to standardized format.
- [ ] 2.7 Refactor `mariadb-system-versioned-tables/SKILL.md` to standardized format.
- [ ] 2.8 Refactor `mariadb-mcp/SKILL.md` to standardized format.

## Phase 3: Verification

- [ ] 3.1 Verify all skills load correctly using `skill-registry` (if applicable).
- [ ] 3.2 Ensure consistent frontmatter and structure across all refactored files.
