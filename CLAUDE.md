## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update 'tasks/lessons.md' with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests -> then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management
1. **Plan First**: Write plan to 'tasks/todo.md' with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review to 'tasks/todo.md'
6. **Capture Lessons**: Update 'tasks/lessons.md' after corrections

## Core Principles
- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

# Dev Environment
- Don't start the dev server — assume it is always already running on port 3000
- Use `npm run lint` or `npx eslint` — just run them, never ask

# Git & Commits
- Always use semantic commit messages
- Don't state "you" in commits or PRs
- Don't add co-authoring or creation mentions to commit or PR messages

# Code Quality
- Always develop with mobile first in mind
- Write reusable, manageable code — separate by concern
- File size limits: components max 200 lines, utilities max 100, API routes max 150 — extract logic to services if larger

# Supabase
- Always use the supabase cli: `supabase <commands>`
- To push to remote: `supabase db push`
- When writing db migrations, always use the full timestamp down to the second in the filename

# Next.js App Router
- Use App Router (`app/` directory) with Server Components by default
- Add `'use client'` only when needed (hooks, event handlers, browser APIs)
- Route handlers go in `app/api/*/route.ts` with named exports (GET, POST, etc.)
- Use Server Actions for mutations: `'use server'` in action files or inline
- Colocate loading.tsx and error.tsx with page.tsx for loading/error states
- Use generateMetadata() for dynamic SEO, export metadata for static

# Vercel AI SDK
- Use `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google` for providers
- Prefer `streamText()` for chat, `generateText()` for one-shot completions
- Tool/function calling pattern:
  ```ts
  const result = await streamText({
    model: openai('gpt-4o'),
    messages,
    tools: {
      toolName: {
        description: 'what it does',
        parameters: z.object({ ... }),
        execute: async (params) => { ... }
      }
    }
  })
  ```
- Return `result.toDataStreamResponse()` from route handlers
- Use `useChat()` hook on client for streaming UI

# shadcn/ui
- Install components via: `npx shadcn@latest add <component>`
- Components go in `components/ui/` (shadcn defaults)
- Use `cn()` utility for conditional classes
- Forms: react-hook-form + zod + shadcn Form components
- Customize via CSS variables in globals.css, not component edits

# Testing (Proactive)
- After implementing features, proactively suggest writing tests
- Use Vitest for unit/integration tests, Playwright for E2E
- Test files: `*.test.ts` or `*.spec.ts` colocated or in `__tests__/`
- Mock AI SDK with MSW or vi.mock(), mock Supabase client
- Suggest test coverage for: API routes, server actions, utility functions, critical UI flows

# Security Rules
- Never commit .env, credentials, or API keys
- Sanitize all user input before database queries
- Use parameterized queries only (Supabase handles this)
- Validate with Zod on both client and server
- Check authentication before any data mutation

# Error Handling
- Wrap external API calls in try/catch
- Use error boundaries for React component trees
- Log errors with context: `{ error, userId, action, timestamp }`
- Return user-friendly messages, log technical details

# Performance
- Use dynamic imports for heavy components: `dynamic(() => import(...))`
- Implement virtual scrolling for lists > 50 items
- Cache with useMemo/useCallback for expensive computations
- Debounce search inputs (300ms default)
- Use Suspense boundaries for loading states

# Umami Cloud API (Browser Automation)

When configuring Umami Cloud via browser automation, modal dialogs block the Chrome extension. Use the internal API directly via JavaScript `fetch()` instead.

## Auth
- Token stored in `localStorage.getItem('umami.auth')` (raw string, not JSON)
- Pass as `Authorization: Bearer ${token}` header

## Base URL
- Format: `https://cloud.umami.is/analytics/{region}/api` (e.g., `eu` for EU region)
- Website ID from the URL: `/websites/{websiteId}/...`

## Segments & Cohorts
Segments and cohorts share the same endpoint, differentiated by `type`:
```js
// POST /api/websites/{websiteId}/segments
{ websiteId, name, type: 'segment'|'cohort', parameters: { filters: [{ type, value }] } }
// Filter types: device, language, referrer, browser, country, path, event, etc.
// GET /api/websites/{websiteId}/segments?type=segment (or type=cohort)
// DELETE /api/websites/{websiteId}/segments/{segmentId}
```

## Reports (Goals, Funnels, Journeys, Retention, UTM, Attribution, Breakdown, Revenue)
All report types share the `/api/reports` endpoint:
```js
// POST /api/reports
{ websiteId, name, type: 'goal'|'funnel'|'journey'|'retention'|'utm'|..., parameters: {...} }
// GET /api/reports?websiteId={id}
// DELETE /api/reports/{reportId}
```

**Goal parameters** (schema: `goalReportSchema`):
```js
{ startDate, endDate, type: 'event', value: 'event_name' }
// Optional: operator ('count'|'sum'|'average'), property (string)
```

**Funnel parameters** (schema: `funnelReportSchema`):
```js
{ startDate, endDate, window: 7, steps: [{ type: 'path'|'event', value: '...' }] }
// Min 2 steps, max 8
```

**Journey parameters** (schema: `journeyReportSchema`):
```js
{ startDate, endDate, steps: 5, startStep: '/path', endStep: '' }
// steps: 2-7
```

**Retention/UTM** just need `{ startDate, endDate }` (retention optionally takes `timezone`)

## Pattern: Batch Creation via fetch()
When the UI has modal issues, execute in the page context:
```js
(async () => {
  const token = localStorage.getItem('umami.auth');
  const baseUrl = 'https://cloud.umami.is/analytics/eu/api';
  const res = await fetch(`${baseUrl}/reports`, {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' },
    body: JSON.stringify({ websiteId, name, type, parameters })
  });
  return await res.json();
})();
```

## Schema Reference
Full Zod schemas: `github.com/umami-software/umami` -> `src/lib/schema.ts`
