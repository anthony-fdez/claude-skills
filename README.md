# ai-skills

A ready-to-go `.claude/` directory you can drop into any project. It comes with skills, slash commands, coding rules, and permission settings that make Claude Code way more useful out of the box.

The whole thing is generic — no assumptions about your stack beyond Next.js/React/TypeScript. Take what you need, ignore what you don't.

## Setup

Copy the whole thing:

```bash
cp -r .claude/ /path/to/your/project/.claude/
```

Or just grab individual files. Every skill and command is a standalone markdown file — they don't depend on each other.

## What's in here

```
.claude/
├── settings.json     # Permission tiers (allow/ask/deny)
├── workflow.md       # Step-by-step dev workflow guide
├── rules/            # Always-on coding conventions (12 rules)
│   ├── async-patterns.md
│   ├── code-style.md
│   ├── error-handling.md
│   ├── file-naming.md
│   ├── function-design.md
│   ├── git-conventions.md
│   ├── import-conventions.md
│   ├── naming-conventions.md
│   ├── performance-awareness.md
│   ├── react-patterns.md
│   ├── testing-principles.md
│   └── type-design.md
├── skills/           # 17 skills (patterns Claude follows automatically)
└── commands/         # 6 slash commands (things you trigger manually)
```

## Skills

Skills are patterns that Claude picks up automatically based on what you're working on. You don't need to invoke them — if you're writing a React component, Claude will follow the `writing-react` skill on its own.

| Category     | Skills                                                                                                         |
| ------------ | -------------------------------------------------------------------------------------------------------------- |
| **Planning** | `planning`, `ask-questions-if-unspecified`                                                                     |
| **Frontend** | `writing-react`, `writing-nextjs`, `writing-forms`, `managing-state`, `writing-react-query`, `frontend-design` |
| **Backend**  | `designing-apis`, `writing-logs`, `creating-packages`                                                          |
| **Language** | `writing-typescript`                                                                                           |
| **Testing**  | `writing-tests`                                                                                                |
| **Quality**  | `handling-errors`, `securing-code`                                                                             |
| **Docs**     | `writing-design-docs`, `writing-skills`                                                                        |

Each skill lives in `.claude/skills/{name}/SKILL.md` and covers things like:

- When to use `getStaticProps` vs `getServerSideProps`
- How to structure React Query hooks
- Zustand store conventions
- Logging patterns with DataDog tags
- Security rules for API routes and payment flows
- TypeScript patterns (no `any`, no enums, named params, etc.)

## Commands

Slash commands you run explicitly in Claude Code. Type them in the chat.

| Command           | What it does                                                                     |
| ----------------- | -------------------------------------------------------------------------------- |
| `/create-pr`      | Diffs your branch, generates a PR title + description, pushes and creates the PR |
| `/review-pr`      | Reviews a PR — summarizes changes, flags issues, checks for test gaps            |
| `/pr-description` | Generates a structured PR description and writes it to a file                    |
| `/verify-branch`  | Checks your branch against design docs, flags stale behaviors                    |
| `/investigate`    | Traces an error through the codebase to find root cause                          |
| `/explain-system` | Deep-dives a module — dependencies, data flow, gotchas                           |

## Settings

The `settings.json` sets up three permission tiers so Claude doesn't ask you to confirm every little thing:

- **Allow** — Read-only commands, file editing, git status/diff/log, running tests. These just work.
- **Ask** — Git commits/pushes, package installs, network requests, file deletion. Claude asks before running these.
- **Deny** — ssh, sudo, docker, brew, system-level stuff. Blocked entirely.

## Workflow

`workflow.md` describes a full dev loop from prompt to PR:

1. Write a detailed prompt
2. Review the plan Claude generates
3. Let it implement
4. Review the code
5. Generate docs (`/writing-design-docs`)
6. Verify the branch (`/verify-branch`)
7. Create the PR (`/create-pr`)

You don't have to follow this exactly — it's a reference for getting the most out of the tools.

## Customizing

**Add your own skills** — Create `.claude/skills/{name}/SKILL.md` with a YAML frontmatter (`name` + `description` with a "Use when..." trigger). Check `writing-skills` for the full pattern.

**Add your own rules** — Drop markdown files in `.claude/rules/`. These are always-on conventions Claude follows regardless of context.

**Tweak permissions** — Edit `settings.json` to match your workflow. Move commands between allow/ask/deny as needed.

**Remove what you don't need** — Using Vue instead of React? Delete `writing-react` and `writing-nextjs`. Not using Playwright? Delete `writing-tests`. Everything is independent.
