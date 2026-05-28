## Defaults Changed in 11.5–11.8 LTS

The current LTS (11.8) flipped several long-standing defaults. New installations behave differently from older ones — relevant when migrating or comparing behavior:

- **Default character set:  → ** (11.6+, MDEV-19123) — new tables use  unless overridden. Replication to MariaDB 10.6 or older replicas needs care (older replicas may not understand all  collations).
- **Default Unicode collation: ** (11.5+, MDEV-25829) — modern Unicode collation with proper SMP (supplementary multilingual plane) support including emoji. Replaces the older  default.
- ** deprecated and ignored** (11.5+, MDEV-33655) — specify  on the statement itself instead.
- **TIMESTAMP range extended** (11.5+ 64-bit, MDEV-32188) — upper bound raised from  to . Storage format unchanged; old servers can still read values within the old range.
- ** default ON** — see next section.
