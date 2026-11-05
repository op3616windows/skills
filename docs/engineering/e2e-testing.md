## What it does

`e2e-testing` authors, maintains, executes, and diagnoses resilient automated end-to-end browser tests using modern frameworks such as Playwright and Cypress. It restricts browser tests strictly to Critical User Journeys (CUJ) rather than duplicating unit or integration tests across edge cases.

It enforces accessibility-first locators, auto-retrying web assertions, and reusable authentication storage states so browser suites run fast and never flake on CI.

## When to reach for it

Type `/e2e-testing`, or the agent reaches for it automatically when authoring or debugging browser-based test suites.

| Your situation | What to do |
| --- | --- |
| Verifying a multi-step user journey across real browser pages | Use [e2e-testing](https://aihero.dev/skills-e2e-testing) |
| Testing core business logic, API endpoints, or edge case variations at a code seam | Use [tdd](https://aihero.dev/skills-tdd) instead |
| Conducting an interactive, exploratory testing walk with the user before a PR | Use [qa-session](https://aihero.dev/skills-qa-session) instead |
| An automated Playwright test fails intermittently or flakes on CI | Use [e2e-testing](https://aihero.dev/skills-e2e-testing) to diagnose the trace and race conditions |

## Prerequisites

The repository must have an end-to-end testing harness configured (such as `@playwright/test` or Cypress) and a script to launch the local application server or test database.

## Critical User Journeys over full matrix coverage

The skill's defining concept is **Critical User Journeys (CUJ)**. Testing every input variation, null check, and validation error in a real browser inverts the testing pyramid into a slow, fragile ice-cream cone.

Browser tests are reserved for high-leverage workflows where multiple services connect:
- User onboarding, invitation acceptance, and organization creation.
- Checkout, subscription upgrades, and billing webhooks.
- Multi-step asynchronous file exports, data imports, and report generation.

All other permutations (validation warnings, boundary values, state machines) belong to fast integration tests written via [tdd](https://aihero.dev/skills-tdd).

## Zero hard waits and accessibility locators

The skill enforces two technical disciplines that eliminate flakiness:

1. **Accessibility-first locators**: Every selector targets user-facing semantics (`getByRole`, `getByLabel`, `getByText`) rather than implementation details like CSS class names or XPath. When styling refactors happen, the tests remain green.
2. **Web-first auto-retrying assertions**: The skill forbids `page.waitForTimeout()` and hard sleeps. Tests synchronize against explicit network responses or DOM visibility (`await expect(locator).toBeVisible()`).

## Common questions

**Should every ticket have an E2E test?**

No. Only tickets introducing or modifying complete, multi-step critical user journeys require an E2E test. Internal refactors, database schema expansions, and minor UI text changes should rely on unit and integration tests.

**How do we avoid slow logins in every test?**

Use Playwright's `storageState` fixture. Authenticate once during global setup or via a direct backend API call, save session tokens to a JSON file, and load that authenticated state into each browser context directly.

**What is the difference between qa-session and e2e-testing?**

`qa-session` is an interactive, human-in-the-loop exploratory session conducted once before opening a pull request to catch edge cases and visual flaws. `e2e-testing` codifies those verified critical paths into permanent, automated browser tests that run in continuous integration.

## It's working if

- Tests pass consistently across repeated local runs with `--repeat-each=3`.
- No test file contains `page.waitForTimeout()` or arbitrary sleep calls.
- Selectors use semantic roles and labels rather than generated CSS classes.
- Test suites run with pre-authenticated storage states rather than UI login forms.

## Where it fits

`e2e-testing` is a **chain step** in the main flow between implementation and review. It codifies acceptance criteria verified during [qa-session](https://aihero.dev/skills-qa-session) into permanent regression tests, builds upon [tdd](https://aihero.dev/skills-tdd) code seams, and protects the codebase before opening a pull request with [pr](https://aihero.dev/skills-pr). See [ask-matt](https://aihero.dev/skills-ask-matt) for the full map.
