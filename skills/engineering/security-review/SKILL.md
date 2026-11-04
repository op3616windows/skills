---
name: security-review
description: "Audit a git diff, branch, or PR for security vulnerabilities, authorization flaws, PII leaks, and injection risks. Use when conducting security reviews, auditing PRs for security, or verifying tenant isolation."
---

# Security Review

A dedicated security and data privacy review of the diff between `HEAD` and a fixed point (or staged and working-tree changes).

This skill scans code specifically for Application Security (AppSec) vulnerabilities, broken authorization logic, multi-tenant data leaks, sensitive data exposure, and configuration weaknesses.

When running as a sub-agent within `code-review`, always run in a **fresh context window** to prevent contamination from earlier code authoring discussions.

## The Five Security Review Pillars

Evaluate the diff against these five specific pillars:

### 1. Authorization & Tenant Isolation (AuthZ)
- **Multi-Tenant Boundaries**: Does every database query, lookup, or mutation strictly scope records by `tenant_id` or `organization_id` derived from the verified session? Look for missing `where: { tenantId }` clauses.
- **Insecure Direct Object Reference (IDOR)**: Does the code accept an ID from the client (e.g. URL param or request body) and fetch or modify the entity without verifying that the requesting user owns or has access to it?
- **Role & Permission Checks**: Are privileged endpoints guarded by role checks (e.g. `isAdmin`, permission guards)? Can a standard user invoke admin operations?
- **Horizontal & Vertical Privilege Escalation**: Can an attacker forge headers, IDs, or parameters to act as another user or escalate their privileges?

### 2. Sensitive Data & Privacy (PII & Secrets)
- **Hardcoded Secrets**: Are API keys, private certificates, database passwords, or auth tokens committed in code or configuration files?
- **PII Exposure in Logs & Telemetry**: Does the diff log user request bodies, email addresses, passwords, credit card numbers, or session tokens into application logs or analytics events?
- **Data at Rest & in Transit**: Are sensitive credentials hashed with strong algorithms (e.g. bcrypt, Argon2)? Is sensitive data encrypted?

### 3. Injection & Input Validation
- **SQL / ORM Injection**: Are queries constructed via string concatenation instead of parameterized queries or ORM methods? Are raw query fragments properly escaped?
- **Command Injection**: Does the code pass user input to shell commands (`exec`, `spawn`, `popen`) without strict whitelisting?
- **Cross-Site Scripting (XSS)**: Does frontend code use `dangerouslySetInnerHTML`, unescaped template tags, or raw HTML injection with untrusted input?
- **Server-Side Request Forgery (SSRF)**: Does the code fetch URLs provided by the user without checking against private IP ranges (`127.0.0.1`, `169.254.169.254`, `10.0.0.0/8`, `192.168.0.0/16`)?
- **Path Traversal**: Can user-supplied filenames traverse directories via `../`?

### 4. Network, Transport & Security Headers
- **CORS Configuration**: Does CORS allow arbitrary origins (`*`) alongside `credentials: true`?
- **Content Security Policy (CSP) & Headers**: Are appropriate security headers present?
- **CSRF Protection**: Are state-changing mutations (`POST`, `PUT`, `DELETE`) protected against Cross-Site Request Forgery?
- **Rate Limiting**: Are unauthenticated or sensitive endpoints (login, signup, password reset, payment, OTP) protected by rate limits?

### 5. Dependencies & Supply Chain
- Are newly introduced third-party packages reputable, actively maintained, and free of known CVEs?
- Does the package addition introduce excessive attack surface?

## Review Process

### 1. Identify the Diff
Run `git diff <fixed-point>...HEAD` (or evaluate the requested diff). Verify the ref exists and the diff is non-empty.

### 2. Scan and Classify Findings
For every finding in the diff, categorize severity:
- **CRITICAL**: Immediate vulnerability enabling remote code execution, database compromise, tenant isolation bypass, or sensitive secret leakage.
- **HIGH**: Exploitable IDOR, authentication bypass, unauthenticated data exposure, or missing authorization check on mutations.
- **MEDIUM**: Missing rate limiting, permissive CORS, potential SSRF, verbose error leaks, or PII in logs.
- **LOW**: Missing security headers, weak defaults, or defense-in-depth suggestions.

### 3. Report Findings
Every finding must cite the exact file, line number or hunk, attack scenario, and remediation code:

```markdown
## Security & Privacy Findings

### [HIGH] IDOR in Invoice Retrieval Endpoint
- **Location**: `src/api/invoices.ts:45-52`
- **Vulnerability**: Insecure Direct Object Reference (AuthZ)
- **Attack Scenario**: An authenticated user can pass any `invoiceId` in the path param. The database query `db.invoice.findUnique({ where: { id: params.invoiceId } })` fails to filter by the requesting user's `organizationId`. An attacker can read competitors' invoices.
- **Remediation**:
  ```ts
  const invoice = await db.invoice.findFirst({
    where: { id: params.invoiceId, organizationId: session.organizationId }
  });
  if (!invoice) throw new NotFoundError();
  ```

### [MEDIUM] Plaintext Email in Application Logs
- **Location**: `src/auth/service.ts:88`
- **Vulnerability**: PII Leakage
- **Observation**: `logger.info("Login attempt", { email: user.email })` records customer PII to shared log aggregates.
- **Remediation**: Hash or mask the email: `{ emailHash: sha256(user.email) }`.
```

If no security vulnerabilities are detected in the diff, state: "No security or privacy vulnerabilities identified across the 5 pillars."
