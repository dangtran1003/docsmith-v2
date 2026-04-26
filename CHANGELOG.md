# Changelog — dangtran1003/docsmith-v2

All notable changes to this skill are documented in this file.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/).

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
