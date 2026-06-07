# Contributing Guide

Reference document for all repositories under the Giralabs organization. Defines how to branch, commit, comment code, open Pull Requests, and maintain a clean, professional history. Anyone should be able to read the project history, understand what changed, why it changed, and follow the codebase without asking.

## Table of Contents

- [General Principles](#general-principles)
- [Branch Strategy](#branch-strategy)
- [Branch Naming](#branch-naming)
- [Daily Workflow](#daily-workflow)
- [Commit Convention](#commit-convention)
- [Code Comments](#code-comments)
- [Pull Requests](#pull-requests)
- [Code Review](#code-review)
- [Conflict Resolution](#conflict-resolution)
- [Dos and Donts](#dos-and-donts)
- [Release Versioning](#release-versioning)

---

## General Principles

- **Never work directly on `main`.** Every change enters through a branch and a Pull Request.
- **Small, atomic commits.** Each commit represents one coherent and complete change. If a commit message needs the word "and", it should probably be two commits.
- **The history tells a story.** It must be readable top to bottom and reflect the evolution of the project.
- **No secrets in the repository.** Keys, tokens, passwords, and `.env` files are never committed. That is what `.gitignore` is for.
- **Sync before you start.** Every session begins by pulling the base branch to reduce conflicts.
- **Code in English.** All code, comments, commit messages, branch names, and PR descriptions are written in English.

---

## Branch Strategy

We use a `main`-based model with short-lived branches. Simple, scalable, and appropriate for the pace of the organization.

### Permanent branches

- **`main`** — Stable and deployable at all times. Reflects what is (or can be) in production. Protected: no direct push allowed.
- **`develop`** *(optional, only in repos with grouped releases)* — Integration branch where features accumulate before a release. For repos that deploy continuously, work directly against `main`.

### Temporary branches

Created from the base branch, alive while the work lasts, deleted after merging.

- **`feature/`** — New functionality.
- **`fix/`** — Bug fix during development.
- **`hotfix/`** — Urgent fix that goes directly from `main` to production.
- **`refactor/`** — Code restructuring without changing behavior.
- **`docs/`** — Documentation-only changes.
- **`chore/`** — Maintenance tasks (dependencies, configuration, tooling).

---

## Branch Naming

Format: `type/short-description-in-kebab-case`.

```
feature/google-sign-in
fix/auth-token-expiry
hotfix/supabase-connection-drop
refactor/user-repository
docs/update-setup-guide
chore/update-flutter-dependencies
```

Rules: all lowercase, words separated by hyphens, no special characters, short but meaningful description.

---

## Daily Workflow

1. **Move to the base branch and pull the latest changes.**
   ```bash
   git checkout main
   git pull origin main
   ```

2. **Create your working branch.**
   ```bash
   git checkout -b feature/descriptive-name
   ```

3. **Work and commit frequently in small, atomic steps.**
   ```bash
   git add specific_file.dart
   git commit -m "feat: add email validation on registration screen"
   ```
   Prefer `git add` per file or per block (`git add -p`) over `git add .` to avoid committing unintended changes.

4. **Keep your branch up to date with the base** (especially on branches that live more than one day).
   ```bash
   git fetch origin
   git rebase origin/main
   ```

5. **Push your branch to the remote.**
   ```bash
   git push -u origin feature/descriptive-name
   ```

6. **Open a Pull Request** toward the base branch.

7. **After approval and merge, clean up.**
   ```bash
   git checkout main
   git pull origin main
   git branch -d feature/descriptive-name
   ```

---

## Commit Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/). This standard allows automatic changelog generation, version inference, and a readable history.

### Structure

```
<type>(<optional scope>): <description>

[optional body]

[optional footer]
```

### Types

| Type | Use |
|------|-----|
| `feat` | New functionality for the user |
| `fix` | Bug fix |
| `docs` | Documentation changes only |
| `style` | Formatting, whitespace, missing semicolons; no logic changes |
| `refactor` | Code restructuring without changing behavior or fixing bugs |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Changes to the build system or dependencies |
| `ci` | Changes to CI/CD configuration |
| `chore` | Maintenance tasks that do not affect production code |
| `revert` | Reverts a previous commit |

### Scope

Optional. Indicates the area of the codebase affected. Useful in projects with clear modules.

```
feat(auth): add biometric login support
fix(profile): correct avatar upload on iOS
chore(deps): upgrade supabase_flutter to 2.5.0
```

### Writing rules

- Description in **lowercase**, **imperative mood**, and **no trailing period**: "add", not "added" or "adds.".
- Maximum 50 characters recommended for the first line; details go in the body.
- The body explains the **what** and the **why**, not the how (the how is in the code).
- Breaking changes are marked with `!` and explained in the footer with `BREAKING CHANGE:`.

### Examples

Simple commit:

```
feat: add pull-to-refresh on home feed
```

Commit with body:

```
fix: resolve session loss after app background

Supabase session was not being persisted correctly when the app
moved to background on Android. Switched to secure storage for
token persistence to match iOS behavior.

Closes
```

Breaking change:

```
feat(api)!: replace userId with UUID across all endpoints

BREAKING CHANGE: all consumers must migrate from integer userId
to UUID string. See migration guide in docs/api-migration.md.
```

---

## Code Comments

Comments are part of the codebase. They must be kept up to date, written in English, and add information that the code alone cannot convey. Never comment the obvious; explain the intent, the constraint, or the edge case.

We use three types of comments depending on what they describe.

---

### Section comments

Used to divide a file into clearly named logical blocks. Written in **ALL CAPS**, with a short opening dash group and a long closing dash group.

Format: `// ----- SECTION NAME --------------------`

```dart
// ----- CONSTANTS --------------------

const int maxRetryAttempts = 3;
const Duration sessionTimeout = Duration(minutes: 30);

// ----- STATE --------------------

bool _isLoading = false;
UserModel? _currentUser;

// ----- LIFECYCLE --------------------

@override
void initState() {
  super.initState();
  _loadUserProfile();
}

// ----- HANDLERS --------------------

void _onLoginPressed() {
  // ...
}

// ----- HELPERS --------------------

String _formatDisplayName(String raw) {
  // ...
}
```

Rule: one section comment per logical group. Do not overuse them — a file with ten sections is a file that needs splitting.

---

### Block comments

Used above a function, method, or chunk of logic to explain what it does, why it exists, or what constraint it respects. **Start with an uppercase letter.** Written as full sentences. Placed immediately above the code they describe with no blank line between the comment and the code.

For public APIs, use Dart doc comments (`///`) so documentation appears in IDEs:

```dart
/// Returns the current authenticated user from the active Supabase session.
/// Returns null if no session exists or the token has expired.
Future<UserModel?> getCurrentUser() async {
  final session = supabase.auth.currentSession;
  if (session == null) return null;
  return UserModel.fromJson(session.user.toJson());
}
```

For private methods and internal logic, use regular block comments (`//`):

```dart
// Retries the upload up to maxRetryAttempts times with exponential backoff.
// Necessary because Supabase Storage can return transient 503 errors
// on large file uploads over mobile connections.
Future<String> _uploadWithRetry(File file) async {
  // ...
}
```

```dart
// Normalizes the phone number before sending it to the auth provider.
// Supabase expects E.164 format (+34XXXXXXXXX) regardless of user input.
String _normalizePhone(String raw) {
  // ...
}
```

Do not use block comments to restate what the function signature already says.

Wrong:

```dart
// Gets the user by id
Future<UserModel?> getUserById(String id) async { ... }
```

Correct:

```dart
// Fetches the user from remote only if the local cache is stale or invalidated.
// Falls back to cached data on network error to avoid blocking the UI.
Future<UserModel?> getUserById(String id) async { ... }
```

---

### Inline comments

Used at the end of a single line to clarify a specific value, condition, or decision that is not immediately obvious. **Start with a lowercase letter.** Keep them short. Two spaces before the `//`.

```dart
final timeout = const Duration(seconds: 10);  // supabase default is 15, reduced for UX
final bucket = 'avatars';  // must match the bucket name in supabase storage settings
final maxFileSize = 5 * 1024 * 1024;  // 5 MB limit enforced by storage policy

if (response.statusCode == 409) {  // conflict: email already registered
  _showDuplicateEmailError();
}

await Future.delayed(const Duration(milliseconds: 300));  // wait for keyboard to dismiss
```

Do not inline-comment things the code already expresses clearly.

Wrong:

```dart
final user = await getUser();  // gets the user
```

---

### TODO and FIXME

Use `TODO` for planned work and `FIXME` for known issues that need attention. Always include context.

```dart
// TODO: replace with real pagination once the endpoint supports cursor-based results
// FIXME: crashes on iPad when the keyboard is open — needs layout constraint fix
```

---

## Pull Requests

The Pull Request is the quality gate before integrating code. It is not a formality: it is where work is reviewed, discussed, and where the reasoning behind decisions is recorded.

### Before opening a PR

- The branch is up to date with the base (`rebase` or recent `merge`).
- The code compiles and passes the linter (`flutter analyze`).
- No `print()` statements, commented-out code, or temporary files.
- No secrets or files that should be in `.gitignore`.

### How to write the PR

- **Title** following the same style as commits: `feat: add biometric login support`.
- **Description** that answers: what does this do, why, and how to test it.
- **Small PRs.** A massive PR is hard to review properly. Split the work when possible.

### Suggested template

```markdown
## Description
Brief summary of what this PR resolves.

## Type of change
- [ ] New feature (feat)
- [ ] Bug fix (fix)
- [ ] Refactor
- [ ] Documentation

## How to test
Steps to verify the change works as expected.

## Checklist
- [ ] Code compiles and passes flutter analyze
- [ ] Tests added or updated
- [ ] Documentation updated if needed
- [ ] No secrets or sensitive files included

Closes
```

---

## Code Review

- Every PR toward `main` requires at least **one approval** before merging.
- **Reviewers** comment with respect and precision, suggest rather than demand, and approve only what they understand.
- **Authors** do not take reviews personally: the goal is product quality, not a judgment of the person.
- Resolve all comments before merging. If you disagree, discuss it in the thread instead of ignoring it.

### Merge strategy

- **Squash and merge** (recommended by default): condenses all branch commits into one on `main`. Keeps the `main` history clean and readable.
- **Rebase and merge**: when individual commits are well-formed and worth preserving.
- **Avoid the standard merge commit** except for long-lived branch integrations (e.g. `develop` into `main`).

After merging, **delete the branch** from GitHub.

---

## Conflict Resolution

Conflicts are normal. What matters is resolving them carefully.

1. Update your branch with the base:
   ```bash
   git fetch origin
   git rebase origin/main
   ```
2. Git will mark the conflicting files. Edit them manually, keeping what is correct from both sides.
3. Mark as resolved and continue:
   ```bash
   git add resolved_file.dart
   git rebase --continue
   ```
4. If you make a mistake or want to abort:
   ```bash
   git rebase --abort
   ```
5. After rebasing an already-published branch, push with `--force-with-lease`:
   ```bash
   git push --force-with-lease
   ```

Never use `--force` on shared branches: you can overwrite someone else's work.

---

## Dos and Donts

**Git and workflow:**

- Never push directly to `main` or `develop`.
- Never use commit messages like `changes`, `fix`, `wip`, `.`, or `asdf`.
- Never accumulate weeks of work into a single giant commit.
- Never commit `.env` files, API keys, tokens, or credentials of any kind.
- Never mix unrelated changes in a single PR.
- Never use `--force` on shared branches (use `--force-with-lease`).
- Never merge your own PR without review when the repository requires approval.
- Never leave review comments unresolved before merging.

**Code comments:**

- Never comment the obvious. Comments explain intent and constraints, not mechanics.
- Never leave outdated comments. A wrong comment is worse than no comment.
- Never use comments to disable code permanently. Delete it; Git history has it.
- Always write comments in English.
- Always write section comments in ALL CAPS using the format `// ----- SECTION --------------------`.
- Always start block comments with an uppercase letter.
- Always start inline comments with a lowercase letter.
- Always keep inline comments short and placed at the end of the line, two spaces after the code.

---

## Release Versioning

For the full versioning policy — when to bump MAJOR, MINOR, or PATCH, how to tag
releases in Git, how to write changelogs, and how to handle hotfixes — see
[VERSIONING.md](./VERSIONING.md).
