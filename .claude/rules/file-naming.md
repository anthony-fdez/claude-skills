---
description: File naming conventions, flat structure, and organization patterns
globs: ["src/**"]
---

# File Naming & Structure

## Core Principles

1. **Flat over nested** - Avoid arbitrary grouping folders
2. **Self-describing names** - Files should not rely on folder context
3. **One export per file** - Each file exports exactly one function/component
4. **Consistency** - Same patterns across the entire codebase

## Quick Reference

| Type      | Pattern                            | Example                      |
| --------- | ---------------------------------- | ---------------------------- |
| Component | `PascalCase.tsx`                   | `StatusBadge.tsx`            |
| Hook      | `use-{descriptive-name}.ts`        | `use-debounced-search.ts`    |
| Function  | `{action}-{resource}.ts`           | `format-date-display.ts`     |
| Service   | `{service}-{action}-{resource}.ts` | `github-get-repo-data.ts`    |
| Type      | `{domain}.types.ts`                | `notification.types.ts`      |
| Constant  | `{domain}.constants.ts`            | `locale.constants.ts`        |
| Folder    | `kebab-case/`                      | `address-restrictions/`      |

## When Nesting Is Acceptable

- Feature domains: `components/`, `lib/`, `pages/`
- Component-specific helpers used ONLY by that component
- Co-located test files
- **Domain sub-folders** for many related items of the same type:

```text
lib/utils/address-restrictions/
├── address-restriction.types.ts       # Types at root
├── create-address-restriction.ts      # Factory at root
├── get-all-address-restrictions.ts    # Aggregator at root
└── restrictions/                      # Related items in subfolder
    ├── united-states-to-puerto-rico.ts
    ├── po-box-restriction.ts
    └── military-address.ts
```

## Avoid

- `folder/index.tsx` — Use `PascalCase.tsx` directly
- Generic names that rely on folder context (`utils/format.ts` → `format-date-display.ts`)
- Deeply nested folders (`services/github/repos/helpers/`)
- Generic groupings (`utils/helpers/`, `common/shared/`)
- Type-only folders (`types/`, `interfaces/`)
- camelCase or PascalCase folder names
