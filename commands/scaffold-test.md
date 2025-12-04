# Scaffold Test

Generate a comprehensive test file for the specified file or component.

**Usage**: `/scaffold-test <file-path>` or `/scaffold-test` (uses current context)

## Instructions

1. **Identify the target**: If a file path is provided as argument `$ARGUMENTS`, analyze that file. Otherwise, ask what file/component to test.

2. **Analyze the code**:
   - Identify all exported functions, components, hooks, or classes
   - Map dependencies that need mocking (Supabase, AI SDK, external APIs)
   - Note edge cases, error paths, and boundary conditions

3. **Determine test type**:
   - **React components** → React Testing Library + Vitest
   - **API routes** → Integration tests with mocked request/response
   - **Server Actions** → Unit tests with mocked Supabase client
   - **Utility functions** → Pure unit tests
   - **Hooks** → renderHook from React Testing Library

4. **Generate test file** at appropriate location:
   - Colocated: `<filename>.test.ts` next to source
   - Or in `__tests__/<filename>.test.ts`

5. **Include in the test file**:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
// ... imports

// Mock setup
vi.mock('@/lib/supabase', () => ({
  createClient: vi.fn(() => mockSupabaseClient)
}))

describe('ComponentOrFunction', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('happy path', () => {
    it('should ...', async () => {
      // Arrange
      // Act
      // Assert
    })
  })

  describe('error handling', () => {
    it('should handle ... error', async () => {
      // Test error scenarios
    })
  })

  describe('edge cases', () => {
    it('should handle empty input', () => {})
    it('should handle null/undefined', () => {})
  })
})
```

6. **Mock patterns to use**:

For Supabase:
```typescript
const mockSupabaseClient = {
  from: vi.fn(() => ({
    select: vi.fn().mockReturnThis(),
    insert: vi.fn().mockReturnThis(),
    update: vi.fn().mockReturnThis(),
    delete: vi.fn().mockReturnThis(),
    eq: vi.fn().mockReturnThis(),
    single: vi.fn().mockResolvedValue({ data: mockData, error: null })
  })),
  auth: {
    getUser: vi.fn().mockResolvedValue({ data: { user: mockUser }, error: null })
  }
}
```

For AI SDK:
```typescript
vi.mock('ai', () => ({
  streamText: vi.fn().mockResolvedValue({
    toDataStreamResponse: vi.fn(() => new Response('mock stream'))
  })
}))
```

For React components:
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

it('should render and handle interaction', async () => {
  const user = userEvent.setup()
  render(<Component />)

  await user.click(screen.getByRole('button', { name: /submit/i }))
  await waitFor(() => {
    expect(screen.getByText('Success')).toBeInTheDocument()
  })
})
```

7. **Provide run command**:
```bash
npx vitest run <test-file-path>
# or for watch mode
npx vitest <test-file-path>
```
