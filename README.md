# CLI Agent Skills Collection

Reusable skills for CLI coding agents. Each skill is a self-contained markdown document that any agent supporting custom instructions can pick up and execute.

Tested with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [OpenAI Codex CLI](https://github.com/openai/codex). Should work with any CLI agent that supports loading custom instruction files.

## Available Skills

### [Android Release Bump](./android-release-bump.md)

Complete release automation for Android projects:

- Intelligent semantic versioning from git commit analysis
- Multi-locale release notes generation
- Pre-commit verification (build, lint, tests)
- Signed bundle building
- Optional Google Play Console upload to Internal Testing

## Installation

Skills are plain markdown files. Copy them into wherever your agent reads custom instructions from.

### Claude Code

Claude Code loads skills from `.claude/skills/*/SKILL.md` inside your project:

```bash
cd your-android-project
mkdir -p .claude/skills/android-release-bump
cp /path/to/android-release-bump.md .claude/skills/android-release-bump/SKILL.md
```

Invoke with `/android-release-bump` or describe what you want: "bump the release version".

### OpenAI Codex CLI

Codex reads instructions from `AGENTS.md` at the project root (or in subdirectories for scoped instructions):

```bash
cd your-android-project
cp /path/to/android-release-bump.md AGENTS.md
```

Then ask Codex to "bump the release version" or "cut a new release".

If you already have an `AGENTS.md`, append the skill content or reference it:

```bash
echo -e "\n\n$(cat /path/to/android-release-bump.md)" >> AGENTS.md
```

### Other Agents

Most CLI agents support some form of custom instruction loading. Common patterns:

| Agent | Instruction file | Notes |
|-------|-----------------|-------|
| Claude Code | `.claude/skills/<name>/SKILL.md` | Auto-loaded, invocable via `/<name>` |
| Codex CLI | `AGENTS.md` (project root) | Loaded automatically on startup |
| Cursor | `.cursor/rules/*.md` | Place skill as a rule file |
| Aider | `.aider.conf.yml` `read:` field | Reference the skill markdown file |
| Generic | Project README or system prompt | Paste skill content into agent context |

The skill files are agent-agnostic markdown — no special syntax required. If your agent can read a markdown file as instructions, it can use these skills.

## Customization

After copying a skill into your project, you may want to adjust:

- **Tag naming convention**: Default is `v<app-name>-X.Y.Z`. Some projects prefer `v1.2.3` or `release/1.2.3`.
- **Verification commands**: Default is `./gradlew assembleDebug && ./gradlew detekt && ./gradlew test`. Swap `detekt` for `lint` or add your own checks.
- **Bundle task name**: Default is `copyReleaseBundle`. Adjust if your Gradle setup differs.
- **Commit message format**: Default is `Updating Release version to X.Y.Z`.

## Contributing

1. Fork this repository
2. Add your skill as a `.md` file at the repo root
3. Test with at least two different CLI agents
4. Submit a PR with a description of what the skill does

### Skill Guidelines

- **Agent-agnostic**: Write for any agent, not just one tool. Use "the agent" instead of "Claude" or "Codex".
- **Self-contained**: A single markdown file should have everything the agent needs.
- **Safe**: Include confirmation checkpoints for destructive or irreversible actions.
- **Generic**: Use placeholders instead of hardcoded project-specific values.
- **Tested**: Verify in real projects before submitting.

## License

MIT
