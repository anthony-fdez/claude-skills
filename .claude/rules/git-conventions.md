---
description: Git commit messages, branch naming, and version control hygiene
---

# Git Conventions

## Conventional Commits

Every commit message follows this format:

```
type: short description in lowercase
```

| Type       | When to use                                         |
| ---------- | --------------------------------------------------- |
| `feat`     | New feature or capability                           |
| `fix`      | Bug fix                                             |
| `refactor` | Code change that doesn't fix a bug or add a feature |
| `chore`    | Tooling, config, dependencies, CI                   |
| `docs`     | Documentation only                                  |
| `test`     | Adding or fixing tests                              |
| `style`    | Formatting, whitespace (no logic change)            |
| `perf`     | Performance improvement                             |

```bash
# GOOD
feat: add global search to navigation
fix: prevent duplicate submissions on double-click
refactor: extract date formatting to shared utility
chore: update eslint config
docs: add API route documentation
test: add onboarding flow e2e tests

# BAD
Updated stuff
fix bug
WIP
changes
Add feature and fix bug and update docs  # multiple concerns
```

## One Concern Per Commit

Each commit does exactly one thing. If you can't describe it without "and", split it.

```bash
# BAD: Two concerns in one commit
feat: add search bar and fix timezone conversion

# GOOD: Separate commits
feat: add search bar to navigation
fix: correct timezone conversion for UTC offsets
```

## Branch Naming

```
type/short-description
```

```bash
# GOOD
feat/global-search
fix/timezone-rounding
refactor/onboarding-flow
chore/upgrade-next-15

# BAD
my-branch
feature
fix-stuff
john/working-on-it
```

## Never Commit These

- `.env` or `.env.local` (secrets)
- `node_modules/`
- Build output (`dist/`, `.next/`, `out/`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Editor config (`.vscode/settings.json` with personal prefs)
- Debug logs or `console.log` statements
- Commented-out code

## Commit Message Description

Keep it under 72 characters. Use imperative mood (tell the code what to do).

```bash
# GOOD: Imperative mood
feat: add error boundary to settings page
fix: handle null response from notifications API

# BAD: Past tense / descriptive
feat: added error boundary
fix: fixed the bug with notifications
feat: this adds a new feature for searching
```
