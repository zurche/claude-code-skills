# Adding Plugins

This repo keeps one plugin directory per reusable workflow. Each plugin directory is the single source of truth for both Claude Code and Codex packaging.

## Required Layout

```text
plugins/<plugin-name>/
├── .claude-plugin/plugin.json
├── .codex-plugin/plugin.json
├── skills/<skill-name>/SKILL.md
├── skills/<skill-name>/references/   # optional
├── skills/<skill-name>/scripts/      # optional
├── agents/                           # optional Claude agents
├── .mcp.json                         # optional MCP config
└── assets/                           # optional icons, screenshots
```

## Checklist

1. Create `plugins/<plugin-name>/`.
2. Add `.claude-plugin/plugin.json`.
3. Add `.codex-plugin/plugin.json`.
4. Add at least one skill under `skills/<skill-name>/SKILL.md`.
5. Keep every referenced file inside the plugin directory.
6. Add the plugin to [../.claude-plugin/marketplace.json](../.claude-plugin/marketplace.json).
7. Add the plugin to [../.agents/plugins/marketplace.json](../.agents/plugins/marketplace.json).
8. Validate both marketplace files and both manifests before committing.

## Manifest Rules

### Claude

- Use `.claude-plugin/plugin.json`
- For same-repo relative-path plugins, store the published version in the root marketplace entry
- Keep plugin component paths relative and inside the plugin root

Minimal example:

```json
{
  "name": "example-plugin",
  "description": "Reusable workflow for a specific task",
  "skills": "./skills"
}
```

### Codex

- Use `.codex-plugin/plugin.json`
- Keep the plugin `version` in this manifest
- Point `skills`, `mcpServers`, and `apps` at plugin-local paths with `./`

Minimal example:

```json
{
  "name": "example-plugin",
  "version": "1.0.0",
  "description": "Reusable workflow for a specific task",
  "skills": "./skills/"
}
```

## MCP Packaging

When a plugin ships MCP servers:

- Put the config at `plugins/<plugin-name>/.mcp.json`
- Reference it from `.codex-plugin/plugin.json` with `"mcpServers": "./.mcp.json"`
- For Claude, either keep `.mcp.json` at the plugin root or point `mcpServers` at it from `.claude-plugin/plugin.json`
- Do not reference files outside the plugin root

## Versioning

- Bump the Claude marketplace entry version and the Codex manifest version together
- Use semver
- Treat plugin directory structure changes as plugin version changes

## Validation

Recommended local checks:

```bash
python3 -m json.tool .claude-plugin/marketplace.json >/dev/null
python3 -m json.tool .agents/plugins/marketplace.json >/dev/null
python3 -m json.tool plugins/<plugin-name>/.claude-plugin/plugin.json >/dev/null
python3 -m json.tool plugins/<plugin-name>/.codex-plugin/plugin.json >/dev/null
```
