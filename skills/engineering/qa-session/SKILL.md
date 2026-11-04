---
name: qa-session
description: Lead an interactive acceptance and exploratory QA session against a feature or bugfix, verifying acceptance criteria and filing standardized bug reports.
disable-model-invocation: true
---

# QA Session

Automated unit tests and static linters check code invariants, but they cannot verify whether the assembled application delivers the expected user experience or withstands exploratory edge-case testing.

This skill leads an interactive Quality Assurance (QA) session between the human and the agent to validate a built feature or bugfix against its originating spec and acceptance criteria before shipping.

The issue tracker configuration should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## Process

### 1. Locate the Spec & Acceptance Criteria
Look for the originating spec or ticket:
1. An issue number or spec path supplied by the user.
2. A spec file in `docs/`, `specs/`, or `.scratch/` matching the branch or feature.
3. Commit messages or recent PR descriptions.
4. If no spec exists, ask the user what intended behavior and acceptance criteria should be verified.

Extract the list of acceptance criteria, user stories, and expected behaviors.

### 2. Build the Test Matrix
Construct an exploratory test matrix divided into three tiers:

- **Tier 1: Core Acceptance (Happy Path)**: Verify that the primary user flows work as specified.
- **Tier 2: Boundary & Edge Cases**: Test zero states, maximum inputs, unusual unicode characters, rapid repeated clicks, pagination limits, and permission boundaries.
- **Tier 3: Failure & Recovery Modes**: Test behavior during network disconnection, invalid server responses, aborted requests, or missing dependencies.

### 3. Conduct Interactive Testing
Walk through the test matrix step by step:

1. **Local environment check**: Check if the application or test environment is running locally (e.g. dev server, Docker container, CLI binary, browser automation). If automated end-to-end tools (such as Playwright, Cypress, or curl) are available, offer to execute smoke checks directly.
2. **Step-by-step guidance**: Present each test scenario clearly:
   - Action to take: exact URL, button to click, or command to run.
   - Input data to supply.
   - Expected observable outcome.
3. **Record findings**: Ask the user what happened, or observe the automated execution result. Compare the observed result against the expected behavior.

### 4. File Discovered Defects
When a bug, edge-case failure, visual flaw, or regression is identified:

Do NOT fix it silently during the QA session. Instead, log and file a standardized defect report so it can be prioritized and tracked properly.

Check `docs/agents/issue-tracker.md` for the configured tracker workflow (GitHub CLI, GitLab CLI, local markdown files under `.scratch/`, or custom tracker).

Check `docs/agents/triage-labels.md` for the label vocabulary (e.g. apply `needs-triage` or `ready-for-agent`).

Format each discovered bug report using the template below:

```markdown
# Bug: <Concise Bug Title>

## Environment
- Branch / Commit: `<git-hash-or-branch>`
- OS / Browser / Node version: `<environment-details>`

## Steps to Reproduce
1. Navigate to `<URL or screen>`
2. Perform action `<step>`
3. Enter `<input>` and submit

## Expected Behavior
<Clear description of what should happen according to spec>

## Actual Behavior
<What actually happened: error message, broken layout, incorrect data>

## Evidence
```
<Paste relevant browser console error, server stack trace, or terminal output>
```

## Severity
- **P1 - Blocker**: Core flow broken, data loss, or crash with no workaround.
- **P2 - Major**: Key feature impaired, but workaround exists.
- **P3 - Minor**: Visual defect, awkward copy, minor edge case.
```

### 5. Wrap Up & Routing
Once all test cases are executed, summarize:
- Total test scenarios executed.
- Passed criteria count.
- Defects filed (with issue numbers or file paths).
- Release recommendation: **Pass (Ready to Ship)** or **Fail (Blocked on P1/P2 fixes)**.

Direct next actions:
- For complex defects that need root-cause investigation: guide user to `/diagnosing-bugs`.
- For triage and backlog organization: guide user to `/triage`.
- If all acceptance criteria pass: guide user to `/pr` to prepare the pull request.
