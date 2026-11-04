---
name: shape-problem
description: Interview and synthesize raw customer feedback, support logs, or metrics into a focused problem statement before discussing technical solutions.
disable-model-invocation: true
---

# Shape Problem

Clarify and frame a product problem before jumping into technical solutions or architecture.

Most software efforts fail because the team built the wrong thing, not because they built it poorly. This skill conducts an upfront discovery interview to isolate the user's struggle, identify measurable outcomes, and explicitly bound what is out of scope.

## Process

### 1. Gather starting evidence

Examine whatever raw inputs exist: customer feedback quotes, bug reports, customer support tickets, incident notes, analytics drops, or vague feature requests.

If `GLOSSARY.md` exists in the repository, read it to use established domain terms.

### 2. Interview the problem space

Call the Skill tool with "grilling".

Run structured interview rounds focused entirely on the problem space. Refuse to discuss code, architecture, frameworks, or database schemas during this phase. If the user suggests a solution ("we need a Redis cache" or "add a modal"), pivot back: "What user struggle or business breakdown makes that necessary?"

Probe these six areas:

1. **Target user and persona**: Who experiences this pain directly? Who experiences it second-hand?
2. **Job to be done (JTBD)**: What is the user trying to accomplish when they run into this friction?
3. **Current pain and workaround**: What happens today when they attempt the task? What broken workarounds do they employ?
4. **Evidence and frequency**: How often does this occur? What logs, support ticket volumes, or user quotes prove it happens?
5. **Success indicators**: How will we objectively measure whether this problem is solved (e.g. drop in bounce rate, reduction in support tickets, task completion speed)?
6. **Explicit non-goals**: What related user pains or adjacent problems are we deliberately NOT tackling?

### 3. Sharpen domain vocabulary

Call the Skill tool with "domain-modeling".

If the problem discussion revealed new business concepts, overloaded terms, or conflicting definitions, resolve them now. If working inside a repository, update `GLOSSARY.md` inline with the agreed terms.

### 4. Produce the Problem Frame

Synthesize the interview into a structured Problem Frame using the template below:

```markdown
# Problem Frame: <Short Problem Name>

## Problem Statement
A concise, one or two sentence declaration of the core user pain, written strictly from the user's perspective without mentioning technical implementation.

## Target User & Context
- **Primary Persona**: Who suffers directly.
- **Context of Struggle**: When and where the friction occurs.

## Job To Be Done (JTBD)
When <situation or trigger>, I want to <motivation or action>, so I can <expected outcome>.

## Current Friction & Workarounds
How users cope today, and why existing alternatives fall short.

## Measurable Success Indicators
- Metric or signal 1 (e.g. reduce checkout abandonment by 15%)
- Metric or signal 2 (e.g. zero manual reconciliation tickets per week)

## Explicit Non-Goals
- Adjacent problem 1 that we will not solve in this initiative.
- Feature idea 2 that is rejected for now to preserve scope.
```

### 5. Transition to technical flow

Once the Problem Frame is approved by the user, guide them to the next step:

- **Single-session technical shaping in a repo**: Run `/grill-with-docs` to explore technical architecture, module boundaries, and ADRs.
- **Foggy, multi-session greenfield build**: Run `/wayfinder` to chart a map of technical decision tickets on the issue tracker.
