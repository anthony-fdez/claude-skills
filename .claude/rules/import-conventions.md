---
description: Import ordering, path aliases, and module conventions
globs: ["src/**/*.{ts,tsx}"]
---

# Import Conventions

- `import type` for type-only imports (or inline `type` keyword when mixing)
- `@/` absolute paths for anything outside current directory; `./` for same directory only
- Group order (blank line between): external libs → `@/` internal → `./` relative → type-only
- No barrel files (`index.ts` re-exports) — import directly from source
- No unused imports
