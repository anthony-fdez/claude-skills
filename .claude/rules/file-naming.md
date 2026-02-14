---
description: File naming conventions, flat structure, and organization patterns
globs: ['src/**']
---

# File Naming & Structure

| Type      | Pattern                            | Example                   |
| --------- | ---------------------------------- | ------------------------- |
| Component | `PascalCase.tsx`                   | `StatusBadge.tsx`         |
| Hook      | `use-{descriptive-name}.ts`        | `use-debounced-search.ts` |
| Function  | `{action}-{resource}.ts`           | `format-date-display.ts`  |
| Service   | `{service}-{action}-{resource}.ts` | `github-get-repo-data.ts` |
| Type      | `{domain}.types.ts`                | `notification.types.ts`   |
| Constant  | `{domain}.constants.ts`            | `locale.constants.ts`     |
| Folder    | `kebab-case/`                      | `address-restrictions/`   |

Flat over nested. Self-describing names. One export per file. No `index.tsx` barrels. No generic groupings (`utils/helpers/`, `common/shared/`).
