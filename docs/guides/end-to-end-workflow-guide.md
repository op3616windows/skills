# End-to-End Product Lifecycle Workflow Guide

> [Bản Tiếng Việt](end-to-end-workflow-guide.vi.md)

This guide walks through a complete, production-grade feature build from customer discovery to incident prevention using the full suite of modular engineering and product lifecycle skills.

It uses a real-world SaaS feature: **Self-Serve Organization Data Export with Asynchronous Processing, Zero-Downtime Migration, and Multi-Tenant Isolation**.

---

## Workflow Overview

The product lifecycle flows across nine disciplined stages:

```
[1. /shape-problem] ──> [2. /grill-with-docs] ──> [3. /to-spec (+ ux-states, telemetry-design)]
                                                                  │
                                                                  ▼
[6. /qa-session] <── [5. /implement (+ /tdd)] <── [4. /to-tickets (+ expand-contract)]
        │
        ▼
[7. /code-review (+ security-review)] ──> [8. /pr & /release-notes] ──> [9. /post-mortem & /retro]
```

---

## Stage 1: Problem Discovery (`/shape-problem`)

### Context
Customers are flooding support tickets because they cannot independently extract historical invoices for quarterly accounting audits. Support engineers run ad-hoc database queries manually, which risks sending confidential data to the wrong recipient and wastes engineering time.

### Developer Prompt
```text
/shape-problem
Enterprise customers are submitting numerous support tickets complaining that they cannot export invoice and transaction records to CSV for quarter-end accounting. Customer support currently asks backend developers to run raw SQL queries. Frame this problem cleanly before we discuss any technical solutions.
```

### Under the Hood
1. Runs `grilling` rounds focused entirely on the problem space: persona, Job to Be Done (JTBD), current workarounds, measurable outcomes, and explicit non-goals.
2. Refuses to discuss implementation details (Redis, background queues, S3 buckets).
3. Invokes `domain-modeling` to update `GLOSSARY.md` with terms like `ExportJob` and `TenantScope`.

### Concrete Artifact Output
```markdown
# Problem Frame: Self-Serve Organization Data Export

## Problem Statement
Finance managers cannot independently extract their organization's historical transaction records for quarterly audits, forcing manual intervention from customer support and creating compliance delays.

## Target User & Context
- Primary Persona: Organization Billing Admins and Finance Leads.
- Context of Struggle: End-of-month and end-of-quarter accounting reconciliation.

## Job To Be Done (JTBD)
When closing monthly books, I want to download all organization invoice data in structured formats (CSV/JSON), so I can reconcile financial records without contacting support.

## Current Friction & Workarounds
Support engineers manually run ad-hoc SQL queries and email CSV files, introducing risk of sending data to the wrong recipient and wasting engineering hours.

## Measurable Success Indicators
- Reduction in support export requests to zero per week.
- 95% of export files generated and available for download within 60 seconds.

## Explicit Non-Goals
- Scheduled recurring automated exports (deferred to future milestone).
- Direct integration with third-party ERPs such as SAP or NetSuite.
```

---

## Stage 2: Technical Architecture & ADR (`/grill-with-docs`)

### Context
With the problem framed, the team decides technical architecture, persistence, and external dependencies.

### Developer Prompt
```text
/grill-with-docs
Based on the approved Problem Frame, interview me on the technical architecture, background processing, storage, and rate-limiting strategy for this repository.
```

### Under the Hood
1. Runs structured technical interview rounds.
2. Recommends asynchronous queue processing rather than long-running HTTP connections.
3. Decides on temporary presigned object storage URLs with 15-minute expiration.
4. Writes Architectural Decision Record `docs/adr/0014-async-export-processing.md`.

---

## Stage 3: Specification with UX & Telemetry (`/to-spec`)

### Context
Synthesizes the conversation into an issue tracker specification without re-interviewing.

### Developer Prompt
```text
/to-spec
Synthesize all decisions into a comprehensive specification and publish it to the configured issue tracker with the ready-for-agent label.
```

### Under the Hood
1. Recognizes user-facing UI components and invokes **`ux-states`** to generate the UI State Matrix.
2. Invokes **`telemetry-design`** to define behavioral analytics events, structured logging context, system metrics, and alerting thresholds.

### Concrete Artifact Output (Spec Excerpt)
```markdown
## UI State Matrix (via ux-states)

| State Dimension | Visual & Interaction Behavior | Technical Invariant |
| --- | --- | --- |
| **Empty (Zero Records)** | Modal states "No records match selected date range" with a reset filter button | `transactions.length === 0` |
| **Loading (Queued)** | Skeleton placeholder and progress indicator "Preparing export..." | `job.status === "QUEUED"` |
| **Optimistic Action** | Button instantly disables and transitions to "Export Requested" | Debounce handler prevents duplicate job dispatch |
| **Error Recovery** | Warning banner displays "Generation timed out. Retry?" with inline retry button | Catch block exposes single-click retry action |
| **Accessibility (a11y)** | Focus trapped in modal; status announced via `aria-live="polite"` | WCAG AA compliance |

## Telemetry & Observability (via telemetry-design)

- Behavioral Event: `export_requested` with `{ format: "csv" | "json", date_range_days: number }`
- Behavioral Event: `export_downloaded` with `{ export_id: string, file_size_bytes: number }`
- Structured Log: `INFO export.started` with `{ export_id, tenant_id, user_id }`
- Structured Log: `ERROR export.failed` with `{ export_id, tenant_id, error_code, stack_trace }`
- Metric: `exports_processing_seconds` (Histogram, P95 target < 60s)
- Alert: Trigger P2 alert when `rate(export_failed[5m]) / rate(export_started[5m]) > 0.05`
```

---

## Stage 4: Ticket Breakdown & Zero-Downtime Migration (`/to-tickets`)

### Context
The spec must be decomposed into tracer-bullet tickets. Because database schema changes and background workers are introduced, zero-downtime safety is required.

### Developer Prompt
```text
/to-tickets
Decompose the spec into tracer-bullet tickets. Apply expand-contract for the database migration and API additions.
```

### Under the Hood
1. Decomposes work into sequential tickets with explicit blocking edges.
2. Invokes **`expand-contract`** to sequence database schema changes across Expand, Backfill, Switch, and Contract phases.
3. Classifies each ticket as a Two-Way Door (reversible) or One-Way Door (irreversible).

### Concrete Artifact Output
```markdown
### Ticket Breakdown:
1. **Ticket #101 [Expand]**: Add `export_jobs` table and nullable `last_exported_at` column to `invoices`.
   - Risk: Two-way door (Safe rollback via down migration)
   - Blocked by: None
2. **Ticket #102 [Core Logic & Worker]**: Implement asynchronous export worker and S3 upload seam using TDD.
   - Blocked by: #101
3. **Ticket #103 [UI & State Matrix]**: Build Export Modal adhering to the UX State Matrix.
   - Blocked by: #102
4. **Ticket #104 [Telemetry & Alerts]**: Wire up behavioral events, logging context, and metric probes.
   - Blocked by: #102
5. **Ticket #105 [Contract]**: Add strict foreign key constraints and index on `export_jobs.tenant_id`.
   - Risk: One-way door (Requires database snapshot before execution)
   - Blocked by: #101, #102, #103
```

---

## Stage 5: Test-Driven Implementation (`/implement` with `/tdd`)

### Context
Building Ticket #102 (Worker and S3 storage seam).

### Developer Prompt
```text
/implement
Pick up Ticket #102. Drive implementation with strict TDD, testing behavior at the highest seam and asserting telemetry emissions.
```

### Under the Hood
1. Writes a failing integration test (`test/export-worker.test.ts`) that exercises job execution, file streaming, and S3 presigned URL generation.
2. Asserts that telemetry and structured logger are called with exact metadata (including `tenant_id`).
3. Writes minimal implementation code to turn the test green.
4. Refactors to eliminate code smells while keeping tests passing.

---

## Stage 6: Interactive Acceptance QA (`/qa-session`)

### Context
Implementation is complete and automated unit tests pass. The local server is running at `http://localhost:3000`.

### Developer Prompt
```text
/qa-session
I have finished implementing the export feature on branch feat/data-export. Walk me through an interactive QA session against the spec.
```

### Under the Hood
1. Loads the spec acceptance criteria and constructs a 3-tier test matrix: Core Happy Path, Edge Cases, and Failure Modes.
2. Guides the developer through interactive execution steps.
3. **Core Discipline:** Does NOT modify code when an anomaly is observed; instead, logs a standardized defect report to the issue tracker.

### Concrete Artifact Output
```markdown
# Bug: Rapid Multi-Click Triggers Duplicate Export Jobs

## Environment
- Branch: `feat/data-export` (Commit: `8a9f3b1`)
- Browser: Chrome 130 / macOS

## Steps to Reproduce
1. Open Billing > Export Data dialog.
2. Rapidly click the "Start Export" button 5 times in under 1 second.

## Expected Behavior
Button disables immediately on first click, queuing exactly 1 job.

## Actual Behavior
Five duplicate jobs were inserted into `export_jobs`, causing redundant worker load.

## Severity
- **P2 - Major**: Redundant processing load on background queue.
```

---

## Stage 7: Tri-Axis Code Review (`/code-review`)

### Context
Reviewing the branch diff before opening the pull request.

### Developer Prompt
```text
/code-review main
Review the diff between main and HEAD.
```

### Under the Hood
Spawns three sub-agents in parallel with fresh context windows:
1. **Standards Sub-agent**: Checks repo standards and Fowler smell baseline.
2. **Spec Sub-agent**: Checks missing requirements or scope creep.
3. **Security Sub-agent (calling `security-review`)**: Audits the diff against the five security pillars (AuthZ, PII, Injection, Network, Dependencies).

### Concrete Artifact Output (Security Excerpt)
```markdown
## Security & Privacy Findings (via security-review)

### [CRITICAL] Missing Tenant Scope in Export Download Endpoint
- **Location**: `src/api/routes/exports.ts:64-72`
- **Vulnerability**: Insecure Direct Object Reference (AuthZ)
- **Attack Scenario**: An authenticated user in Tenant A can pass an `exportId` belonging to Tenant B and receive a valid presigned S3 URL, exposing competitor invoices.
- **Remediation**:
  ```ts
  const job = await db.exportJob.findFirst({
    where: { id: params.exportId, tenantId: session.tenantId }
  });
  if (!job) throw new NotFoundError();
  ```
```

---

## Stage 8: Pull Request & Customer Release Notes (`/pr` & `/release-notes`)

### 1. Opening the Pull Request
The agent uses `/pr` to assess migration risk via `expand-contract`:
```markdown
## Summary
Enables self-serve CSV and JSON data export for organization billing admins with asynchronous worker processing.

## Merge Danger & Migration Safety
- **Phase**: Expand (Phase 1)
- **Door Classification**: Two-Way Door
- **Rollback Strategy**: Can be reverted immediately via feature flag `ff_self_serve_export = false` without data loss.
```

### 2. Synthesizing Customer Release Notes
After merging the PR into main:

### Developer Prompt
```text
/release-notes
Branch feat/data-export was merged into main. Synthesize merged PRs and commits into customer-facing release notes.
```

### Concrete Artifact Output
```markdown
# Release Notes: Version 2.8.0

## Highlights
Organizations can now independently export their complete transaction and invoice history directly from their billing dashboard, eliminating wait times for accounting audits.

## New Features
- **Self-Serve Data Export**: Billing administrators can download CSV and JSON summaries of all historical transactions with one click from Settings > Billing > Export.

## Improvements
- Background generation ensures large data requests process reliably without slowing down your active dashboard session.

## Bug Fixes
- Resolved an issue where filtered transaction tables did not persist date selections across page refreshes.
```

---

## Stage 9: Operational Incident & Prevention (`/post-mortem` & `/retro`)

### Context
In production, a tenant with 500,000 records triggered an export that caused worker Out-Of-Memory (OOM) crashes. The incident was mitigated by switching the worker to database streaming cursors.

### Developer Prompt
```text
/post-mortem
The export worker OOM incident has been mitigated. Guide me through a blameless post-mortem, construct the timeline, run 5 Whys, and record lasting prevention assets.
```

### Under the Hood
1. Computes Time to Detect (TTD = 18m) and Time to Mitigate (TTM = 25m).
2. Runs blameless 5 Whys uncovering structural root causes (loading all rows into memory instead of streaming chunks).
3. Records Architectural Decision Record `docs/adr/0015-stream-large-database-exports.md`.
4. Files preventive issue tracker tickets.
5. Prompts the developer to run `/retro` to update `CODING_STANDARDS.md` and add automated linter checks.

---

## Summary of Benefits

1. **Zero Wasted Development**: `shape-problem` prevents building the wrong capability before code is touched.
2. **Predictable User Experience**: `ux-states` guarantees all network, loading, and recovery states are specified.
3. **Safe Deployments**: `expand-contract` eliminates database locks and breaking API cutovers.
4. **Data Isolation**: `security-review` catches multi-tenant leaks and IDOR vulnerabilities before merge.
5. **Continuous Learning**: `post-mortem` and `retro` transform production incidents into automated guardrails.
