---
name: stage-and-ship
description: Automate the full git workflow from uncommitted changes to merged PR. Use when user says "stage and ship", "ship my changes", "ship it", or wants to turn uncommitted/staged changes into a properly branched, tested, reviewed, and merged PR. Handles semantic branch creation, test verification, semantic commits, code review, PR creation, conflict resolution, and squash merge with cleanup.
---

# Stage and Ship

## Overview

This skill automates the complete git workflow from local changes to merged PR:

1. Analyze uncommitted changes AND unpushed commits
2. Create semantic feature branch from staging
3. Run tests (blocking - must pass)
4. Create semantic commit via smart-commit
5. Simplify code via code-simplifier agent
6. Push and create PR via pr skill
7. Review via review skill
8. Squash merge and cleanup

### Prerequisites

- Git repository with `staging` as main development branch
- Changes to ship (uncommitted, staged, or unpushed commits)
- Tests available via `npm run lint` and project test commands

## Workflow Decision Tree

```
START
  │
  ├─ No changes? → Abort gracefully
  │
  ├─ Not on staging? → Confirm with user or switch
  │
  ├─ Has uncommitted changes?
  │   └─ Yes → Stash temporarily, create branch, pop stash
  │
  ├─ Has unpushed commits on staging?
  │   └─ Yes → Create branch from staging~N, cherry-pick
  │
  └─ Continue to Phase 1
```

## Phase 1: Analyze Changes

First, understand what we're working with:

```bash
# Check current branch
git branch --show-current

# Check for uncommitted changes
git status --porcelain

# Check for staged changes
git diff --cached --stat

# Check for unstaged changes
git diff --stat

# Check for unpushed commits (if on staging)
git log origin/staging..HEAD --oneline 2>/dev/null || echo "No remote tracking"
```

### Summarize Changes

Create a summary covering:
- **Uncommitted changes**: Files modified, added, deleted
- **Unpushed commits**: Commit messages and files affected
- **Primary type**: feat, fix, refactor, etc. (see `references/semantic-conventions.md`)
- **Scope**: Main area affected (derived from file paths)

Present summary to user:
```
I found the following changes to ship:

**Uncommitted:**
- Modified: features/chat/components/ChatInput.tsx
- Added: lib/utils/validation.ts

**Unpushed commits (2):**
- abc1234: add input validation
- def5678: fix edge case in chat

**Suggested branch:** feat/chat-input-validation

Proceed with this workflow?
```

## Phase 2: Branch Creation

### If Starting Fresh (No Unpushed Commits)

```bash
# Ensure we're on staging and up to date
git checkout staging
git pull origin staging

# Create semantic branch
git checkout -b <type>/<short-description>
```

### If Has Uncommitted Changes

```bash
# Stash changes
git stash push -m "stage-and-ship: temporary stash"

# Create branch from staging
git checkout staging
git pull origin staging
git checkout -b <type>/<short-description>

# Pop stash
git stash pop
```

### If Has Unpushed Commits on Staging

```bash
# Count unpushed commits
COMMIT_COUNT=$(git rev-list --count origin/staging..HEAD)

# Create branch from before those commits
git branch <type>/<short-description> origin/staging

# Cherry-pick the commits
git checkout <type>/<short-description>
git cherry-pick origin/staging..staging

# Reset staging to origin
git checkout staging
git reset --hard origin/staging
git checkout <type>/<short-description>
```

### Branch Naming

Use semantic conventions from `references/semantic-conventions.md`:
- `feat/` for new features
- `fix/` for bug fixes
- `refactor/` for code improvements
- `chore/` for maintenance

## Phase 3: Test Verification

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

Check for test commands:
```bash
# Check package.json for test script
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

## Phase 4: Semantic Commit

Invoke the `smart-commit` skill to create a properly formatted semantic commit.

The smart-commit skill will:
1. Analyze the staged changes
2. Generate a semantic commit message
3. Create the commit

If there are unstaged changes, stage them first:
```bash
git add -A
```

Then invoke: `/smart-commit`

## Phase 5: Code Simplification

**This phase is MANDATORY. Always run the code-simplifier after committing.**

Invoke the `code-simplifier:code-simplifier` agent using the Task tool:

```
Task tool call:
  subagent_type: "code-simplifier:code-simplifier"
  prompt: "Review and simplify the code changes in the current branch. Focus on the files modified in the most recent commit(s). Improve clarity, consistency, and maintainability while preserving all functionality."
```

After the code-simplifier agent completes, check if any files were changed:

```bash
git status --porcelain
```

If changes were made by the simplifier:
```bash
# Stage simplification changes
git add -A

# Commit separately
git commit -m "refactor: simplify code for clarity"
```

If no changes were made, proceed to Phase 6.

## Phase 6: Push & Create PR

### Push Branch

```bash
git push -u origin <branch-name>
```

If push fails due to conflicts with remote:
- See `references/merge-conflicts.md`
- Never force push
- Present options to user

### Create PR

Invoke the `/pr` skill to create a pull request.

The PR should:
- Target `staging` branch
- Have a clear title matching the primary commit
- Include a summary of changes
- Reference any related issues

## Phase 7: Review PR

Invoke the `/review` skill to review the created PR.

### Handle Review Findings

**Critical issues:**
```
The review found critical issues:
[list issues]

Options:
1. Fix now (recommended)
2. Proceed anyway (not recommended)
3. Abort workflow
```

If fixing:
1. Make fixes
2. Commit with: `fix: address review feedback`
3. Push: `git push`
4. Re-run review

**Minor issues:**
```
The review found minor suggestions:
[list suggestions]

These are non-blocking. Options:
1. Address them now
2. Proceed to merge (can address later)
```

## Phase 8: Merge & Cleanup

### Pre-Merge Checks

```bash
# Ensure branch is up to date with staging
git fetch origin staging
git log HEAD..origin/staging --oneline
```

If staging has new commits:
```bash
git merge origin/staging
```

Handle any conflicts per `references/merge-conflicts.md`.

### Check PR Status

Verify CI checks pass (if applicable):
```bash
gh pr checks <pr-number>
```

If checks pending:
```
PR checks are still running. Options:
1. Wait for checks to complete
2. Proceed anyway (not recommended if checks are required)
```

### Squash Merge

```bash
gh pr merge <pr-number> --squash --delete-branch
```

Or if gh CLI not available:
```bash
# Manual squash merge
git checkout staging
git merge --squash <branch-name>
git commit  # Uses PR title/description
git push origin staging
git branch -d <branch-name>
git push origin --delete <branch-name>
```

### Return to Staging

```bash
git checkout staging
git pull origin staging
```

### Completion Message

```
Successfully shipped your changes!

Summary:
- Branch: <branch-name> (deleted)
- PR: #<number> (merged)
- Commit: <sha> - <message>

You're now on staging with the latest changes.
```

## Safety Guardrails

### Never Do

- **Never force push** - Preserves history and collaborator work
- **Never auto-resolve conflicts** - Requires human judgment
- **Never delete without confirmation** - Branches, commits, or changes
- **Never skip failed tests** - Tests are blocking

### Always Do

- **Confirm destructive actions** - Branch deletion, hard resets
- **Preserve user's work** - Stash before risky operations
- **Allow abort at any phase** - User can stop anytime
- **Show what's happening** - Transparency in git operations

### Recovery Options

If something goes wrong:
```bash
# Recover stashed changes
git stash list
git stash pop

# Recover deleted branch (within 30 days)
git reflog
git checkout -b <branch-name> <sha>

# Abort merge/rebase in progress
git merge --abort
git rebase --abort
```

## Edge Cases

### No Changes Detected

```
No changes found to ship:
- No uncommitted changes
- No unpushed commits

Nothing to do. Make some changes first!
```

### Already on Feature Branch

```
You're on branch 'feat/existing-work', not staging.

Options:
1. Continue with current branch (skip branch creation)
2. Switch to staging first
3. Abort
```

### Uncommitted Changes on Wrong Branch

```
You have uncommitted changes on 'main' instead of 'staging'.

Options:
1. Stash changes, switch to staging, create branch, pop stash
2. Abort and let me handle manually
```

### PR Already Exists

```bash
gh pr list --head <branch-name>
```

If PR exists:
```
A PR already exists for this branch: #<number>

Options:
1. Update existing PR (push new commits)
2. Close existing and create new
3. Abort
```

### Merge Conflicts

See `references/merge-conflicts.md` for detailed handling.

Never auto-resolve. Present options:
1. Manual resolution
2. Abort workflow
3. Guided walkthrough of conflicts

## Quick Reference

| Phase | Command/Action | Blocking? |
|-------|---------------|-----------|
| 1. Analyze | `git status`, `git log` | No |
| 2. Branch | `git checkout -b` | No |
| 3. Test | `npm run lint`, `npm test` | **Yes** |
| 4. Commit | `/smart-commit` | No |
| 5. Simplify | `code-simplifier:code-simplifier` agent (Task tool) | No |
| 6. Push/PR | `git push`, `/pr` | No |
| 7. Review | `/review` | Critical issues only |
| 8. Merge | `gh pr merge --squash` | No |

## Invocation Examples

User says any of:
- "stage and ship"
- "ship my changes"
- "ship it"
- "take my changes to staging"
- "create a PR from my changes and merge it"

Response:
```
Starting stage-and-ship workflow...

Let me analyze your current changes.
[Proceed to Phase 1]
```
