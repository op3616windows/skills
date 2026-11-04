## What it does

`security-review` audits a git diff or pull request specifically along the Application Security (AppSec) and Data Privacy axis.

The defining constraint is that it focuses exclusively on security invariants, authorization boundaries, sensitive data exposure, and injection vectors, refusing to mix them with stylistic coding preferences or generic smell complaints. A diff can follow every clean-code guideline and still leak customer data or expose an Insecure Direct Object Reference (IDOR). This skill isolates security as an independent review discipline.

## When to reach for it

Type `/security-review`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when auditing diffs, reviewing pull requests, or verifying access control.

| Your situation | What to run |
| --- | --- |
| Auditing a diff for security vulnerabilities, IDOR, or data leaks | `security-review` |
| Running a complete review covering repo standards, spec compliance, and security | [code-review](https://aihero.dev/skills-code-review), which spawns this skill as a third parallel axis |
| Hunting functional runtime bugs or off-by-one errors | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| Designing zero-downtime database migrations | [expand-contract](https://aihero.dev/skills-expand-contract) |

## The five security pillars

`security-review` evaluates changes against five essential AppSec pillars:

1. **Authorization & Tenant Isolation (AuthZ)**: Verifies that all database queries and mutations scope records by verified session tenant identifiers (`tenant_id`, `organization_id`). Scans for IDOR vulnerabilities where user-supplied IDs are queried without ownership verification.
2. **Sensitive Data & Privacy (PII & Secrets)**: Checks for committed credentials, API keys, unmasked tokens, or plaintext PII (passwords, emails, credit cards) leaking into application logs, error traces, or telemetry streams.
3. **Injection & Input Validation**: Checks for raw SQL concatenation, shell command execution with untrusted arguments, unescaped HTML injection (XSS), Server-Side Request Forgery (SSRF) against internal metadata endpoints, and path traversal flaws.
4. **Network & Configuration**: Verifies CORS origins, Content Security Policy, CSRF protection on state-changing requests, and rate-limiting on sensitive endpoints (authentication, password reset, payment flows).
5. **Dependencies & Supply Chain**: Flags untrusted new third-party packages or dependencies with known CVEs.

## Sub-agent isolation

When invoked as part of a multi-axis review in [code-review](https://aihero.dev/skills-code-review), `security-review` must be spawned in a **fresh [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)** as a dedicated [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent).

Running security analysis in the same context that wrote the code creates confirmation bias: the agent assumes its own implementation decisions were sound. Isolating the security review in a clean context ensures adversarial, objective scrutiny.

## Common questions

**Does this replace automated SAST/DAST security scanning tools?**

No. Static analysis tools (like Snyk, Semgrep, or SonarQube) excel at detecting known signature-based flaws and outdated dependencies. `security-review` excels at contextual and business logic vulnerabilities: missing tenant filters in ORM queries, subtle IDOR flaws, and multi-step authorization bypasses that automated pattern matchers miss.

**Why shouldn't security findings be merged into the general code review report?**

Merging security findings into a general list allows critical authorization vulnerabilities to be treated as routine stylistic feedback. Keeping security findings in a dedicated report ensures they receive immediate prioritization.

**What happens if no vulnerabilities are found?**

The skill explicitly reports: "No security or privacy vulnerabilities identified across the 5 pillars." It does not fabricate low-value warnings just to produce output.

## It's working if

- Every finding cites the exact file, line number, vulnerability classification, and concrete attack scenario.
- Findings distinguish between critical vulnerabilities (e.g. IDOR, tenant leak) and defense-in-depth suggestions (e.g. security headers).
- It catches queries that accept an entity ID without validating tenant ownership.
- It flags any plaintext logging of credentials or personal user data.

## Where it fits

`security-review` runs as a gate before code merges into main:

- In [code-review](https://aihero.dev/skills-code-review), it runs as the third parallel sub-agent alongside Standards and Spec.
- It can be invoked directly on any branch or PR diff before merging.
- If a security incident slips through, [post-mortem](https://aihero.dev/skills-post-mortem) identifies why the review missed it and updates detection criteria.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever security or privacy checks are requested.
