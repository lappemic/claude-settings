# Cross-Session Memory

Use `@MEMORY.md` in prompts to reference this file, or `/memory` to edit.

## Active Projects

- **leadershipbuddy**: Next.js + Supabase AI coaching platform
- **sustainabilityjobs**: Job board with scrapers
- **localepay**: Payment/pricing solution
- **bytebricks-new**: Next.js project

## Tech Stack Preferences

- **Frontend**: Next.js App Router, TypeScript, Tailwind, shadcn/ui
- **Backend**: Supabase (Postgres, Auth, Edge Functions)
- **AI**: Vercel AI SDK, LangChain, OpenAI/Anthropic/Gemini
- **Package manager**: npm (consider pnpm)
- **Validation**: Zod everywhere

## Coding Preferences

- Mobile-first responsive design
- Server Components by default, 'use client' only when needed
- Feature-based file organization
- Semantic git commits
- Separate by concern, no long files

## Common Patterns

### Supabase Client
```typescript
// Server: import { createClient } from '@/lib/supabase/server'
// Client: import { createClient } from '@/lib/supabase/client'
```

### AI Route
```typescript
import { streamText } from 'ai'
import { openai } from '@ai-sdk/openai'
// Return: result.toDataStreamResponse()
```

## Known Issues & Solutions

<!-- Add issues as you encounter them -->
- Hydration errors: Check for browser-only code in server components
- RLS issues: Verify auth context and policy conditions

## TODO / Backlog

<!-- Add items to work on later -->
- [ ] Set up testing infrastructure
- [ ] Add E2E tests with Playwright
