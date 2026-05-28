# Tasks: Modularize Skill Documentation

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | 200-350 |
| 400-line budget risk | Low |
| Chained PRs recommended | No |
| Suggested split | single PR |
| Delivery strategy | single-pr |
| Chain strategy | size-exception |

Decision needed before apply: Yes
Chained PRs recommended: No
Chain strategy: size-exception
400-line budget risk: Low

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|------|------|-----------|-------|
| 1 | Migrate docs and refactor all skills | PR 1 | Base branch: main |

## Phase 1: Preparation
- [ ] 1.1 Create `mariadb-features/docs` directory
- [ ] 1.2 Create `mariadb-vector/docs` directory
- [ ] 1.3 Create `docs` directories for remaining 6 skills

## Phase 2: Content Migration
- [ ] 2.1 Migrate long-form content to `mariadb-features/docs`
- [ ] 2.2 Migrate content to `mariadb-vector/docs`
- [ ] 2.3 Migrate content to remaining 6 `docs` directories

## Phase 3: Refactoring
- [ ] 3.1 Refactor `mariadb-features/SKILL.md` (root) to reference new docs
- [ ] 3.2 Refactor `mariadb-vector/SKILL.md` (root) to reference new docs
- [ ] 3.3 Refactor `SKILL.md` for remaining 6 skills

## Phase 4: Cleanup & Verification
- [ ] 4.1 Verify root `SKILL.md` files are under 100 lines
- [ ] 4.2 Verify all moved docs are accessible and linked correctly
