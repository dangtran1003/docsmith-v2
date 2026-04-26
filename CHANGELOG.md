# Changelog — dangtran1003/docsmith-v2

All notable changes to this skill are documented in this file.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/).

## [1.2.1] - 2026-04-26

### Added

- **`HOW_IT_WORKS.md`**: end-user walkthrough of the skill's operating model — 12 sections covering the two-layer model, command pipeline, init, config, authoring loop, deploy, categorize, locales, mental model, daily cheat-sheet, troubleshooting, and reading list. Read this before using docsmith on a real project.
- **`HOW_IT_WORKS.vi.md`**: Vietnamese translation of the same document.
- README "How it works" section linking to both language versions.

### Notes

Patch release — documentation only. No SKILL.md or command logic changes from v1.2.0.

## [1.2.0] - 2026-04-26

Standalone-first refactor + deploy to host project (Docusaurus preset).
This release combines what was originally planned as v1.2 (foundation) and v1.3 (deploy).

### Added

- **`init` command**: scaffold workspace with `.docsmithrc.yaml` config, `documentation/` folder tree, locale-aware draft directories. Detects host project context (`docusaurus.config.*`, `CLAUDE.md`) and suggests preset.
- **`deploy` command**: copy/sync workspace to host project with transforms. Supports `--dry-run`, `--target <path>`, `--force`, `--locale <locale>`. Detects target via `CLAUDE.md` then `docusaurus.config.*`. Generates audit trail in `deployments/<timestamp>-<target>/`.
- **`categorize` command**: generate Docusaurus `_category_.json` files from sitemap. Normalizes titles (acronyms uppercase, articles lowercase). Flags undocumented folders.
- **`.docsmithrc.yaml` config schema**: source of truth for product slug, locales, paths, deploy preset, collision strategy, validation rules. Every command reads it before writing. See [.docsmithrc.example.yaml](.docsmithrc.example.yaml).
- **Path scoping enforcement**: every write validated against allowed roots (workspace + deploy target). Reject writes outside.
- **Standalone preset** (`presets/standalone.yaml`): default. No deploy target. Workspace is the publishable artifact.
- **Docusaurus preset** (`presets/docusaurus.yaml`): full deploy mappings, frontmatter injection, image namespacing, MDX escaping, category generation.
- **Locale-aware draft structure**: `documentation/drafts/<locale>/` per locale. Source locale gets actual drafts; target locales get scaffolded folders for v1.6 auto-translation.
- **In-place mode**: `deploy.default_target = .` runs deploy within same project (workspace + Docusaurus folders coexist).
- **Image namespacing**: `product.slug` becomes URL prefix `/img/<slug>/`. Globally unique across products sharing a target.
- **Deploy reference doc** ([deploy-reference.md](deploy-reference.md)): detection, plan, action determination, dry-run output format, audit trail, in-place mode, categorize subcommand, title normalization rules, known limitations.
- **Category file template** ([templates/CATEGORY_FILE_TEMPLATE.md](templates/CATEGORY_FILE_TEMPLATE.md)): `_category_.json` schema, examples, acronym preservation.

### Changed

- **`draft` command**: now writes to `documentation/drafts/<locales.source>/<path>.md` (locale-aware). Image refs use workspace-absolute paths `/images/<feature>/<asset>.png`. Note: drafts in non-source locales are NOT auto-generated in 1.2.0 (deferred to v1.6 auto-translation).
- **`publish` command**: now positions itself after `deploy` (was the only deployment step before). Checklist updated to reflect deploy-driven workflow ending in `git commit/push` on target.
- **Process Flow diagram**: includes `init`, `categorize`, and `deploy`.
- **File Organization section**: split into Workspace + Target + In-place mode for clarity.
- **plugin.json description and keywords**: reflect Docusaurus, i18n, deploy capabilities.

### Deferred to future versions

- **`translate` command**: scaffolding the locale folder structure is in 1.2.0; AI auto-translation is on the **v1.6** roadmap. Until then, users with multi-locale needs must populate `drafts/<target-locale>/` manually.
- **`adopt` command**: convert existing Docusaurus docs into a docsmith workspace. Targeting **v1.5**.
- **Sidebar generation** (`sidebars.generated.js` from sitemap): config flag exists (`generate_sidebars`) but generator unimplemented in 1.2.0. Default false, so no impact. Targeting **v1.4**.
- **Hide undocumented folders**: report-only in 1.2.0. Auto-hide via sidebars on **v1.4** roadmap.

### Migration from 1.1.0

If you were using docsmith 1.1.0 with the old flat layout (`docs/drafts/`, `docs/images/`):

1. Run `/docsmith init` in your project to create `.docsmithrc.yaml` and the new `documentation/` workspace
2. Move existing files: `docs/drafts/*.md` → `documentation/drafts/<source-locale>/`, `docs/images/` → `documentation/images/`, `docs/walkthrough/` → `documentation/walkthrough/`
3. Update image refs in markdown: change relative `./images/foo.png` to workspace-absolute `/images/foo.png`
4. Run `/docsmith verify <product>` to confirm no broken references
5. (Docusaurus users) Run `/docsmith deploy <product> --dry-run` to preview the new flow

The 1.1.0 layout is no longer recommended but will still work with the existing commands; only `init`, `deploy`, and `categorize` require the new layout.

## [1.1.0] - 2026-04-26

### Added

- New `record` command (alias `rec`) for capturing short tutorial videos from `<!-- VIDEO ... -->` markers via browser screen recording
- `templates/SCREENSHOT_POLICY_TEMPLATE.md` — caption rules (state-not-action, specific data, label-not-appearance, placement-after-step), density rules, file naming conventions
- `templates/VIDEO_MARKER_TEMPLATE.md` — marker syntax with structured fields (id, duration, type, start, actions, end, highlight, pacing, caption), good/bad examples, re-record metadata convention
- `templates/WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md` — capture plan format mirroring screenshot capture plan, blocked-marker handling, execution notes
- `videos/` and `videos/raw/` folders in the documented file organization
- `walkthrough/video-plan/` folder for video capture plans
- `standards/screenshot-policy.md` listed in standards directory

### Changed

- `draft` command now references `SCREENSHOT_POLICY_TEMPLATE.md` and enforces explicit caption rules and density rules inline
- `walkthrough` command now documents caption-driven matching mechanism and pre-execution caption review; explicitly skips `<!-- VIDEO ... -->` markers (handled by `record`)
- Process Flow diagram includes `[record (AI, optional)]` between `walkthrough` and `peer-review`

### Notes

- `process-reference.md`, `subprocess-010a.md`, `tools-reference.md` unchanged in this release; the `record` command is documented self-contained in SKILL.md

## [1.0.0] - 2026-03-11

### Added
- Initial skill structure based on PRC-010 Documentation Creation Process
- Command-based invocation with aliases: `start`, `audience`, `plan`, `review-plan`, `sitemap`, `voice`, `draft`, `edit`, `walkthrough`, `validate`, `test`, `verify`, `peer-review`, `tech-review`, `incorporate`, `publish`
- `help` command showing full command reference
- `verify` command with 10 verification checks (technical accuracy, completeness, structure, clarity, voice compliance, link audit, placeholder audit, traceability, sitemap consistency, template compliance) — supports project-wide or scoped to specific docs via path/glob
- `validate` command for re-running walkthrough test cases without modifying docs
- `test` command for generating test cases from existing docs without executing them
- 9 templates: audience profile, documentation plan, traceability matrix, content type templates, voice chart, UX text patterns, UX content scorecard, walkthrough test cases, walkthrough test execution
- Process reference (PRC-010), subprocess reference (PRC-010A), tools reference (browser automation)
- File organization convention under `docs/`
- Versioning with YAML frontmatter and this changelog

### Sources
- *Docs for Developers* (Bhatti et al., 2021)
- *Strategic Writing for UX* (Podmajersky, 2019)
