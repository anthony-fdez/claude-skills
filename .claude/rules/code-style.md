---
description: Universal code style rules to reduce AI slop and enforce clean, minimal code
globs: ["src/**"]
---

# Code Style

## No Comments (Unless Non-Obvious "Why")

```typescript
// BAD: Narrating what the code does
// Get the user's name
const userName = user.name

// Check if the list is empty
if (tasks.length === 0) {

// BAD: JSDoc that repeats the signature
/**
 * Formats a date for display.
 * @param date - The date to format
 * @returns The formatted date string
 */
const formatDate = ({ date }: { date: Date }) => {

// GOOD: No comment needed — the code is clear
const userName = user.name

// GOOD: Explaining a non-obvious decision
// setTimeout defers focus until after React's commit phase
setTimeout(() => inputRef.current?.focus(), 0)
```

## No console.log in Committed Code

Use the logger for anything that should persist. `console.log` is for local debugging only — remove before committing.

```typescript
// BAD
console.log("user data:", user);
console.log("API response:", response);

// GOOD: Use structured logging
logger.info("[Auth] Session created", {
  metadata: { sessionId: session.id },
});
```

## Early Returns Over Nesting

```typescript
// BAD: Nested conditionals
const processTask = ({ task }: { task: Task }) => {
  if (task) {
    if (task.subtasks.length > 0) {
      if (task.status === "pending") {
        return executeTask(task);
      }
    }
  }
  return null;
};

// GOOD: Guard clauses, happy path at the end
const processTask = ({ task }: { task: Task }) => {
  if (!task) return null;
  if (task.subtasks.length === 0) return null;
  if (task.status !== "pending") return null;

  return executeTask(task);
};
```

## One Function, One Responsibility

Every function does exactly one thing. There's no line limit — a 50-line function that does one thing is fine. A 15-line function that does three things is not.

The test: if you need the word "and" to describe what a function does, it's doing too much.

```typescript
// BAD: This function validates, transforms, AND saves
const processDocument = async ({ doc, user }: ProcessParams) => {
  // validation logic...
  // content transformation logic...
  // storage logic...
}

// GOOD: Each function has one job
// A coordinating function that calls others is fine — its one job is orchestration
const processDocument = async ({ doc, user }: ProcessParams) => {
  const validated = validateDocument({ doc })
  const transformed = applyFormatting({ content: validated.content })
  return saveDocument({ user, content: transformed })
}
```

Don't split functions just to make them shorter. Split them when they're doing more than one thing. A long function that reads top-to-bottom and does one job is better than five tiny functions you have to jump between to understand what's happening.

## Named Exports Only

No `export default` except where the framework requires it (Next.js pages, route handlers).

```typescript
// BAD
export default function formatDate() {}
export default class NotificationService {}

// GOOD
export const formatDate = () => {};
export class NotificationService {}

// EXCEPTION: Next.js pages
export default function DashboardPage() {}
```

## No Dead Code

Delete unused code. Don't comment it out "just in case" — that's what git history is for.

```typescript
// BAD
// const oldCalculation = (x: number) => x * 1.1
const newCalculation = ({ value }: { value: number }) => value * 1.2;

// BAD: Unused imports
import { something } from "./utils"; // never used

// GOOD: Just delete it
const newCalculation = ({ value }: { value: number }) => value * 1.2;
```

## No Unnecessary Abstractions

Don't create helpers, wrappers, or utilities for things used once.

```typescript
// BAD: Wrapper that adds nothing
const fetchUserData = async ({ userId }: { userId: string }) => {
  return fetch(`/api/users/${userId}`);
};
// Called exactly once

// GOOD: Just inline it where it's used
const response = await fetch(`/api/users/${userId}`);
```

Three similar lines of code is better than a premature abstraction.
