---
name: android-release-bump
description: Bump Android release versions in app/build.gradle.kts, generate localized Play release notes, run verification, commit/tag/push, build a signed AAB, and optionally upload to Google Play Internal Testing. Use when asked to cut a release, bump major/minor/patch (or medium/main), update versionName/versionCode, or publish an Android app release.
---

# Android Release Bump

## Workflow

1. Validate preconditions: run on `main`, require clean working tree, require `keystore.properties`, and require `play-api-key.json` only when upload is requested.
2. Resolve app name in this order: existing tag prefix, `settings.gradle.kts` `rootProject.name`, `app/build.gradle.kts` `namespace` or `applicationId`, then ask the user.
3. Read current `versionName` and `versionCode` from `app/build.gradle.kts`.
4. Discover the latest release tag and commits since that tag.
5. Recommend `major`, `minor`, or `patch` using [references/versioning.md](references/versioning.md), then wait for explicit confirmation.
6. Update `versionName` and increment `versionCode` by `+1` from its current configured value.
7. Detect locales, generate `app/src/main/play/release-notes/<locale>/default.txt`, and enforce limits from [references/locales.md](references/locales.md).
8. Show release notes for review, then wait for explicit confirmation before committing.
9. Run verification: `./gradlew assembleDebug && ./gradlew detekt && ./gradlew test`.
10. Commit release notes and version bump as separate commits, then create annotated tag `v<app-name>-X.Y.Z`.
11. Show pre-push summary and wait for explicit confirmation before pushing `main` and the tag.
12. Build signed bundle with `./gradlew :app:copyReleaseBundle`.
13. If upload is requested, show upload plan and wait for explicit confirmation before `./gradlew publishReleaseBundle`.
14. Print completion summary format from [references/recovery.md](references/recovery.md).

## Mandatory Confirmations

- Version bump recommendation
- Release notes content
- Pre-push commit/tag summary
- Play Internal Testing upload

## Guardrails

- Abort on dirty working tree, wrong branch, or missing required credential files.
- Never skip verification.
- Never force-push, rewrite existing tags, or amend pushed commits.
- Never read, print, or store secret values from `keystore.properties` or `play-api-key.json`.
- On failure, stop at the safe boundary and follow [references/recovery.md](references/recovery.md).

## Load Only When Needed

- Versioning and ambiguous commit handling: [references/versioning.md](references/versioning.md)
- Locale mapping and release-note rules: [references/locales.md](references/locales.md)
- Google Play upload setup and plugin wiring: [references/play-console.md](references/play-console.md)
- Failure matrix and completion summary: [references/recovery.md](references/recovery.md)
