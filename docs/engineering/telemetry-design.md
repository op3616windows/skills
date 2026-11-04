## What it does

`telemetry-design` defines the product analytics events, structured logs, system metrics, and error boundaries for any new feature or system change.

The defining constraint is that it treats observability as a first-class feature requirement defined before implementation, rather than an afterthought added after a production incident. Features built without explicit event taxonomies and log contexts cannot measure user adoption or diagnose distributed runtime failures. This skill guarantees that every new capability has verifiable tracking and alerting baked into its design.

## When to reach for it

Type `/telemetry-design`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when designing, specifying, or implementing features that touch user behavior or backend services.

| Your situation | What to run |
| --- | --- |
| Designing analytics tracking, log structures, or service metrics | `telemetry-design` |
| Writing a comprehensive feature specification | [to-spec](https://aihero.dev/skills-to-spec), which includes telemetry |
| Implementing behavior and asserting event emissions | [tdd](https://aihero.dev/skills-tdd), which verifies tracking calls |
| Investigating an unobserved production outage | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## The four observability pillars

`telemetry-design` organizes instrumentation into four complementary tiers:

1. **Behavioral Analytics**: Focuses on user actions and funnel progression (e.g. `signup_completed`, `plan_upgraded`). Enforces strict `object_action` naming, required context properties, and absolute exclusion of sensitive Personally Identifiable Information (PII).
2. **Structured Logging**: Focuses on runtime debugging for engineers. Enforces machine-readable JSON formats with mandatory correlation IDs, tenant identifiers, execution timings, and secret redaction.
3. **Service Level Indicators (SLIs)**: Defines time-series metrics (counters, histograms, and gauges) along with alert thresholds (e.g. error rate exceeding 1% over 5 minutes) to monitor health and performance.
4. **Error Boundaries & Breadcrumbs**: Defines how exceptions are caught, tagged with operational context, and routed to error aggregation platforms without leaking user credentials.

## Testing telemetry with TDD

Because telemetry requirements are formalized in advance, they are fully verifiable through automated tests.

During [tdd](https://aihero.dev/skills-tdd), you write unit tests against the domain [seam](https://www.aihero.dev/ai-coding-dictionary/seam) asserting that:
- The analytics dispatcher is called with the exact event name and expected payload.
- Sensitive fields (passwords, tokens, credit card numbers) are never passed to the tracking client.
- Structured logger methods record proper error codes and duration metrics on failure.

## Common questions

**Why not just let developers add log lines as they see fit?**

Unstructured logging results in noisy, inconsistent strings that cannot be indexed or alerted on effectively. One engineer logs `user_id`, another logs `uid`, and a third logs a plaintext email. `telemetry-design` establishes a consistent schema so logs can be aggregated across distributed services.

**Should every function emit an analytics event?**

No. High-volume internal operations belong in structured logs or metrics counters. Behavioral analytics events should be reserved for meaningful user interactions and business milestones.

**How does this prevent PII leaks?**

The specification explicitly categorizes all payload properties. Any field containing personal information must be hashed, anonymized, or excluded from third-party tracking payloads.

## It's working if

- Every newly specified feature includes a dedicated Telemetry & Success Metrics table.
- Analytics event names follow a consistent `noun_action` syntax.
- All log lines specify required context keys (`correlation_id`, `tenant_id`) and exclude plaintext secrets.
- Automated tests assert that event dispatchers fire with the specified properties.

## Where it fits

`telemetry-design` connects feature specification with verification:

- Upstream, [to-spec](https://aihero.dev/skills-to-spec) incorporates this skill to produce the Telemetry section.
- Downstream, [tdd](https://aihero.dev/skills-tdd) writes failing tests asserting that telemetry is emitted at agreed seams.
- During code review, [code-review](https://aihero.dev/skills-code-review) checks that implementation matches the telemetry contract.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever observability questions arise.
