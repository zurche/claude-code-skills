# Android Release Skills for Claude Code

A collection of reusable Claude Code skills for Android development and release management.

## Overview

This repository contains production-ready skills that automate common Android development workflows, particularly around release management, versioning, and Google Play Console integration.

## Skills

### 🚀 [Android Release Bump](./android-release-bump.md)

Comprehensive release automation skill that handles the complete Android release workflow:

- **Intelligent Semantic Versioning**: Analyzes git commits to recommend MAJOR/MINOR/PATCH version bumps
- **Automated Release Notes**: Generates localized release notes for all supported languages
- **Pre-commit Verification**: Runs build, linting, and tests before committing
- **Google Play Integration**: Optionally uploads to Internal Testing track
- **Multi-locale Support**: Auto-detects app locales and generates appropriate release notes

**Key Features:**
- Commit message analysis for version recommendations
- Automated `versionCode` and `versionName` updates
- Git tagging with customizable naming conventions
- Signed bundle generation
- Gradle Play Publisher integration

## Installation

### Using as Claude Code Skills

1. **Clone or copy the skill files** to your project:
   ```bash
   mkdir -p .claude/skills/android-release-bump
   cp android-release-bump.md .claude/skills/android-release-bump/SKILL.md
   ```

2. **Customize the skill** for your project:
   - Update tag naming convention (e.g., `v<your-app-name>-X.Y.Z`)
   - Adjust bundle filename pattern
   - Configure pre-commit verification commands

3. **Invoke in Claude Code**:
   ```
   /android-release-bump
   ```
   Or simply ask: "bump the release version"

## Requirements

- **Android Project**: Gradle-based Android project with `app/build.gradle.kts`
- **Git Repository**: Version-controlled project with semantic versioning
- **Keystore** (optional): For signed release builds (`keystore.properties`)
- **Play Console Access** (optional): Service account for automated uploads (`play-api-key.json`)

## Configuration

Each skill includes detailed configuration instructions. Key customization points:

- **Tag Naming**: Customize git tag format per project
- **Bundle Naming**: Configure output AAB filename
- **Verification Steps**: Adjust pre-commit checks (detekt, lint, tests)
- **Locales**: Automatically detected from `app/src/main/res/values*` directories

## Workflow Example

Here's what the Android Release Bump skill does:

```bash
# 1. Analyzes commits since last release
git log --oneline <last-tag>..HEAD

# 2. Recommends version bump (MAJOR/MINOR/PATCH)
# Based on commit keywords: feat:, fix:, breaking:, etc.

# 3. Updates version in build.gradle.kts
versionCode = 15
versionName = "1.5.0"

# 4. Generates localized release notes
app/src/main/play/release-notes/en-US/default.txt
app/src/main/play/release-notes/de-DE/default.txt
...

# 5. Runs verification
./gradlew assembleDebug && ./gradlew detekt && ./gradlew test

# 6. Commits and tags
git commit -m "Updating Release version to 1.5.0"
git tag -a "vmy-app-1.5.0" -m "vmy-app-1.5.0"

# 7. Builds signed bundle
./gradlew :app:copyReleaseBundle

# 8. Optionally uploads to Play Console
./gradlew publishReleaseBundle
```

## Best Practices

### Commit Message Conventions

Use conventional commit formats to enable intelligent version bumping:

```
feat: Add new feature → MINOR bump (1.4.0 → 1.5.0)
fix: Bug fix → PATCH bump (1.4.0 → 1.4.1)
breaking: Breaking change → MAJOR bump (1.4.0 → 2.0.0)
```

### Release Notes

- Keep under 500 characters per locale (Play Console limit)
- Focus on user-facing changes
- Use bullet points (•) for readability
- Translate appropriately for each locale

### Security

- Never commit `keystore.properties` or `play-api-key.json`
- Store credentials securely (e.g., encrypted cloud storage)
- Use `.gitignore` to exclude sensitive files

## Contributing

This is a public collection of reusable Android development skills. Contributions are welcome!

To contribute:
1. Fork the repository
2. Add or improve skills
3. Test thoroughly in your projects
4. Submit a pull request with clear documentation

## License

MIT License - Feel free to use, modify, and distribute these skills in your projects.

## Credits

Created for use with [Claude Code](https://claude.ai/code) by Anthropic.

Skills developed and tested in production Android projects.

## Support

For issues or questions:
- Open an issue in this repository
- Check the [Claude Code documentation](https://docs.anthropic.com/claude/docs/claude-code)
- Review individual skill documentation for detailed setup instructions

---

**Note**: These skills are provided as-is. Always test in your specific project environment before relying on them for production releases.
