---
description: Function signature and design patterns for clean, predictable code
globs: ["src/**"]
---

# Function Design

## Always Use Named Object Parameters

Every function takes a single destructured object. No positional arguments.

```typescript
// BAD: Positional args
const createUser = (name: string, email: string, role: string) => {};
createUser("John", "john@test.com", "admin");

// BAD: Even single params
const getUserById = (id: string) => {};

// GOOD: Named object params
const createUser = ({ name, email, role }: CreateUserParams) => {};
createUser({ name: "John", email: "john@test.com", role: "admin" });

const getUserById = ({ id }: { id: string }) => {};
```

**Only exception:** Built-in callbacks (`.map()`, `.filter()`, `.reduce()`, etc.)

## No Boolean Parameters

Boolean params are unreadable at the call site. Use separate functions or string unions.

```typescript
// BAD: What does `true` mean here?
formatDate({ date, showTime: true, useRelative: false });
fetchUsers({ includeDeactivated: true });

// GOOD: Separate functions
formatDateTime({ date });
formatRelativeDate({ date });

// GOOD: String union when options are related
formatDate({ date, format: "date-only" });
formatDate({ date, format: "date-time" });
fetchUsers({ status: "all" });
fetchUsers({ status: "active" });
```

## Guard Clauses First, Happy Path Last

Validate and bail out early. The main logic should be at the lowest indentation level.

```typescript
// BAD: Happy path buried in nesting
const getRateLimit = ({ user, plan }: RateLimitParams) => {
  if (user) {
    if (plan.endpoints.length > 0) {
      if (user.tier === "enterprise") {
        return plan.baseLimit * 10;
      } else {
        return plan.baseLimit * 2;
      }
    }
  }
  return 0;
};

// GOOD: Guards first
const getRateLimit = ({ user, plan }: RateLimitParams) => {
  if (!user) return 0;
  if (plan.endpoints.length === 0) return 0;

  const multiplier = user.tier === "enterprise" ? 10 : 2;
  return plan.baseLimit * multiplier;
};
```

## Pure by Default

Functions should not mutate inputs or cause side effects. Side effects belong in React hooks, event handlers, or explicitly named functions (`sendEmail`, `logEvent`).

```typescript
// BAD: Mutates input
const markComplete = ({ task }: { task: Task }) => {
  task.status = "done"; // mutating the argument
  return task;
};

// GOOD: Returns new value
const completeTask = ({ task }: { task: Task }) => {
  return { ...task, status: "done" as const };
};
```

## Single Clear Return Type

Functions should return one clear shape. Avoid `string | null | undefined | false`.

```typescript
// BAD: Ambiguous return
const findUser = ({ id }: { id: string }) => {
  if (!id) return undefined;
  const user = db.find(id);
  if (!user) return null;
  if (user.deleted) return false;
  return user;
};

// GOOD: One return type
const findUser = ({ id }: { id: string }): User | null => {
  if (!id) return null;
  const user = db.find(id);
  if (!user || user.deleted) return null;
  return user;
};
```

## One Function, One Job

If you can't describe what a function does without using "and", split it.

```typescript
// BAD: Does two things
const validateAndSubmitForm = async ({ data }: { data: FormData }) => {
  const errors = validateFields({ data });
  if (errors.length > 0) throw new ValidationError(errors);
  return submitToApi({ data });
};

// GOOD: Split responsibilities
const validateForm = ({ data }: { data: FormData }) => {
  const errors = validateFields({ data });
  if (errors.length > 0) throw new ValidationError(errors);
  return data;
};

const submitForm = async ({ data }: { data: FormData }) => {
  return submitToApi({ data });
};
```
