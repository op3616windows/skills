---
name: post-mortem
description: "Lead a blameless post-mortem analysis after an incident or severe bug: establish timeline, run 5 Whys, identify detection gaps, record ADRs, and trigger retro."
disable-model-invocation: true
---

# Post-Mortem

When a production outage, data corruption incident, or critical regression occurs, fixing the immediate symptom is only half the job. If the root cause, detection gap, and architectural vulnerability are not formally addressed, the incident will recur.

This skill leads a blameless post-mortem analysis once an incident has been mitigated. It establishes the factual timeline, uncovers structural causes, documents lasting architectural decisions in `docs/adr/`, files preventive tickets on the issue tracker, and feeds improvements into `/retro`.

The issue tracker and domain docs configuration should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## Process

### 1. Gather Incident Facts
Assemble the raw evidence from the event:
- When was the bad change deployed or triggered?
- When did the first customer impact occur?
- How was the incident discovered (automated alert, customer escalation, internal report)?
- What mitigation or rollback stopped the bleeding?
- What was the overall user impact (affected users, error rates, degraded services, downtime duration)?

### 2. Build the Incident Timeline
Construct an exact chronological sequence using UTC timestamps:

- **T0 (Introduced)**: Bad commit merged, migration executed, or configuration toggled.
- **T1 (First Symptom)**: Errors first recorded in logs or metrics.
- **T2 (Detected)**: Pager fired or incident reported.
- **T3 (Acknowledged)**: Responders mobilized and incident room opened.
- **T4 (Diagnosed)**: Root cause or failing component identified.
- **T5 (Mitigated)**: Rollback completed, traffic redirected, or hotfix deployed; user impact ended.
- **T6 (Resolved)**: System fully stabilized and verified healthy.

Calculate the key operational metrics:
- Time to Detect (TTD = T2 - T1)
- Time to Mitigate (TTM = T5 - T2)

### 3. Five Whys & Gap Analysis
Conduct a blameless root cause investigation. Never stop at human error ("developer forgot a null check" or "reviewer missed a typo"). Ask why five times until structural, architectural, or tooling gaps are revealed:

1. Why did the service crash? (Uncaught null reference exception on user payload.)
2. Why was the reference null? (A newly introduced optional field was expected to be present.)
3. Why did the deploy assume it was present? (The migration and code were deployed in a single step instead of using the Expand-Contract pattern.)
4. Why wasn't this caught before production? (The staging integration test suite did not simulate mixed-version deployment states.)
5. Why wasn't that test required? (No automated guardrail or architectural standard checked for backward compatibility on migrations.)

Identify the two critical system gaps:
- **Detection Gap**: Why didn't automated tests, canary releases, or alerting fire before users were impacted?
- **Containment Gap**: Why did the blast radius spread to unrelated services or tenants? Why did mitigation take as long as it did?

### 4. Record ADRs & Prevention Tickets
Turn lessons learned into lasting assets:

1. **Architectural Decisions**: If the incident demonstrated that an existing design pattern is dangerous, record an Architectural Decision Record in `docs/adr/` (e.g. "ADR 0012: Enforce Expand-Contract for All Database Migrations").
2. **Preventive Action Items**: File concrete tickets on the configured tracker (`docs/agents/issue-tracker.md`) with explicit owners and priority:
   - Immediate preventative ticket: add the missing automated regression test or lint check.
   - Observability ticket: add SLI metric and alert threshold for the missing detection signal.
   - Containment ticket: isolate the failure boundary or improve rollback tooling.

### 5. Feed into `/retro`
Close the post-mortem by prompting the user:

"Run `/retro` to update our agent environment, `CODING_STANDARDS.md`, and automated checks so future code reviews catch this class of failure automatically."

## Post-Mortem Document Template

Synthesize the analysis into `docs/incidents/YYYY-MM-DD-<incident-slug>.md`:

```markdown
# Incident Post-Mortem: <Incident Title>

- **Date**: YYYY-MM-DD
- **Severity**: P1 / P2
- **Lead Investigator**: <Name>
- **Impact Duration**: <X> minutes

## Executive Summary
A 2-3 sentence overview of what broke, the customer impact, and how it was mitigated.

## User Impact
- Total requests affected / error rate spike.
- Customer accounts impacted.
- Revenue or business disruption.

## Incident Timeline (UTC)
- **YYYY-MM-DD HH:MM** - Bad deploy merged (`<commit-hash>`).
- **YYYY-MM-DD HH:MM** - First customer error recorded.
- **YYYY-MM-DD HH:MM** - Alert `<alert-name>` fired.
- **YYYY-MM-DD HH:MM** - Rollback initiated.
- **YYYY-MM-DD HH:MM** - Impact mitigated.

## Root Cause & Five Whys
Detailed causal chain explaining why the failure occurred structurally.

## Gaps Identified
- **Detection Gap**: <Why alerts or tests failed to catch it earlier>
- **Containment Gap**: <Why blast radius or rollback was slower than needed>

## Action Items
| Action Item | Type | Tracker Issue | Priority |
| --- | --- | --- | --- |
| Add regression integration test | Prevention | #<issue_id> | P1 |
| Add P99 latency alert on payment queue | Detection | #<issue_id> | P2 |
| Adopt Expand-Contract ADR | Process | #<issue_id> | P2 |
```
