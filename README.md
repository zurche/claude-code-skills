# Agent Skills Marketplace

Shared plugin marketplace for Claude Code and OpenAI Codex.

This repo is the canonical source for reusable cross-repo plugins. Each plugin is self-contained and can ship:

- `skills/`
- `agents/`
- `.mcp.json`
- hooks or other plugin assets

The repository publishes both marketplace formats documented by Anthropic and OpenAI:

- Claude Code: [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json)
- Codex: [`.agents/plugins/marketplace.json`](./.agents/plugins/marketplace.json)

## Layout

```text
.
├── .claude-plugin/marketplace.json
├── .agents/plugins/marketplace.json
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/plugin.json
        ├── .codex-plugin/plugin.json
        ├── skills/
        ├── agents/            # optional
        ├── .mcp.json          # optional
        └── assets/            # optional
```

Keep every file a plugin needs inside that plugin directory. Claude installs plugins into a cache, and relative references outside the plugin root will break after installation.

## Included Plugins

### Android Release Bump

- Plugin: [plugins/android-release-bump](./plugins/android-release-bump)
- Skill: [plugins/android-release-bump/skills/android-release-bump/SKILL.md](./plugins/android-release-bump/skills/android-release-bump/SKILL.md)
- Purpose: bump Android app versions, generate Play release notes, run verification, tag releases, build signed bundles, and optionally upload to Internal Testing

## Install In Claude Code

Claude can consume this repo directly as a Git-backed marketplace.

```bash
claude plugin marketplace add https://github.com/zurche/agent-skills.git
claude plugin install android-release-bump@zurche-shared-plugins
```

Inside Claude Code, the equivalent interactive commands are:

```text
/plugin marketplace add https://github.com/zurche/agent-skills.git
/plugin install android-release-bump@zurche-shared-plugins
```

Use GitHub or another Git remote when sharing this marketplace. Anthropic's docs note that relative plugin paths only work for Git-backed marketplaces, not for a raw `marketplace.json` URL.

## Install In Codex

Codex currently documents repo-scoped and personal marketplaces as local JSON catalogs that point at plugin directories with `./`-prefixed relative paths. This repo therefore serves as the plugin source, and each consuming repo or home directory owns the marketplace file that points into it.

### Personal Marketplace

If this repo is cloned to `~/Repo/agent-skills`, create `~/.agents/plugins/marketplace.json`:

```json
{
  "name": "zurche-shared-plugins",
  "interface": {
    "displayName": "Zurche Shared Plugins"
  },
  "plugins": [
    {
      "name": "android-release-bump",
      "source": {
        "source": "local",
        "path": "./Repo/agent-skills/plugins/android-release-bump"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Release Engineering"
    }
  ]
}
```

Restart Codex after updating the marketplace file.

### Repo-Scoped Marketplace

If another repo vendors this repo under `./vendor/agent-skills`, that repo can expose only the plugins it wants by adding its own `.agents/plugins/marketplace.json`:

```json
{
  "name": "project-plugins",
  "interface": {
    "displayName": "Project Plugins"
  },
  "plugins": [
    {
      "name": "android-release-bump",
      "source": {
        "source": "local",
        "path": "./vendor/agent-skills/plugins/android-release-bump"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Release Engineering"
    }
  ]
}
```

To install only one plugin in Codex, keep just that plugin entry in the consuming marketplace file. To expose the whole catalog from this repo as-is, reuse the entries from [`.agents/plugins/marketplace.json`](./.agents/plugins/marketplace.json) and adjust `source.path` so it is relative to the consuming marketplace root.

## Adding Plugins

Add new plugins under `plugins/<plugin-name>/` and update both marketplace catalogs.

Authoring rules:

- Put Claude metadata in `.claude-plugin/plugin.json`
- Put Codex metadata in `.codex-plugin/plugin.json`
- Put shared skills in `skills/<skill-name>/SKILL.md`
- Put plugin-local references and scripts next to the skill
- Put optional MCP config in `.mcp.json`
- Keep paths relative and inside the plugin root

Versioning rules:

- Claude marketplace entries should carry the version for same-repo relative-path plugins
- Codex plugin versions live in `.codex-plugin/plugin.json`
- Bump both together when releasing a plugin change

See [docs/ADDING_PLUGINS.md](./docs/ADDING_PLUGINS.md) for the exact checklist.

## License

MIT
