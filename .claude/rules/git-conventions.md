---
description: Commit message format, branch naming, and git hygiene
alwaysApply: true
---

# Git Conventions

## Commit Messages

Format: `type: short description` (imperative mood, under 72 chars)

| Type       | When                                |
| ---------- | ----------------------------------- |
| `feat`     | New feature                         |
| `fix`      | Bug fix                             |
| `refactor` | Code change (no new feature or fix) |
| `chore`    | Tooling, config, dependencies       |
| `docs`     | Documentation only                  |
| `test`     | Adding or fixing tests              |
| `style`    | Formatting, whitespace              |
| `perf`     | Performance improvement             |

One concern per commit. If you need "and" to describe it, split it.

## Branch Naming

Format: `type/short-description` (e.g., `feat/global-search`, `fix/timezone-rounding`)

## Never Commit

`.env`, `node_modules/`, build output, `.DS_Store`, `console.log` statements, commented-out code.
