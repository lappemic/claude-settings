# Code Review

Review staged or unstaged changes for issues before committing.

**Usage**: `/review` (staged changes) or `/review all` (all changes)

## Instructions

1. **Get the changes to review**:
   - Default: `git diff --cached` (staged changes only)
   - With `all` argument: `git diff` (all uncommitted changes)
   - If no changes, inform user and exit

2. **Analyze each changed file** for:

### Security Issues (Critical)
- Hardcoded secrets, API keys, passwords
- SQL injection vulnerabilities
- XSS vulnerabilities in React/Next.js
- Insecure data handling
- Missing authentication/authorization checks
- Exposed sensitive endpoints

### Logic Errors (High)
- Off-by-one errors
- Null/undefined access without checks
- Race conditions in async code
- Incorrect error handling
- Missing edge cases

### Performance Issues (Medium)
- N+1 queries
- Missing database indexes (for new queries)
- Unnecessary re-renders in React
- Large bundle imports
- Missing memoization for expensive operations

### Code Quality (Low)
- Inconsistent naming conventions
- Missing TypeScript types
- Dead code or unused imports
- Overly complex functions (consider refactoring)
- Missing error boundaries

### Project Patterns
- Deviations from CLAUDE.md conventions
- Inconsistent with existing codebase patterns
- Missing mobile-first responsive design

3. **Output format**:

```
## Code Review Summary

### 🔴 Critical Issues (must fix)
- [file:line] Description of security/critical issue

### 🟡 Warnings (should fix)
- [file:line] Description of logic/performance issue

### 🔵 Suggestions (nice to have)
- [file:line] Description of code quality improvement

### ✅ What looks good
- Brief note on well-implemented aspects
```

4. **If no issues found**:
   - Confirm the changes look good
   - Suggest proceeding with commit

## Notes
- Focus on actionable feedback
- Don't nitpick formatting (that's what linters are for)
- Prioritize security and correctness over style
