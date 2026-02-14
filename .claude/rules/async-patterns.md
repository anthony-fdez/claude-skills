---
description: Async/await patterns — parallel execution, error handling, and common pitfalls
globs: ["src/**"]
---

# Async Patterns

## Parallel When Independent, Sequential When Dependent

If operations don't depend on each other, run them in parallel with `Promise.all`.

```typescript
// BAD: Sequential when they could be parallel
const user = await getUser({ id: userId });
const projects = await getProjects({ userId });
const preferences = await getPreferences({ userId });

// GOOD: Independent operations run in parallel
const [user, projects, preferences] = await Promise.all([
  getUser({ id: userId }),
  getProjects({ userId }),
  getPreferences({ userId }),
]);

// GOOD: Sequential when there's a real dependency
const user = await getUser({ id: userId });
const workspace = await getWorkspace({ workspaceId: user.workspaceId });
```

## No Floating Promises

Every promise must be `await`ed, returned, or explicitly handled. Fire-and-forget promises hide errors.

```typescript
// BAD: Floating promise — errors are silently swallowed
saveAnalyticsEvent({ event: "page_view" });
logger.flush();

// GOOD: Await it
await saveAnalyticsEvent({ event: "page_view" });

// GOOD: If you truly don't need to wait, be explicit about ignoring errors
void saveAnalyticsEvent({ event: "page_view" }).catch((error) => {
  logger.warn("[Analytics] Failed to save event", { error });
});
```

## No Unnecessary `async`

Don't mark a function `async` if it doesn't `await` anything. It adds overhead and obscures intent.

```typescript
// BAD: async with no await
const getDefaultConfig = async () => {
  return { timeout: 5000, retries: 3 };
};

// BAD: async just to wrap a return
const getUser = async ({ id }: { id: string }) => {
  return fetchUser({ id }); // already returns a promise
};

// GOOD: Just return the value
const getDefaultConfig = () => {
  return { timeout: 5000, retries: 3 };
};

// GOOD: Just return the promise
const getUser = ({ id }: { id: string }) => {
  return fetchUser({ id });
};

// GOOD: async is needed because we await
const getUserWithProjects = async ({ id }: { id: string }) => {
  const user = await fetchUser({ id });
  const projects = await fetchProjects({ userId: user.id });
  return { ...user, projects };
};
```

## Add Timeouts to External Calls

Network requests can hang forever without a timeout. Always set one for external APIs.

```typescript
// BAD: No timeout — can hang indefinitely
const response = await fetch("https://api.external.com/data");

// GOOD: AbortController with timeout
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 10_000);

try {
  const response = await fetch("https://api.external.com/data", {
    signal: controller.signal,
  });
  return response.json();
} finally {
  clearTimeout(timeout);
}

// GOOD: Using fetch with built-in signal timeout (Node 18+)
const response = await fetch("https://api.external.com/data", {
  signal: AbortSignal.timeout(10_000),
});
```

## Use `Promise.allSettled` When Partial Failure Is Acceptable

`Promise.all` fails fast — if one promise rejects, you lose everything. Use `Promise.allSettled` when you want all results regardless.

```typescript
// BAD: One failure kills all results
const [posts, comments, reactions] = await Promise.all([
  fetchPosts({ channelId }),
  fetchComments({ channelId }), // if this fails...
  fetchReactions({ channelId }), // ...you lose this too
]);

// GOOD: Partial results are fine
const results = await Promise.allSettled([
  fetchPosts({ channelId }),
  fetchComments({ channelId }),
  fetchReactions({ channelId }),
]);

const posts = results[0].status === "fulfilled" ? results[0].value : [];
const comments = results[1].status === "fulfilled" ? results[1].value : [];
const reactions =
  results[2].status === "fulfilled" ? results[2].value : [];
```

## Don't Await in Loops

Awaiting inside a loop runs things sequentially. Use `Promise.all` with `.map()` instead.

```typescript
// BAD: Sequential — each iteration waits for the previous one
const results = [];
for (const id of userIds) {
  const user = await getUser({ id });
  results.push(user);
}

// GOOD: Parallel
const results = await Promise.all(userIds.map((id) => getUser({ id })));
```
