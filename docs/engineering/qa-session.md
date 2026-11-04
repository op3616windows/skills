## What it does

`qa-session` leads an interactive acceptance and exploratory testing session against a running application to verify a feature or bugfix against its originating [spec](https://www.aihero.dev/ai-coding-dictionary/spec).

The defining constraint is that it refuses to fix defects silently during the test run. When an anomaly, broken acceptance criterion, or visual flaw is discovered, the skill logs and files a standardized defect issue on the project's tracker instead of immediately hacking a fix. This keeps the testing process objective, prevents untracked side-effects, and preserves reproducible evidence for triage.

## When to reach for it

You invoke this by typing `/qa-session`, and the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

Reach for this once code is implemented and automated tests pass, but before opening a pull request or shipping to users:

| Your situation | What to run |
| --- | --- |
| Verifying a finished feature against acceptance criteria | `/qa-session` |
| Diagnosing a known, reproducible bug with failing tests | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| Running code review across standards, spec, and security | [code-review](https://aihero.dev/skills-code-review) |
| Structuring the pull request once QA passes | [pr](https://aihero.dev/skills-pr) |

## Prerequisites

`qa-session` files standardized defects into the issue tracker, so [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) must have configured `docs/agents/issue-tracker.md` and `docs/agents/triage-labels.md`. Local markdown files under `.scratch/` work immediately if no remote tracker is configured.

## The three test tiers

`qa-session` structures the interactive verification into three progressive tiers:

1. **Tier 1: Core Acceptance (Happy Path)**
   Walks through the primary user stories and criteria defined in the originating spec. Confirms that the feature solves the intended problem when valid data is supplied.
2. **Tier 2: Boundary & Edge Cases**
   Tests empty inputs, maximum text limits, unusual unicode characters, rapid double-clicks, pagination boundaries, and cross-role permissions.
3. **Tier 3: Failure & Recovery Modes**
   Tests how the UI and backend respond to network disconnections, server 500 errors, aborted requests, and missing dependencies. Verifies that user-friendly error banners and retry buttons appear.

## Defect reporting discipline

When a test step fails, the skill formats a complete defect report:
- Exact reproduction steps with input data.
- Expected behavior cited directly from the spec.
- Actual behavior observed, including browser console errors or server logs.
- Severity rating: P1 (Blocker), P2 (Major), or P3 (Minor).

The defect is published to the tracker with the appropriate triage label (`needs-triage` or `ready-for-agent`), ensuring full traceability.

## Common questions

**Can `qa-session` execute automated browser checks?**

Yes. If browser automation tools like Playwright or Puppeteer, or CLI execution tools like curl, are configured in the environment, the skill will run automated smoke checks and inspect console output directly. Where automation is absent, it guides the human through manual testing steps.

**Why shouldn't the agent fix bugs as soon as they are spotted?**

Fixing bugs mid-session disrupts the test plan, resets the local environment, and introduces untested changes that can mask other defects. Filing the issue preserves a clean record of what failed, allowing fixes to be prioritized and executed through disciplined [tdd](https://aihero.dev/skills-tdd).

**What if the spec does not have explicit acceptance criteria?**

The skill infers criteria from the user stories and implementation decisions in the spec. If no spec exists at all, it asks the user for the intended behavior before starting testing.

## It's working if

- Testing systematically progresses from core happy paths to edge cases and error states.
- Discovered defects are filed to the tracker with full reproduction steps and error logs.
- It produces an end-of-session release recommendation: Pass (Ready to Ship) or Fail (Blocked on P1/P2 defects).
- No untracked, ad-hoc code edits are made during the verification session.

## Where it fits

`qa-session` sits between code implementation and pull request creation:

```txt
implement → qa-session → code-review → pr → retro
```

- Upstream, [implement](https://aihero.dev/skills-implement) finishes building the code and tests.
- Discovered bugs funnel directly to [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) or the triage queue in [triage](https://aihero.dev/skills-triage).
- Once QA passes, [pr](https://aihero.dev/skills-pr) prepares the pull request.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever exploratory testing is required.
