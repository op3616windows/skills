---
name: expand-contract
description: "Plan and execute zero-downtime database schema migrations, breaking API changes, or wide refactors using the Expand-Contract pattern. Use when modifying database schemas, altering API contracts, or refactoring high-blast-radius code."
---

# Expand Contract

When modifying a database schema, public API contract, or shared core module, making the change in a single step causes downtime, runtime crashes, or massive merge conflicts.

This skill sequences changes using the **Expand-Contract** (or Parallel Run) pattern across four disciplined phases, ensuring old and new versions coexist safely during transitions.

## The Four Migration Phases

### Phase 1: Expand (Non-breaking Addition)
Add the new capability or schema element alongside the old one without breaking existing callers or data.

- **Database**: Add the new column or table as nullable or with a safe default. Never add a `NOT NULL` constraint without a default in this phase. Never drop or rename existing columns.
- **Application Code**: Begin dual-writing to both the old and new columns or endpoints if both must stay updated.
- **API**: Add new fields or endpoints. Keep old fields and endpoints fully functional.
- **Blast Radius**: Zero. Old clients and queries run completely unchanged.

### Phase 2: Backfill (Asynchronous Data Migration)
Populate the new structure with historical data while the application runs normally.

- Execute backfill jobs in small, bounded batches (e.g. 500 records per transaction) to prevent database table locking or replication lag.
- Write scripts to verify data parity between old and new structures.
- Once backfill completes, verify that 100% of rows have valid data in the new structure.

### Phase 3: Switch (Controlled Traffic Cutover)
Redirect reads and business logic to use the new structure, guarded by runtime controls.

- **Feature Flags**: Wrap the switch behind a dynamic feature flag or canary deployment.
- **Incremental Rollout**: Test the switch internally, then expand to 5%, 25%, and 100% of production traffic.
- **Rollback Readiness**: If unexpected anomalies occur, toggle the feature flag back. Dual-writing from Phase 1 guarantees the old read path is still valid and up to date.

### Phase 4: Contract (Safe Removal of Legacy Code)
Once all readers and writers have successfully operated on the new structure for a soak period with zero traffic hitting the old path:

- Stop dual-writing.
- Add database constraints (e.g. `NOT NULL`, foreign keys, indexes) if required.
- Remove old database columns, deprecated API routes, legacy functions, and temporary feature flags.
- Drop temporary fallback code.

## Risk Classification: One-Way vs Two-Way Doors

Evaluate every phase against the door test:

- **Two-Way Door**: Reversible in seconds by toggling a flag or deploying a rollback commit (e.g. Phase 1 Expand, Phase 3 Switch with dual-write).
- **One-Way Door**: Irreversible or expensive to undo once executed (e.g. dropping a column, deleting historical data, dropping an API endpoint).
  - One-Way Doors must be scheduled strictly in Phase 4 (Contract).
  - Every One-Way Door ticket requires an explicit backup snapshot and user sign-off before running.

## Generating Migration Tickets for `/to-tickets`

When called during ticket breakdown, decompose the work into sequential tickets with explicit blocking edges:

```markdown
### Migration Plan: <Change Name>

1. **Ticket 1 [Expand]**: Add `<new_field_or_table>` to schema with default/nullability. Deploy dual-write logic in repository layer.
   - Blocked by: None (can start immediately)
   - Risk: Two-way door

2. **Ticket 2 [Backfill]**: Run batch backfill script to populate historical records. Verify parity across all environments.
   - Blocked by: Ticket 1 [Expand]
   - Risk: Two-way door

3. **Ticket 3 [Switch]**: Flip reads and downstream consumers to `<new_field_or_table>`, guarded by feature flag `ff_<feature_name>`.
   - Blocked by: Ticket 2 [Backfill]
   - Risk: Two-way door

4. **Ticket 4 [Contract]**: Remove old column/endpoints, remove feature flag branches, apply final schema constraints.
   - Blocked by: Ticket 3 [Switch]
   - Risk: One-way door (requires database backup confirmation)
```

## Supplying Risk Assessment to `/pr`

When a PR touches a schema or API contract, summarize:
- **Phase**: Expand, Backfill, Switch, or Contract.
- **Merge Danger**: State whether this PR is a One-Way Door or Two-Way Door.
- **Rollback Strategy**: What exact command or flag toggle rolls back the change without data loss.
