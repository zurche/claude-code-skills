# Google Play Upload Reference

Use this file only when Play upload is requested or failing.

## Required Files

- `keystore.properties` in project root for signed release builds.
- `play-api-key.json` in project root for automated Play upload.

Never print secret values from either file.

## Service Account Setup

1. Create a Google Cloud service account and JSON key.
2. Grant `roles/androidpublisher.admin` (or equivalent release permission scope).
3. Link the Cloud project in Play Console API access.
4. Ensure the service account has Play Console access for release management.

## Gradle Play Publisher Wiring

Example `gradle/libs.versions.toml`:

```toml
[versions]
playPublisher = "3.10.0"

[plugins]
gradle-play-publisher = { id = "com.github.triplet.play", version.ref = "playPublisher" }
```

Example `app/build.gradle.kts`:

```kotlin
plugins {
    id("com.github.triplet.play")
}

play {
    serviceAccountCredentials.set(file("${rootProject.rootDir}/play-api-key.json"))
    track.set("internal")
}
```

## Upload Command

Run:

```bash
./gradlew publishReleaseBundle
```

If automation is not configured, upload the AAB from `app/build/release/` manually in Play Console.
