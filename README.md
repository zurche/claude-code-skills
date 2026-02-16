# Claude Code Skills Collection

A curated collection of production-ready Claude Code skills for software development, automation, and productivity.

## Overview

This repository contains reusable skills that extend Claude Code's capabilities across various development workflows. Each skill is battle-tested in real projects and designed to be easily customized for your specific needs.

## Available Skills

### 🚀 [Android Release Bump](./android-release-bump.md)

Comprehensive release automation for Android projects:

- **Intelligent Semantic Versioning**: Analyzes git commits to recommend MAJOR/MINOR/PATCH version bumps
- **Automated Release Notes**: Generates localized release notes for all supported languages
- **Pre-commit Verification**: Runs build, linting, and tests before committing
- **Google Play Integration**: Optionally uploads to Internal Testing track
- **Multi-locale Support**: Auto-detects app locales and generates appropriate release notes

**Perfect for**: Android developers releasing apps to Google Play Store

---

## Coming Soon

More skills will be added to this collection covering:
- **iOS Release Management**: Similar automation for iOS/Xcode projects
- **Code Review Automation**: PR review checklist generation
- **Documentation Generation**: Auto-generate docs from code
- **Test Coverage Analysis**: Coverage reports and improvement suggestions
- **Dependency Management**: Automated dependency updates and security audits

## Installation

### Using as Claude Code Skills

Each skill can be installed independently:

1. **Navigate to your project**:
   ```bash
   cd your-project
   ```

2. **Copy the skill file** to your Claude skills directory:
   ```bash
   mkdir -p .claude/skills/<skill-name>
   cp <skill-file>.md .claude/skills/<skill-name>/SKILL.md
   ```

3. **Customize the skill** for your project (tag naming, commands, etc.)

4. **Invoke in Claude Code**:
   ```
   /<skill-name>
   ```
   Or describe what you want: "bump the release version"

## Skill Structure

Each skill follows this format:

```markdown
# Skill Name

Brief description and trigger phrases.

## Overview
What the skill does

## Workflow
Step-by-step process

## Configuration
How to customize

## Usage
How to use the skill

## Requirements
What you need

## Examples
Real-world usage
```

## Contributing

Contributions are welcome! This collection aims to be a community resource for Claude Code users.

### How to Contribute

1. **Fork this repository**
2. **Add your skill**:
   - Create a new `.md` file with clear documentation
   - Follow the existing skill structure
   - Include configuration instructions
   - Add real-world examples
3. **Test thoroughly** in actual projects
4. **Submit a pull request** with:
   - Clear description of what the skill does
   - Any prerequisites or setup requirements
   - Example usage

### Skill Guidelines

- **Generic**: Remove project-specific references
- **Documented**: Include setup and configuration instructions
- **Tested**: Verify it works in multiple scenarios
- **Customizable**: Use placeholders for project-specific values
- **Safe**: Include guardrails and validation checks

## Best Practices

### Organizing Skills

```
.claude/
└── skills/
    ├── android-release/
    │   └── SKILL.md
    ├── code-review/
    │   └── SKILL.md
    └── test-coverage/
        └── SKILL.md
```

### Customization

Each skill includes placeholders for project-specific values:
- `<app-name>`: Your application name
- `<tag-format>`: Your preferred git tag format
- `<verification-commands>`: Your build/test commands

### Security

- Never commit credentials (`keystore.properties`, API keys, etc.)
- Use `.gitignore` to exclude sensitive files
- Store secrets in secure locations outside version control

## Use Cases

### Development Workflows
- Release management and versioning
- Code review automation
- Test coverage tracking
- Documentation generation

### DevOps & CI/CD
- Automated deployments
- Build verification
- Release note generation
- Change log automation

### Project Management
- Sprint planning assistance
- Issue triaging
- PR review checklists
- Status report generation

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude/docs/claude-code)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)

## Support

For issues or questions:
- **Skill-specific issues**: Check the skill's documentation
- **General questions**: Open an issue in this repository
- **Feature requests**: Submit an issue with the `enhancement` label

## License

MIT License - Feel free to use, modify, and distribute these skills in your projects.

## Credits

Created for use with [Claude Code](https://claude.ai/code) by Anthropic.

Skills developed and tested in production software projects.

---

**Note**: These skills are provided as-is. Always test in your specific environment before relying on them for production use.
