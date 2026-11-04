## What it does

`shape-problem` clarifies and bounds a customer problem from raw feedback, support requests, or metrics before any technical discussion begins.

It refuses to entertain implementation details until the user's struggle, observable pain, and measurable success indicators are explicitly defined. A problem statement that specifies an architecture or a database table is rejected. The skill isolates why an intervention is needed, who experiences the friction, and what related concerns are deliberately ruled out of scope.

## When to reach for it

You invoke this by typing `/shape-problem`, and the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

Reach for this at the earliest stage of product discovery, when ideas arrive as vague requests or customer complaints:

| Your situation | What to run |
| --- | --- |
| You have raw customer complaints or a drop in metrics | `/shape-problem` to frame the problem |
| The problem is clear and you want to design technical architecture in a repo | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| The effort is massive, foggy, and spans many sessions | [wayfinder](https://aihero.dev/skills-wayfinder) |
| Architecture is settled and you need to build code | [implement](https://aihero.dev/skills-implement) |

## Prerequisites

None. It can run in an empty directory or an existing repository. When a `GLOSSARY.md` file exists, it reads existing terms to maintain consistent domain language.

## The problem frame

Most software projects waste time building technically sound solutions to poorly understood problems. When engineers jump straight to solution design, they solve the easiest technical problem rather than the hardest product friction.

`shape-problem` drives a structured [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) interview across six constraints:

1. **Target Persona**: Who hurts when this happens?
2. **Job To Be Done**: What are they trying to achieve in that moment?
3. **Current Friction**: What breaks, confuses, or blocks them today?
4. **Evidence**: What logs, feedback quotes, or metrics prove this is real?
5. **Success Indicators**: What objective metric proves the problem was solved?
6. **Non-Goals**: What adjacent problems will this effort deliberately refuse to touch?

The resulting artifact is a **Problem Frame**. It serves as the primary input when entering technical architecture discussions.

## Common questions

**How is this different from `/grill-with-docs`?**

`/shape-problem` explores the problem space, while `grill-with-docs` explores the solution space. `shape-problem` asks what the user is trying to accomplish and why existing workflows fail. It forbids discussing schemas, APIs, and libraries. Once the problem is bounded, you take the Problem Frame into `grill-with-docs` to figure out how to build the software to solve it.

**What if the user already asked for a specific feature, like an export button?**

Treat the feature request as an initial symptom, not a verified problem. Ask why the user wants to export data. Are they generating weekly executive reports, reconciling billing discrepancies, or transferring data into another system? Each struggle points to a different product solution.

**Does it create an issue on our issue tracker?**

Not directly. The output is a markdown Problem Frame meant to anchor the subsequent technical design session. If you want to log it as a long-term discovery ticket, paste it into your tracker, but typically it feeds immediately into `/grill-with-docs` or `/wayfinder`.

## It's working if

- It actively deflects technical proposals ("let's use Redis") back to the user struggle ("what latency is the user seeing?").
- The non-goals section excludes real, tempting features rather than obvious nonsense.
- The success metric is verifiable (e.g. "support ticket volume drops by 30%") rather than vague ("users are happier").
- You finish with clarity on what *not* to build.

## Where it fits

`shape-problem` is the discovery step positioned immediately before the engineering pipeline:

```txt
shape-problem → grill-with-docs → to-spec → to-tickets → implement → code-review
```

- Upstream, raw user feedback and bug queues funnel into this skill.
- Downstream, [grill-with-docs](https://aihero.dev/skills-grill-with-docs) takes the Problem Frame and explores implementation choices.
- On large greenfield initiatives, the Problem Frame feeds into [wayfinder](https://aihero.dev/skills-wayfinder) to chart a map of technical decisions.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes across all skills when you are unsure where to begin.
