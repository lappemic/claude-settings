# New AI Route

Scaffold a new AI-powered API route with streaming support.

**Usage**: `/new-ai-route <route-name>` (e.g., `/new-ai-route chat` creates `app/api/chat/route.ts`)

## Instructions

1. **Get route details**:
   - Route name from `$ARGUMENTS` or ask user
   - Ask which provider: OpenAI, Anthropic, or Google
   - Ask if tools/function calling is needed

2. **Create route file** at `app/api/<route-name>/route.ts`:

### Basic streaming chat route:

```typescript
import { openai } from '@ai-sdk/openai'
// or: import { anthropic } from '@ai-sdk/anthropic'
// or: import { google } from '@ai-sdk/google'
import { streamText } from 'ai'

export const runtime = 'edge' // optional: for edge runtime

export async function POST(req: Request) {
  try {
    const { messages } = await req.json()

    const result = await streamText({
      model: openai('gpt-4o'), // or anthropic('claude-3-5-sonnet-20241022') or google('gemini-1.5-pro')
      system: 'You are a helpful assistant.',
      messages,
    })

    return result.toDataStreamResponse()
  } catch (error) {
    console.error('AI route error:', error)
    return new Response(
      JSON.stringify({ error: 'Failed to process request' }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    )
  }
}
```

### With tools/function calling:

```typescript
import { openai } from '@ai-sdk/openai'
import { streamText, tool } from 'ai'
import { z } from 'zod'

export async function POST(req: Request) {
  try {
    const { messages } = await req.json()

    const result = await streamText({
      model: openai('gpt-4o'),
      system: 'You are a helpful assistant with access to tools.',
      messages,
      tools: {
        getWeather: tool({
          description: 'Get current weather for a location',
          parameters: z.object({
            location: z.string().describe('City name'),
          }),
          execute: async ({ location }) => {
            // Implement tool logic
            return { temperature: 72, condition: 'sunny', location }
          },
        }),
      },
      maxSteps: 5, // allow multi-step tool use
    })

    return result.toDataStreamResponse()
  } catch (error) {
    console.error('AI route error:', error)
    return new Response(
      JSON.stringify({ error: 'Failed to process request' }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    )
  }
}
```

### With authentication (Supabase):

```typescript
import { createClient } from '@/lib/supabase/server'
import { openai } from '@ai-sdk/openai'
import { streamText } from 'ai'

export async function POST(req: Request) {
  const supabase = await createClient()
  const { data: { user }, error: authError } = await supabase.auth.getUser()

  if (authError || !user) {
    return new Response(
      JSON.stringify({ error: 'Unauthorized' }),
      { status: 401, headers: { 'Content-Type': 'application/json' } }
    )
  }

  try {
    const { messages } = await req.json()

    const result = await streamText({
      model: openai('gpt-4o'),
      system: `You are helping user ${user.email}.`,
      messages,
    })

    return result.toDataStreamResponse()
  } catch (error) {
    console.error('AI route error:', error)
    return new Response(
      JSON.stringify({ error: 'Failed to process request' }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    )
  }
}
```

3. **Create client hook usage example**:

```typescript
// In your component:
'use client'

import { useChat } from 'ai/react'

export function Chat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/<route-name>',
  })

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id}>
          <strong>{m.role}:</strong> {m.content}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Say something..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading}>
          Send
        </button>
      </form>
    </div>
  )
}
```

4. **Remind about environment variables**:
   - `OPENAI_API_KEY` for OpenAI
   - `ANTHROPIC_API_KEY` for Anthropic
   - `GOOGLE_GENERATIVE_AI_API_KEY` for Google
