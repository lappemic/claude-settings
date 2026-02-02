# Merge Conflict Resolution

## Safety First

**Never auto-resolve merge conflicts.** Conflicts require human judgment to ensure correctness.

## Detection

Conflicts are detected when:
- `git merge` or `git rebase` reports conflicts
- `git pull` fails with conflict markers
- Files contain `<<<<<<<`, `=======`, `>>>>>>>` markers

## Safe Resolution Options

When conflicts occur, present these options to the user:

### Option 1: Manual Resolution (Recommended)
```
I've detected merge conflicts. Would you like to:
1. Resolve them manually in your editor
2. I can explain each conflict to help you decide
```

### Option 2: Abort and Reassess
```bash
git merge --abort   # If during merge
git rebase --abort  # If during rebase
```

### Option 3: Guided Resolution
Walk through each conflict:
1. Show the conflicting sections
2. Explain what each version does
3. Let user decide which to keep
4. Never choose for them

## Conflict Resolution Workflow

1. **Stop** - Don't proceed with any git operations
2. **List** - Show all conflicted files
3. **Explain** - Describe what caused the conflict
4. **Options** - Present resolution choices
5. **Wait** - User must explicitly confirm resolution
6. **Verify** - After resolution, confirm no markers remain

## Post-Resolution

After user resolves conflicts:
```bash
git add <resolved-files>
git commit  # Or continue rebase/merge
```

Always verify the resolution compiles/runs before proceeding.
