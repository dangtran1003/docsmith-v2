# Publishing & Installation — dangtran1003/docsmith-v2

This repository is a **Claude Code plugin marketplace** containing the `docsmith` plugin. It follows the official Claude Code plugin format introduced in late 2025.

## Repository structure

```
docsmith-v2/
├── .claude-plugin/
│   ├── marketplace.json          # marketplace catalog (required)
│   └── plugin.json               # plugin manifest (required)
├── skills/
│   └── docsmith/                 # the actual skill content
│       ├── SKILL.md
│       ├── deploy-reference.md
│       ├── intake-reference.md
│       ├── translate-reference.md
│       ├── process-reference.md
│       ├── tools-reference.md
│       ├── presets/
│       └── templates/
└── (top-level docs: README, CHANGELOG, INTAKE_GUIDE, HOW_IT_WORKS, COMPARISON)
```

When `/plugin marketplace add dangtran1003/docsmith-v2` runs, Claude Code reads `.claude-plugin/marketplace.json` to discover plugins, then installs the `docsmith` plugin which lives at `skills/docsmith/`.

## Installation methods

### 1. Marketplace install (recommended)

```bash
# In Claude Code:
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2
```

The `@` syntax: `<plugin-name>@<marketplace-name>`. Marketplace name comes from `.claude-plugin/marketplace.json` `name` field (`dangtran1003-docsmith-v2`).

### 2. Update via marketplace

```bash
/plugin marketplace update dangtran1003-docsmith-v2
```

This pulls latest from GitHub and refreshes plugin metadata. Reinstall plugins to apply updates.

### 3. Direct skill install (without marketplace)

If you don't want the marketplace overhead, copy just the skill folder:

```bash
# Personal (all projects)
git clone https://github.com/dangtran1003/docsmith-v2.git /tmp/docsmith-v2
cp -r /tmp/docsmith-v2/skills/docsmith ~/.claude/skills/docsmith

# Or symlink (for live updates):
ln -s /tmp/docsmith-v2/skills/docsmith ~/.claude/skills/docsmith
```

```bash
# Project-level (committed to repo)
mkdir -p .claude/skills
cp -r /path/to/docsmith-v2/skills/docsmith .claude/skills/
```

### 4. External directory (for development)

When working on the skill itself:

```bash
claude --add-dir /path/to/docsmith-v2/skills/docsmith
```

Claude Code will pick up live changes during the session.

## Update commands

| Install method | Update command |
|---|---|
| Marketplace | `/plugin marketplace update dangtran1003-docsmith-v2` |
| Direct clone | `cd ~/repos/docsmith-v2 && git pull` |
| Symlinked | `cd /tmp/docsmith-v2 && git pull` |
| Manual copy | Recopy from latest clone |

## Publishing to the official Anthropic marketplace

If you want the plugin discoverable via the built-in `/plugin Discover` tab:

1. **Claude.ai**: Submit at [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit)
2. **Console**: Submit at [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)

Both forms ask for the GitHub repository URL. They review the marketplace.json + skill content before listing.

## Versioning

- Version lives in **3 places** that must match:
  - `.claude-plugin/marketplace.json` → `version` and `plugins[0].version`
  - `.claude-plugin/plugin.json` → `version`
  - Git tag (e.g., `v1.5.2`)
- Changes documented in `CHANGELOG.md`
- Follow [Semantic Versioning](https://semver.org/):
  - **Patch** (1.5.x): bug fixes, doc clarifications, template tweaks
  - **Minor** (1.x.0): new commands, new templates, new capabilities
  - **Major** (x.0.0): breaking changes to commands or file structure

## Tagging and pushing a release

```bash
# After committing changes:
git tag v1.5.2 -m "Release v1.5.2 — <one-line summary>"
git push origin main
git push origin v1.5.2
```

Marketplaces use the latest tag by default. Users running `/plugin marketplace update` get the new version.

## Cross-platform compatibility

The skill content (`skills/docsmith/`) follows the open Agent Skills format, so it works in:

- **Claude Code** — native, full support including `/docsmith` slash commands
- **Claude Desktop** — reads SKILL.md, no slash commands
- **OpenClaw** — same skill folder works (under `~/.openclaw/skills/`)
- **Cursor** — supports Agent Skills via `.cursor/skills/`

The plugin marketplace format (`.claude-plugin/marketplace.json`) is **Claude Code specific**. Other tools install just the `skills/docsmith/` folder directly.

## Scopes

| Scope        | Location                           | Who gets it                  |
|--------------|------------------------------------|------------------------------|
| **Personal** | `~/.claude/skills/docsmith/`       | You, all projects on machine |
| **Project**  | `.claude/skills/docsmith/`         | Anyone working on this repo  |
| **Plugin**   | Via `/plugin install`              | Anyone with marketplace      |
| **Enterprise** | Managed settings / Cowork        | Org-wide distribution        |
