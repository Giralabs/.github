# Versioning Guide

Reference document for all Giralabs repositories. Defines when to bump a version,
what each version number means, how to tag a release in Git, and how to publish it
on GitHub. Applies to Flutter applications using `pubspec.yaml` as the version source.

---

## Version format

We follow [Semantic Versioning](https://semver.org): `MAJOR.MINOR.PATCH+BUILD`.

```
2.1.3+47
│ │ │  └─ build number  — increments on every release, never resets
│ │ └──── patch         — bug fixes
│ └────── minor         — new features
└──────── major         — breaking changes or full redesigns
```

In Flutter, the version lives in `pubspec.yaml`:

```yaml
version: 2.1.3+47
```

The part before `+` is the version name shown to users (`versionName` on Android,
`CFBundleShortVersionString` on iOS). The part after `+` is the build number used
by the stores to identify each upload (`versionCode` on Android,
`CFBundleVersion` on iOS). **The build number must always go up. It never resets,
not even when MAJOR changes.**

---

## When to bump each number

### PATCH — `x.x.1`, `x.x.2`, `x.x.3`

Increment PATCH for changes that fix something without adding new functionality
and without breaking anything existing.

Use it for:

- Bug fixes (crashes, incorrect behavior, broken UI).
- Copy or translation corrections.
- Performance improvements with no API or UI changes.
- Dependency updates with no visible impact.

Examples:

```
1.0.0 → 1.0.1   fixed crash on logout when session was already expired
1.0.1 → 1.0.2   corrected date format on invoice screen for iOS
1.0.2 → 1.0.3   upgraded supabase_flutter to resolve auth refresh bug
```

### MINOR — `x.1.x`, `x.2.x`, `x.3.x`

Increment MINOR and reset PATCH to zero when new functionality is added that does
not break existing behavior. Users get something new; nothing they relied on stops working.

Use it for:

- New screens or flows.
- New settings or preferences.
- New integrations (notifications, payments, third-party services).
- UI improvements that extend what already exists.

Examples:

```
1.0.3 → 1.1.0   added biometric login support
1.1.0 → 1.2.0   added push notification preferences screen
1.2.0 → 1.3.0   integrated Stripe payments on checkout flow
```

### MAJOR — `2.x.x`, `3.x.x`

Increment MAJOR and reset MINOR and PATCH to zero for changes that fundamentally
alter how the application works, looks, or integrates with external systems.

Use it for:

- Complete redesigns of the UI or navigation structure.
- Replacing the backend or authentication system.
- Removing or renaming features users depend on.
- Changes that require users to re-authenticate or migrate data.
- Initial public release moving out of beta (`0.x.x → 1.0.0`).

Examples:

```
0.9.2 → 1.0.0   first stable public release
1.3.0 → 2.0.0   full redesign with new navigation structure
2.0.0 → 3.0.0   migrated from Firebase to Supabase, all users must re-authenticate
```

---

## The 0.x.x range — pre-release and beta

While the app is in active development before the first stable release, the MAJOR
version stays at `0`. This signals that the API and feature set are not stable yet
and breaking changes can happen between MINOR bumps.

```
0.1.0   first internal build
0.2.0   first testflight / internal testing build
0.5.0   feature-complete beta
0.9.0   release candidate
1.0.0   first stable public release
```

Once `1.0.0` is published, the rules above apply strictly.

---

## Build number

The build number (`+47`) is independent from the version name. It increments by one
on every single release to any channel (internal, beta, production). It never resets.

```
1.0.0+1    first internal build
1.0.0+2    hotfix before launch
1.0.1+3    patch release
1.1.0+4    minor release
2.0.0+5    major release
```

This ensures the stores always accept the upload and there is never ambiguity about
which binary a build number refers to.

---

## Release process

### 1. Finish the work and merge to main

All feature branches for this release are merged to `main` via Pull Request
following the standard workflow.

### 2. Bump the version in pubspec.yaml

```yaml
# before
version: 1.2.0+31

# after
version: 1.3.0+32
```

Commit this change alone, with nothing else in the same commit:

```bash
git add pubspec.yaml
git commit -m "chore: bump version to 1.3.0+32"
git push origin main
```

### 3. Create and push the Git tag

Tags are created on `main` after the version commit. Use annotated tags.

```bash
git tag -a v1.3.0 -m "Release 1.3.0: notification preferences screen"
git push origin v1.3.0
```

Tag format: `v` followed by the version name, without the build number.

```
v1.0.0
v1.3.0
v2.0.0
```

### 4. Create the GitHub Release

Go to the repository on GitHub → Releases → Draft a new release.

- **Tag**: select the tag you just pushed (`v1.3.0`).
- **Title**: `v1.3.0 — Notification preferences`.
- **Description**: list what changed, grouped by type. See the changelog format below.
- **Mark as pre-release** if this is a beta or release candidate.

### 5. Build and submit to the stores

Run the release build pointing to the tagged commit:

```bash
# Android
flutter build appbundle --release

# iOS
flutter build ipa --release
```

Submit the output to Google Play and App Store Connect as usual.

---

## Changelog format

Each GitHub Release description follows this structure. Only include sections that
have entries for that release.

```markdown
## What's new
- Added notification preferences screen with per-category toggles.
- Added badge count on the home tab for unread items.

## Improvements
- Reduced initial load time by caching the user profile locally.
- Smoother transition animation on the onboarding flow.

## Bug fixes
- Fixed crash on logout when the session was already expired (#54).
- Corrected date format on invoice screen for iOS devices (#61).

## Internal
- Upgraded supabase_flutter to 2.5.0.
- Migrated CI workflow to GitHub Actions.
```

---

## Hotfixes

A hotfix is an urgent patch that cannot wait for the next planned release cycle.
It branches directly from the latest tag on `main`, not from `develop`.

```bash
# branch from the current production tag
git checkout v1.3.0
git checkout -b hotfix/crash-on-logout

# fix the bug, commit
git commit -m "fix: resolve crash on logout when session is expired"

# merge back to main via PR
# then tag immediately after merge
git tag -a v1.3.1 -m "Release 1.3.1: hotfix crash on logout"
git push origin v1.3.1
```

Bump only PATCH for hotfixes. Increment the build number as always.

---

## Version summary

| Situation | Before | After | Reason |
|---|---|---|---|
| Fixed a crash on login | `1.2.3+20` | `1.2.4+21` | PATCH: bug fix |
| Added a new settings screen | `1.2.4+21` | `1.3.0+22` | MINOR: new feature |
| Full redesign, new navigation | `1.3.0+22` | `2.0.0+23` | MAJOR: breaking change |
| Hotfix before first launch | `0.9.0+8` | `0.9.1+9` | PATCH: bug fix in pre-release |
| First stable public release | `0.9.1+9` | `1.0.0+10` | MAJOR: leaving pre-release |
| Two patches in the same week | `1.0.0+10` → `1.0.1+11` → `1.0.2+12` | | each fix is its own release |
