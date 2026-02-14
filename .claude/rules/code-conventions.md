---
description: Code style, naming, and function design conventions
alwaysApply: true
---

# Code Conventions

## Function Parameters

- **Always named object parameters** — single destructured object, no positional args
- Exception: built-in callbacks (`.map()`, `.filter()`, `.reduce()`)
- **No boolean parameters** — use string unions or separate functions

## Naming

- Variables describe what they hold, not how obtained (`user` not `fetchedUserData`)
- Booleans: `is`, `has`, `should`, `can`, `was` prefix
- Event handlers: `handle` + noun + verb (`handleFormSubmit`)
- Props callbacks: `on` + noun + verb (`onFormSubmit`)
- Collections plural, items singular (`users.map((user) => ...)`)
- Constants: `UPPER_SNAKE_CASE` for compile-time values only
- No type info in names (`users` not `userArray`)
- No abbreviations except: `id, url, ref, props, params, config, auth, env, db, api, src, btn, img, nav, max, min`

## Style

- Named exports only (except Next.js pages)
- Early returns over nesting — guard clauses first, happy path last
- One function, one job — if you need "and" to describe it, split it
- Pure by default — don't mutate inputs, side effects in handlers/hooks
- Single clear return type — avoid `string | null | undefined | false`
- No comments unless explaining a non-obvious "why"
- No `console.log` in committed code — use a structured logger
- No dead code — delete unused code, don't comment it out
- No unnecessary abstractions — don't create helpers for one-time operations
