# Design: Modularizing MariaDB Skill Documentation

## Technical Approach

Refactor root `SKILL.md` files for 8 MariaDB-related skills by adopting a `docs/` subdirectory architecture. This modular approach reduces the size of root `SKILL.md` files to under 100 lines, improving maintainability and readability by separating concerns into granular, linked documentation files.

## Architecture Decisions

### Decision: Modular Documentation Structure (`docs/` subdirectory)

**Choice**: Move detailed sections from `SKILL.md` into `docs/<topic>.md` files.
**Alternatives considered**: Maintaining a single large file, splitting into subdirectories without a standard convention.
**Rationale**: Root files were exceeding 200+ lines, making them difficult to navigate. A standardized `docs/` directory per skill provides better discoverability and enforces manageable content limits.

### Decision: PR-per-Skill Rollout

**Choice**: One PR per skill.
**Alternatives considered**: One massive PR for all 8 skills, or individual PRs per section.
**Rationale**: Smaller, focused PRs reduce risk, simplify reviews, and allow for easier rollbacks if documentation breaks.

## File Changes

| File | Action | Description |
|------|--------|-------------|
| `mariadb-*/docs/*.md` | Create | Modular documentation files for each skill |
| `mariadb-*/SKILL.md` | Modify | Strip detailed content, add links to `docs/` files |

## Testing Strategy

| Layer | What to Test | Approach |
|-------|-------------|----------|
| Unit | Root `SKILL.md` line count | Ensure `< 100` lines |
| Integration | Link integrity | Verify all links in `SKILL.md` correctly point to `docs/` files |
| E2E | Agent activation | Validate that `mariadb-features` (and others) are still recognized as valid skill directories |

## Migration / Rollout

Iterate through each of the 8 MariaDB-related skill folders sequentially.
1. Create `docs/` directory.
2. Refactor `SKILL.md` content into new `docs/` files.
3. Verify links and line counts.
4. Submit PR.
5. Proceed to next skill.

## Open Questions

- [ ] Confirm if all 8 skills need the exact same `docs/` structure, or if they should adapt to their specific needs.
