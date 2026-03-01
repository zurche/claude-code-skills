# CLI Agent Skills Collection

Reusable skills for CLI coding agents.

## Skill Format

Skills in this repo follow canonical folder format for progressive disclosure and lower token usage:

```text
<skill-name>/
|-- SKILL.md
|-- agents/openai.yaml
`-- references/
```

Keep only core workflow and guardrails in `SKILL.md`. Move detailed tables, setup guides, and fallback matrices into `references/`.

## Available Skills

### Android Release Bump

Canonical path: [android-release-bump/SKILL.md](./android-release-bump/SKILL.md)

What it handles:

- Recommend and apply `major`/`minor`/`patch` bumps
- Update `versionName` and `versionCode`
- Generate localized Play release notes
- Run release verification checks
- Commit, tag, push, build signed AAB, and optionally upload to Internal Testing

## Installation

### Claude Code

```bash
cd your-android-project
mkdir -p .claude/skills
cp -R /path/to/agent-skills/android-release-bump .claude/skills/
```

### Codex

Use the skill folder directly in your Codex skills path or copy `SKILL.md` plus `references/` into your project skill directory.

### Other Agents

If an agent only supports a single markdown file, use `android-release-bump/SKILL.md` as the primary instructions and keep the referenced files available.

## License

MIT
