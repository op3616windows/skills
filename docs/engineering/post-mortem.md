## What it does

`post-mortem` leads a blameless operational post-mortem analysis following a production outage, data incident, or critical bug.

The defining constraint is that it refuses to stop at human error. Blaming an engineer for a typo or missed edge case fixes nothing. This skill uncovers the structural, architectural, and tooling reasons why the failure was possible, why automated tests failed to catch it, and why alerting did not detect it sooner. It converts lessons into lasting Architectural Decision Records in `docs/adr/`, files preventative tickets, and triggers `/retro` to update coding standards.

## When to reach for it

You invoke this by typing `/post-mortem`, and the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

Reach for this immediately after mitigating a high-severity production incident or unpredicted regression:

| Your situation | What to run |
| --- | --- |
| Production incident is mitigated and you need to investigate root cause | `/post-mortem` |
| An active bug needs disciplined diagnosis and a failing reproduction test | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| Routine retro after a normal engineering session | [retro](https://aihero.dev/skills-retro) |
| Systematically reviewing codebase design and deep modules | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |

## Prerequisites

The skill files follow-up action items to the issue tracker and records architectural decisions, so [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) must have configured `docs/agents/issue-tracker.md` and `docs/agents/domain.md`.

## The incident timeline and metrics

`post-mortem` reconstructs an exact chronological sequence using UTC timestamps:
- **T0 (Introduced)**: The bad commit or migration was deployed.
- **T1 (First Symptom)**: The first error or degraded behavior occurred.
- **T2 (Detected)**: Monitoring alerts fired or a user reported the issue.
- **T3 (Acknowledged)**: Responders mobilized.
- **T4 (Diagnosed)**: Root cause was identified.
- **T5 (Mitigated)**: Rollback, hotfix, or traffic switch ended user impact.

From this timeline, it calculates critical operational indicators:
- **Time to Detect (TTD)**: Time between first symptom and alert acknowledgment.
- **Time to Mitigate (TTM)**: Time between detection and impact cessation.

## Five Whys and gap analysis

The investigation drives deep causal analysis using the Five Whys technique to identify two systemic gaps:

1. **Detection Gap**: Why didn't local linters, unit tests, integration tests, canary deployments, or automated health checks catch this before users were affected?
2. **Containment Gap**: Why did the blast radius expand across tenants or services? Why wasn't rollback instant?

## Converting lessons into lasting assets

A post-mortem without action items is wasted effort. `post-mortem` creates three durable assets:

1. **Architectural Decision Records**: If the failure revealed that a pattern is unsafe (such as direct schema modifications without parallel runs), it drafts an ADR in `docs/adr/`.
2. **Preventive Tracker Tickets**: Files prioritized tickets on the configured issue tracker to add missing regression tests, implement circuit breakers, or improve alerting.
3. **Agent Environment Tuning**: Triggers [retro](https://aihero.dev/skills-retro) to update `CODING_STANDARDS.md` and agent instructions so future code reviews automatically catch this pattern.

## Common questions

**How soon after an incident should we run this?**

Ideally within 24 to 48 hours while details, timestamps, and log contexts are fresh in everyone's memory.

**What is the difference between `/post-mortem` and `/retro`?**

`/retro` evaluates an agent coding session: navigation pointers, tool ergonomics, and coding conventions. `/post-mortem` evaluates a live system failure: customer impact, incident timeline, alert effectiveness, and system architecture. `/post-mortem` concludes by calling `/retro` to translate operational lessons into agent guardrails.

**Can this be shared with external customers?**

The internal post-mortem contains sensitive technical details and logs. You can use [release-notes](https://aihero.dev/skills-release-notes) or a sanitized summary for external status communication.

## It's working if

- The post-mortem assigns zero blame to individuals and focuses on systemic vulnerabilities.
- A precise timeline with TTD and TTM metrics is established.
- Concrete follow-up tickets are filed on the tracker for both detection and prevention.
- Relevant architectural rules are documented in `docs/adr/`.

## Where it fits

`post-mortem` connects incident mitigation back into continuous improvement:

```txt
diagnosing-bugs → post-mortem → adr / tickets → retro
```

- Upstream, [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) locates and fixes the immediate bug.
- Downstream, [retro](https://aihero.dev/skills-retro) updates agent environment rules based on post-mortem findings.
- Architectural decisions are recorded in `docs/adr/` using the layout from [domain-modeling](https://aihero.dev/skills-domain-modeling).
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever an operational incident occurs.
