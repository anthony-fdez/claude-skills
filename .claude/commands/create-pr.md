# Create Pull Request

Create a GitHub pull request from the current branch.

## Steps

### 1. Determine target branch

Ask the user which branch to open the PR against using AskUserQuestion. Provide common options like `main`, `develop`, or let them specify a custom branch.

### 2. Gather context

Run these commands to understand the changes:

```bash
git log <target_branch>..HEAD --oneline
git diff <target_branch>...HEAD --stat
```

### 3. Generate PR description

Generate a concise GitHub PR description in markdown format with the following structure:

1. **Description section** (< 100 words, or < 120 words if explaining complex changes): Brief explanation of what was implemented/changed and why
2. **Technical Details** (if applicable): Explanation of complicated code changes or architectural decisions
3. **Note section** (if applicable): Any dependencies, requirements, or follow-up tasks needed
4. **Testing Checklist**: 5-7 actionable test items using `- [ ]` checkbox format

Requirements:
- Keep description under 100 words (can extend to 120 words if explaining complex technical changes)
- If code changes involve complex patterns, new architecture, or non-obvious implementations, add a Technical Details section
- Include relevant testing URLs or tools (e.g., Facebook Debugger for OG tags)
- Make checklist items specific and actionable
- Include both happy path and error/fallback scenario tests
- If external dependencies exist (API keys, translations, etc.), mention them in a **Note**

### 4. Generate PR title

The PR title MUST follow this exact pattern:

```
<target_branch> <- <current_branch> (<very brief description>)
```

Examples:
- `main <- feat/add-auth (add user authentication)`
- `develop <- fix/cart-total (fix cart total calculation)`
- `main <- chore/update-deps (update dependencies)`

The description in parentheses should be a very brief summary (3-7 words) of the changes.

### 5. Show the user the PR title and description for review

Present the full PR title and body to the user. Ask them to confirm or request changes before creating the PR.

### 6. Create the PR

Once the user confirms, push the branch and create the PR:

```bash
git push -u origin HEAD
gh pr create --base <target_branch> --title "<title>" --body "<body>"
```

Return the PR URL to the user.
