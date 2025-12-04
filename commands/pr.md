# Create Pull Request

Analyze the current branch and create a well-structured pull request.

**Usage**: `/pr` or `/pr <base-branch>`

## Instructions

1. **Gather context** (run in parallel):
   - `git status` - check for uncommitted changes
   - `git branch --show-current` - get current branch name
   - `git log main..HEAD --oneline` or `git log <base>..HEAD --oneline` - commits to include
   - `git diff main...HEAD --stat` - files changed summary

2. **Check prerequisites**:
   - If there are uncommitted changes, ask if user wants to commit first
   - Verify we're not on main/master branch
   - Check if branch is pushed to remote (`git status -sb`)

3. **Analyze all commits** in the PR:
   - Read the full diff: `git diff main...HEAD`
   - Understand the scope: feature, bugfix, refactor, etc.
   - Identify breaking changes or migrations

4. **Generate PR content**:
   - **Title**: Concise, semantic format (e.g., "feat: add user authentication flow")
   - **Summary**: 2-4 bullet points explaining what and why
   - **Test plan**: How to verify the changes work

5. **Create the PR** using gh CLI:

```bash
# Push if needed
git push -u origin $(git branch --show-current)

# Create PR
gh pr create --title "title here" --body "$(cat <<'EOF'
## Summary
- Point 1
- Point 2

## Test plan
- [ ] Step 1
- [ ] Step 2

EOF
)"
```

6. **Return the PR URL** to the user

## Notes
- Default base branch is `main`, use argument to override (e.g., `/pr develop`)
- If PR already exists, show its URL instead of creating a new one
- Check with `gh pr view --web` first
