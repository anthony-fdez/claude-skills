---
description: Async/await patterns, parallel execution, and promise handling
globs: ["src/**/*.{ts,tsx}"]
---

# Async Patterns

- `Promise.all` for independent operations, sequential when dependent
- No floating promises — always `await`, return, or explicitly `void` with `.catch()`
- No unnecessary `async` — don't mark functions async if they don't `await`
- Timeout external API calls (`AbortSignal.timeout()`)
- `Promise.allSettled` when partial failure is acceptable
- No `await` in loops — use `Promise.all` with `.map()`
