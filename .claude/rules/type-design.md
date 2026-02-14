---
description: How to design types — narrow over wide, derive over duplicate, domain over implementation
globs: ["src/**"]
---

# Type Design

## Types Describe the Domain, Not the Implementation

Name and structure types after business concepts, not technical details.

```typescript
// BAD: Implementation-focused types
type StringOrNull = string | null
type ApiResponseWrapper<T> = { data: T; status: number }
type DatabaseRow = { col1: string; col2: number }

// GOOD: Domain-focused types
type UserEmail = string
type TaskStatus = 'pending' | 'in_progress' | 'review' | 'done'
type GeoLocation = {
  latitude: number
  longitude: number
  altitude: number | null
  accuracy: number
}
```

## Prefer Narrow Types Over Wide Types

The narrower the type, the more the compiler helps you. Use string literals, specific objects, and exact shapes instead of `string`, `object`, or `Record<string, unknown>`.

```typescript
// BAD: Wide types accept anything
type Config = {
  mode: string
  theme: string
  locale: string
}
// mode could be "banana" and TypeScript wouldn't care

// GOOD: Narrow types catch mistakes at compile time
type Config = {
  mode: 'development' | 'staging' | 'production'
  theme: 'light' | 'dark'
  locale: 'en-US' | 'es-MX' | 'fr-FR'
}

// BAD: Accepts any object shape
const processEvent = ({ event }: { event: Record<string, unknown> }) => {}

// GOOD: Specific shape
const processEvent = ({ event }: { event: AnalyticsEvent }) => {}
```

## Derive Types from Runtime Sources

Don't hand-write types that can be derived from schemas, constants, or functions. Single source of truth.

```typescript
// BAD: Hand-written type that duplicates the schema
const TaskSchema = z.object({
  id: z.string(),
  status: z.enum(['pending', 'in_progress', 'done']),
  priority: z.number(),
})

type Task = {
  id: string
  status: 'pending' | 'in_progress' | 'done'
  priority: number
}

// GOOD: Derive from the schema
const TaskSchema = z.object({
  id: z.string(),
  status: z.enum(['pending', 'in_progress', 'done']),
  priority: z.number(),
})

type Task = z.infer<typeof TaskSchema>

// GOOD: Derive from constants
const ROLES = ['admin', 'editor', 'viewer'] as const
type Role = (typeof ROLES)[number] // 'admin' | 'editor' | 'viewer'

// GOOD: Derive from functions
const createTask = ({ title, assigneeId }: { title: string; assigneeId: string }) => {
  return { id: generateId(), title, assigneeId, status: 'pending' as const }
}
type Task = ReturnType<typeof createTask>
```

## Make Impossible States Unrepresentable

Use discriminated unions to ensure your types can't represent invalid combinations.

```typescript
// BAD: Boolean flags allow impossible states
type RequestState = {
  isLoading: boolean
  isError: boolean
  data?: SearchResult[]
  error?: Error
}
// Can have isLoading=true AND isError=true AND data AND error — nonsense

// GOOD: Discriminated union — only valid states exist
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: SearchResult[] }
  | { status: 'error'; error: Error }
```

```typescript
// BAD: Optional fields create ambiguous states
type User = {
  role: 'guest' | 'member' | 'admin'
  email?: string        // required for member/admin, absent for guest
  permissions?: string[] // only for admin
}

// GOOD: Each role has exactly the fields it needs
type User =
  | { role: 'guest' }
  | { role: 'member'; email: string }
  | { role: 'admin'; email: string; permissions: string[] }
```

## Use Indexed Access Types Over Duplicating Nested Types

When you need a sub-type from a larger type, use indexed access instead of creating a separate type by hand.

```typescript
// BAD: Duplicating a nested type
type PlaylistResponse = {
  items: Array<{
    id: string
    title: string
    duration: number
    tags: Array<{ slug: string; label: string }>
  }>
}

// Manually rewriting the item type
type Track = {
  id: string
  title: string
  duration: number
  tags: Array<{ slug: string; label: string }>
}

// GOOD: Index into the parent type
type Track = PlaylistResponse['items'][number]
type Tag = Track['tags'][number]
```

## Avoid `type` Keyword for Simple Aliases

If a type alias is just renaming a primitive with no additional constraints, it adds noise.

```typescript
// BAD: Aliases that add nothing
type Id = string
type Name = string
type Count = number

// GOOD: Just use the primitive in the signature
const getUser = ({ id }: { id: string }) => {}

// GOOD: Aliases are useful when they carry domain meaning AND are used widely
type Currency = 'USD' | 'EUR' | 'GBP' // constrains the values
type TaskId = string & { readonly __brand: 'TaskId' } // branded type for safety
```
