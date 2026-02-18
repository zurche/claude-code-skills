# Android Release Bump

Bump release version in app/build.gradle.kts, commit, tag, push to origin/main, build a signed release bundle, and generate multi-locale release notes. Use when asked to bump the release, update versionName/versionCode, bump major/minor/patch, or cut a new release.

## Workflow

1. **Pre-flight checks**: Ensure repo is on `main` with clean working tree and `keystore.properties` present. If Play Console upload is intended, also verify `play-api-key.json` exists.
2. **Discover app name** (see [Configuration > App Name Discovery](#discovering-app-name)).
3. **Analyze changes**:
   - Get latest release tag via `git describe --tags --abbrev=0` and diff commits since then
   - Scan commit messages for change type indicators (see [Commit Analysis Keywords](#commit-analysis-keywords))
   - Recommend appropriate version bump (MAJOR/MINOR/PATCH)
4. **Present version recommendation** to user with reasoning — user can accept or override.
5. **Update `app/build.gradle.kts`**: Increment `versionCode` by 1 from its current value, set `versionName` to new version.
6. **Generate release notes** for all auto-detected locales:
   - Scan `app/src/main/res/values*` directories and map to Play Store locale codes
   - Create `app/src/main/play/release-notes/<locale>/default.txt` files (max 500 chars each)
7. **Run pre-commit verification**: `./gradlew assembleDebug && ./gradlew detekt && ./gradlew test`
8. **Commit and tag**:
   - Commit release notes: `git commit -m "feat: Add localized release notes for vX.Y.Z"`
   - `git commit -m "Updating Release version to X.Y.Z"`
   - `git tag -a "v<app-name>-X.Y.Z" -m "v<app-name>-X.Y.Z"`
   - Note: Release notes commit is intentionally separate. The git tag is applied to the version bump commit only.
9. **Push** to origin/main and push the tag.
10. **Build signed bundle**: `./gradlew :app:copyReleaseBundle` — outputs to `app/build/release/<app-name>-vX.Y.Z-release.aab`
11. **Upload to Internal Testing** (optional): `./gradlew publishReleaseBundle`
12. **Present completion report** (see [Completion Report](#completion-report)).

## Confirmation Checkpoints

The agent MUST pause and wait for explicit user confirmation at these points. Do NOT proceed autonomously past any checkpoint.

| Checkpoint | When | What to show |
|-----------|------|--------------|
| **Version recommendation** | After step 3 | Recommended bump type, reasoning, old → new version |
| **Release notes review** | After step 6, before committing notes | Full release notes for at least the primary locale |
| **Pre-push** | After step 8, before `git push` | Summary of commit, tag name, target branch |
| **Play Console upload** | Before step 11 | Bundle path, target track, version being uploaded |

If the user rejects at any checkpoint, ask what they want to change. Do not restart the entire workflow — resume from the relevant step.

## Versioning Rules

### Bump Categories

| Type | When to use | Example |
|------|-------------|---------|
| **MAJOR** (`X.0.0`) | Complete rewrites, architectural refactors, breaking changes to user workflows, major functionality additions/removals | `1.4.1 → 2.0.0` |
| **MINOR** (`1.Y.0`) | New features, noticeable UI/UX enhancements, behavioral improvements (performance, reliability), localization updates | `1.4.1 → 1.5.0` |
| **PATCH** (`1.0.Z`) | Bug fixes, minor UI tweaks, dependency updates, code refactoring (no user-facing changes), documentation updates. This is the **default**. | `1.4.1 → 1.4.2` |

`versionCode` always increments by +1 from the current configured value regardless of bump type. For first release workflows, do not reset or downgrade an existing `versionCode`. If no explicit `versionCode` exists, initialize it to `1`.

### Analysis Process

1. **Scan commits** since last release tag
2. **Categorize** using keywords (see table below) and PR titles
3. **Recommend** based on the most critical change found
4. **Allow override** — user can specify major/minor/patch

### Handling Ambiguous Commits

When commits lack conventional prefixes:

1. **Check diffstat** (`git diff --stat <last-tag>..HEAD`):
   - 1-5 files, small diffs → PATCH
   - 5-20 files or new files → MINOR
   - 20+ files or structural changes → consider MAJOR
2. **Check file types**: Only test/doc files → PATCH. New source files in feature dirs → MINOR. Core architecture files (DI, navigation, base classes) → possibly MAJOR.
3. **When uncertain**: Default to PATCH. Tell the user: "Commits don't follow conventional prefixes, so I'm defaulting to PATCH. Override if this includes user-facing features."

## Commit Analysis Keywords

| Category | Keywords | Version Impact |
|----------|----------|----------------|
| **Features** | `feat:`, `feature:`, `add:`, `implement:` | MINOR |
| **Bug Fixes** | `fix:`, `bugfix:`, `patch:`, `resolve:` | PATCH |
| **UI/UX** | `ui:`, `ux:`, `design:`, `style:` | MINOR/PATCH based on scope |
| **Refactoring** | `refactor:`, `refactoring:`, `cleanup:` | PATCH |
| **Performance** | `perf:`, `performance:`, `optimize:` | MINOR/PATCH based on impact |
| **Architecture** | `arch:`, `architecture:`, `restructure:` | MAJOR if breaking |
| **Breaking** | `breaking:`, `breaking change:` | MAJOR |
| **Localization** | `i18n:`, `localize:`, `locale:` | MINOR |
| **Testing** | `test:`, `tests:` | PATCH |
| **Docs** | `docs:`, `documentation:` | PATCH |

## Configuration

### Discovering App Name

The skill uses `<app-name>` in tags, bundle filenames, and commit messages. Resolve dynamically — never hardcode or guess.

**Discovery order** (use the first that succeeds):
1. **Existing git tags**: `git tag --list 'v*'` → extract app name from pattern (e.g., `vmy-app-1.4.1` → `my-app`)
2. **`settings.gradle.kts`**: Read `rootProject.name`
3. **`app/build.gradle.kts`**: Read `namespace` or `applicationId`, use last segment
4. **Ask the user**

Cache the resolved name for the workflow duration.

### Naming Convention

- **Git tag**: `v<app-name>-X.Y.Z`
- **Commit message**: `Updating Release version to X.Y.Z`
- **Version name** (build.gradle.kts): `X.Y.Z`
- **Bundle filename**: `<app-name>-vX.Y.Z-release.aab`

### Signing Configuration

The project uses `keystore.properties` (gitignored):

```properties
storeFile=/path/to/your-app-keystore.jks
storePassword=YOUR_STORE_PASSWORD
keyAlias=your-app-key
keyPassword=YOUR_KEY_PASSWORD
```

**Security**: The agent MUST only check that `keystore.properties` exists (`test -f keystore.properties`). NEVER read, log, or display its contents. The same applies to `play-api-key.json`.

### Build Configuration

Add a custom copy task to `app/build.gradle.kts` to rename the release bundle:

```kotlin
tasks.register<Copy>("copyReleaseBundle") {
    val bundleTask = tasks.named("bundleRelease")
    dependsOn(bundleTask)
    val versionName = android.defaultConfig.versionName
    from("build/outputs/bundle/release/app-release.aab")
    into("build/release")
    rename { "your-app-name-v$versionName-release.aab" }
}
```

## Release Notes

**IMPORTANT**: Generate release notes BEFORE building the bundle so they are automatically included in the Play Store upload.

### Locale Detection

Auto-detect supported locales by scanning `app/src/main/res/values*` directories. Map to Play Store locale codes:

| Android resource dir | Play Store locale |
|---------------------|-------------------|
| `values/` | `en-US` |
| `values-de` | `de-DE` |
| `values-es` | `es-ES` |
| `values-fr` | `fr-FR` |
| `values-pt-rBR` | `pt-BR` |
| `values-hi` | `hi-IN` |
| `values-id` | `id` |
| `values-it` | `it-IT` |
| `values-ja` | `ja-JP` |
| `values-ko` | `ko-KR` |
| `values-pl` | `pl-PL` |
| `values-tr` | `tr-TR` |
| `values-zh-rCN` | `zh-CN` |
| `values-zh-rHK` | `zh-HK` |
| `values-zh-rTW` | `zh-TW` |
| `values-es-rUS` | `es-US` |
| `values-es-r419` | `es-419` |

### File Structure

```
app/src/main/play/release-notes/
├── en-US/default.txt
├── de-DE/default.txt
├── es-ES/default.txt
└── fr-FR/default.txt
```

### Guidelines

- **User-focused**: Write for end users, not developers
- **Concise**: 3-5 bullet points per locale, max 500 characters (Play Console limit)
- **Format**: Use bullet points (`•`) for readability
- **Translate naturally**: Ensure translations are accurate and natural for each locale
- **Highlight**: New features, UI/UX improvements, bug fixes, performance gains, security updates

## Google Play Console Integration

### Setup

**1. Create a Service Account** in [Google Cloud Console](https://console.cloud.google.com/):
- Create or select a project → **Service Accounts** → **Create Service Account**
- Grant role: `roles/androidpublisher.admin`
- Create a JSON key → save as `play-api-key.json`

**2. Link to Google Play Console**:
- In [Play Console](https://play.google.com/console) → **Settings** → **API access** → link the Cloud project
- Under **Users and permissions**, verify the service account has "Admin" or "Release management" access

**3. Store `play-api-key.json`** in project root (gitignored). Never commit to version control.

### Gradle Plugin Configuration

Use **Gradle Play Publisher**:

**`gradle/libs.versions.toml`**:
```toml
[versions]
playPublisher = "3.10.0"

[plugins]
gradle-play-publisher = { id = "com.github.triplet.play", version.ref = "playPublisher" }
```

**`app/build.gradle.kts`**:
```kotlin
plugins {
    id("com.github.triplet.play")
}

play {
    serviceAccountCredentials.set(file("${rootProject.rootDir}/play-api-key.json"))
    track.set("internal")
}
```

### Upload

`./gradlew publishReleaseBundle` reads `play-api-key.json`, uploads the signed AAB to the Internal Testing track, and includes the release notes from `app/src/main/play/release-notes/`.

If automated upload is not configured, manually upload the bundle from `app/build/release/` to Play Console.

## Completion Report

After the workflow finishes (fully or partially), present this summary:

```
## Release Summary

| Field | Value |
|-------|-------|
| Previous version | 1.4.1 (versionCode 42) |
| New version | 1.5.0 (versionCode 43) |
| Bump type | MINOR |
| Git tag | v<app-name>-1.5.0 |
| Commits included | 12 commits since v<app-name>-1.4.1 |
| Release notes | Generated for 5 locales (en-US, de-DE, es-ES, fr-FR, ja-JP) |
| Bundle path | app/build/release/<app-name>-v1.5.0-release.aab |
| Pushed to remote | Yes — origin/main |
| Play Console upload | Uploaded to Internal Testing / Skipped / Failed (reason) |
```

Note any skipped or failed steps so the user knows exactly where things stand.

## Guardrails

- Abort if not on `main`, working tree is dirty, or `keystore.properties` is missing
- For Play Console upload: also verify `play-api-key.json` exists
- ALWAYS fetch latest release tag, scan commits, and present recommendation with reasoning
- ALWAYS allow the user to override the version recommendation
- ALWAYS increment `versionCode` by +1
- ALWAYS generate release notes BEFORE building the bundle
- ALWAYS ensure release notes are under 500 characters per locale
- Never skip pre-commit verification
- Never force-push or amend already-pushed commits
- Never release with uncommitted changes
- Never read, log, or display contents of `keystore.properties` or `play-api-key.json`

## Error Recovery

| Failure | Recovery |
|---------|----------|
| **Gradle build** (`assembleDebug`) | Run `./gradlew clean` and retry once. If it fails again, report error and stop. Do NOT commit or tag. |
| **Detekt / Tests** | Report which checks failed. Do NOT retry — these require code changes. Ask user how to proceed. |
| **Git tag exists** | Do NOT overwrite. Inform user and suggest incrementing to next version (e.g., `1.5.1`). |
| **Git push failure** | Check for diverged branches. Report conflict — do NOT force-push. Local commit and tag remain intact. |
| **Play Console upload** | Check `play-api-key.json` exists. Auth error → verify service account permissions. Network error → offer retry or manual upload. Bundle is still at `app/build/release/`. |
| **First release (no tags)** | Default to `1.0.0`. Set `versionCode` to current configured value +1 (or `1` if no explicit `versionCode` exists). Present to user: "No previous tags found. Starting at v1.0.0. Override?" Use all commits as changelog source. |
