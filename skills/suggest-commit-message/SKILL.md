---
name: suggest-commit-message
description: Suggest a commit message for staged or unstaged git changes. Use when the user asks for a commit message.
argument-hint: "[staged|unstaged]"
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git branch:*), Bash(git log:*), Bash(git rev-parse:*), Read
model: haiku
effort: low
---

Suggest a commit message. Do not commit and do not change any files.

## Context
- Repo root: !`git rev-parse --show-toplevel`
- Branch: !`git branch --show-current`
- Status: !`git status --short`
- Recent commits: !`git log --oneline -15 --all`

## Where the rules come from

Use these sources. When two disagree, the higher one wins.

1. **Repo rules file.** Read `<repo root>/.claude/commit-message.md` with the Read tool.
   If it exists, follow it. It can set the ticket prefix, add or remove types,
   list domain words and acronyms, or change the format.
   If it does not exist, skip this step and say nothing about it.
2. **Recent commits.** If most recent commits follow a clear style that differs from the
   defaults below (for example a scope like `feat(auth):`, a ticket in brackets like
   `[PROJ-123]`, or a different casing), match that style.
   If there are few commits or no clear style, use the defaults.
3. **The defaults below.**

## Format

`<TICKET> <type>: <subject>`

- `<TICKET>`: the ticket key from the branch name.
  A ticket key is 2 or more letters, a dash, then digits, for example `PROJ-123` or `ENG-42`.
  It can be anywhere in the branch name, like `feature/proj-123-add-login`.
  Always write it in uppercase: `PROJ-123`.
  If the branch has no ticket key, leave it out and start with `<type>`.
- `<type>`: exactly one type from the list below.
- `<subject>`: follow the casing rules below.

### Types

| Type | Use for |
|---|---|
| `feat` | A new feature or new behaviour for the user |
| `fix` | A bug fix |
| `refactor` | A code change that does not change behaviour |
| `perf` | A change that makes the code faster or smaller |
| `test` | Adding or updating tests only |
| `docs` | Documentation only |
| `style` | Formatting only (spaces, commas), no logic change |
| `build` | Build system or dependencies (`package.json`, lockfiles, bundler config) |
| `ci` | CI config and scripts (GitHub Actions, GitLab CI) |
| `chore` | Other maintenance (config, tooling) that fits no type above |
| `revert` | Reverting an earlier commit |

If the change mixes types, pick the type of the main change.

### Casing rules for the subject

- Start the subject with a lowercase verb in the imperative mood ("add", not "added").
- Write every other word in lowercase, except:
  - Important words that name the target of the change (the feature, page or domain),
    for example `Login`, `Checkout`, `Settings`. Start these with an uppercase letter.
  - Acronyms, like `API`, `URL`, `MFA`. Keep them all uppercase.
  - Code names (components, hooks, functions). Keep their exact casing, like `useAuth`.

### Examples

- `PROJ-123 fix: align Submit button on Checkout page`
- `PROJ-123 feat: add Login retry for MFA errors`
- `PROJ-123 refactor: move useAuth hook to Account domain`
- `chore: update ESLint config` (no ticket in the branch name)

## Steps
1. Read the repo rules file, if it exists (see "Where the rules come from").
2. If the argument is "unstaged", read `git diff`. If it is "staged" or empty, read `git diff --cached`.
   If that diff is empty, say so and offer the other one.
3. Pick the type that fits the main change.
4. Write the first line in the format above, 72 characters or less.
5. Add a short body only if the "why" is not clear from the subject.
6. Show 1 main suggestion and 1 shorter alternative.
