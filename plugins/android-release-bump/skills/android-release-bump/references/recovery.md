# Failure Recovery Matrix

## Safe Boundaries

- Before version file changes: stop and report the blocking issue.
- After version file changes but before commit: revert only if the user explicitly asks. Otherwise report the modified files and next steps.
- After commit but before tag: report the commit SHA and propose the next safe action.
- After tag but before push: report the tag name and commit SHA and wait for direction.
- After push but before signed bundle: report exactly what is already published to the remote.
- After signed bundle but before Play upload: report artifact path and whether the repo and tag are already pushed.
- After Play upload failure: report whether the upload was attempted, the exact task that failed, and whether the bundle remains valid for retry.

## Completion Summary Format

When the workflow completes or stops at a safe boundary, print:

```text
Result: <completed|stopped>
Version: <X.Y.Z>
Version code: <N>
Commits: <sha list or none>
Tag: <tag name or none>
Pushed: <yes|no>
Bundle: <path or none>
Play upload: <not requested|completed|failed>
Next action: <single best next step>
```
