---
name: ux-states
description: "Design and specify a comprehensive UI state matrix (empty, loading, optimistic UI, error recovery, accessibility, and localization) for frontend components or user flows. Use when designing, reviewing, or implementing user interfaces."
---

# UX States

Every user interface exists in more states than just the ideal "happy path" with complete data. This skill enforces a disciplined state matrix for every user-facing component or flow before code is written or merged.

## The Six UI State Dimensions

When designing, specifying, prototyping, or implementing a frontend surface, systematically define all six dimensions:

### 1. Empty States
- **First use**: What does a new user see before any data has been created? Provide clear guidance and a prominent call to action.
- **Zero search or filter results**: What happens when a search or filter yields nothing? Tell the user which filter caused the empty list and offer a single-click reset.
- **Cleared state**: What does the screen display after a user deletes or clears all existing items?

### 2. Loading & Latency States
- **Initial fetch**: Use skeleton loaders matching the target layout to prevent layout shift (CLS).
- **Subsequent refetch or filtering**: Use subtle inline spinners or progress indicators that preserve existing content rather than blanking the screen.
- **Debounced input**: For search inputs or live validations, define the debounce delay (typically 200ms to 300ms) and show an active evaluating state if the operation exceeds 150ms.
- **Perceived speed**: Defer non-critical widgets so the primary interaction renders first.

### 3. Optimistic UI & Reversion
- **Immediate feedback**: For high-frequency, low-risk mutations (likes, toggles, status tags, additions), apply the change locally before the server responds.
- **Reversion on failure**: If the network request fails, rollback the state immediately, display an error banner or toast explaining the reversion, and provide an inline retry action.
- **In-flight conflict handling**: Prevent stale overwrite if another mutation occurs while the first request is pending.

### 4. Error Recovery & Partial Failures
- **Granular error boundaries**: Wrap independent widgets in separate error boundaries so a failure in a secondary widget (e.g. sidebar or analytics graph) does not crash the entire page.
- **Inline recovery**: Provide explicit "Retry" buttons for transient network failures.
- **Form validation errors**: Display inline field-level errors right beneath the invalid input, alongside an overarching error summary at the top of the form for screen readers.

### 5. Accessibility & Navigation (a11y)
- **Keyboard navigation**: Every interactive element must be reachable and actionable via Tab, Shift+Tab, Enter, and Space.
- **Focus management**: When a modal or dialog opens, trap focus within it. Return focus to the trigger element when closed.
- **ARIA attributes**: Use semantic HTML first (`<button>`, `<dialog>`, `<nav>`). Use `aria-expanded`, `aria-busy`, and `aria-live` regions for dynamic state updates.
- **Contrast and targets**: Maintain minimum 4.5:1 contrast for text and minimum 44x44px touch targets.

### 6. Copy, Localization & Boundary Conditions
- **Text overflow**: How does the layout handle unusually long strings (e.g. 50-character names, localized German phrases)? Specify ellipsis, line-clamp, or dynamic wrapping.
- **Internationalization (i18n)**: Externalize all strings to translation keys. Support pluralization rules and variable interpolation.
- **Date, time, and currency**: Format dates and numbers using the user's locale settings.

## Output Format

When generating a UI state specification, format the output as a Markdown table:

```markdown
### UI State Matrix: <Component / Flow Name>

| State Dimension | Visual & Interaction Behavior | Technical Invariant |
| --- | --- | --- |
| **Empty (First Use)** | Centered illustration, explanatory message, "Create First Item" button | Displays when `items.length === 0` and `filter === null` |
| **Empty (No Results)** | "No matches for '{query}'", "Clear filters" link | Displays when `items.length === 0` and `filter !== null` |
| **Loading (Initial)** | Skeleton cards matching exact grid dimensions | Prevents layout shift during `isLoading` |
| **Loading (Refetch)** | Subtle spinner on action button; existing list dimmed | `isFetching && !isLoading` |
| **Optimistic Action** | Toggle state flips instantly; item badge updates | Local cache update with rollback snapshot |
| **Mutation Failure** | Rollback badge, show warning toast with "Retry" action | Catch block executes cache rollback |
| **Partial Error** | Widget card displays error icon and "Reload widget" button | Isolated React/Component ErrorBoundary |
| **Accessibility (a11y)** | Keyboard trap in modal; `aria-live="polite"` on status changes | WCAG AA compliance, focus restored to trigger |
| **Copy & Overflow** | Title truncates at 2 lines with tooltip; numbers localized via `Intl` | CSS `line-clamp: 2`, i18n key `dashboard.title` |
```
