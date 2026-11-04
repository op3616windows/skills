## What it does

`ux-states` produces an exhaustive UI state matrix covering empty, loading, optimistic action, error recovery, accessibility, and localization states for frontend components and flows.

The defining constraint is that it treats the loaded "happy path" as only one state out of six required dimensions. Components designed only for ideal data break when latency spikes, networks disconnect, search queries return zero items, or text strings overflow. This skill forces explicit specification of every failure, transition, and edge state before frontend code is implemented.

## When to reach for it

Type `/ux-states`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when designing, prototyping, or specifying user-facing interfaces.

| Your situation | What to run |
| --- | --- |
| Designing a component, page, or interactive user flow | `ux-states` |
| Building a throwaway HTML prototype to test UI interactions | [prototype](https://aihero.dev/skills-prototype), which calls this skill |
| Writing a full product specification for frontend features | [to-spec](https://aihero.dev/skills-to-spec), which includes this matrix |
| Reviewing code for general repo standards | [code-review](https://aihero.dev/skills-code-review) |

## The six state dimensions

Frontend bugs rarely come from complex math; they come from unhandled state transitions. `ux-states` systematically evaluates six core dimensions:

1. **Empty States**: Distinguishes first-run onboarding, empty search or filter results, and states after all items are deleted. Each requires a distinct message and recovery action.
2. **Loading States**: Specifies skeleton loaders to eliminate Cumulative Layout Shift (CLS), subtle spinners for background refetches, and debounce intervals for live search.
3. **Optimistic UI & Reversion**: Mandates that every instant local state update defines its exact failure rollback mechanism and conflict resolution policy.
4. **Error Recovery & Partial Failures**: Enforces granular error boundaries so that an unhandled error in one card does not crash the entire application screen.
5. **Accessibility (a11y)**: Defines keyboard tab stops, focus traps in modals, ARIA live region announcements, and minimum 44x44px touch targets.
6. **Copy & Localization (i18n)**: Dictates behavior for text truncation, multiline wrapping, pluralization rules, and localized date and currency formats.

## Common questions

**Does this slow down rapid prototyping?**

No. It speeds up delivery by preventing costly redesigns. Finding out during QA that a table lacks a zero-results state or that long email addresses break the layout costs far more than sketching the state matrix up front.

**Should every small button have a 6-row matrix?**

Size the matrix to the surface. A simple button needs only disabled, loading, and active focus states. A rich interactive widget (a data table, checkout form, or modal dialog) requires all six dimensions.

**Can this be automated with tests?**

Yes. Each row in the UI State Matrix translates directly into a test case for [tdd](https://aihero.dev/skills-tdd). For example, verifying that an optimistic toggle rolls back when the network request rejects becomes a simple red-green unit test.

## It's working if

- The frontend specification lists explicit empty states for both first-run and zero-search results.
- Every async mutation specifies what happens visually if the server request fails.
- Modals and popovers specify which element receives focus on open and where focus returns on close.
- Long text truncation rules (e.g. line-clamp or ellipsis) are established before styling is written.

## Where it fits

`ux-states` operates in the frontend design phase between idea sharpening and implementation:

- It is called during [grill-with-docs](https://aihero.dev/skills-grill-with-docs) and [prototype](https://aihero.dev/skills-prototype) when user interfaces are discussed.
- It supplies the interaction details embedded into [to-spec](https://aihero.dev/skills-to-spec).
- During [tdd](https://aihero.dev/skills-tdd), developers turn each state in the matrix into concrete automated component tests.
- [ask-matt](https://aihero.dev/skills-ask-matt) maps this alongside the full skill catalog.
