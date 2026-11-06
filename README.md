# git-helper plugin

A Claude Code plugin with git helpers. It suggests a commit message for your git changes.
It only suggests. It never commits and never changes files.

## Install

In Claude Code, run:

```
/plugin install git-helper --marketplace arashmanteghi/git-helper
```

On Claude Code versions older than 2.1.275, add the marketplace first, then install:

```
/plugin marketplace add arashmanteghi/git-helper
/plugin install git-helper@git-helper-marketplace
```

The plugin works in all your projects once it is installed.

To get a newer version, run `/plugin marketplace update git-helper-marketplace`.

## Migrating from commit-message

This plugin used to be called `commit-message`. If you installed it under that name, remove the old one and install the new one:

```
/plugin uninstall commit-message@commit-message-marketplace
/plugin marketplace remove commit-message-marketplace
/plugin marketplace add arashmanteghi/git-helper
/plugin install git-helper@git-helper-marketplace
```

The command is now `/git-helper:suggest-commit-message`. Your `.claude/commit-message.md` rules files keep working as before.

## Commands

| Command | What it reads |
|---|---|
| `/git-helper:suggest-commit-message` | Staged changes (the default) |
| `/git-helper:suggest-commit-message staged` | Staged changes |
| `/git-helper:suggest-commit-message unstaged` | Unstaged changes |

You can also just ask "suggest a commit message". Claude may use the skill without the slash command.

## Message format

`<TICKET> <type>: <subject>`

- `<TICKET>` comes from the branch name. Any key like `PROJ-123` or `ENG-42` works (Jira, Linear and similar). It can be anywhere in the branch name, like `feature/proj-123-add-login`, and is always written in uppercase. If the branch has none, it is left out.
- `<type>` is one of the types below.
- `<subject>` follows the casing rules below.

Example: `PROJ-123 fix: align Submit button on Checkout page`

## Types

The type tells the reader what kind of change this is, before they read the code.
Pick one type. If the change mixes types, pick the type of the main change.

| Type | Use it when | Why it helps | Example |
|---|---|---|---|
| `feat` | You add a new feature or new behaviour for the user. | Shows what is new for the user. Good input for release notes. | `PROJ-123 feat: add Login retry for MFA errors` |
| `fix` | You fix a bug. | Shows a broken thing now works. Helps when you look for the change that fixed (or caused) a bug. | `PROJ-123 fix: align Submit button on Checkout page` |
| `refactor` | You change how the code is written, but not what it does. | Tells the reviewer to expect no change in behaviour. | `PROJ-123 refactor: move useAuth hook to Account domain` |
| `perf` | You make the code faster or the bundle smaller. | Shows the change is about speed or size, not a feature or a fix. | `PROJ-123 perf: lazy load Product list images` |
| `test` | You add or change tests only. | Shows production code did not change, so the risk is low. | `PROJ-123 test: add tests for Signup form validation` |
| `docs` | You change documentation only (README, `docs/*.md`, comments). | Shows no code changed. Easy to review. | `PROJ-123 docs: explain Signup flow in auth README` |
| `style` | You change formatting only (spaces, commas, import order). No logic change. | Tells the reviewer they can skim it. | `PROJ-123 style: format Checkout files with Prettier` |
| `build` | You change the build system or dependencies (`package.json`, lockfiles, bundler config). | Shows the change can affect how the project builds or installs. | `PROJ-123 build: upgrade Next.js to 15.2` |
| `ci` | You change CI config or scripts (GitHub Actions, GitLab CI). | Shows only the pipeline changed, not the app. | `PROJ-123 ci: cache dependencies in Checks workflow` |
| `chore` | Other maintenance that fits no type above (config, tooling, cleanup). | Keeps small housekeeping changes apart from real features and fixes. | `chore: update ESLint config` |
| `revert` | You undo an earlier commit. | Shows clearly that an old change was taken back. | `PROJ-123 revert: add Login retry for MFA errors` |

## Casing rules for the subject

- Start with a lowercase verb in the imperative mood ("add", not "added").
- Write every other word in lowercase, except:
  - Important words that name the target of the change (feature, page or domain), like `Login`, `Checkout`, `Settings`. These start with uppercase.
  - Acronyms, like `API`, `URL`, `MFA`. These stay all uppercase.
  - Code names, like `useAuth`. These keep their exact casing.

## Customize for your repo

The skill picks its rules in this order. A higher one wins.

1. **A rules file in your repo:** `.claude/commit-message.md`
2. **Your recent commits:** if most of them follow a clear style (for example `feat(auth):` scopes or `[PROJ-123]` in brackets), the skill matches it.
3. **The defaults** in this README.

So it works with no setup. Add the rules file only for what is different in your team. Commit it, so everyone on the repo gets the same rules.

The file is plain Markdown. Write only what you want to change. Example:

```markdown
# Commit message rules

- Ticket prefix: COREX. If the branch has no COREX ticket, leave the ticket out.
- Acronyms (keep all uppercase): KYC, RG, SGA, MFA
- Domain words (start with uppercase): Deposit, Withdrawal, Cashier, Login
- Extra type: `i18n` for translation-only changes
- Example: `COREX-12345 fix: align Deposit button on Cashier page`
```

Other things you can set: a scope (`feat(auth): ...`), a different max length, or "never add a body".

## Files

| File | Purpose |
|---|---|
| `skills/suggest-commit-message/SKILL.md` | The instructions Claude follows. Edit this to change the rules. |
| `.claude-plugin/plugin.json` | The plugin name, version and description. |
| `.claude-plugin/marketplace.json` | Lists the plugin, so this repo works as a marketplace. |
| `README.md` | This doc, for people. Claude does not load it. |
| `LICENSE` | The MIT license. |

## Release a new version

1. Change `skills/suggest-commit-message/SKILL.md`.
2. Raise `version` in `.claude-plugin/plugin.json` and in `.claude-plugin/marketplace.json`. Keep both the same.
3. Commit and push. Developers get the change when they run `/plugin marketplace update git-helper-marketplace`.

If you push without raising the version, developers who already installed the plugin do not get the change.

## License

[MIT](LICENSE)
