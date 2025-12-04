# Smart Semantic Commit

Analyze all unstaged and staged changes in the current repository and create semantic commits, logically grouping related changes.

## Instructions

1. **Gather changes**: Run `git status` and `git diff` to see all modified, added, and deleted files (both staged and unstaged).

2. **Analyze and group**: Examine each changed file and identify logical groups based on:
   - Related functionality (e.g., all changes to authentication)
   - Type of change (bug fix, feature, refactor, style, docs, test, chore)
   - Affected component or module
   - If changes span multiple concerns, split them into separate commits

3. **For each logical group**:
   - Stage only the files belonging to that group using `git add <files>`
   - Create a semantic commit with the appropriate prefix and scope:
     - `feat(scope):` for new features
     - `fix(scope):` for bug fixes
     - `refactor(scope):` for code refactoring
     - `style(scope):` for formatting/style changes
     - `docs(scope):` for documentation
     - `test(scope):` for test additions/changes
     - `chore(scope):` for maintenance tasks
     - `perf(scope):` for performance improvements
   - Write a concise, descriptive commit message focusing on the "why"

4. **Commit message format**:
   - Always include a scope in parentheses: `type(scope): message`
   - Scope should be the module, component, or area affected (e.g., auth, api, ui, db, mcp, config)
   - Use lowercase after the prefix
   - Keep the first line under 72 characters
   - Add a blank line and bullet points for details if multiple files are involved
   - Do NOT include co-authored-by or AI generation notes

5. **Execution order**: If there are dependencies between changes (e.g., a refactor that a feature depends on), commit the foundational changes first.

6. **Report**: After all commits are made, show a summary of what was committed.

## Example output

If there are changes to:
- `src/auth/login.ts` (bug fix)
- `src/auth/session.ts` (bug fix)
- `src/components/Button.tsx` (new feature)
- `README.md` (docs update)

Create 3 commits:
1. `fix(auth): resolve session persistence issue`
2. `feat(ui): add loading state to Button component`
3. `docs(readme): update with new button props`
