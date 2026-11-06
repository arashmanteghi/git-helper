---
name: suggest-pr
description: Suggest a pull request title, description, or both for the changes on a branch. Use when the user asks for a PR title or PR description.
argument-hint: "[title|description|both] [source-branch] [destination-branch]"
allowed-tools: Bash(git diff:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(git symbolic-ref:*), Bash(git status:*), Read
model: sonnet
effort: medium
---

Suggest a pull request title and/or description. Do not create the PR, do not push,
do not commit and do not change any files.

## Context
- Repo root: !`git rev-parse --show-toplevel`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -15 --all`

## Arguments

`$ARGUMENTS` holds up to three words, in this order:

1. **What to suggest:** `title`, `description` or `both`. Empty means `both`.
2. **Source branch:** the branch with the changes. Empty means the current branch.
3. **Destination branch:** the branch the PR goes into. Empty means the repo's main branch.

To set a branch but get both, pass `both` first, like `both feature/login develop`.
If the first word is not `title`, `description` or `both`, say so and stop.

### Find the main branch

Run `git symbolic-ref --short refs/remotes/origin/HEAD` and drop the `origin/` part.
If that fails, use `main` if it exists, else `master`.

### Find the branches

For each branch, use the local branch if it exists, else `origin/<branch>`.
If neither exists, say which one is missing and stop.

## What to read

1. `git log --oneline <destination>..<source>`: the commits in the PR.
2. `git diff --stat <destination>...<source>`: the changed files.
3. `git diff <destination>...<source>`: the changes. If the diff is very large,
   read the stat and the commits first, then read only the files that matter most.

If there are no commits between the two branches, say so and stop.
If the source is the current branch and `git status --short` shows changes that are not
committed, tell the user these are not part of the PR.

## Title

Use the same format and rules as commit messages:

`<TICKET> <type>: <subject>`

- Follow `<repo root>/.claude/commit-message.md` if it exists, then the style of recent
  commits, the same way the `suggest-commit-message` skill does.
- `<TICKET>`: the ticket key from the **source** branch name, like `PROJ-123`.
  2 or more letters, a dash, then digits. Always uppercase. Leave it out if there is none.
- `<type>`: one of `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `style`, `build`,
  `ci`, `chore`, `revert`. Pick the type of the main change across all commits.
- `<subject>`: starts with a lowercase imperative verb ("add", not "added").
  Keep target words (`Login`, `Checkout`), acronyms (`API`, `MFA`) and code names
  (`useAuth`) in their casing. Everything else lowercase.
- 72 characters or less. Describe the whole PR, not only the last commit.

## Description

Rules, in order. A higher one wins.

1. **A PR template in the repo.** Look for these files with the Read tool, under the repo root:
   `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`,
   `docs/pull_request_template.md`, `pull_request_template.md`.
   If one exists, fill in its sections. Keep its headings and checkboxes.
   If none exists, say nothing about it.
2. **A rules file:** `<repo root>/.claude/pull-request.md`. If it exists, follow it.
3. **The default below.**

### Default description

```markdown
## Summary
<1 to 3 sentences: what this PR does and why.>

## Changes
- <one bullet per main change, grouped by area, not per commit>

## How to test
1. <steps a reviewer can follow>
```

- Write for the reviewer. Say what changed and why, not how the diff looks.
- Leave out a section that would be empty or obvious. Do not make up test steps
  or ticket links you cannot see in the changes.
- Mention breaking changes, migrations or new config in a **Notes** section, only if there are any.

## Output

- Start with one line: `<source> → <destination>, <N> commits`.
- `title`: show 1 main title and 1 shorter alternative.
- `description`: show the description in one fenced `markdown` block, ready to paste.
- `both`: show the title first, then the description.
