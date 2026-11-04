## What it does

`expand-contract` sequences database schema migrations, breaking API updates, and high-blast-radius refactors into safe, zero-downtime phases.

The defining constraint is that it forbids executing a breaking change in a single step. Renaming a database column or dropping an API parameter in one deployment guarantees downtime or crashes when running against active traffic or rolling deployments. This skill enforces the Expand-Contract (parallel run) pattern, ensuring old and new versions run side-by-side until all callers have safely migrated.

## When to reach for it

Type `/expand-contract`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when designing or executing database schema alterations, breaking API changes, or wide refactors.

| Your situation | What to run |
| --- | --- |
| Altering a database schema, column, or table | `expand-contract` |
| Breaking a shared API contract or function interface | `expand-contract` |
| Slicing a large spec into tracer-bullet tickets | [to-tickets](https://aihero.dev/skills-to-tickets), which calls this skill for migrations |
| Evaluating merge danger on a pull request | [pr](https://aihero.dev/skills-pr), which reads risk classifications from this skill |

## The four migration phases

`expand-contract` breaks risky changes into four distinct, sequentially gated phases:

1. **Phase 1: Expand (Non-breaking Addition)**
   Add the new database column, table, or API endpoint alongside the existing one. Make new fields nullable or assign safe defaults. Never apply a `NOT NULL` constraint without a default in this phase. Begin dual-writing to both old and new paths.
2. **Phase 2: Backfill (Asynchronous Parity)**
   Migrate historical data into the new structure using small, bounded background batches. Verify 100% parity between legacy and new representations before proceeding.
3. **Phase 3: Switch (Guarded Cutover)**
   Redirect read queries and downstream consumers to the new structure. Guard the cutover with a feature flag or canary rollout so that any unexpected issue can be reversed instantly.
4. **Phase 4: Contract (Safe Retirement)**
   After a verified soak period with zero traffic hitting the old path, remove the old column or endpoint, apply permanent schema constraints, and delete the feature flag.

## One-way vs two-way doors

The skill classifies every migration phase as a **one-way door** or a **two-way door**:

- **Two-Way Door**: Reversible in seconds by toggling a feature flag or deploying a rollback commit. Phases 1, 2, and 3 are two-way doors because the legacy structure remains intact and synchronized via dual-writes.
- **One-Way Door**: Irreversible or expensive to undo once executed, such as dropping a database column or deleting historical records. One-way doors are restricted entirely to Phase 4 (Contract) and require explicit database backups and user sign-off.

## Common questions

**Does this mean I need four pull requests for one database change?**

Often yes, or at least four separate deployment steps. In continuous delivery environments, separating Expand, Backfill, Switch, and Contract ensures zero downtime and makes rollback trivial. Combining them into one PR creates a high-risk, all-or-nothing deployment.

**What if my application has only a few hundred users and can tolerate downtime?**

On very early prototypes with no production users, you can combine phases. However, for any system with active traffic, background workers, or rolling deployments, Expand-Contract is non-negotiable.

**How does this integrate with `/to-tickets`?**

`/to-tickets` uses this skill to create dedicated migration tickets with explicit blocking edges (`Blocked by`). The application logic ticket is blocked by the Expand ticket, and the Contract ticket is blocked by all consumer updates.

## It's working if

- New columns are added as nullable or with defaults before any code relies on them.
- Historical data is backfilled in bounded batches rather than one massive lock-inducing table rewrite.
- Read traffic switches to the new schema under feature flag control before legacy columns are dropped.
- Dropping legacy code occurs only in a final Contract ticket after zero callers remain.

## Where it fits

`expand-contract` sits at the intersection of ticket planning and deployment risk:

- [to-tickets](https://aihero.dev/skills-to-tickets) calls this skill to sequence migration tickets with proper dependency edges.
- [pr](https://aihero.dev/skills-pr) references this skill's risk assessment to document Merge Danger and rollback instructions.
- After a production incident caused by a rushed migration, [post-mortem](https://aihero.dev/skills-post-mortem) points to this skill as the preventive standard.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever database or API migrations are planned.
