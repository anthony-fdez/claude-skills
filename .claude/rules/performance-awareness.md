---
description: Performance patterns for imports, memoization, images, and lazy loading
globs: ["src/**/*.{ts,tsx}"]
---

# Performance Awareness

- Import specific functions (`import sortBy from 'lodash/sortBy'`), not entire libraries
- `useMemo`/`useCallback` only for expensive computations or referential stability — not trivial operations
- Stable references for objects/arrays passed as props (avoid inline `{}` or `.map()` in JSX)
- Batch API calls — no N+1 queries (one request per item in a list)
- Images: always `next/image` with `width`/`height` and `priority` for above-fold
- Lazy load heavy components with `dynamic()` from `next/dynamic`
