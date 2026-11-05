---
name: e2e-testing
description: "Author, maintain, run, and debug resilient end-to-end browser tests (Playwright, Cypress) for critical user journeys. Use when writing automated E2E tests, codifying acceptance criteria into browser suites, or diagnosing flaky browser tests."
---

# End-to-End Testing

Automated end-to-end (E2E) browser tests verify that integrated application subsystems (frontend, backend, database, external APIs) deliver the intended user experience in real browsers.

This skill is the reference for writing, maintaining, running, and diagnosing resilient browser tests with modern frameworks (Playwright, Cypress) without flakiness or maintenance nightmares.

## The Five E2E Invariants

Every automated browser test must uphold these five invariants:

### 1. Scope: Critical User Journeys (CUJ) Only
Do not invert the testing pyramid into an ice-cream cone. Edge cases, input boundary validations, and detailed matrix variations belong to unit and integration tests at the code seam (see the `tdd` skill). E2E browser tests verify only high-value, multi-step critical user journeys:
- User signup, email verification, and workspace creation.
- Core checkout, payment, and subscription transitions.
- Critical workflow handoffs (e.g. upload dataset, trigger async export, download verified file).

### 2. Locators: Accessibility-First
Never use implementation-coupled CSS selectors (`div > button.btn-primary.css-1a2b`), class names, or XPaths. Always use user-facing locators matching how assistive technologies and users see the page:
- `page.getByRole('button', { name: /submit order/i })`
- `page.getByLabel('Work Email')`
- `page.getByText('Invoice #1042')`
- `page.getByTestId('tenant-selector')` (used only when semantic roles do not exist)

### 3. Synchronization: Zero Hard Waits
Never introduce arbitrary sleep calls (`page.waitForTimeout(3000)` or `sleep(3)`). Hard waits cause test flakiness and slow down CI suites:
- Use web-first auto-retrying assertions: `await expect(locator).toBeVisible()`.
- Wait on explicit network responses or domain events: `await page.waitForResponse(resp => resp.url().includes('/api/exports') && resp.status() === 200)`.
- Wait on UI state transitions rather than time.

### 4. Authentication: Fast Storage State
Do not navigate through login pages before every individual test:
- Authenticate once in a global setup fixture or via direct API call.
- Persist browser cookies and local storage tokens into `storageState.json`.
- Initialize test worker contexts directly with the pre-authenticated storage state.

### 5. Hermetic Test Data & Isolation
Each test run must be independent and repeatable:
- Avoid sharing mutable state across tests.
- Scope created resources using unique prefixes or timestamps (e.g. `test-org-${Date.now()}`).
- Clean up test artifacts in fixtures or use ephemeral database environments.

## Process: Authoring an E2E Test

### 1. Codify Acceptance Criteria
Extract the user journey from the spec or recent `/qa-session` defect/acceptance notes. Outline the journey as sequential user actions and observable milestones.

### 2. Scaffold the Test File
Place E2E tests in the repo's designated test directory (e.g. `e2e/`, `tests/e2e/`, or `playwright/`). Use descriptive test titles:

```typescript
import { test, expect } from "@playwright/test";

test.describe("Organization Data Export", () => {
  test.use({ storageState: "playwright/.auth/admin.json" });

  test("billing admin triggers async export and downloads verified CSV", async ({ page }) => {
    // 1. Navigate directly to target surface
    await page.goto("/settings/billing/export");

    // 2. Interact using accessible roles
    await page.getByRole("button", { name: /request export/i }).click();

    const dialog = page.getByRole("dialog", { name: /confirm export/i });
    await expect(dialog).toBeVisible();

    await dialog.getByRole("combobox", { name: /format/i }).selectOption("csv");
    await dialog.getByRole("button", { name: /start export/i }).click();

    // 3. Verify optimistic transition and status polling
    await expect(page.getByText(/export requested/i)).toBeVisible();

    // 4. Await asynchronous job completion and download event
    const downloadPromise = page.waitForEvent("download");
    const downloadButton = page.getByRole("button", { name: /download ready file/i });
    await expect(downloadButton).toBeEnabled({ timeout: 30000 });
    await downloadButton.click();

    const download = await downloadPromise;
    expect(download.suggestedFilename()).toMatch(/^export-.*\.csv$/);
  });
});
```

### 3. Run and Validate Locally
Run the suite with headless and UI modes:
```bash
npx playwright test e2e/export.spec.ts --project=chromium
```
Verify that the test passes consistently across multiple local runs without intermittent failures:
```bash
npx playwright test e2e/export.spec.ts --repeat-each=3
```

## Diagnosing Flaky or Failing Tests

When an E2E test fails locally or on CI:
1. **Inspect Artifacts**: Never guess what happened. Open the Playwright trace file, screenshot, and video recording:
   ```bash
   npx playwright show-trace test-results/trace.zip
   ```
2. **Classify Failure Root Cause**:
   - **Locator Drifting**: UI label or accessible role changed. Fix: update locator to match new accessible label.
   - **Race Condition**: Assertion checked before asynchronous network fetch resolved. Fix: replace manual checks with auto-retrying `expect(locator).toBeVisible()` or explicit `waitForResponse`.
   - **Shared State Contamination**: Test failed because a previous test mutated or deleted shared data. Fix: isolate test database records.
   - **Genuine Bug**: Application regression. File defect or route to `/diagnosing-bugs`.
