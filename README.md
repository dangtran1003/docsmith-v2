# docsmith

A Claude Code skill that builds technical documentation from filled markdown intake forms. Drafts content, captures screenshots via browser walkthrough, generates voiceover videos, translates to multiple locales, and deploys to Docusaurus.

**Current version**: 1.5.11 ([CHANGELOG](CHANGELOG.md))

> 🇻🇳 Tiếng Việt: [README.vi.md](README.vi.md) (coming soon — for now, [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md) and [SETUP.vi.md](SETUP.vi.md) cover the basics)

## What it does

You fill out two markdown forms (project + per-feature module). docsmith does the rest:

```
You fill intake forms          docsmith does
├── project.md (1 file)        ├── Drafts content from BA docs (Notion/GDoc/MD)
└── modules/<name>.md (1+)     ├── Captures screenshots from live product
                               ├── Records short tutorial videos (silent or voiced)
                               ├── Translates to target locales (with glossary)
                               └── Deploys to Docusaurus repo
```

No YAML to write. Checkboxes and backtick fields only. AI parses them deterministically.

## Quick start

### 1. One-time setup

Read **[SETUP.md](SETUP.md)** to install browser tools (Claude in Chrome OR Playwright) and set credentials env vars. Skip if you only need drafting/translating without product screenshots.

### 2. Install the skill

```bash
# In Claude Code:
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2
```

Or clone directly into your skills folder — see [PUBLISHING.md](PUBLISHING.md) for alternatives.

### 3. Initialize a project

```bash
mkdir my-product-docs && cd my-product-docs
/docsmith init                              # scaffolds documentation/intake/project.md
```

`init` creates the workspace folder structure and a project intake form pre-filled with detected values (slug from package.json, etc.).

**OR auto-fill from BA doc / PRD (v1.5.9+)**:

```bash
/docsmith init --from-source documentation/sources/ba-doc.md
# AI reads BA doc, infers fields, asks 5-10 questions, writes project.md
# Marks uncertain fields with "← AI guess" for you to verify
```

Source can be local file, Notion URL, GitHub path, or Google Drive ID. See [INTAKE_GUIDE.md](INTAKE_GUIDE.md) § "Source-driven auto-fill" for details.

### 4. Add modules (per feature area)

```bash
/docsmith module instances                  # creates documentation/intake/modules/instances.md
/docsmith module storage
```

Each module is one feature area. Create as many as you need.

**Auto-fill from source** (v1.5.9+):

```bash
/docsmith module instances --from-source documentation/sources/ba-doc.md
# AI extracts features for "instances" from BA doc, fills module intake
```

### 5. Fill the intake forms

Open the two files and fill them out. Each field has an inline hint explaining what it's for.

- `documentation/intake/project.md` — project-wide settings (product, audience, languages, deploy, voice, sources)
- `documentation/intake/modules/<name>.md` — per-module scope (features, content types)

**For your first project, fill only sections 1-6 in project.md and sections 1-2 in module.md.** Leave Advanced sections collapsed — defaults work for ~80% of cases.

For more guidance: **[INTAKE_GUIDE.md](INTAKE_GUIDE.md)** (or **[INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md)** in Vietnamese).

### 6. Run the pipeline

```bash
/docsmith run                               # auto-chains: audience → plan → voice → draft → edit
                                            # → walkthrough → [record] → translate
                                            # pauses at after-draft (default)
```

AI generates audience profile, documentation plan, voice chart, drafts, then pauses for your review.

Review drafts in `documentation/drafts/<source-locale>/<module>/`. Edit by hand if needed (re-run preserves edits).

```bash
/docsmith continue                          # resumes pipeline: walkthrough → record → translate
```

### 7. Deploy

```bash
/docsmith deploy --dry-run                  # preview file changes
/docsmith deploy                            # apply to Docusaurus repo

cd ../my-docusaurus-site
git diff && git add . && git commit -m "Update docs" && git push
```

`deploy` only writes files. The git commit + push is your manual safety net.

### Update flow when sources change

```bash
/docsmith update                            # detects Notion/GitHub/GDrive changes
                                            # AND new/orphan modules in source structure (v1.5.10+)
                                            # proposes targeted re-drafts of affected docs
```

Cheap metadata calls (no full re-fetch) detect what changed. Three-layer change report (v1.5.10+):
- **Content drift**: source content changed → affected drafts to re-evaluate
- **Module diff**: modules in source vs in workspace (new / orphan / scope drift)
- **Scope drift**: existing module's features lag source

You review proposed deltas before applying. For projects with many new modules (>5), AI MAY use parallel processing.

## All commands at a glance

| Command       | What it does                                                          |
| ------------- | --------------------------------------------------------------------- |
| `init`        | Scaffold workspace + project intake form                              |
| `module`      | Create / list / archive a module intake                               |
| `intake-help` | Print field reference for intake forms                                |
| `fetch`       | Pull external sources (Notion/GitHub/GDrive/URL/file) into local cache |
| `run`         | Orchestrated pipeline. Pauses at the configured gate.                 |
| `continue`    | Resume `run` after a gate                                             |
| `audience`    | Generate audience profile from intake                                 |
| `plan`        | Generate documentation plan + sitemap                                 |
| `voice`       | Generate voice chart                                                  |
| `draft`       | Draft docs in `documentation/drafts/<source-locale>/`                 |
| `edit`        | 5-pass self-review                                                    |
| `walkthrough` | Verify against live product + capture screenshots                     |
| `record`      | Record short tutorial videos (silent or voiced)                       |
| `translate`   | Translate to target locales with glossary                             |
| `verify`      | 11-check audit (placeholders, voice score, links, etc.)               |
| `update`      | Detect external source changes, propose draft updates                 |
| `categorize`  | Generate `_category_.json` for Docusaurus                             |
| `deploy`      | Sync workspace to Docusaurus repo                                     |
| `publish`     | Final checklist (human step — git commit + push)                      |

Most users only need: `init`, `module`, `run`, `deploy`. The rest are individual stages of the pipeline.

## What gets generated

After a successful run on a project with `instances` module:

```
documentation/
├── intake/
│   ├── project.md                           # you filled this
│   ├── modules/instances.md                 # you filled this
│   └── sources.lock.yaml                    # auto-managed
├── plan/
│   ├── audience-profile.md                  # AI-generated
│   ├── documentation-plan.md
│   └── sitemap.md
├── standards/
│   ├── voice-chart.md
│   ├── glossary.vi.yaml                     # if Vietnamese target locale set
│   └── glossary.jp.yaml
├── drafts/
│   ├── en/instances/                        # source-locale drafts
│   │   ├── overview.md
│   │   ├── create-instance-tutorial.md
│   │   ├── create-instance.md (how-to)
│   │   └── ...
│   └── vi/instances/                        # translated by `translate`
│       └── (mirrors en/ structure)
├── walkthrough/
│   ├── test-cases/instances/
│   ├── drift/<timestamp>/                   # drift reports per run
│   └── active-product-bugs.yaml
├── images/
│   └── instances/                           # screenshots from walkthrough
│       ├── create-form-empty.png
│       └── create-form-filled.png
├── scripts/                                 # v1.5.7+
│   └── instances/
│       └── instance-create-tour.md          # video script (multi-locale)
├── videos/
│   ├── raw/                                 # gitignored
│   ├── instance-create-tour.mp4             # final video
│   ├── voiceover/                           # if voiceover strategy != silent
│   │   └── instance-create-tour.en.mp3
│   └── subtitles/
│       └── instance-create-tour.en.vtt
├── deployments/<timestamp>-<target>/        # audit trail per deploy
│   ├── manifest.yaml
│   └── diff.md
├── archive/<timestamp>/                     # backups from re-run protocol
└── .cache/sources/                          # gitignored, fetched content
```

## Key concepts

### Layered config (project + module)

- **Project intake** (`documentation/intake/project.md`): applies to ALL modules
- **Module intake** (`documentation/intake/modules/<name>.md`): per-feature-area; OVERRIDES project where set
- Sources are CUMULATIVE (project sources + module sources both used)

### Re-run safety

Every command checks if output exists before writing. If it does, you get:
- **Update** mode (recommended): existing content read as KB, AI proposes deltas only. Your manual edits preserved.
- **Overwrite**: discard existing, regenerate. Backup to `archive/<timestamp>/`.
- **Side-by-side**: keep both, new file gets `-v2` suffix.
- **Cancel**: abort.

### Drift detection (walkthrough)

`walkthrough --check` compares your drafts against live product UI. Reports drift (UI changed, label different, button missing). You decide per item: auto-fix the doc, mark as product bug, or skip.

### Source change detection (update)

`update` checks if external sources (Notion/GitHub/GDoc) have new content. Cheap metadata calls (Notion edit time, GitHub commit SHA, file mtime) — no full re-fetch unless something changed.

### Multi-locale

Set source + target locales in project intake. After drafts are ready in source language, `translate` produces translated drafts in `documentation/drafts/<target>/<module>/`. Glossary at `documentation/standards/glossary.<locale>.yaml` enforces consistent terminology.

### Sitemap consistency (v1.5.4+)

Pick one of 3 patterns at project level (Learning path / Task-first / Custom). All modules follow it. AI warns if a module is missing a section the pattern includes.

### Media policy (v1.5.5+)

Screenshot density rules per content type. Video voiceover strategy (silent default; AI per-locale, source+sub, human as options). 6 TTS providers (local-piper default, free + offline). Subtitle generation. All configurable via project intake § 11; per-module overrides via module intake § 8.

### Video scripts (v1.5.7+)

Each video has a script file at `documentation/scripts/<module>/<id>.md`. Source language + per-locale translations under `## en` / `## vi` / `## jp` headings in same file. AI generates initial script from your draft prose; you review and edit. `translate` processes scripts alongside drafts.

## Documentation map

| You want to | Read |
|---|---|
| Install browser tools, set up env vars, install TTS provider | [SETUP.md](SETUP.md) / [SETUP.vi.md](SETUP.vi.md) |
| Walk through real scenarios from basic to advanced (multi-locale, video, drift, deploy) | [USAGE_GUIDE.md](USAGE_GUIDE.md) / [USAGE_GUIDE.vi.md](USAGE_GUIDE.vi.md) |
| Understand each intake field, what AI does with it | [INTAKE_GUIDE.md](INTAKE_GUIDE.md) / [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md) |
| See full operating model, lifecycle, troubleshooting | [HOW_IT_WORKS.md](HOW_IT_WORKS.md) / [HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md) |
| Compare with v1.1 (was the design too aggressive?) | [COMPARISON.md](COMPARISON.md) / [COMPARISON.vi.md](COMPARISON.vi.md) |
| Install or update the skill | [PUBLISHING.md](PUBLISHING.md) |
| See what changed in each version | [CHANGELOG.md](CHANGELOG.md) |

For deep technical reference (parsers, file formats, edge cases), see the reference docs inside `skills/docsmith/`:
- `intake-reference.md` — intake parsing, layered config, fetch/update workflow
- `process-reference.md` — PRC-010 process detail
- `deploy-reference.md` — deploy/categorize logic
- `translate-reference.md` — translation block-level rules
- `tools-reference.md` — browser automation reference

## File structure

This repo is a Claude Code plugin marketplace containing one plugin (`docsmith`):

```
docsmith-v2/
├── .claude-plugin/
│   ├── marketplace.json               # marketplace catalog
│   └── plugin.json                    # plugin manifest
├── skills/
│   └── docsmith/                      # the actual skill
│       ├── SKILL.md                   # main skill spec
│       ├── deploy-reference.md
│       ├── intake-reference.md
│       ├── translate-reference.md
│       ├── process-reference.md
│       ├── tools-reference.md
│       ├── presets/
│       └── templates/                 # 14 templates
└── (top-level docs: README, CHANGELOG, PUBLISHING, INTAKE_GUIDE, SETUP, HOW_IT_WORKS, COMPARISON)
```

When installed via `/plugin marketplace add`, Claude Code picks up `skills/docsmith/SKILL.md` automatically.

## Roadmap

Future releases (driven by real-world feedback — no dates):

- Visual regression in walkthrough (pixel-diff)
- Translation drift tracking (`translate --check` mode, `<!-- translation-locked -->` markers)
- Per-locale image namespacing for products with localized UI
- `adopt` command — convert existing Docusaurus docs into docsmith workspace
- `health` command — one-shot wrapper for verify + drift + source change
- Per-doc source provenance — surgical re-evaluation in `update`
- Additional presets — mkdocs, mintlify
- YAML config removal — `.docsmithrc.yaml` deprecated since 1.5; will be removed in v1.6

Priorities will be set by what hurts most when using v1.5.7 on real projects.

## License

MIT — see `.claude-plugin/plugin.json`.
