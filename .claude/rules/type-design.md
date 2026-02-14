---
description: TypeScript type patterns, discriminated unions, and type derivation
globs: ["src/**/*.{ts,tsx}"]
---

# Type Design

- Types describe domain concepts, not implementation details
- Narrow types over wide — string literals, specific objects, exact shapes
- Derive types from runtime sources: `z.infer<typeof Schema>`, `as const`, `ReturnType<typeof fn>`
- Discriminated unions for mutually exclusive states (no boolean flag combos)
- Indexed access types (`Type['field'][number]`) over duplicating nested types
- No simple aliases for primitives (`type Id = string` adds nothing)
