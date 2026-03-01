# Recovery And Completion Reference

Use this file only when a step fails or when printing the final summary.

## Failure Matrix

| Failure | Recovery |
|---|---|
| `assembleDebug` fails | Run `./gradlew clean` and retry once. If it still fails, stop before commit/tag and report the error. |
| `detekt` or tests fail | Report failed checks and stop. Ask whether to fix code or abort release. |
| Tag already exists | Do not overwrite. Suggest next version (for example `1.5.1`). |
| Push fails | Report divergence/conflict. Do not force-push. Keep local commit/tag intact. |
| Play upload fails | Verify `play-api-key.json` and service account permissions. Offer retry or manual upload. |
| No previous tags | Recommend `1.0.0`; set `versionCode` to current configured value +1 (or `1` if no explicit `versionCode` exists), then request confirmation. |

## Completion Summary Template

```markdown
## Release Summary

| Field | Value |
|---|---|
| Previous version | <prevVersion> (versionCode <prevCode>) |
| New version | <newVersion> (versionCode <newCode>) |
| Bump type | <major/minor/patch> |
| Git tag | v<app-name>-<newVersion> |
| Commits included | <count> since <lastTag or start> |
| Release notes | Generated for <N> locales (<list>) |
| Bundle path | app/build/release/<app-name>-v<newVersion>-release.aab |
| Pushed to remote | Yes/No |
| Play upload | Uploaded/Skipped/Failed (<reason>) |
```
