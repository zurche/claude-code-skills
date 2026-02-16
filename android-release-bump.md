# Android Release Bump

Automate Android app release version management with intelligent semantic versioning, multi-locale release notes generation, and Google Play Console integration.

## Overview

This skill automates the complete Android release workflow:
- Analyzes git commits to recommend semantic version bumps (MAJOR/MINOR/PATCH)
- Updates version code and name in `app/build.gradle.kts`
- Generates localized release notes for all supported languages
- Runs pre-commit verification (build, linting, tests)
- Creates git tags and commits
- Builds signed release bundles
- Optionally uploads to Google Play Console Internal Testing track

## Workflow

1. **Pre-flight checks**: Ensure repo is on `main` with clean working tree, `keystore.properties`, and `play-api-key.json` present.
2. **Analyze changes**:
   - Get latest release tag and diff commits since last release
   - Scan commit messages for change type indicators
   - Recommend appropriate version bump (MAJOR/MINOR/PATCH)
3. **Determine version**:
   - Present recommendation with reasoning to user
   - User can accept recommendation or override
4. **Update `app/build.gradle.kts`**:
   - Increment `versionCode` by 1
   - Set `versionName` to new version
5. **Generate release notes** for all supported locales (auto-detected):
   - Analyze commits since last release
   - Create `app/src/main/play/release-notes/<locale>/default.txt` files
   - Write user-friendly release notes (max 500 chars per locale)
   - Commit release notes files
6. **Run pre-commit verification**: `./gradlew assembleDebug && ./gradlew detekt && ./gradlew test`
7. **Commit and tag**:
   - Commit message: `Updating Release version to X.Y.Z`
   - Tag: `v<app-name>-X.Y.Z` (configurable prefix)
   - **Note**: The release notes commit (step 5) is intentionally separate from the version bump commit (step 7). The git tag is applied to the version bump commit only. This keeps release notes changes isolated for cleaner history.
8. **Push to origin/main** and push the tag.
9. **Build signed bundle**: `./gradlew :app:copyReleaseBundle`
10. **Report bundle location**: `app/build/release/<app-name>-vX.Y.Z-release.aab`
11. **Upload to Internal Testing track**: Push bundle with embedded release notes to Google Play Console (optional, requires confirmation)

## Confirmation Checkpoints

The agent MUST pause and wait for explicit user confirmation at these points. Do NOT proceed autonomously past any checkpoint.

| Checkpoint | When | What to show the user |
|-----------|------|-----------------------|
| **Version recommendation** | After analyzing commits (step 2-3) | Recommended bump type, reasoning, old → new version |
| **Release notes review** | After generating notes, before committing (step 5) | Full release notes for at least the primary locale |
| **Pre-push** | After commit + tag, before `git push` (step 8) | Summary of commit, tag name, target branch |
| **Play Console upload** | Before `publishReleaseBundle` (step 11) | Bundle path, target track, version being uploaded |

If the user rejects at any checkpoint, ask what they want to change. Do not restart the entire workflow — resume from the relevant step.

## Versioning Rules & Intelligent Bumping

The skill analyzes commits since the last release to recommend the appropriate version bump:

### Version Bump Categories

**MAJOR** (`X.0.0`) - Use when:
- Complete feature rewrites or architectural refactors
- Significant UI redesigns across the app
- Breaking changes to user workflows
- Major functionality additions/removals
- Example: `1.4.1 → 2.0.0`

**MINOR** (`1.Y.0`) - Use when:
- New features or feature improvements
- Noticeable UI/UX enhancements
- Behavioral improvements (performance, reliability)
- Localization updates
- Example: `1.4.1 → 1.5.0`

**PATCH** (`1.0.Z`) - Use when:
- Bug fixes and stability improvements
- Minor UI tweaks or polish
- Dependency updates
- Code refactoring (no user-facing changes)
- Documentation updates
- Example: `1.4.1 → 1.4.2` (default)

### Intelligent Analysis Process

1. **Scan commits** since last release tag
2. **Categorize changes**:
   - Look for keywords: `feat:`, `fix:`, `refactor:`, `ui:`, `perf:`
   - Analyze PR titles and commit messages
   - Check for architectural changes vs cosmetic changes
3. **Calculate risk level**:
   - HIGH RISK: Major rewrites, breaking changes → Major bump
   - MEDIUM RISK: New features, significant improvements → Minor bump
   - LOW RISK: Bug fixes, patches → Patch bump
4. **Recommend version** based on most critical change found
5. **Allow override** - User can specify major/minor/patch if recommendation differs from intent

### Handling Ambiguous Commits

Not all repositories use conventional commit prefixes. When commits lack clear keywords:

1. **Check the diffstat**: Run `git diff --stat <last-tag>..HEAD` to gauge scope
   - 1-5 files changed, small diffs → likely PATCH
   - 5-20 files changed or new files added → likely MINOR
   - 20+ files or major structural changes → consider MAJOR
2. **Look at file types changed**:
   - Only test/doc files → PATCH
   - New source files in feature directories → MINOR
   - Changes to core architecture files (DI, navigation, base classes) → possibly MAJOR
3. **When still uncertain**: Default to PATCH and present your reasoning. Explicitly tell the user: "Commits don't follow conventional prefixes, so I'm defaulting to PATCH. Override if this includes user-facing features."

### Examples

| Scenario | Last Release | Commits | Recommendation |
|----------|--------------|---------|-----------------|
| Bug fixes + polish | v1.4.1 | "Fix widget layout", "Normalize docs" | PATCH → v1.4.2 |
| New features | v1.3.2 | "Add widget", "Add Focus sessions" | MINOR → v1.4.0 |
| Refactor + features | v1.3.1 | "Material 3 compliance", "Fix issues" | MINOR → v1.4.0 |
| Major redesign | v1.0.0 | [Major UI rewrite] | MAJOR → v2.0.0 |

### versionCode

`versionCode` always increments by +1 regardless of version strategy.

## Commit Analysis Keywords

Use these keywords in commit messages to help the skill categorize changes accurately:

| Category | Keywords | Version Impact |
|----------|----------|-----------------|
| **Features** | `feat:`, `feature:`, `add:`, `implement:` | MINOR (1.Y.0) |
| **Bug Fixes** | `fix:`, `bugfix:`, `patch:`, `resolve:` | PATCH (1.0.Z) |
| **UI/UX** | `ui:`, `ux:`, `design:`, `style:` | MINOR/PATCH based on scope |
| **Refactoring** | `refactor:`, `refactoring:`, `cleanup:` | PATCH (1.0.Z) |
| **Performance** | `perf:`, `performance:`, `optimize:` | MINOR/PATCH based on impact |
| **Architecture** | `arch:`, `architecture:`, `restructure:` | MAJOR if breaking (X.0.0) |
| **Breaking** | `breaking:`, `breaking change:` | MAJOR (X.0.0) |
| **Localization** | `i18n:`, `localize:`, `locale:` | MINOR (1.Y.0) |
| **Testing** | `test:`, `tests:` | PATCH (1.0.Z) |
| **Docs** | `docs:`, `documentation:` | PATCH (1.0.Z) |

**Example well-formatted commits**:
```
feat: Add dark mode support across app
fix: Resolve timer crash on orientation change
perf: Optimize database queries for faster startup
refactor: Extract timer logic into dedicated service
breaking: Change notification permission flow
```

## Configuration

### Discovering `<app-name>`

The skill uses `<app-name>` in tags, bundle filenames, and commit messages. The agent must resolve this dynamically — never hardcode or guess it.

**Discovery order** (use the first that succeeds):
1. **Existing git tags**: Run `git tag --list 'v*'` and extract the app name from the most recent tag pattern (e.g., `vmy-app-1.4.1` → `my-app`)
2. **`settings.gradle.kts`**: Read `rootProject.name` (e.g., `rootProject.name = "my-app"`)
3. **`app/build.gradle.kts`**: Read `namespace` or `applicationId` and use the last segment (e.g., `com.example.myapp` → `myapp`)
4. **Ask the user**: If none of the above yields a clear name, ask: "What app name should I use for tags and bundle filenames?"

Cache the resolved name for the duration of the workflow. Do not re-discover mid-release.

### Naming Convention

Customize version references to match your project:
- **Git tag**: `v<app-name>-X.Y.Z` (e.g., `vmy-app-1.4.2`)
- **Commit message**: `Updating Release version to X.Y.Z`
- **App version name** (in build.gradle.kts): `X.Y.Z` (e.g., `1.4.2`)
- **Release bundle filename**: `<app-name>-vX.Y.Z-release.aab`

### Signing Configuration

The project uses `keystore.properties` (gitignored) with:
- `storeFile` - Path to keystore (e.g., `~/secure/my-app-keystore.jks`)
- `storePassword` - Keystore password
- `keyAlias` - Key alias (e.g., `my-app-key`)
- `keyPassword` - Key password

**Example `keystore.properties`**:
```properties
storeFile=/path/to/your-app-keystore.jks
storePassword=YOUR_STORE_PASSWORD
keyAlias=your-app-key
keyPassword=YOUR_KEY_PASSWORD
```

**Security**: The agent MUST only check that `keystore.properties` exists (`test -f keystore.properties`). NEVER read, log, or display its contents — it contains signing credentials. The same applies to `play-api-key.json`.

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

## Google Play Console Integration

### Setup Requirements

To enable automated uploads to Google Play Console Internal Testing track:

**1. Create a Service Account in Google Cloud Console**:
- Go to [Google Cloud Console](https://console.cloud.google.com/)
- Create a new project or use existing one
- Go to **Service Accounts** → **Create Service Account**
- Grant role: `Editor` (or more restrictive: `roles/androidpublisher.admin`)
- Create a JSON key and save as `play-api-key.json`

**2. Add Service Account to Google Play Console**:
- In [Google Play Console](https://play.google.com/console), go to **Settings** → **API access**
- Click **Link a Google Cloud project** or use existing one
- The service account will appear in the list automatically

**3. Grant Service Account Permissions**:
- In Google Play Console → **Users and permissions**
- The service account should have access to the app
- Verify it has "Admin" or "Release management" permissions

**4. Store `play-api-key.json` Securely**:
- Add to project root (gitignored)
- Never commit to version control
- Keep in secure location (e.g., `~/secure/play-api-key.json`)

### Gradle Plugin Configuration

Use **Gradle Play Publisher** for automation:

**In `gradle/libs.versions.toml`**:
```toml
[versions]
playPublisher = "3.10.0"  # Latest version

[plugins]
gradle-play-publisher = { id = "com.github.triplet.play", version.ref = "playPublisher" }
```

**In `app/build.gradle.kts`**:
```kotlin
plugins {
    id("com.github.triplet.play")
}

play {
    serviceAccountCredentials.set(file("${rootProject.rootDir}/play-api-key.json"))
    track.set("internal")  // Internal Testing track
}
```

### Upload Command

After building the bundle, upload to Internal Testing:
```bash
./gradlew publishReleaseBundle
```

This will:
1. Read `play-api-key.json`
2. Upload the signed AAB to Internal Testing track
3. Use the release notes from the generated template
4. Update version code on Play Console

## Release Notes Generation

**IMPORTANT**: Generate release notes BEFORE building the bundle so they are automatically included in the Play Store upload.

### Supported Locales (Auto-Detected)

The skill automatically detects supported locales by scanning `app/src/main/res/values*` directories and maps them to Google Play locale codes:

**Locale Mapping Reference**:
- `values/` → `en-US` (default English)
- `values-de` → `de-DE` (German)
- `values-es` → `es-ES` (Spanish)
- `values-fr` → `fr-FR` (French)
- `values-pt-rBR` → `pt-BR` (Portuguese Brazil)
- `values-hi` → `hi-IN` (Hindi)
- `values-id` → `id` (Indonesian)
- `values-it` → `it-IT` (Italian)
- `values-ja` → `ja-JP` (Japanese)
- `values-ko` → `ko-KR` (Korean)
- `values-pl` → `pl-PL` (Polish)
- `values-tr` → `tr-TR` (Turkish)
- `values-zh-rCN` → `zh-CN` (Chinese Simplified)
- `values-zh-rHK` → `zh-HK` (Chinese Hong Kong)
- `values-zh-rTW` → `zh-TW` (Chinese Traditional)
- `values-es-rUS` → `es-US` (Spanish US)
- `values-es-r419` → `es-419` (Spanish Latin America)

### Release Notes File Structure

Release notes are stored in `app/src/main/play/release-notes/<locale>/default.txt` and are automatically included when uploading to Play Console via Gradle Play Publisher.

**Directory structure**:
```
app/src/main/play/release-notes/
├── en-US/
│   └── default.txt
├── de-DE/
│   └── default.txt
├── es-ES/
│   └── default.txt
└── fr-FR/
    └── default.txt
```

**Example `default.txt` content**:
```
• Added dark mode support for better night-time viewing
• Fixed crash when rotating device during data sync
• Improved app startup time by 30%
• Updated translations for German and French
```

**Character limit**: 500 characters per locale (Play Console limit)

**Note**: When new locales are added to the app (by creating new `values-*` directories), the skill will automatically create release notes for them.

### Release Notes Guidelines

1. **Review commits**: Check `git log` since the last release to understand what changed
2. **User-focused**: Write for end users, not developers
3. **Concise**: Keep each locale to 3-5 bullet points maximum (500 character limit)
4. **Highlight**:
   - New features
   - UI/UX improvements
   - Bug fixes
   - Security updates (if applicable)
   - Performance improvements
5. **Translate appropriately**: Ensure translations are natural and accurate for each locale
6. **Format**: Use bullet points (•) for readability

### Automated Release Notes Generation

The skill will:
1. **Scan** `app/src/main/res/` for all `values*` directories to detect supported locales
2. **Map** each directory to its corresponding Google Play locale code
3. **Analyze commits** since last release to generate user-friendly release notes
4. **Create files** in `app/src/main/play/release-notes/<locale>/default.txt` for each locale
5. **Commit** release notes files to the repository
6. **Embed** release notes automatically in the bundle upload via Gradle Play Publisher

This ensures release notes are always included in Play Store uploads and versioned with the code.

## Command Template

Use these commands in order:

```bash
# 1. Generate release notes (before building)
# Create app/src/main/play/release-notes/<locale>/default.txt files
# Then commit them
git add app/src/main/play/release-notes/
git commit -m "feat: Add localized release notes for vX.Y.Z"

# 2. Pre-commit verification
./gradlew assembleDebug && ./gradlew detekt && ./gradlew test

# 3. Commit version bump and tag
git add app/build.gradle.kts
git commit -m "Updating Release version to X.Y.Z"
git tag -a "v<app-name>-X.Y.Z" -m "v<app-name>-X.Y.Z"
git push origin main
git push origin "v<app-name>-X.Y.Z"

# 4. Build signed bundle (with embedded release notes)
./gradlew :app:copyReleaseBundle

# 5. Upload to Play Console Internal Testing (optional)
./gradlew publishReleaseBundle
```

## Bundle Output

The signed bundle is output to:
```
app/build/release/<app-name>-vX.Y.Z-release.aab
```

The bundle naming is automatically handled by the `copyReleaseBundle` task in `app/build.gradle.kts`.

## Completion Report

After the workflow finishes (whether fully or partially), present a summary to the user:

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

If any step was skipped or failed, note it in the summary so the user knows exactly where things stand.

## Google Play Console Upload

### Optional Internal Testing Upload

After generating release notes and building the bundle, the skill can optionally upload the release candidate to Google Play Console's Internal Testing track.

**Prerequisites**:
- `play-api-key.json` must exist in project root
- Google Cloud service account must be configured (see Setup Requirements above)
- Gradle Play Publisher plugin configured in `app/build.gradle.kts`

**Upload Process**:
1. Generate release notes in `app/src/main/play/release-notes/<locale>/default.txt` files
2. Commit release notes to repository
3. Build signed bundle with embedded release notes
4. Ask user for confirmation to upload to Internal Testing
5. Run `./gradlew publishReleaseBundle` to upload AAB with release notes
6. Monitor upload progress and report status
7. Provide Play Console link for testing/review

**Automatic Release Notes Inclusion**:
- Release notes in `app/src/main/play/release-notes/` are automatically included in the upload
- No manual copy-paste required
- Release notes are versioned with the code in git
- Future uploads will include updated release notes automatically

**Manual Alternative**:
If automated upload is not configured, you can manually upload the bundle from `app/build/release/` to Play Console and the release notes will still be included in the AAB metadata.

## Guardrails

**Pre-flight**:
- Abort if not on `main`, the working tree is dirty, or `keystore.properties` is missing.
- For Play Console upload: Also verify `play-api-key.json` exists and is valid.
- Stop and report errors if Gradle or Git commands fail.
- Do NOT commit if pre-commit verification fails.

**Analysis & Versioning**:
- ALWAYS fetch latest release tag using `git describe --tags --abbrev=0`
- ALWAYS scan commits since last release tag
- ALWAYS analyze commit messages for change type keywords
- ALWAYS present version bump recommendation with reasoning (MAJOR/MINOR/PATCH)
- ALWAYS allow user to override recommendation if they disagree
- ALWAYS increment `versionCode` by +1 regardless of version strategy

**Naming & Format**:
- ALWAYS use consistent tag naming format (configurable per project)
- ALWAYS use X.Y.Z semantic versioning (e.g., 1.4.1, 2.0.0)
- ALWAYS use version commit message format: `Updating Release version to X.Y.Z`

**Release Notes**:
- ALWAYS scan `app/src/main/res/` to detect supported locales dynamically
- ALWAYS generate release notes BEFORE building the bundle
- ALWAYS create `app/src/main/play/release-notes/<locale>/default.txt` files for all detected locales
- ALWAYS analyze commits since last release to write user-friendly, localized release notes
- ALWAYS commit release notes files to the repository before building
- ALWAYS ensure release notes are under 500 characters per locale (Play Console limit)

**Quality Gates**:
- Never skip pre-commit verification
- Never force-push or amend already-pushed commits
- Never release with uncommitted changes

## Error Recovery

When a step fails, follow these recovery paths instead of aborting immediately:

### Gradle Build Failure (`assembleDebug`)
1. Run `./gradlew clean` and retry the build once
2. If it fails again, report the build error output to the user and stop
3. Do NOT proceed to commit or tag — the release is blocked

### Gradle Detekt / Test Failure
1. Report which checks failed with the relevant output
2. Do NOT retry automatically — linting and test failures require code changes
3. Ask the user how they want to proceed (fix issues or skip this release)

### Git Tag Already Exists
1. If the computed tag (e.g., `v<app-name>-1.5.0`) already exists, do NOT overwrite it
2. Inform the user: "Tag `vX.Y.Z` already exists. This likely means the version was already released."
3. Suggest incrementing to the next version (e.g., `1.5.1`) and ask the user to confirm

### Git Push Failure
1. Check if the failure is due to diverged branches (`git status` / `git log --oneline origin/main..HEAD`)
2. Report the conflict to the user — do NOT force-push or rebase automatically
3. The local commit and tag are still intact; the user can resolve and retry manually

### `publishReleaseBundle` Failure (Play Console Upload)
1. Check if `play-api-key.json` exists and is readable
2. If auth error: remind the user to verify service account permissions in Google Play Console
3. If network error: ask the user if they want to retry or upload manually
4. The signed bundle is still available at `app/build/release/` for manual upload

### First Release (No Prior Tags)
1. If `git describe --tags --abbrev=0` fails (no tags found), this is a first release
2. Default to version `1.0.0` with `versionCode = 1`
3. Present this to the user: "No previous release tags found. Starting at v1.0.0. Override?"
4. For release notes, use all commits on the branch (or since initial commit) as the changelog source

## Usage as a Claude Code Skill

To use this as a skill in Claude Code:

1. Create a skill directory: `.claude/skills/android-release-bump/`
2. Copy this content to `.claude/skills/android-release-bump/SKILL.md`
3. Customize the skill title and trigger description for your project
4. Update tag naming convention and bundle filename patterns
5. Invoke with: `/android-release-bump` or ask Claude to "bump the release version"

**Skill Header Example**:
```markdown
# Android Release Bump

Bump release version in app/build.gradle.kts, commit, tag, push to origin/main, build a signed release bundle, and generate multi-locale release notes. Use when asked to bump the release, update versionName/versionCode, bump major/minor/patch, or cut a new release.
```

## License

This skill documentation is provided as-is for use in Android projects. Customize as needed for your workflow.
