# Publishing & Installation — dangtran1003/docsmith-v2

## Installation Methods

### 1. Claude Code Plugin (recommended)

If published to a marketplace repo:

```bash
# Add marketplace (one-time)
/plugin marketplace add dangtran1003/docsmith-v2

# Install
/plugin install dangtran1003/docsmith-v2

# Update
/plugin marketplace update
```

### 2. Direct Git Install

```bash
# Clone into personal skills
git clone https://github.com/dangtran1003/docsmith-v2.git ~/.claude/skills/docsmith

# Or project-level
git clone https://github.com/dangtran1003/docsmith-v2.git .claude/skills/docsmith
```

### 3. Add as External Directory

```bash
# Point Claude Code at the skill directory (live reload)
claude --add-dir /path/to/docsmith
```

### 4. Manual Copy

```bash
# Personal (all projects)
cp -r docsmith/ ~/.claude/skills/docsmith/

# Project-level (this repo only)
cp -r docsmith/ .claude/skills/docsmith/
```

## Publishing to a Marketplace

### GitHub Repository Setup

1. Create repo `dangtran1003/docsmith-v2`
2. Push the skill directory as the repo root:

```
docsmith/                  ← repo root
├── SKILL.md               # Main skill (required)
├── plugin.json            # Plugin manifest
├── CHANGELOG.md           # Version history
├── PUBLISHING.md          # This file
├── process-reference.md   # PRC-010 process
├── subprocess-010a.md     # PRC-010A subprocess
├── tools-reference.md     # Browser automation tools
└── templates/             # All output templates
```

3. Tag releases with semantic versions:

```bash
git tag v1.0.0
git push origin v1.0.0
```

### Register as a Marketplace

Others can add your repo as a marketplace source:

```bash
/plugin marketplace add dangtran1003/docsmith-v2
```

### Cross-Platform Compatibility (agentskills.io)

The SKILL.md format is compatible with:
- **Claude Code** — native support via `/docsmith`
- **OpenAI Codex CLI** — reads SKILL.md
- **GitHub Copilot (VS Code)** — supports Agent Skills
- **Cursor** — supports Agent Skills

Note: Slash commands, argument parsing, and template references are Claude Code-specific. Other tools will use the SKILL.md as context/instructions but won't support `/docsmith start` invocation.

## Scopes

| Scope | Location | Who Gets It |
|-------|----------|-------------|
| **Personal** | `~/.claude/skills/docsmith/` | You, all projects |
| **Project** | `.claude/skills/docsmith/` | Anyone on this project |
| **Enterprise** | Managed settings / Cowork | Org-wide distribution |

## Versioning

- Version lives in `SKILL.md` frontmatter (`version: x.y.z`) and `plugin.json`
- Changes documented in `CHANGELOG.md`
- Git tags match versions: `v1.0.0`, `v1.1.0`, etc.
- Follow [Semantic Versioning](https://semver.org/):
  - **Patch** (1.0.x): Template fixes, typo corrections, clarifications
  - **Minor** (1.x.0): New commands, new checks, new templates
  - **Major** (x.0.0): Breaking changes to command names, file organization, or process flow
