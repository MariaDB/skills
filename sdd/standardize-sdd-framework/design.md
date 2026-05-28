# Design: Standardization of MariaDB SKILL.md Framework

## Technical Approach

We will standardize all eight MariaDB-related `SKILL.md` files by refactoring them into a unified Markdown schema. This reduces context noise and enforces consistency. We will implement shared versioning via a new `skills/_shared/versioning.md` file, which all other `SKILL.md` files will reference. We will follow a PR-per-skill strategy, starting with `mariadb-features` as a blueprint.

## Architecture Decisions

### Decision: Centralized Versioning Boilerplate

**Choice**: Create `skills/_shared/versioning.md` and reference it from all other skills.
**Alternatives considered**: Hardcoding versioning in each file, or using a complex templating system.
**Rationale**: Hardcoding is redundant and error-prone during upgrades (e.g., when moving past 11.8 LTS). A shared file ensures single-point-of-truth.

### Decision: PR-per-skill Workflow

**Choice**: One pull request per skill refactor.
**Alternatives considered**: A single, massive PR for all skills.
**Rationale**: Smaller, focused PRs are easier to review, verify, and revert if necessary. It adheres to the 400-line budget per PR.

## Data Flow

    [Shared Versioning] <──┐
                           │
    [Skill A/SKILL.md] ────┤ (Refactor to Schema)
    [Skill B/SKILL.md] ────┤
    [.../SKILL.md] ────────┘

## File Changes

| File | Action | Description |
|------|--------|-------------|
| `skills/_shared/versioning.md` | Create | Contains the standardized versioning snippet (MariaDB 11.8 LTS). |
| `skills/mariadb-*/SKILL.md` | Modify | Refactor existing skill files to the new standardized schema. |

## Interfaces / Contracts

The standardized schema:
```markdown
---
name: [skill-name]
description: "[Summary]"
---
# [Title]
## 💡 Quick Context
- **Baseline Version:** [Reference shared/versioning.md]
...
```

## Testing Strategy

| Layer | What to Test | Approach |
|-------|-------------|----------|
| Unit | Structural integrity | Validate schema compliance using a script or manual inspection against spec #139. |
| Integration | Accuracy of content | Compare refactored content against original `SKILL.md` to ensure no loss of critical information. |

## Migration / Rollout

The `mariadb-features` skill will serve as the initial blueprint to validate the schema and refactoring process. Once validated, we will proceed with the remaining skills in the order specified in the proposal.

## Open Questions

- [ ] None.
