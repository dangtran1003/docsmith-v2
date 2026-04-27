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

## How it works

For a complete walkthrough of the skill's operating model — every command, every file, the workspace/target/deploy lifecycle, troubleshooting — read **[HOW_IT_WORKS.md](HOW_IT_WORKS.md)** (or **[HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md)** in Vietnamese).

**For BAs and content owners** filling out intake forms (no technical background needed): read **[INTAKE_GUIDE.md](INTAKE_GUIDE.md)** (or **[INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md)**).

## Quick start

### Standalone (drafting first, deploy later)

```bash
mkdir my-product-docs && cd my-product-docs
/docsmith init                              # interactive: slug, locales, preset
/docsmith audience MyProduct                # define audience
/docsmith plan MyProduct                    # AI generates plan
# ... iterate plan / sitemap / voice
/docsmith draft MyProduct                   # writes to documentation/drafts/en/
/docsmith wt MyProduct                      # capture screenshots
/docsmith rec MyProduct                     # (optional) record videos
/docsmith verify MyProduct
```

### Deploy to Docusaurus

```bash
# In .docsmithrc.yaml: set deploy.preset = docusaurus and deploy.default_target
/docsmith deploy MyProduct --dry-run        # preview: detect target, plan, show diff
/docsmith deploy MyProduct                  # apply
cd ../my-docusaurus-site && git diff && git commit
```

### In-place inside an existing Docusaurus project

```bash
cd my-docusaurus-site
/docsmith init --in-place                   # auto-detects, sets default_target = .
# ... draft / wt / rec / deploy as above
```

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

```
.
├── SKILL.md                         # Main skill spec — entry point for Claude
├── plugin.json                      # Plugin manifest
├── .docsmithrc.example.yaml         # Config schema with comments
├── presets/
│   ├── standalone.yaml              # Default preset
│   └── docusaurus.yaml              # Docusaurus deploy mappings
├── deploy-reference.md              # Deploy command detailed logic
├── process-reference.md             # PRC-010 process detail
├── subprocess-010a.md               # PRC-010A UX content subprocess
├── tools-reference.md               # Browser automation reference
├── templates/                       # 13 templates
└── CHANGELOG.md / PUBLISHING.md / README.md
```

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

MIT — see plugin.json.

