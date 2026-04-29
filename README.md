# docsmith-v2

A Claude Code skill for systematic documentation: plan → draft → walkthrough → record → deploy. Standalone-first workspace with optional deploy to Docusaurus or other host projects.

**Sources**: *Docs for Developers* (Bhatti et al., 2021), *Strategic Writing for UX* (Podmajersky, 2019).

## What it does

Guides you through 19 commands covering the full documentation lifecycle:

- **Init**: `init` scaffolds a self-contained workspace with `.docsmithrc.yaml`
- **Plan**: audience profile → documentation plan → sitemap → voice/UX standards
- **Author**: draft (locale-aware) with content type templates → 5-pass self-review
- **Verify**: live-product walkthrough with caption-matched screenshots → test cases → 10-check verification
- **Record** *(1.1)*: short tutorial videos from inline `<!-- VIDEO ... -->` markers via browser screen recording
- **Deploy** *(1.2)*: copy/sync workspace to host project with frontmatter injection, image namespacing, MDX escaping. Supports Docusaurus preset out of the box. `--dry-run` before any write.
- **Publish**: deploy → review diff → `git commit/push` on target

## Install

### As a Claude Code plugin (recommended)

```bash
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2
```

### As a personal skill (without marketplace)

If you prefer cloning directly:

```bash
# Clone the whole repo, then symlink/copy the skill folder into your skills dir
git clone https://github.com/dangtran1003/docsmith-v2.git ~/repos/docsmith-v2
ln -s ~/repos/docsmith-v2/skills/docsmith ~/.claude/skills/docsmith
```

Or copy just the skill folder:

```bash
git clone https://github.com/dangtran1003/docsmith-v2.git /tmp/docsmith-v2
cp -r /tmp/docsmith-v2/skills/docsmith ~/.claude/skills/docsmith
```

### As a project skill

```bash
# In your project root
mkdir -p .claude/skills
cp -r /path/to/docsmith-v2/skills/docsmith .claude/skills/
```

See [PUBLISHING.md](PUBLISHING.md) for all installation methods and marketplace publishing.

## Update

```bash
# If installed via marketplace
/plugin marketplace update dangtran1003-docsmith-v2

# If cloned directly
cd ~/repos/docsmith-v2 && git pull
```

## How it works

For a complete walkthrough of the skill's operating model — every command, every file, the workspace/target/deploy lifecycle, troubleshooting — read **[HOW_IT_WORKS.md](HOW_IT_WORKS.md)** (or **[HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md)** in Vietnamese).

**For a hands-on tour from basic to advanced** (first project, multi-locale, video voiceover, drift detection, deploy patterns, troubleshooting): read **[USAGE_GUIDE.md](USAGE_GUIDE.md)** (or **[USAGE_GUIDE.vi.md](USAGE_GUIDE.vi.md)**).

**Before your first `walkthrough` or `record` run**: read **[SETUP.md](SETUP.md)** (or **[SETUP.vi.md](SETUP.vi.md)**) to install the browser extension, set env vars, and configure credentials.

**For BAs and content owners** filling out intake forms (no technical background needed): read **[INTAKE_GUIDE.md](INTAKE_GUIDE.md)** (or **[INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md)**).

## Quick start

### 1. One-time setup (per machine)

Read **[SETUP.md](SETUP.md)** to install browser tools and set credentials env vars. Skip if you only need drafting/translating without product verification.

### 2. Initialize project

```bash
mkdir my-product-docs && cd my-product-docs
/docsmith init                              # scaffolds documentation/intake/project.md
```

### 3. Add modules (per feature area)

```bash
/docsmith module instances                  # creates documentation/intake/modules/instances.md
/docsmith module storage
```

### 4. Fill the intake forms

Edit `documentation/intake/project.md` (project-wide settings: product, audience, locales, deploy, voice, sources) and each `documentation/intake/modules/<name>.md` (per-module scope). See [INTAKE_GUIDE.md](INTAKE_GUIDE.md) for what each field means.

### 5. Run the pipeline

```bash
/docsmith run                               # auto-chains: audience → plan → voice → draft → edit
                                            # → walkthrough → [record] → translate
                                            # pauses at after-draft (default) for your review
# Review drafts in documentation/drafts/<locale>/<module>/
/docsmith continue                          # resume to deploy plan
```

### 6. Deploy

```bash
/docsmith deploy --dry-run                  # preview
/docsmith deploy                            # apply
cd ../my-docusaurus-site && git add . && git commit && git push
```

### Update flow (when sources change)

```bash
/docsmith update                            # detects Notion/GitHub/GDrive changes
                                            # proposes targeted re-drafts
```

### In-place inside an existing Docusaurus project

```bash
cd ~/my-docusaurus-site
/docsmith init --in-place                   # detects docusaurus.config and uses target_path = .
/docsmith module pricing
# fill intakes
/docsmith run pricing
/docsmith deploy
```

## What's new in 1.5.7

- **Per-video script files** — voiceover content lives in dedicated files at `documentation/scripts/<module>/<id>.md`. Source language + per-locale translations under `## en` / `## vi` / `## jp` headings in same file.
- **Simplified VIDEO marker** — `<!-- VIDEO id: <id> -->` only. All other config in script file frontmatter.
- **`translate` extended** — now processes script files alongside drafts. Same review gate, same glossary.
- **`record --migrate-scripts`** — extract inline scripts from v1.5.6 drafts, create new script files.
- New template: `VIDEO_SCRIPT_TEMPLATE.md`.

## What's new in 1.5.6

- **Collapsed Advanced sections** in intake forms. BA mới thấy ~80 dòng essentials thay vì 356 dòng (project) hay 245 dòng (module). Click để expand khi cần customize.
- **Same fields, same parsing, no new commands** — pure UX patch. AI parser ignores `<details>` wrappers.
- **INTAKE_GUIDE Rule 4** — explicit minimum-fill checklist for first project: chỉ điền 6 essential sections, skip mọi Advanced.

## What's new in 1.5.5

- **Comprehensive media policy** — screenshot density per content type, style options, aspect ratio, per-locale strategy. Video density, length caps, voiceover strategy (Silent / AI per locale / source+sub / human), 6 TTS providers (local-piper default, ElevenLabs, OpenAI, Google, Azure, Coqui), subtitle generation.
- **Per-module media override** — modules can override project-wide media policy.
- **TTS provider abstraction** — pick via project intake; install instructions in SETUP.md.
- **`verify` check #11** — media compliance: density rules, length caps, voiceover/subtitle file existence.
- **Cost transparency** — explicit cost estimates so users can budget (default config = ~$0; premium = ~$5-10 per cycle).
- New template: `MEDIA_POLICY_TEMPLATE.md`.

## What's new in 1.5.4

- **Sitemap consistency**: 3 project-level patterns (Learning path / Task-first / Custom) + 11 canonical section types. Fixes navigation drift across modules — was a real problem where module A had "Quick Starts → Tutorials" while module B had "Guides → Reference → Glossary".
- **Per-module section checklist**: each module ticks which canonical sections to include. AI suggests missing sections.
- **Display name overrides** at project and module level — slugs stay canonical for folder paths.
- **`verify` check #10** updated: now "sitemap consistency" (replaces less-actionable "no orphan test cases").
- New template: `SITEMAP_PATTERNS_TEMPLATE.md`.

## What's new in 1.5.3

- **SETUP.md guide** (en + vi): comprehensive prerequisites for walkthrough (Claude in Chrome OR Playwright), record (ffmpeg), and source fetching (auth tokens). Step-by-step token acquisition for Notion, GitHub, Google Drive. Pre-walkthrough/pre-record checklists. Common errors and fixes.
- **Quick start refreshed** to v1.5+ flow (intake-driven via `run`/`continue`).
- Prerequisites notes added to `walkthrough`, `record`, `fetch` command sections in SKILL.md.

## What's new in 1.5.2

- **Plugin marketplace compliance**: restructured to follow official Claude Code plugin format. `.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json` at root, skill content in `skills/docsmith/`. `/plugin marketplace add dangtran1003/docsmith-v2` now actually works.
- **Updated install docs**: correct `/plugin install docsmith@dangtran1003-docsmith-v2` syntax. Update commands for all install methods.

## What's new in 1.5.1

- **Path conflict fixes**: 6 issues found in audit and fixed. Footprint at project root is now exactly 1 folder (`documentation/`). Per-module run state. In-place mode collision detection.
- **INTAKE_GUIDE for BAs**: comprehensive practical guide (en + vi) explaining how to fill intake forms — what each field means, how AI uses it, common patterns.
- Smart `.gitignore` append (BEGIN/END markers, idempotent).

## What's new in 1.5.0

- **Markdown intake forms**: BAs configure docs by ticking checkboxes and filling `backtick fields`. No YAML to write. Layered: project + module.
- **External knowledge fetching**: AI pulls context from Notion, GitHub, Google Drive, public URLs, and local files. Auth via env var references.
- **`run` orchestration**: one command chains audience → plan → voice → draft → edit → wt → record → translate. Pauses at configurable gate (default after-draft).
- **`update` change detection**: cheap metadata calls detect when external sources change. Propose targeted re-drafts.
- **Lean refactor**: removed obsolete commands (`start`, `validate`, `test`, `peer-review`, `tech-review`, `review-plan`, `incorporate`, `sitemap`). 5 templates removed. 2 reference docs inlined.
- **Module management**: `module <n>`, `module list`, `module archive` for per-feature config files.

## What's new in 1.4.0

- **`translate` command**: AI translation from source locale to each `locales.targets`. Per-block review gate by default; user approves / edits / skips per block. `--auto-approve` flag for speed.
- **Glossary support**: optional `documentation/standards/glossary.<locale>.yaml` enforces consistent terminology. Build iteratively from review corrections.
- **Translation metadata**: every translated file tracks `translated_from`, `source_hash`, `glossary_version` for forward-compatible drift tracking (v1.6+).
- **Translation completeness check in `deploy`**: warns when target locales have missing translations.
- **Re-run safety extends to translate**: Update mode preserves manually-edited translations when source unchanged.
- 2 new templates: `GLOSSARY_TEMPLATE.yaml`, `TRANSLATION_DECISIONS_TEMPLATE.md`. New: `translate-reference.md`.

## What's new in 1.3.0

- **Re-run safety**: every command checks output existence first. 4-option gate (Update / Overwrite / Side-by-side / Cancel) when found. Default `prompt`; configurable.
- **KB inheritance**: in Update mode, AI reads existing artifact as knowledge base. Proposes deltas (NEW / UPDATE / REMOVE / KEEP). Untouched content NEVER regenerated — preserves team's manual edits.
- **Walkthrough drift detection**: refactored into 3 phases — VERIFY (read-only) → user gate via `decisions.yaml` → APPLY → CAPTURE. New flags: `--check`, `--apply`, `--skip-drift`, `--auto-apply-high-confidence`.
- **Product-bug tracking**: drift items can be marked as product regressions. Tracked across runs in `active-product-bugs.yaml`. Auto-resolved when UI matches doc again.
- **Delete propagation**: `deploy` always lists orphan target files. New `--sync-deletes` flag opts into actual cleanup with backup.
- 2 new templates: `DRIFT_REPORT_TEMPLATE.md`, `MERGE_DECISION_TEMPLATE.md`. New: `update-reference.md`.

## What's new in 1.2.0

- **Standalone-first**: workspace lives independently of host project. Run docsmith anywhere, deploy when ready.
- **`init` command**: one-shot setup with `.docsmithrc.yaml`, locale-aware folder structure, preset auto-detection.
- **`deploy` command**: sync workspace to Docusaurus (or other) host project. Frontmatter injection, image namespacing (`/img/<slug>/...`), MDX escaping, audit trail. Always supports `--dry-run`.
- **`categorize` command**: generate Docusaurus `_category_.json` from sitemap with normalized titles.
- **Locale folders**: `drafts/<source>/` + `drafts/<target>/` per language. Source gets drafts; targets get scaffolded for v1.6 auto-translation.
- **Path scoping**: writes validated against allowed roots. No accidental writes outside the configured workspace + deploy target.
- **Audit trail**: every deploy creates `deployments/<timestamp>-<target>/` with manifest, target config snapshot, diff, pre-deploy hashes.

See [CHANGELOG.md](CHANGELOG.md) for full history including v1.1.0 (caption rules + tutorial videos).

## File structure

This repo is a Claude Code plugin marketplace containing one plugin (`docsmith`):

```
docsmith-v2/                           # marketplace repo root
├── .claude-plugin/
│   ├── marketplace.json               # marketplace catalog
│   └── plugin.json                    # plugin manifest
├── skills/
│   └── docsmith/                      # the actual skill
│       ├── SKILL.md                   # main skill spec
│       ├── .docsmithrc.example.yaml   # legacy config schema (deprecated)
│       ├── deploy-reference.md
│       ├── intake-reference.md
│       ├── translate-reference.md
│       ├── process-reference.md
│       ├── tools-reference.md
│       ├── presets/
│       │   ├── standalone.yaml
│       │   └── docusaurus.yaml
│       └── templates/                 # 14 templates
├── README.md                          # this file
├── CHANGELOG.md
├── PUBLISHING.md
├── HOW_IT_WORKS.md / .vi.md           # user guides at repo root
├── USAGE_GUIDE.md / .vi.md            # workflow guide (basic → advanced)
├── INTAKE_GUIDE.md / .vi.md           # BA-friendly intake guide
├── SETUP.md / .vi.md                  # one-time prerequisites
└── COMPARISON.md / .vi.md             # v1.1.0 vs v1.5.1 self-review
```

When installed via `/plugin marketplace add`, Claude Code picks up `skills/docsmith/SKILL.md` automatically.

## Roadmap

Future releases (no fixed version yet — driven by real-world usage feedback):

- Visual regression in walkthrough — pixel-diff between captured and previous screenshots
- Translation drift tracking — `translate --check` mode, `<!-- translation-locked -->` markers
- Per-locale image namespacing for products with localized UI
- `adopt` command — convert existing Docusaurus docs into docsmith workspace
- `health` command — one-shot wrapper for verify + drift + source change
- Per-doc source provenance — surgical re-evaluation in `update` instead of module-wide
- Additional presets — mkdocs, mintlify
- YAML config removal — `.docsmithrc.yaml` deprecated in v1.5; will be removed in v1.6

Priorities will be set by what hurts most when using v1.5 on real projects.

## License

MIT — see `.claude-plugin/plugin.json`.

