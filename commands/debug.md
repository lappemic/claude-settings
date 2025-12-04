# Debug Helper

Strategic debugging assistance for tracking down issues.

**Usage**: `/debug <error-message-or-description>` or `/debug` (then paste error)

## Instructions

1. **Understand the problem**:
   - Parse the error message or stack trace from `$ARGUMENTS`
   - If no arguments, ask user to paste the error or describe the issue
   - Identify: error type, file location, line numbers

2. **Analyze the error**:

### For Stack Traces
- Read the relevant source files mentioned in the trace
- Focus on the first few frames (closest to the error)
- Identify the root cause vs. symptoms

### For Runtime Errors
- Check for common causes:
  - Undefined/null access
  - Type mismatches
  - Missing imports/exports
  - Async/await issues
  - Environment variable issues

### For Build Errors
- Check package.json dependencies
- Verify TypeScript config
- Check for circular dependencies
- Verify import paths

### For API Errors
- Check request/response format
- Verify authentication
- Check rate limiting
- Validate request payload

3. **Strategic debugging approach**:

```typescript
// Add targeted console.logs at key points:
console.log('[DEBUG] functionName - input:', JSON.stringify(input, null, 2));
console.log('[DEBUG] functionName - state before:', { key: value });
console.log('[DEBUG] functionName - result:', result);
```

4. **Provide solution**:
   - Explain the root cause
   - Show the fix with code
   - Optionally add defensive checks to prevent recurrence

5. **Cleanup reminder**:
   - Note any console.logs that should be removed after debugging
   - Suggest error handling improvements

## Common Patterns

### Next.js Hydration Errors
- Check for browser-only code in server components
- Look for Date/random values without proper handling
- Check for mismatched HTML structure

### Supabase Errors
- Check RLS policies
- Verify auth state
- Check column names match schema

### TypeScript Errors
- Read the error message carefully
- Check type definitions
- Verify generics and inference

### React Errors
- Check hook rules (no conditional hooks)
- Verify key props in lists
- Check for stale closures in effects
