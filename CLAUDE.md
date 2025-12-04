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