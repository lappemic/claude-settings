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

# General Rules
- dont start the dev server. assume it is always already running on port 3000
- always use semantic commit messages
- always develop with mobile first in mind
- write reusable, managable code, dont write too long files, separate by concern
- dont state you in commits or prs
- dont add co creation or creation of claude code to commit or pr messages
- use npm run lint or npx eslint
- never ask for linting commands, just use them

# Supabase
- always use the supabase cli: `supabase <commands>`
- to push to remote: `supabase db push`
- when writing db migrations, always use the full timestamp down to the second in the filename

# Next.js App Router
- use App Router (`app/` directory) with Server Components by default
- add `'use client'` only when needed (hooks, event handlers, browser APIs)
- route handlers go in `app/api/*/route.ts` with named exports (GET, POST, etc.)
- use Server Actions for mutations: `'use server'` in action files or inline
- colocate loading.tsx and error.tsx with page.tsx for loading/error states
- use generateMetadata() for dynamic SEO, export metadata for static

# Vercel AI SDK
- use `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google` for providers
- prefer `streamText()` for chat, `generateText()` for one-shot completions
- tool/function calling pattern:
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
- return `result.toDataStreamResponse()` from route handlers
- use `useChat()` hook on client for streaming UI

# LangChain
- use LangChain.js (`langchain`, `@langchain/openai`, `@langchain/anthropic`)
- for RAG: document loaders → text splitters → embeddings → vector store → retriever
- use `RunnableSequence` for composing chains
- conversation memory: `BufferMemory` or `ConversationSummaryMemory`

# shadcn/ui
- install components via: `npx shadcn@latest add <component>`
- components go in `components/ui/` (shadcn defaults)
- use `cn()` utility for conditional classes
- forms: react-hook-form + zod + shadcn Form components
- customize via CSS variables in globals.css, not component edits

# Testing (Proactive)
- after implementing features, proactively suggest writing tests
- use Vitest for unit/integration tests, Playwright for E2E
- test files: `*.test.ts` or `*.spec.ts` colocated or in `__tests__/`
- mock AI SDK with MSW or vi.mock(), mock Supabase client
- suggest test coverage for: API routes, server actions, utility functions, critical UI flows

# Security Rules
- never commit .env, credentials, or API keys
- sanitize all user input before database queries
- use parameterized queries only (Supabase handles this)
- validate with Zod on both client and server
- check authentication before any data mutation

# Error Handling
- wrap external API calls in try/catch
- use error boundaries for React component trees
- log errors with context: `{ error, userId, action, timestamp }`
- return user-friendly messages, log technical details

# Performance
- use dynamic imports for heavy components: `dynamic(() => import(...))`
- implement virtual scrolling for lists > 50 items
- cache with useMemo/useCallback for expensive computations
- debounce search inputs (300ms default)
- use Suspense boundaries for loading states

# File Size Limits
- components: max 200 lines, split by concern if larger
- utility files: max 100 lines
- API routes: max 150 lines, extract logic to services