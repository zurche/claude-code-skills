# Versioning Reference

Use this file only when deciding or explaining the bump type.

## Bump Categories

| Type | Rule | Example |
|---|---|---|
| `major` | Breaking workflow changes, incompatible behavior changes, or significant architecture rewrites that change user expectations | `1.4.1 -> 2.0.0` |
| `minor` | New features, notable UI/UX improvements, performance/reliability improvements visible to users, or localization expansion | `1.4.1 -> 1.5.0` |
| `patch` | Bug fixes, minor UI tweaks, maintenance refactors, docs/tests/dependency updates without clear user-facing changes | `1.4.1 -> 1.4.2` |

Always increment `versionCode` by `+1` from the current configured value.
Never reset or lower an existing `versionCode` in first-release flows.

## Commit Signal Keywords

| Category | Common prefixes | Suggested impact |
|---|---|---|
| Feature | `feat:`, `feature:`, `add:`, `implement:` | `minor` |
| Bug fix | `fix:`, `bugfix:`, `patch:`, `resolve:` | `patch` |
| UI/UX | `ui:`, `ux:`, `design:`, `style:` | `minor` or `patch` |
| Refactor | `refactor:`, `cleanup:` | `patch` |
| Performance | `perf:`, `optimize:` | `minor` or `patch` |
| Architecture | `arch:`, `architecture:`, `restructure:` | `major` if breaking |
| Breaking | `breaking:`, `breaking change:` | `major` |
| Localization | `i18n:`, `localize:`, `locale:` | `minor` |
| Test/docs | `test:`, `docs:` | `patch` |

## Ambiguous Commit Handling

When commit messages are weak or inconsistent:

1. Use `git diff --stat <last-tag>..HEAD`.
2. Use size and scope heuristics:
   - Small scope (1-5 files, low churn): default `patch`.
   - Medium scope (5-20 files or new feature files): consider `minor`.
   - Large structural scope (20+ files, core architecture paths): consider `major`.
3. Apply file-type hints:
   - Tests/docs only: `patch`.
   - New feature source files: usually `minor`.
   - Navigation/DI/base architecture changes: possibly `major`.
4. Default to `patch` when uncertain and explicitly tell the user why.

## First Release Rule

If no tags exist, recommend `1.0.0` and `versionCode = 1`, then ask for confirmation before proceeding.
