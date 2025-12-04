# Refactor Assistant

Guided refactoring with safety checks and incremental changes.

**Usage**: `/refactor <file-or-pattern>` or `/refactor` (will ask what to refactor)

## Instructions

1. **Identify target**:
   - If `$ARGUMENTS` provided, locate the file(s) matching the pattern
   - If no arguments, ask user what they want to refactor

2. **Analyze current state**:
   - Read the target file(s) completely
   - Identify code smells:
     - Functions > 50 lines
     - Files > 200 lines
     - Deeply nested conditionals (> 3 levels)
     - Duplicate code patterns
     - Mixed concerns in single file
     - Unclear naming
     - Missing TypeScript types

3. **Check for tests**:
   - Look for existing test files (`*.test.ts`, `*.spec.ts`)
   - If tests exist, run them first: `npm test -- --grep "filename"`
   - If no tests, warn user and suggest writing tests first

4. **Propose refactoring plan**:
   ```
   ## Refactoring Plan for [filename]

   ### Issues Found
   - [List specific issues with line numbers]

   ### Proposed Changes
   1. [Change 1 - what and why]
   2. [Change 2 - what and why]

   ### Risk Assessment
   - Breaking changes: [Yes/No - details]
   - Test coverage: [Existing/None/Partial]
   ```

5. **Execute incrementally**:
   - Make ONE change at a time
   - After each change, verify:
     - TypeScript compiles: `npx tsc --noEmit`
     - Tests still pass (if they exist)
     - No runtime errors
   - Commit each logical change separately with `/smart-commit`

6. **Common refactoring patterns**:

### Extract Function
```typescript
// Before: inline logic
const result = data.filter(x => x.active).map(x => x.name).join(', ')

// After: extracted with clear name
const getActiveNames = (data: Item[]) =>
  data.filter(x => x.active).map(x => x.name).join(', ')
```

### Extract Component
```typescript
// Before: large component with multiple concerns
// After: split into Container + Presentational components
```

### Simplify Conditionals
```typescript
// Before: nested if/else
// After: early returns or switch/case
```

## Safety Checklist

- [ ] Tests pass before refactoring
- [ ] Each change is atomic and reversible
- [ ] TypeScript types maintained/improved
- [ ] No functionality changes (pure refactor)
- [ ] Tests pass after refactoring
