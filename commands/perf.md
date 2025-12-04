# Performance Analysis

Analyze and optimize application performance.

**Usage**: `/perf` or `/perf <area>` where area is: `bundle`, `db`, `react`, `api`

## Instructions

### General Analysis (`/perf`)

1. **Identify hot spots**:
   - Check for large files: `find src -name "*.ts*" -size +50k`
   - Look for n+1 queries in API routes
   - Check for missing React optimizations

2. **Bundle analysis**:
   ```bash
   # Next.js bundle analyzer
   ANALYZE=true npm run build
   ```

3. **Database query analysis**:
   - Check for missing indexes on frequently queried columns
   - Look for sequential queries that could be batched

### Bundle Analysis (`/perf bundle`)

1. **Check bundle size**:
   ```bash
   npm run build
   # Look for large chunks in .next/analyze/
   ```

2. **Common issues**:
   - Large dependencies imported entirely (lodash, moment)
   - Missing dynamic imports for heavy components
   - Unused code not tree-shaken

3. **Fixes**:
   ```typescript
   // Bad: imports entire library
   import _ from 'lodash'

   // Good: import only what's needed
   import debounce from 'lodash/debounce'

   // Better: use native or lighter alternatives
   const debounce = (fn, ms) => { /* native impl */ }
   ```

### Database Performance (`/perf db`)

1. **Query analysis**:
   - List all Supabase queries in the codebase
   - Check for missing `.select()` specificity
   - Identify n+1 patterns

2. **Index recommendations**:
   ```sql
   -- Check for slow queries
   SELECT * FROM pg_stat_user_tables;

   -- Add indexes for foreign keys
   CREATE INDEX idx_<table>_<column> ON <table>(<column>);
   ```

3. **Common fixes**:
   ```typescript
   // Bad: fetch all columns
   const { data } = await supabase.from('users').select('*')

   // Good: select only needed columns
   const { data } = await supabase.from('users').select('id, name, email')

   // Bad: n+1 queries
   for (const user of users) {
     const posts = await supabase.from('posts').select().eq('user_id', user.id)
   }

   // Good: single query with join
   const { data } = await supabase.from('users').select('*, posts(*)')
   ```

### React Performance (`/perf react`)

1. **Component analysis**:
   - Look for missing `memo()` on list items
   - Check for inline functions in props
   - Identify missing `useCallback`/`useMemo`

2. **Profiling**:
   - Use React DevTools Profiler
   - Check for unnecessary re-renders

3. **Common fixes**:
   ```typescript
   // Memoize expensive components
   const MemoizedItem = memo(Item)

   // Stable callbacks
   const handleClick = useCallback(() => {}, [deps])

   // Memoize computed values
   const filtered = useMemo(() => items.filter(...), [items])
   ```

### API Performance (`/perf api`)

1. **Check API routes**:
   - Response times
   - Caching headers
   - Streaming where appropriate

2. **Optimizations**:
   ```typescript
   // Add caching headers
   return new Response(data, {
     headers: {
       'Cache-Control': 'public, s-maxage=60, stale-while-revalidate=300'
     }
   })

   // Stream large responses
   return new StreamingTextResponse(stream)
   ```

## Output Format

```
## Performance Report

### Critical Issues
- [Issue]: [Impact] - [Fix]

### Recommendations
1. [Priority 1 optimization]
2. [Priority 2 optimization]

### Metrics
- Bundle size: X MB
- Largest chunks: [list]
- DB queries per page load: X
```
