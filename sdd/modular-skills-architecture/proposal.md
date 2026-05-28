# Proposal: Modularizing SKILL.md entries

## Intent
Improve maintainability and readability of 8 MariaDB skills documentation by adopting a `docs/` folder architecture. The current root `SKILL.md` files are becoming bloated, making it difficult to find critical information.

## Scope

### In Scope
- Create `docs/` subdirectory for each of the 8 MariaDB skills.
- Refactor existing `SKILL.md` content into granular files within `docs/`.
- Enforce a 100-line limit for the root `SKILL.md`.
- Ensure root `SKILL.md` contains clear links to `docs/` files.
- Execute via a PR-per-skill approach.

### Out of Scope
- Changing the functionality of the skills themselves.
- Updating documentation for other non-MariaDB skills.

## Capabilities

### New Capabilities
- `modular-docs-structure`: Standardizing the documentation layout across all skills.

### Modified Capabilities
- None

## Approach
We will iterate through each skill, moving detailed sections into `docs/<section>.md` files, leaving only the summary and links in `SKILL.md`.

## Affected Areas

| Area | Impact | Description |
|------|--------|-------------|
| `skills/mariadb-*/` | Modified | Add `docs/` folder, refactor `SKILL.md` |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Broken links | Medium | Verify all links after refactoring in each PR |

## Rollback Plan
Revert the PR for the specific skill if documentation becomes inaccessible or broken.

## Dependencies
- None

## Success Criteria
- [ ] Every MariaDB skill has a `docs/` folder.
- [ ] Every root `SKILL.md` is under 100 lines.
- [ ] All `SKILL.md` files link correctly to `docs/` files.
