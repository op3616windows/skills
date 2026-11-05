# Skills For Real Engineers & End-to-End Product Lifecycle

[![skills.sh](https://skills.sh/b/op3616windows/skills)](https://skills.sh/op3616windows/skills)

Agent skills for daily software engineering and complete product lifecycle management, designed for real production systems rather than vibe coding.

Developing reliable software with AI agents is difficult. Monolithic agent frameworks often try to control the entire workflow, stripping away your agency and making debugging hard when assumptions break down.

These skills are modular, composable, and compatible with any LLM harness (Claude Code, Codex, Cursor, Gemini CLI, Windsurf, OpenCode, and Copilot). Built upon Matt Pocock's software engineering foundation, this repository expands the workflow across the entire **End-to-End Product Lifecycle**: from customer discovery and UI edge state matrices to zero-downtime database migrations, AppSec security audits, interactive QA sessions, customer release notes, and incident post-mortems.

---

## Installation (30-second setup)

Choose the setup method for your agent. You can install as an auto-updating plugin or copy editable files directly into your project via [skills.sh](https://skills.sh/op3616windows/skills).

### 1. Get the skills

<details>
<summary><strong>Any Agent (Universal CLI: Cursor, OpenCode, Devin, Windsurf, Amp, Pi)</strong></summary>

Install interactively and pick the skills you want:
```bash
npx skills@latest add op3616windows/skills -a <agent>  # cursor, opencode, devin, windsurf, amp, pi; omit -a to choose interactively
```

Or install all 35 promoted skills at once:
```bash
npx skills@latest add op3616windows/skills --all
```

To update installed skills at any time:
```bash
npx skills@latest update
```

</details>

<details>
<summary><strong>Claude Code</strong></summary>

Install directly from GitHub:
```bash
claude plugin install op3616windows/skills
```

Or if you clone this repository locally, run the automatic symlink script:
```bash
./scripts/link-skills.sh
```

</details>

<details>
<summary><strong>Codex</strong></summary>

Add the marketplace and plugin:
```bash
codex plugin marketplace add op3616windows/skills
codex plugin add mattpocock-skills@op3616windows
```

Updates automatically at startup.

</details>

<details>
<summary><strong>GitHub Copilot (CLI and VS Code)</strong></summary>

Add the marketplace and install:
```bash
copilot plugin marketplace add op3616windows/skills
copilot plugin install mattpocock-skills@op3616windows
```

Then add to `~/.copilot/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "op3616windows": { "source": { "source": "github", "repo": "op3616windows/skills" }, "autoUpdate": true }
  }
}
```

In VS Code, run **Chat: Install Plugin From Source** and enter `https://github.com/op3616windows/skills`.

</details>

<details>
<summary><strong>Gemini CLI</strong></summary>

Install both engineering and productivity buckets:
```bash
gemini skills install https://github.com/op3616windows/skills.git --path skills/engineering
gemini skills install https://github.com/op3616windows/skills.git --path skills/productivity
```

Re-run both commands to update.

</details>

### 2. Configure your repository

In your agent, run this command once per repository:
```bash
/setup-matt-pocock-skills
```

It will:
- Configure your issue tracker (GitHub Issues, GitLab, local Markdown files under `.scratch/`, or custom workflows).
- Set up triage label vocabularies (`needs-triage`, `ready-for-agent`, etc.).
- Establish domain documentation layout (`GLOSSARY.md` and `docs/adr/`).

---

## Complete End-to-End Workflow Guide

Here is how to navigate the full product lifecycle from raw idea to production operations:
> For a complete, step-by-step example with exact developer prompts and outputs, read the [End-to-End Workflow Guide (English)](docs/guides/end-to-end-workflow-guide.md) or [Bản Hướng Dẫn Vận Hành (Tiếng Việt)](docs/guides/end-to-end-workflow-guide.vi.md).


```txt
[Discovery]      /shape-problem
                      │
[Architecture]   /grill-with-docs ──(UI check)──> /prototype + ux-states
                      │
[Specification]  /to-spec (telemetry-design)
                      │
[Decomposition]  /to-tickets (expand-contract)
                      │
[Build & Test]   /implement (or /implement-spec) ──> /tdd
                      │
[Acceptance QA]  /qa-session ──(Defect filed)──> /diagnosing-bugs
                      │
[Automated E2E]  /e2e-testing (Playwright CUJ regression)
                      │
[Review & Gate]  /code-review (Standards + Spec + security-review)
                      │
[Ship]           /pr ──> Merge
                      │
[Release & Ops]  /release-notes ──> /retro
                      │
[Outage Response]/diagnosing-bugs ──> /post-mortem ──> ADR / retro
```

### Phase 1: Problem Discovery
Before jumping into code or database schemas, isolate the customer struggle.
- Run **`/shape-problem`** on customer support tickets, user quotes, or metric drops. It drives an interview focused entirely on the problem space, Jobs-to-be-Done (JTBD), measurable success metrics, and explicit non-goals.

### Phase 2: Architecture & Technical Alignment
- Run **`/grill-with-docs`** to explore implementation architecture. It sharpens technical decisions, creates Architectural Decision Records (ADRs) in `docs/adr/`, and captures domain terminology in `GLOSSARY.md`.
- If a UI or interaction design is uncertain, branch out to **`/prototype`** with throwaway code, accompanied by **`ux-states`** to map the complete matrix: Empty, Loading, Optimistic UI, Error recovery, Accessibility (a11y), and Localization (i18n).

### Phase 3: Specification & Ticket Slicing
- Run **`/to-spec`** to synthesize the conversation into an engineering specification. It incorporates **`telemetry-design`** (defining analytics events, structured logs, and service metrics) and **`ux-states`**.
- Run **`/to-tickets`** to break the spec into tracer-bullet vertical slices with explicit dependency edges (`Blocked by`). If the change touches database schemas, public APIs, or wide refactors, it invokes **`expand-contract`** to sequence zero-downtime migration tickets (Expand, Backfill, Switch, Contract).

### Phase 4: Test-Driven Implementation
- Build tickets using **`/implement`** (one ticket at a time, clearing context between tickets) or **`/implement-spec`** (orchestrating multiple implementer subagents across the task graph in parallel).
- Implementation drives **`/tdd`**, enforcing red-green-refactor cycles at pre-agreed seams.

### Phase 5: Interactive Acceptance QA & Automated E2E Suites
- Before opening a pull request, run **`/qa-session`** to test the running application against the originating spec. It verifies happy paths, edge boundaries, and failure recoveries.
- When an anomaly is discovered, it logs a standardized defect ticket to your issue tracker rather than patching code silently.
- Run **`/e2e-testing`** to codify verified critical user journeys (CUJ) into permanent, automated browser tests (Playwright/Cypress) using accessibility-first locators and fast storage state fixtures.

### Phase 6: Three-Axis Review & Pull Request
- Run **`/code-review`** to audit the diff along three independent parallel axes:
  1. **Standards**: Documented repo standards plus Fowler code smell heuristics.
  2. **Spec**: Compliance with the originating issue or specification.
  3. **Security & Privacy**: Dedicated AppSec audit via **`security-review`** (AuthZ/IDOR, tenant isolation, PII leaks, injection vectors, and supply chain).
- Run **`/pr`** to generate the pull request description: minimal visual summary, before/after evidence, and a Merge Danger assessment (One-Way vs Two-Way Door).

### Phase 7: Release, Post-Mortems & Continuous Learning
- Run **`/release-notes`** to synthesize merged PRs and closed issues into polished, customer-facing notes, highlighting breaking changes and migration steps.
- Run **`/retro`** after any session to turn lessons into automated checks, coding standards, and agent steering files.
- If an operational outage or critical regression occurs, resolve the immediate issue via **`/diagnosing-bugs`**, then run **`/post-mortem`** to establish a blameless Five Whys timeline, record preventative ADRs, and feed automated guardrails into `/retro`.

### Unsure where to start?
Type **`/ask-matt`** at any point. It acts as the central router, asking about your current situation and directing you to the exact skill or flow you need.

---

## Why These Skills Exist

These skills address the most frequent failure modes encountered when building software with AI agents:

### #1: The Agent Misunderstands The Goal
- **Problem**: Misalignment between developer expectations and what the agent creates.
- **Fix**: Upfront interview grilling via [`/grill-me`](./skills/productivity/grill-me/SKILL.md) (stateless) or [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) (stateful, building `GLOSSARY.md` and ADRs).

### #2: The Agent Speaks Jargon And Wastes Tokens
- **Problem**: Agents guess domain terminology and invent redundant explanations.
- **Fix**: Ubiquitous language documented in `GLOSSARY.md` through [`domain-modeling`](./skills/engineering/domain-modeling/SKILL.md).

### #3: The Code Doesn't Work
- **Problem**: Hallucinated logic and untested edge conditions.
- **Fix**: Red-green feedback loops via [`/tdd`](./skills/engineering/tdd/SKILL.md) and disciplined bug reproduction loops via [`/diagnosing-bugs`](./skills/engineering/diagnosing-bugs/SKILL.md).

### #4: Codebase Entropy & Architecture Decay
- **Problem**: Rapid coding generates shallow, tightly coupled modules ("ball of mud").
- **Fix**: Deep module design principles via [`codebase-design`](./skills/engineering/codebase-design/SKILL.md) and periodic architecture health scans via [`/improve-codebase-architecture`](./skills/engineering/improve-codebase-architecture/SKILL.md).

### #5: Building The Wrong Product Solution
- **Problem**: Engineering teams jump straight to technical implementations before the customer pain, JTBD, and non-goals are defined.
- **Fix**: Upfront discovery framing via [`/shape-problem`](./skills/productivity/shape-problem/SKILL.md).

### #6: UI Breaks On Edge Conditions
- **Problem**: Frontend components look fine with perfect demo data but break on latency, empty results, network failures, or long text.
- **Fix**: Mandatory UI state matrices (empty, loading, optimistic rollback, error recovery, a11y, i18n) via [`ux-states`](./skills/engineering/ux-states/SKILL.md).

### #7: Flying Blind In Production
- **Problem**: Features ship without observability, making adoption impossible to track and production bugs impossible to diagnose.
- **Fix**: Formalized analytics events, structured logs, and SLI alert rules via [`telemetry-design`](./skills/engineering/telemetry-design/SKILL.md).

### #8: Migration Downtime & Security Blindspots
- **Problem**: Single-step database changes take down services, and routine code reviews miss authorization bypasses (IDOR) and tenant leaks.
- **Fix**: Zero-downtime migrations via [`expand-contract`](./skills/engineering/expand-contract/SKILL.md) and dedicated AppSec audits via [`security-review`](./skills/engineering/security-review/SKILL.md).

---

## Reference

Skills split on one axis: **invocation mode**.
- **User-invoked**: Reachable only when you type their slash command (e.g. `/shape-problem`); their job is to orchestrate workflows.
- **Model-invoked**: Reachable by you or called autonomously by agents when appropriate; they embody reusable engineering disciplines.

### Engineering

**User-invoked**
- **[ask-matt](./skills/engineering/ask-matt/SKILL.md)**: Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)**: Grilling session that also builds your project's domain model, sharpening terminology and updating `GLOSSARY.md` and ADRs inline.
- **[triage](./skills/engineering/triage/SKILL.md)**: Move issues through a state machine of triage roles.
- **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)**: Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick.
- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)**: Configure this repo for the engineering skills (issue tracker, triage labels, domain doc layout). Run once per repo before using the other engineering skills.
- **[to-spec](./skills/engineering/to-spec/SKILL.md)**: Turn the current conversation into a spec and publish it to the issue tracker. No interview, just synthesizes what you have already discussed.
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)**: Break any plan, spec, or conversation into a set of tracer-bullet tickets, each declaring its blocking edges, written as text in a local file, or as native blocking links on a real tracker.
- **[implement](./skills/engineering/implement/SKILL.md)**: Build the work described by a spec or set of tickets, driving `/tdd` at pre-agreed seams and closing out with `/code-review` before committing.
- **[implement-spec](./skills/engineering/implement-spec/SKILL.md)**: Implement a whole spec on one integration branch. Works the tickets as a task graph, running implementer subagents across the ready frontier for maximum concurrency, then closes out with `/code-review`.
- **[wayfinder](./skills/engineering/wayfinder/SKILL.md)**: Plan a huge chunk of work, more than one agent session can hold, as a shared map of decision tickets on the issue tracker, and resolve them one at a time until the way to the destination is clear.
- **[qa-session](./skills/engineering/qa-session/SKILL.md)**: Lead an interactive acceptance and exploratory QA session against a feature or bugfix, verifying acceptance criteria and filing standardized bug reports.
- **[post-mortem](./skills/engineering/post-mortem/SKILL.md)**: Lead a blameless post-mortem analysis after an incident or severe bug: establish timeline, run 5 Whys, identify detection gaps, record ADRs, and trigger retro.
- **[retro](./skills/engineering/retro/SKILL.md)**: Suggest improvements to the coding agent's environment (navigation, automated checks, coding standards, steering files, tooling) after a session, most severe first.

**Model-invoked**
- **[prototype](./skills/engineering/prototype/SKILL.md)**: Build a throwaway prototype to answer a design question, either a single shareable HTML file for state/logic questions, or several radically different UI variations toggleable from one route.
- **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)**: Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug, minimise, hypothesise, instrument, fix, and regression-test.
- **[research](./skills/engineering/research/SKILL.md)**: Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[tdd](./skills/engineering/tdd/SKILL.md)**: Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)**: Actively build and sharpen a project's domain model: challenge terms against the glossary, stress-test with edge-case scenarios, and update `GLOSSARY.md` and ADRs inline.
- **[codebase-design](./skills/engineering/codebase-design/SKILL.md)**: Shared discipline and vocabulary for designing deep modules: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface.
- **[ux-states](./skills/engineering/ux-states/SKILL.md)**: Design and specify a comprehensive UI state matrix (empty, loading, optimistic UI, error recovery, accessibility, and localization) for frontend components or user flows.
- **[telemetry-design](./skills/engineering/telemetry-design/SKILL.md)**: Define product analytics events, structured logs, system metrics, and error boundaries for a feature.
- **[expand-contract](./skills/engineering/expand-contract/SKILL.md)**: Plan and execute zero-downtime database schema migrations, breaking API changes, or wide refactors using the Expand-Contract pattern.
- **[security-review](./skills/engineering/security-review/SKILL.md)**: Audit a git diff, branch, or PR for security vulnerabilities, authorization flaws, PII leaks, and injection risks.
- **[code-review](./skills/engineering/code-review/SKILL.md)**: Three-axis review of the diff since a fixed point: **Standards** (repo coding standards, plus a Fowler smell baseline), **Spec** (originating issue/spec compliance), and **Security** (AppSec and privacy audit via `security-review`), run as parallel sub-agents.
- **[pr](./skills/engineering/pr/SKILL.md)**: The shape a pull request body should take: a summary as the smallest visual that makes the change clear, before/after evidence that it works, and a merge-danger call (one-way or two-way door, plus blast radius).
- **[wizard](./skills/engineering/wizard/SKILL.md)**: Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.
- **[e2e-testing](./skills/engineering/e2e-testing/SKILL.md)**: Author, maintain, run, and debug resilient end-to-end browser tests (Playwright, Cypress) for critical user journeys.

### Productivity

**User-invoked**
- **[shape-problem](./skills/productivity/shape-problem/SKILL.md)**: Interview and synthesize raw customer feedback, support logs, or metrics into a focused problem statement before discussing technical solutions.
- **[grill-me](./skills/productivity/grill-me/SKILL.md)**: Get relentlessly interviewed about a plan or design until every branch of the design tree is resolved.
- **[handoff](./skills/productivity/handoff/SKILL.md)**: Compact the current conversation into a handoff document so another agent can continue the work.
- **[teach](./skills/productivity/teach/SKILL.md)**: Teach the user a new skill or concept over multiple sessions, using the current directory as a stateful teaching workspace.
- **[to-questionnaire](./skills/productivity/to-questionnaire/SKILL.md)**: Turn a decision you can't answer alone into a Markdown questionnaire for the one person who can, filled in async, or together over a meeting. It grills you about the send (who it's for, what you need back), not the subject.
- **[wait-what](./skills/productivity/wait-what/SKILL.md)**: Fire this the moment a message doesn't land. The agent re-pitches it with the context you're missing, in plain English, using your `GLOSSARY.md` vocabulary.
- **[release-notes](./skills/productivity/release-notes/SKILL.md)**: Generate user-facing release notes from merged pull requests, commits, and closed issues since the last release, translating technical diffs into customer value.

**Model-invoked**
- **[grilling](./skills/productivity/grilling/SKILL.md)**: Interview the user relentlessly about a plan, decision, or idea until every branch of the design tree is resolved. The reusable interview primitive behind `grill-me`, `grill-with-docs`, `triage`, `wayfinder` and `improve-codebase-architecture`.
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)**: Writing documents for agents: skills, AGENTS.md/CLAUDE.md, and any doc an agent reaches by a pointer.
