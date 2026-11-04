---
name: release-notes
description: Generate user-facing release notes from merged pull requests, commits, and closed issues since the last release, translating technical diffs into customer value.
disable-model-invocation: true
---

# Release Notes

Engineers write commit messages and pull requests for other engineers. Customers and stakeholders need to know how the software changed from their perspective: what problems were solved, what new capabilities were added, and what breaking changes require action.

This skill synthesizes Git history, merged pull requests, and closed tickets since the last release into polished, customer-facing release notes.

The issue tracker configuration should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## Process

### 1. Establish the Release Boundary
Determine the range of changes to include:
- Find the most recent tag: `git describe --tags --abbrev=0` (or ask the user for the base release tag or commit SHA).
- Determine the target ref (defaults to `HEAD` or the current branch).
- List all commits in the release window: `git log <last-tag>..HEAD --oneline`.

### 2. Gather Rich Context
Read the primary sources:
1. **Pull Request bodies**: If using GitHub or GitLab, inspect merged PRs in this range. Specifically extract the `Summary`, `Evidence`, and `Merge Danger` sections written by the `pr` skill.
2. **Issue Tracker**: Check closed issues and tickets closed within this release window, using the workflow in `docs/agents/issue-tracker.md`.
3. **Domain Vocabulary**: Read `GLOSSARY.md` in the repository to ensure release notes speak the project's canonical customer language rather than raw database or code names.

### 3. Filter and Translate Technical Noise
Exclude internal engineering tasks that provide zero visible change to end users:
- Suppress CI/CD workflow tweaks, internal tooling updates, formatting/lint changes, and test-only refactors.
- Translate technical implementations into product benefits:
  - Technical: "Migrated invoice query to use composite index on tenant_id and created_at."
  - User-facing: "Invoice history now loads up to 5x faster for large organizations."

### 4. Structure the Release Notes
Group changes into clear, user-centric categories:

```markdown
# Release Notes: <Version / Tag / Date>

## Highlights
A 2-3 sentence executive summary of the primary theme or milestone achieved in this release.

## New Features
- **<Feature Title>**: Description of what users can now do, the problem it solves, and how to access it. Link to documentation where applicable.

## Improvements
- **<Improvement Title>**: Enhancements to performance, UX polish, responsiveness, or workflow efficiency.

## Bug Fixes
- **<Fix Title>**: User-facing defect resolved. State the symptom that was experienced and confirm it is corrected.

## Breaking Changes & Migration Guide
Include this section only if breaking changes, deprecations, or manual user actions exist:
- **<Change Description>**: Detail what changed, why the change occurred, and the exact steps required to upgrade (e.g. required environment variables, configuration updates, or database backups identified during Expand-Contract phases).
```

### 5. Final Polish
- Ensure no internal jargon, unexpanded acronyms, or raw database column names leak into the copy.
- Present the draft to the user for review.
- Offer to save the output to `CHANGELOG.md` or output directly as a GitHub / GitLab Release draft.
