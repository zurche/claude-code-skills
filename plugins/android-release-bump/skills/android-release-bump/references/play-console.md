# Google Play Upload Setup

Load this only if the user asks to upload to Google Play Internal Testing.

## Preconditions

- `play-api-key.json` must exist and be configured for the Gradle Play Publisher setup used by the repo.
- The app must already be configured with the correct `applicationId`.
- The user must explicitly confirm before upload.

## Expected Workflow

1. Confirm the release variant and track.
2. Verify that the repo already has Gradle Play Publisher or equivalent upload wiring.
3. Build the signed bundle.
4. Upload with the repository's existing Gradle task, typically `publishReleaseBundle`.
5. Report success or the exact blocking error.

## Guardrails

- Never create or rotate Google Play credentials yourself.
- Never print secrets from credential files.
- Never upload without an explicit user confirmation in the current session.
