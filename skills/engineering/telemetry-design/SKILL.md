---
name: telemetry-design
description: "Define product analytics events, structured logs, system metrics, and error boundaries for a feature. Use when designing telemetry, observability, tracking plans, or verifying event emissions in tests."
---

# Telemetry Design

If you cannot observe how a feature behaves in production, you cannot know whether it solved the user's problem or degraded system stability. This skill specifies the telemetry and observability requirements for any new capability or architectural change.

## The Four Observability Pillars

Every feature specification or technical plan should define its telemetry across four pillars:

### 1. Product Analytics & Behavioral Events
Track user actions and funnel progression to measure feature adoption and success metrics.

- **Naming Taxonomy**: Use lowercase `noun_action` or `object_action` format (e.g. `checkout_started`, `filter_applied`, `invite_sent`).
- **Trigger Condition**: Exactly when the event is emitted (e.g. when the button is pressed versus when the server confirms the mutation).
- **Required Properties**: Explicit context required for every event (e.g. `user_id`, `organization_id`, `feature_variant`, `item_count`).
- **Data Privacy**: Never include Personally Identifiable Information (PII) such as plaintext email addresses, passwords, social security numbers, or raw credit card data in analytics payloads. Use anonymized or hashed identifiers.

### 2. Structured Application Logging
Provide searchable, machine-readable log lines for debugging operational failures.

- **Log Level**: Strict guidelines for levels:
  - `DEBUG`: Verbose diagnostics disabled in production by default.
  - `INFO`: Normal lifecycle events (e.g. worker started, job completed, transaction processed).
  - `WARN`: Recoverable anomalies or degraded operations (e.g. cache miss fallback, rate limit approaching).
  - `ERROR`: Unrecoverable failures affecting a user transaction or background job.
- **Context Keys**: Every log line must include structured metadata keys (e.g. `correlation_id`, `trace_id`, `tenant_id`, `duration_ms`).
- **Sanitization Invariant**: Redact credentials, session tokens, and secrets before log emission.

### 3. Service Level Metrics (SLIs)
Define numeric time-series metrics to monitor system health and performance:

- **Counters**: Monotonically increasing numbers (e.g. `feature_invocations_total`, `payment_failures_total`).
- **Histograms & Timers**: Request and processing latency distributions (e.g. `api_request_duration_seconds{endpoint="upload"}`).
- **Gauges**: Current state measurements (e.g. `active_websocket_connections`, `queue_depth`).
- **SLO & Alert Thresholds**: Concrete thresholds that trigger paging alerts (e.g. error rate exceeding 1% over a 5-minute rolling window, p95 latency exceeding 500ms).

### 4. Error Boundaries & Alerting
- Define how unexpected exceptions are grouped and reported to error monitoring services (e.g. Sentry, Datadog).
- Tag errors with component names, deployment versions, and user impact levels.

## Output Template

When designing telemetry for a spec or feature, format the output as follows:

```markdown
### Telemetry & Success Metrics Specification

#### Behavioral Analytics Events
| Event Name | Trigger Moment | Required Properties | Success Metric Target |
| --- | --- | --- | --- |
| `export_requested` | User clicks Export button | `format: "csv" \| "json"`, `item_count: number` | Track feature usage volume |
| `export_completed` | File generation completes | `duration_ms: number`, `byte_size: number` | P95 latency < 3000ms |
| `export_failed` | Generation errors out | `error_code: string`, `retry_count: number` | Error rate < 0.5% |

#### Structured Logs
- `INFO`: `export.started` with `{ export_id, user_id, format }`
- `ERROR`: `export.failed` with `{ export_id, error_code, stack_trace, duration_ms }`

#### Service Metrics & Alert Rules
- Metric: `exports_processed_total{status="success|failure", format="csv|json"}`
- Metric: `export_processing_seconds` (Histogram with buckets `[0.1, 0.5, 1, 2, 5, 10]`)
- Alert: Trigger P2 alert when `rate(exports_processed_total{status="failure"}[5m]) / rate(exports_processed_total[5m]) > 0.02`.

#### TDD Seam Verification
In test suites, assert that `analytics.track` and `logger.error` are invoked with exact payloads when testing the execution seam.
```
