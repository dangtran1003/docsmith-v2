# docsmith-v2

A Claude Code skill for systematic documentation creation, verification, and short tutorial video recording. Based on the PRC-010 documentation creation process.

**Sources**: *Docs for Developers* (Bhatti et al., 2021), *Strategic Writing for UX* (Podmajersky, 2019).

## What it does

Guides you through 13 commands covering the full documentation lifecycle:

- **Plan**: audience profile → documentation plan → sitemap → voice/UX standards
- **Author**: draft with content type templates → 5-pass self-review
- **Verify**: live-product walkthrough with caption-matched screenshots → test cases → comprehensive 10-check verification
- **Record** *(new in 1.1.0)*: capture short tutorial videos from inline `<!-- VIDEO ... -->` markers via browser screen recording
- **Publish**: peer/tech review → incorporate feedback → publish

## Install

### As a Claude Code plugin (recommended)

```bash
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install dangtran1003/docsmith-v2
```

### As a personal skill

```bash
git clone https://github.com/dangtran1003/docsmith-v2.git ~/.claude/skills/docsmith
```

### As a project skill

```bash
git clone https://github.com/dangtran1003/docsmith-v2.git .claude/skills/docsmith
```

See [PUBLISHING.md](PUBLISHING.md) for all installation methods and marketplace publishing.

## Quick start

```bash
/docsmith help              # Show command reference
/docsmith start MyProduct   # Begin from audience definition
/docsmith plan MyProduct    # AI generates documentation plan
/docsmith draft MyProduct   # AI drafts docs with screenshot placeholders + video markers
/docsmith wt MyProduct      # Walkthrough: capture screenshots, run test cases
/docsmith rec MyProduct     # Record videos from <!-- VIDEO --> markers (optional)
/docsmith verify MyProduct  # Run all 10 verification checks
```

## What's new in 1.1.0

- **`record` command**: tutorial video capture from inline markers, with re-record metadata for product UI changes
- **Caption rules**: explicit rules for state-not-action descriptions, specific data, label-not-appearance — to make screenshot capture deterministic
- **Screenshot density**: every procedure must capture starting state, each visible UI change, and success state
- 3 new templates: `SCREENSHOT_POLICY_TEMPLATE.md`, `VIDEO_MARKER_TEMPLATE.md`, `WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md`

See [CHANGELOG.md](CHANGELOG.md) for the full history.

## File structure

```
.
├── SKILL.md                    # Main skill spec — entry point for Claude
├── plugin.json                 # Plugin manifest
├── process-reference.md        # PRC-010 process detail
├── subprocess-010a.md          # PRC-010A UX content subprocess
├── tools-reference.md          # Browser automation reference
├── templates/                  # 12 templates (audience profile, plan, voice chart, etc.)
├── CHANGELOG.md
├── PUBLISHING.md
└── README.md
```

## License

MIT — see plugin.json.
