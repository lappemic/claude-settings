---
name: ship-to-prod
description: Promote a tested feature from staging to main (production). Use when user says "ship to prod", "promote to main", "deploy to production", or wants to move a staging feature to main via PR. Creates a release branch, cherry-picks feature commits, and opens a PR to main.
---

# Ship to Prod

## Overview

This skill promotes a tested feature from `staging` to `main` by:

1. Identify feature commits on staging (not yet on main)
2. Verify commit scope (only feature-related changes)
3. Create release branch from main with cherry-picked commits
4. Run tests on the release branch (blocking)
5. Open PR to main via `/pr` skill
6. Optionally merge after user approval

### Prerequisites

- Git repository with `staging` and `main` branches
- Feature already merged to `staging` and tested
- Feature identifier: PR number, feature name, or commit keyword

## Phase 1: Identify Feature Commits

User provides a feature identifier (PR number, feature name, or keyword).

```bash
# Ensure up to date
git fetch origin

# List all staging-only commits (not yet on main)
git log --oneline origin/main..origin/staging
```

### Search by PR Number

```bash
git log --oneline origin/main..origin/staging --grep="#<PR>"
```

### Search by Keyword

```bash
git log --oneline origin/main..origin/staging --grep="<keyword>"
```

### Present Findings

```
Found the following staging-only commit(s) matching "<identifier>":

- abc1234 feat(generator): add Kuendigungsschreiben generator (#73)

Is this the commit to promote to main?
```

If no commits found:
```
No staging-only commits found matching "<identifier>".

Current staging-only commits:
[list from git log]

Options:
1. Try a different search term
2. Abort
```

If commit is ambiguous or multiple matches, present all and ask user to confirm.

## Phase 2: Verify Commit Scope

Show the files changed by the identified commit(s):

```bash
git show --stat <commit-sha>
```

Verify all changed files relate to the feature scope. Flag anything suspicious:

```
These files seem unrelated to the feature:
- components/header.tsx (shared component)
- package.json (dependency changes)

Options:
1. Proceed anyway (changes are intentional)
2. Abort and clean up on staging first
```

If scope looks clean, proceed without prompting.

## Phase 3: Create Release Branch

```bash
# Create release branch from main
git checkout -b release/<feature-name> origin/main

# Cherry-pick the feature commit(s)
git cherry-pick <commit-sha> [<commit-sha2> ...]
```

### Branch Naming

Use `release/<feature-scope>` derived from the commit message:
- `release/generator-kuendigungsschreiben`
- `release/chat-titles`
- `release/fix-auth-redirect`

### Cherry-Pick Conflicts

If cherry-pick conflicts occur:
- See `references/merge-conflicts.md`
- Never auto-resolve
- Present conflicts to user with options:
  1. Manual resolution
  2. Abort: `git cherry-pick --abort`
  3. Guided walkthrough

### Empty Cherry-Pick

If cherry-pick produces an empty commit:
```
The cherry-pick produced no changes. This commit may already be on main.

Verify with:
  git log --oneline origin/main --grep="<keyword>"

Options:
1. Skip this commit and continue
2. Abort workflow
```

## Phase 4: Test Verification

**This phase is BLOCKING. Tests must pass before proceeding.**

### Run Linting

```bash
npm run lint
```

If lint errors:
```bash
npm run lint:fix
npm run lint  # Verify fix worked
```

If lint still fails after fix:
```
Lint errors remain after auto-fix. Please review:
[show errors]

Options:
1. Fix manually and tell me when ready
2. Abort workflow
```

### Run Tests (If Available)

```bash
grep -q '"test"' package.json && npm test
```

If tests fail:
```
Tests failed:
[show failures]

This is a blocking issue. Options:
1. Fix the tests and tell me when ready
2. Abort workflow
```

**Do not proceed until tests pass.**

If lint fixes were applied, commit them:
```bash
git add -A
git commit -m "fix: lint fixes for release"
```

## Phase 5: Open PR to Main

### Push Branch

```bash
git push -u origin release/<feature-name>
```

### Create PR

Invoke the `/pr` skill to create a pull request with:

- **Base**: `main`
- **Head**: `release/<feature-name>`
- **Title**: Semantic title matching the feature (reuse the original commit message)
- **Body**:
  - Summary of what's being promoted
  - Link to original staging PR if known (e.g., `Promotes staging PR #73`)
  - Note: "Tested on staging"

## Phase 6: Merge (Optional)

Ask user if they want to merge now:

```
PR #<number> created: <url>

Options:
1. Merge now (squash merge)
2. Leave open for review/CI
```

### If Merging

```bash
gh pr merge <pr-number> --squash --delete-branch
```

### Return to Staging

```bash
git checkout staging
git pull origin staging
```

### Completion Message

```
Successfully promoted to production!

Summary:
- Feature: <commit message>
- Release branch: release/<name> (deleted)
- PR: #<number> (merged to main)
- Original staging PR: #<original> (if known)

You're now on staging.
```

## Safety Guardrails

### Never Do

- **Never force push to main** - Production branch is sacred
- **Never auto-resolve cherry-pick conflicts** - Requires human judgment
- **Never skip failed tests** - Tests are blocking
- **Never merge without user confirmation** - Always ask first

### Always Do

- **Confirm commit scope** - Verify only feature changes are included
- **Show what's being promoted** - Transparency in what goes to production
- **Allow abort at any phase** - User can stop anytime
- **Warn on empty cherry-picks** - Commit may already be on main

### Recovery Options

If something goes wrong:
```bash
# Abort cherry-pick in progress
git cherry-pick --abort

# Delete release branch and start over
git checkout staging
git branch -D release/<feature-name>
git push origin --delete release/<feature-name>

# Recover from failed state
git reflog
```

## Edge Cases

### No Staging-Only Commits

```
staging and main are in sync. No commits to promote.

Nothing to do.
```

### Multiple Commits for One Feature

If a feature spans multiple commits on staging:
```
Found multiple commits for this feature:
- abc1234 feat(generator): add Kuendigungsschreiben generator (#73)
- def5678 refactor: simplify generator code

Cherry-pick all of them? (in chronological order)
```

Cherry-pick in order: oldest first.

### Feature Depends on Other Staging Changes

If cherry-pick fails because the feature depends on other changes not yet on main:
```
Cherry-pick failed - this feature may depend on other staging changes
not yet on main.

Options:
1. Identify and include dependency commits
2. Abort and promote dependencies first
3. Abort workflow
```

## Quick Reference

| Phase | Command/Action | Blocking? |
|-------|---------------|-----------|
| 1. Identify | `git log origin/main..origin/staging` | No |
| 2. Verify | `git show --stat` | No |
| 3. Branch | `git checkout -b release/...`, `git cherry-pick` | No |
| 4. Test | `npm run lint`, `npm test` | **Yes** |
| 5. PR | `git push`, `/pr` | No |
| 6. Merge | `gh pr merge --squash` | User confirmation |

## Invocation Examples

User says any of:
- "ship to prod #73"
- "ship to prod PR 73"
- "promote to main"
- "promote feature X to production"
- "deploy to production"
- "ship the generator feature to prod"
- "promote PR 73 to main"

Response:
```
Starting ship-to-prod workflow...

Let me find the feature commits on staging.
[Proceed to Phase 1]
```
