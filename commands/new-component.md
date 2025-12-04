# New Component

Create a new shadcn-style React component with TypeScript, variants, and mobile-first styling.

**Usage**: `/new-component <ComponentName>` (e.g., `/new-component UserCard`)

## Instructions

1. **Get component details**:
   - Component name from `$ARGUMENTS` or ask user
   - Ask: Is this a client or server component?
   - Ask: What variants does it need? (e.g., size: sm/md/lg, variant: default/outline/ghost)

2. **Determine location**:
   - UI primitives → `components/ui/<component-name>.tsx`
   - Feature components → `components/<component-name>.tsx` or `features/<feature>/components/`

3. **Create component file**:

### Server Component (default):

```typescript
import { cn } from '@/lib/utils'
import { cva, type VariantProps } from 'class-variance-authority'

const componentNameVariants = cva(
  // Base styles (mobile-first)
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-base',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
)

interface ComponentNameProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof componentNameVariants> {
  // Add custom props here
}

export function ComponentName({
  className,
  variant,
  size,
  children,
  ...props
}: ComponentNameProps) {
  return (
    <div
      className={cn(componentNameVariants({ variant, size, className }))}
      {...props}
    >
      {children}
    </div>
  )
}
```

### Client Component (with interactivity):

```typescript
'use client'

import { useState } from 'react'
import { cn } from '@/lib/utils'
import { cva, type VariantProps } from 'class-variance-authority'

const componentNameVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        outline: 'border border-input bg-background hover:bg-accent',
      },
      size: {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-base',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
)

interface ComponentNameProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof componentNameVariants> {
  isLoading?: boolean
}

export function ComponentName({
  className,
  variant,
  size,
  isLoading,
  children,
  ...props
}: ComponentNameProps) {
  const [isHovered, setIsHovered] = useState(false)

  return (
    <button
      className={cn(componentNameVariants({ variant, size, className }))}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      disabled={isLoading}
      {...props}
    >
      {isLoading ? (
        <span className="animate-spin mr-2">⏳</span>
      ) : null}
      {children}
    </button>
  )
}
```

### With Composition (compound component):

```typescript
import { cn } from '@/lib/utils'

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {}

function Card({ className, ...props }: CardProps) {
  return (
    <div
      className={cn('rounded-lg border bg-card text-card-foreground shadow-sm', className)}
      {...props}
    />
  )
}

function CardHeader({ className, ...props }: CardProps) {
  return <div className={cn('flex flex-col space-y-1.5 p-4 md:p-6', className)} {...props} />
}

function CardTitle({ className, ...props }: React.HTMLAttributes<HTMLHeadingElement>) {
  return <h3 className={cn('text-lg md:text-xl font-semibold leading-none tracking-tight', className)} {...props} />
}

function CardContent({ className, ...props }: CardProps) {
  return <div className={cn('p-4 md:p-6 pt-0', className)} {...props} />
}

export { Card, CardHeader, CardTitle, CardContent }
```

4. **Mobile-first responsive patterns**:
   - Start with mobile styles, add `md:` and `lg:` breakpoints
   - Use flexible sizing: `w-full md:w-auto`
   - Stack on mobile, row on desktop: `flex flex-col md:flex-row`
   - Adjust padding: `p-4 md:p-6 lg:p-8`

5. **Export from index** (if using barrel exports):
   ```typescript
   // components/index.ts
   export { ComponentName } from './component-name'
   ```

6. **Suggest test file**:
   After creating, ask if user wants a test file generated with `/scaffold-test`.
