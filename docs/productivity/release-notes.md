## What it does

`release-notes` synthesizes git commit history, merged pull requests, and closed issues since the last tag into customer-facing release notes.

The defining constraint is that it translates internal implementation details into user-facing value and product outcomes. A release note that lists raw commit messages or database column migrations is useless to customers. This skill filters out internal refactoring noise, organizes changes by user impact, uses the project's canonical domain language from `GLOSSARY.md`, and documents required migration steps.

## When to reach for it

You invoke this by typing `/release-notes`, and the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

Reach for this when preparing a new software release, publishing a changelog, or writing customer updates:

| Your situation | What to run |
| --- | --- |
| Preparing customer-facing release notes for a version or tag | `/release-notes` |
| Structuring an individual pull request description | [pr](https://aihero.dev/skills-pr) |
| Reviewing session performance and agent environment improvements | [retro](https://aihero.dev/skills-retro) |
| Documenting an operational outage | [post-mortem](https://aihero.dev/skills-post-mortem) |

## Prerequisites

The skill reads git tags and commit history, and accesses closed tickets through the workflow configured in `docs/agents/issue-tracker.md` by [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills). If `GLOSSARY.md` exists, it uses the established domain vocabulary.

## Translating technical diffs into user value

Engineers describe changes by what they built; users care about what they can now do. `release-notes` systematically converts technical statements into product benefits:

- **Technical commit**: "Refactored invoice table queries with indexed composite key."
- **Customer note**: "Invoice history now loads up to 5x faster for large accounts."
- **Technical commit**: "Added null check to user profile avatar upload handler."
- **Customer note**: "Fixed an issue where uploading large profile pictures could fail silently."

Internal chores such as CI dependency bumps, test-only adjustments, and formatting commits are suppressed unless they affect the end-user experience.

## Sourcing context from PR bodies

When pull requests are written using the [pr](https://aihero.dev/skills-pr) skill, they contain three rich sections:
1. **Summary**: The smallest visual or concise explanation of what changed.
2. **Evidence**: Before and after proof that the change works.
3. **Merge Danger**: Classification of One-Way vs Two-Way doors.

`release-notes` extracts these summaries and uses the Merge Danger notes to populate the **Breaking Changes & Migration Guide** section, ensuring customers receive advance warning of configuration or workflow changes.

## Common questions

**Does it modify `CHANGELOG.md` automatically?**

It drafts the complete release notes in the conversation first for your review. Once approved, it can append the notes to `CHANGELOG.md` or format them for GitHub / GitLab Releases.

**What if we do not use git tags?**

If no git tags exist, the skill prompts you for a starting commit hash or date. However, using semantic version tags (e.g. `v1.2.0`) produces the cleanest automated boundaries.

**How does it handle breaking changes?**

Any change identified as a One-Way Door or breaking API change is placed in a prominent "Breaking Changes & Migration Guide" section with explicit instructions on how users must update their configuration.

## It's working if

- Internal refactors and CI maintenance commits are excluded from the output.
- Notes describe customer capabilities and resolved symptoms rather than function names.
- It uses terms defined in `GLOSSARY.md` instead of technical code identifiers.
- Breaking changes provide step-by-step upgrade instructions.

## Where it fits

`release-notes` sits at the final stage of the delivery lifecycle:

```txt
code-review → qa-session → pr → release-notes → retro
```

- Upstream, [pr](https://aihero.dev/skills-pr) provides structured summaries and merge danger assessments.
- Closed tickets from [to-tickets](https://aihero.dev/skills-to-tickets) provide context on resolved user stories.
- The output can be fed into customer newsletters or GitHub Release notes.
- [ask-matt](https://aihero.dev/skills-ask-matt) routes to this skill whenever release documentation is needed.
