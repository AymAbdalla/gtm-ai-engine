# Template skills

Full recreations of three production skills from the GTM AI Engine, with company-specific parameters removed and replaced by `[CONFIGURE]` blocks. The architecture — triggers, modes, gates, precedence declarations, checklists, hard rules — is exactly what runs in production. The numbers, verticals, customer references, and product specifics are not.

Where a value is needed to make the template readable, an **illustrative** value is used and marked as such. Illustrative values are teaching aids, not the production configuration.

| Template | Recreates | The pattern it demonstrates |
|---|---|---|
| [`account-scorer-template/`](account-scorer-template/SKILL.md) | The qualification engine | Live-research gating, two independent fit lenses, two visible axes with a verdict matrix, lookalike grounding, DQ as a first-class outcome |
| [`outreach-copy-router-template/`](outreach-copy-router-template/SKILL.md) | The copy engine | Universal craft law + persona router + supreme-law protocols with altitude ladders and checklist gates, grounded in published benchmark research |
| [`sequence-state-tracker-template/`](sequence-state-tracker-template/SKILL.md) | The sequence tracker | Rebuilding a blocked API as a memory-backed state machine with human-in-the-loop reconciliation (DRAFTED ≠ SENT) |

To adopt one: read [`docs/REPLICATION.md`](../docs/REPLICATION.md) first — especially the parameters-vs-invariants split. The `[CONFIGURE]` blocks are the parameters. Everything else is the invariant architecture; if you find yourself editing outside a `[CONFIGURE]` block, you're forking the system rather than configuring it.
