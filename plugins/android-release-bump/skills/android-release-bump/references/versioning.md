# Release Bump Recommendation Rules

Recommend the version bump from the user-visible impact of commits since the latest release tag.

## Recommend `major`

Use `major` when commits clearly indicate a breaking change for users, integrators, or supported environments, for example:

- Removal of a feature or supported workflow
- Backward-incompatible configuration or data changes
- Platform or minimum-version jumps that strand existing users

## Recommend `minor`

Use `minor` when there are new user-visible features or meaningful improvements without breaking existing behavior, for example:

- New screens, flows, or settings
- New integrations or capabilities
- Significant enhancements users can notice

## Recommend `patch`

Use `patch` when the release is primarily:

- Bug fixes
- Stability improvements
- Small UX polish
- Dependency or internal maintenance without material feature changes

## Ambiguous Cases

- If the diff mixes feature work and fixes, recommend the highest applicable bump and explain why.
- If commit messages are unclear, inspect the changed files before recommending.
- If there is still uncertainty after inspection, recommend the lower plausible bump and call out the uncertainty explicitly for user confirmation.
