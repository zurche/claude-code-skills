# Locale Mapping and Play Release Note Limits

Use the app's existing `play/release-notes/<locale>/` directories as the source of truth.

## Rules

- Generate one `default.txt` per existing locale directory.
- If no locale directories exist, generate only `en-US/default.txt`.
- Keep each `default.txt` within Google Play's current character limits configured in the target repo.
- Preserve locale directory names exactly as they already exist in the project.

## Writing Guidance

- Prefer one short paragraph or 2 to 4 bullets per locale.
- Focus on user-visible changes, stability, bug fixes, and notable performance improvements.
- Avoid internal implementation jargon unless it explains a user-visible change.
- Do not claim features that are not in the diff.

## Fallback

If the project has locale directories that are unfamiliar or ambiguous, mirror the English content conservatively rather than inventing a translation and ask the user whether they want human-localized copy before publishing.
