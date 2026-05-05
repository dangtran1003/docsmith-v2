# Changelog — dangtran1003/docsmith-v2

All notable changes to this skill are documented in this file.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/).

## [1.5.8] - 2026-04-29

Documentation refresh — inline hints in intake templates, concise guides for v1.5.7.

### Why

After 7 minor patches accumulating features (sitemap consistency, media policy, video scripts), the docs were stale and bloated:
- README still described v1.5.0 quickstart
- INTAKE_GUIDE was 516 lines duplicating template explanations
- Templates required users to flip back and forth between template and guide
- v1.5.7 video scripts feature wasn't reflected in user-facing guides

Solution: put hints WHERE users need them (in templates), keep guides concise for context/decisions only.

### Changed

- **`templates/PROJECT_INTAKE_TEMPLATE.md`** — every field now has an inline `>` hint immediately below it explaining:
  - What to put (with examples)
  - When to skip
  - How AI uses the value
  - Common values for the field
  - Header at top: minimum-fill instructions for first project
- **`templates/MODULE_INTAKE_TEMPLATE.md`** — same inline-hint treatment. BA can fill module without consulting external guide.
- **`README.md`** — refreshed for v1.5.7:
  - Quick start uses current command flow (`init` → `module` → fill → `run` → `continue` → `deploy`)
  - "All commands at a glance" table (20 commands)
  - "Key concepts" section explains layered config, re-run safety, drift detection, source change detection, multi-locale, sitemap consistency, media policy, video scripts
  - "What gets generated" section shows full output tree
  - "Documentation map" routes users to right doc by question
  - File structure reflects plugin marketplace layout
- **`INTAKE_GUIDE.md`** — rewritten as decision/context guide, not field reference (templates do that now):
  - Three editing rules
  - Minimum-fill checklist
  - 7 common patterns with concrete commands
  - Decision matrix tables for voiceover, screenshots, TTS provider
  - "AI uses each field for..." reference table
  - Mistake recovery table
  - When to expand Advanced sections
  - 7 tips for first-time success
  - Reduced from 516 lines to 258 lines (-50%)
- **`INTAKE_GUIDE.vi.md`** — full Vietnamese translation, same structure

### Benefits

For BAs filling intake first time:
- Open template → every field has explanation right there
- Don't need to flip to INTAKE_GUIDE
- INTAKE_GUIDE only consulted for decisions ("which voiceover strategy?")

For experienced users:
- Quick refresher in README "All commands at a glance"
- Decision matrices in INTAKE_GUIDE for second-project tweaks
- Reference docs (in skills/docsmith/) unchanged for deep technical lookups

For maintainers (you):
- Templates are self-documenting
- When you add a new field, just add a `>` hint inline
- Guide doesn't accumulate field descriptions over time

### What didn't change

- SETUP.md — already covered v1.5.5 TTS providers and v1.5.7 setup; unchanged
- HOW_IT_WORKS.md — last updated v1.4.0 multi-locale; could use refresh but lower priority
- Reference docs (intake-reference, deploy-reference, translate-reference, etc.) — technical detail, unchanged
- Plugin schema — same
- Commands — same (no new flags or behaviors)

### Migration from v1.5.7

For new projects: `init` scaffolds the new self-explaining templates.

For existing projects with old templates:
- Old templates still parse correctly (inline hints are markdown blockquotes, ignored by parser)
- To upgrade existing intake files in place:
  ```bash
  /docsmith init --reformat-intake
  ```
  Re-renders project.md and modules/*.md with v1.5.8 inline hints. Your filled values preserved. Backup at `documentation/intake/.backup-pre-v1.5.8/`.

### Why this is a patch (not minor)

Pure documentation. No new commands, no new fields, no new templates, no schema changes. Plugin functionality identical to 1.5.7.

## [1.5.7] - 2026-04-29

Per-video script files. Voiceover content lives in dedicated script files instead of inline VIDEO marker comments.

### Why

After v1.5.5 added voiceover/TTS, the question "where do scripts live?" was unresolved. Inline scripts in VIDEO markers (the implicit assumption) had problems:
- Bloated drafts (VIDEO marker with 40-line script comment)
- Hard to translate scripts independently of prose
- Multi-locale scripts had no clear home

v1.5.7 externalizes scripts to per-video files at `documentation/scripts/<module>/<id>.md`.

### Added

- **`templates/VIDEO_SCRIPT_TEMPLATE.md`** — definitive format for per-video script files:
  - Frontmatter: id, module, asset_path, duration_target, voiceover_strategy, voiceover_provider, source_locale, generation timestamp
  - `# Source script (<lang>)` section: BA-edited source-language script
  - `## <locale>` sections: per-locale translations (auto-managed by translate command)
  - `# Notes` section: internal notes, NOT voiced, NOT translated
  - Workflow integration with `record` and `translate` commands
  - Multi-locale strategy mapping (silent, AI per-locale, source+sub, human)
  - Path mapping table for source script + audio files + subtitles
- **`/docsmith record --check`** flag — validate script files exist for all VIDEO markers
- **`/docsmith record --re-record <id>`** flag — force re-record one video
- **`/docsmith record --migrate-scripts`** flag — extract inline scripts from old VIDEO markers, create new script files, simplify markers

### Changed

- **VIDEO marker simplified to** `<!-- VIDEO id: <id> -->` only. All other config moved to script file frontmatter.
- **Old expanded VIDEO marker syntax** still parsed by `record` for backward compatibility with v1.5.4-1.5.6 drafts. New drafts use simplified marker.
- **`record` command workflow updated**:
  1. Scan drafts for VIDEO markers
  2. Look up script file at `documentation/scripts/<module>/<id>.md`
  3. If missing → AI generates initial script from surrounding draft prose, pauses for user review
  4. If exists but stale → warn user
  5. User reviews/edits
  6. TTS generates audio per locale (skipped if silent), capture screen, encode video
- **`translate` command extended** to process script files alongside drafts. Translates `# Source script` content into `## <locale>` sections. Same per-block / batch review gate. Same glossary.
- **VIDEO_MARKER_TEMPLATE.md** — added v1.5.7+ note at top about marker simplification
- **MEDIA_POLICY_TEMPLATE § 5** — added reference to script files as voiceover content source
- **translate-reference.md § 1** — clarifies what gets translated (drafts AND scripts)
- **intake-reference.md path mapping table** — added rows for script file, voiceover audio, subtitle paths
- **SKILL.md `record` command section** — full workflow update
- **SKILL.md File organization** — adds `documentation/scripts/<module>/` directory
- **INTAKE_GUIDE en/vi § 11** — added script file note in Videos section

### Migration from v1.5.6

For new projects: just use v1.5.7. `record` creates script files automatically when needed.

For existing v1.5.6 workspaces with inline scripts in VIDEO markers:
```bash
/docsmith record --migrate-scripts
```

This:
1. Scans drafts for inline-script VIDEO markers
2. Creates `documentation/scripts/<module>/<id>.md` files from inline content
3. Updates draft VIDEO markers to short form `<!-- VIDEO id: <id> -->`
4. Backs up old drafts to `documentation/archive/<ts>/`

For drafts without VIDEO markers: nothing to migrate.

### Limitations (v1.5.7)

- **No automatic script re-write** when draft prose changes — user manually updates scripts. v1.6+ may add change detection with diff suggestions.
- **No SSML support** — TTS providers that support SSML (pause, emphasis, prosody) accept full markdown but ignore SSML tags. Adding SSML passthrough is roadmap.
- **No multiple voices per script** — entire script uses one voice per locale. Dialog-style scripts not supported.
- **No audio mixing** — voiceover only, no background music or SFX in `record`. User can post-process video manually.

### Why this is a patch (not minor)

1 new template, 5 modified templates/references, 3 new flags on `record`. No new commands. No schema-breaking changes for new projects. Old VIDEO marker format still parses for backward compat. Plugin functionality identical to 1.5.6 for users who don't use video.

## [1.5.6] - 2026-04-29

Smart defaults UX patch — collapse advanced sections by default in intake forms. Same fields, same parsing logic, just less visual noise for BAs.

### Why

After v1.5.5, intake templates accumulated to 356 lines (project) + 245 lines (module) with 71 + 41 checkboxes. BAs filling first project felt overwhelmed. They didn't need to see TTS provider list, sitemap pattern selection, or 6 other Advanced fields when defaults work for 80% of cases.

Solution: wrap Advanced sections in HTML `<details>` blocks. GitHub/VS Code/Cursor render these as collapsed clickable headers. BA sees ~80 lines top-level (essentials only). Power user clicks to expand and customize.

No new fields, no new logic, no new commands. Pure UX.

### Changed

- **`PROJECT_INTAKE_TEMPLATE.md`** — wrapped 7 Advanced sections in `<details>` blocks:
  - Secondary personas (under § 2 Audience)
  - Glossary settings (under § 3 Languages)
  - Collision behavior (under § 4 Deploy)
  - Voice and tone (separated from § 4)
  - MFA / SSO (under § 5 Credentials)
  - Additional sources (under § 6 Knowledge sources)
  - Auto-run behavior, Sitemap pattern, Media policy (whole top-level sections)
- **`MODULE_INTAKE_TEMPLATE.md`** — wrapped 7 Advanced sections in `<details>` blocks:
  - Add more features (under § 2 Scope)
  - Out of scope, Module priority (under § 2)
  - Voice override
  - Module-specific sources
  - Walkthrough setup
  - Special handling (sensitive fields, status)
  - Sitemap sections, Media override
- **`intake-reference.md` § 1** — added subsection explaining `<details>` parsing rule (AI strips wrapper, parses content, treats collapsed defaults as canonical)
- **`INTAKE_GUIDE.md` and `.vi.md`** — added "Rule 4: Skip Advanced sections" with explicit minimum-fill checklist for first project (only § 1, 2, 3, 4, 5, 6 essentials)

### Default tick state

All Advanced sections have defaults pre-ticked. Examples:

```markdown
<details>
<summary><b>Advanced — voice and tone</b> (using defaults)</summary>

Tone:
- [ ] Casual
- [x] Friendly-professional (default)
- [ ] Technical-direct
- [ ] Formal

</details>
```

When BA leaves Advanced section collapsed, AI uses ticked default. When BA expands and changes a tick, AI uses new value. Same parsing, no special-case logic.

### Visual impact

Before v1.5.6 in default GitHub render:
- Project intake: ~356 lines all visible. BA scrolls through every section.
- Module intake: ~245 lines all visible.

After v1.5.6:
- Project intake: ~80 lines essentials visible. 7 collapsed `<details>` headers indicate Advanced sections.
- Module intake: ~50 lines essentials visible. 7 collapsed `<details>` headers.

Total file size barely changed (354 / 242 lines now). Perceived complexity dropped ~75%.

### Editor compatibility

| Editor | Renders `<details>` collapsed? |
|---|---|
| GitHub web preview | ✅ Yes |
| VS Code markdown preview | ✅ Yes |
| Cursor | ✅ Yes |
| GitHub.dev | ✅ Yes |
| Plain text editor | ❌ Shows raw markup (still readable) |

For editors that don't render `<details>`, BA sees `<details>` and `<summary>` tags as plain text. Not ideal but parseable. AI still parses correctly regardless of editor rendering.

### Migration from v1.5.5

For new projects: just use v1.5.6. New init scaffolds collapsed templates.

For existing v1.5.5 workspaces:
- Existing intake files keep flat structure — no `<details>` wrapping
- AI parser still works (it ignores `<details>` whether present or not)
- To convert existing intake to collapsed form, run `/docsmith init --reformat-intake`
  - Re-renders project.md / modules/*.md with `<details>` wrappers
  - User content (filled values, ticked checkboxes) preserved
  - Backup at `documentation/intake/.backup-pre-v1.5.6/`

OR just leave existing intakes as-is. Both render correctly; only new projects benefit from collapsed default.

### Why this is a patch (not minor)

No schema changes, no new fields, no new commands, no new templates. Pure markup change to existing templates. Plugin functionality identical to 1.5.5.

## [1.5.5] - 2026-04-28

Comprehensive media policy — screenshot density rules, per-locale strategy, video voiceover/TTS configuration, subtitle generation.

### Why

User raised real gap: docsmith had screenshot caption rules and video markers but no policy for:
- How many screenshots per content type (density)
- Style options (viewport, full window, cropped, annotated)
- Aspect ratio (desktop, mobile, square)
- Per-locale screenshot strategy (1 EN ảnh dùng chung hay capture riêng cho VI/JP?)
- Video density per content type
- Voiceover language strategy (silent? AI per locale? human?)
- TTS provider abstraction
- Subtitle generation method

These decisions affect cost/effort by 3-10× depending on choice. Can't be inferred — must be explicit project decision.

### Added

- **`templates/MEDIA_POLICY_TEMPLATE.md`** — definitive reference covering:
  - Screenshot density per content type (Tutorial=1/step, How-to=1/heading, etc.)
  - 4 screenshot styles (viewport-only, full-window, cropped-element, annotated)
  - 5 aspect ratios (16:9 desktop default, 4:3, mobile, square, custom)
  - 3 per-locale strategies (source-only default, per-locale, hybrid)
  - Video density rules with length caps per content type (Tutorial ≤90s, How-to ≤30s, etc.)
  - 5 voiceover strategies (silent default, AI per locale, source+sub, human, none)
  - 6 TTS providers with config schemas (local-piper default, local-coqui, openai, elevenlabs, google-cloud, azure-cognitive)
  - 3 subtitle generation methods (auto from script, STT, manual)
  - Sidecar vs burn-in caption packaging
  - Cost estimation tables (default vs premium config)
  - Migration guide
- **Project intake § 11 "Media policy"** — full media config block in PROJECT_INTAKE_TEMPLATE
- **Module intake § 8 "Media override"** — per-module overrides for screenshot density, video density, per-locale forcing, voiceover override
- **TTS provider install instructions** in SETUP.md / SETUP.vi.md (Piper, Coqui, OpenAI, ElevenLabs, Google Cloud, Azure)
- **Whisper STT install** for human voiceover subtitle generation
- **INTAKE_GUIDE en/vi § 11** explanation for BAs (decision matrix, cost reality check)
- **INTAKE_GUIDE en/vi § 8** explanation of module media override common cases

### Changed

- **`walkthrough` command** — respects screenshot density rules and per-locale strategy from project intake § 11
- **`record` command** — respects video density rules, length caps, voiceover strategy, TTS provider config; warns on length cap exceeded; checks TTS prerequisites
- **`verify` command** — added check #11: media compliance (screenshots match density policy, videos within length caps, voiceover/subtitle files exist when expected)
- **File organization** in SKILL.md — adds `documentation/videos/voiceover/` and `documentation/videos/subtitles/` paths; per-locale image namespace under `images/<module>/<locale>/`
- **Project intake validation** — if "AI synthetic voice" selected, TTS provider must be selected; if non-local TTS, auth env var required
- **Module intake validation** — media overrides reference valid content types and providers

### Default decisions (designed for cheap-first, upgrade-later)

| Config | Default | Reason |
|---|---|---|
| Screenshot per-locale | Source-only | 1× capture cost; note "EN UI" in translated docs |
| Screenshot style | viewport-only | Clean for Docusaurus content |
| Aspect ratio | 16:9 desktop (1280×720) | Standard |
| Voiceover | Silent + on-screen captions | No TTS needed; multi-locale via text overlay |
| TTS provider | local-piper | Free, offline, no API key |
| Subtitle generation | Auto from script | Deterministic for Silent + AI voice strategies |
| Subtitle packaging | Sidecar `.vtt` | Flexibility, smaller storage |

User can upgrade strategy by re-editing project intake § 11 and re-running `walkthrough`/`record`. Existing media not regenerated unless explicit re-run.

### Deferred (v1.5.5 limitations)

- **Pixel-perfect annotation** for `annotated` style — requires manual edit (Figma, Photoshop)
- **Video editing** — record produces single-take captures; no transitions, intros/outros
- **Music library** — ambient music must be user-provided file
- **Lip-sync verification** — when AI voice + screen capture, no check that timing matches
- **Background blur** for general aesthetic (only for redaction)
- **STT subtitle from voiceover audio** — works only with text-script paths in v1.5.5

### Cost reality

Default config (silent + source-only) for 3 locales × 5 modules × 30 docs:
- ~1 hour total time, $0 TTS cost, ~65 MB storage

Premium config (AI voice per locale + per-locale screenshots):
- ~3 hours first run, ~$5-10 TTS cost, ~200 MB storage

Documented explicitly in MEDIA_POLICY_TEMPLATE § 11 so users can budget.

### Migration from v1.5.4

For new projects: just use v1.5.5. Project intake template includes Media policy section.

For existing v1.5.4 workspaces:
```bash
/docsmith plan --migrate-media
```
AI:
1. Inspects existing screenshots and videos in workspace
2. Proposes default config (silent + source-only screenshots + viewport-only style)
3. User confirms or adjusts
4. AI updates project intake § 11

Existing screenshots and videos NOT regenerated. Only new captures from next walkthrough/record follow new policy.

### Why this is a patch (not minor)

1 new template (MEDIA_POLICY), 2 modified templates, 2 modified guide files, 2 modified setup files, 4 SKILL.md sections updated. No new commands. No schema-breaking changes. Behavior is additive: project intake without § 11 falls back to all-default media policy; module intake without § 8 inherits from project.

Plugin functionality identical to 1.5.4 for projects that don't fill in the new sections.

## [1.5.4] - 2026-04-28

Sitemap consistency feature — fixes navigation drift across modules.

### Why

Real-world example from a Cloud Advisor doc + Tagging doc: same project, two modules with completely different sitemap structures. Tagging used "Quick Starts → Tutorials"; Cloud Advisor used "Guides → Reference → Glossary → Troubleshooting". Users navigating between modules felt lost. AI generated each sitemap independently with no consistency rules.

v1.5.4 adds:

1. **Canonical section types** — fixed list of 11 types every section must map to (overview, initial-setup, quickstarts, tutorials, guides, concepts, dashboard, reference, api-reference, glossary, troubleshooting)
2. **3 project patterns**:
   - **Pattern A** (Learning path, default): `overview → initial-setup → quickstarts → tutorials → guides → concepts → reference → api-reference → troubleshooting → glossary`
   - **Pattern B** (Task-first): `overview → quickstarts → guides → concepts → reference → troubleshooting → glossary`
   - **Pattern C** (Custom): user defines order in project intake
3. **Per-module section selection** — module intake has explicit "Sitemap sections" subsection with checkboxes for which canonical types this module includes
4. **AI suggestions** — `plan` warns when a module is missing a section the project pattern includes (e.g., "Pattern A includes glossary but module 'storage' didn't tick it; suggest adding")
5. **Display name overrides** — both project-level and module-level allow renaming canonical sections (slugs stay canonical for deploy paths)

### Added

- `templates/SITEMAP_PATTERNS_TEMPLATE.md` — definitive reference for section types, patterns, generation logic, validation rules
- Project intake § 9 "Sitemap pattern" — pick A/B/C and optional display name overrides
- Module intake § 7 "Sitemap sections" — per-module checkbox list of canonical types + per-module display name overrides
- New `verify` check #10: sitemap consistency (was "no orphan test cases" before; consolidated into other checks)
- INTAKE_GUIDE en/vi: added § 9 (project sitemap pattern) and § 7 (module sitemap sections) explanations
- `plan --migrate-sitemap` flag for v1.5.3-or-earlier workspaces

### Changed

- **`plan` command** — now reads project pattern and per-module sections; generates `sitemap.md` with consistency warnings inline
- **`categorize` command** — uses sitemap pattern order to compute Docusaurus `position` fields; respects display name overrides for `label`
- **`verify` command** — check #10 replaced: was "no orphan test cases" (rarely actionable), now "sitemap consistency" (actionable, surfaces real navigation drift)
- **SKILL.md** — `plan`, `verify`, `categorize` sections updated with sitemap pattern references

### Deferred

- **Auto-fix mode** for sitemap inconsistency — explicitly NOT implemented per user decision. AI warns but doesn't rename or restructure without user input. User remains the IA authority.
- **Pattern templates for specific industries** (B2B SaaS, dev tools, infrastructure) — could be Pattern D/E/F. Defer until real demand.
- **Cross-locale sitemap consistency** — translated docs follow source sitemap automatically; per-locale customization deferred.

### Migration from v1.5.3

For new projects: just use v1.5.4. Project intake template includes sitemap pattern selection.

For existing v1.5.3 workspaces:
```bash
/docsmith plan --migrate-sitemap
```
AI:
1. Inspects existing sitemap.md and modules
2. Proposes Pattern A as default
3. Suggests per-module section selections based on existing draft folder structure
4. User confirms; AI updates project intake (§ 9) and module intakes (§ 7)
5. Re-runs `plan` to regenerate sitemap.md per new rules

Existing drafts are NOT modified. Only intake files and sitemap.md change.

If you skip migration: `plan` still works (uses Pattern A defaults) but warnings about missing pattern config will appear until you complete intakes.

### Why this is a patch (not minor)

Adds 1 template, modifies 2 templates, 1 verify check changes meaning. No new commands. No schema-breaking changes for existing workspaces. Behavior is additive: project intake without § 9 falls back to Pattern A; module intake without § 7 falls back to AI inferring sections from drafts.

## [1.5.3] - 2026-04-28

Documentation patch — adds prerequisites guide for `walkthrough` and `record` commands. Bridges a gap between SKILL.md and operational reality.

### Added

- **`SETUP.md`** (English) — comprehensive prerequisites guide:
  - Path 1: Claude in Chrome extension install (recommended for beginners)
  - Path 2: Playwright MCP server (for headless / CI use)
  - Path 3: ffmpeg setup for `record` (video tutorials)
  - Environment variables for sources (NOTION_TOKEN, GITHUB_TOKEN, GOOGLE_DRIVE_TOKEN) and credentials (test account)
  - How to obtain auth tokens for each source type (step-by-step Notion integration setup, GitHub fine-grained PAT, Google Drive service account)
  - Per-project `.env` setup with direnv support
  - Pre-walkthrough and pre-record checklists
  - Common errors and fixes (login failures, browser timeouts, missing ffmpeg, wrong locale captures)
  - Network/firewall allowlist requirements
  - Performance tuning for slow walkthrough/fetch/translate
  - "docsmith without browser" path for users who only need drafts + translation
- **`SETUP.vi.md`** (Vietnamese) — full Vietnamese translation
- **README "How it works" section** now links SETUP.md as required reading before first walkthrough/record run

### Changed

- **SKILL.md** — Prerequisites notes added to:
  - `walkthrough` command (links to SETUP § Path 1 or Path 2)
  - `record` command (links to SETUP § Path 3)
  - `fetch` command (links to SETUP § Environment variables)
- **README Quick start** — fully refreshed to v1.5+ flow:
  - Step 1: SETUP reference (one-time)
  - Step 2: init
  - Step 3: module commands per feature
  - Step 4: fill intake forms (links INTAKE_GUIDE)
  - Step 5: run + continue
  - Step 6: deploy
  - Update flow section
  - In-place section consolidated
- **README "What's new"** — v1.5.3 entry added
- Removed leftover content from old Quick start

### Why this is a patch (not minor)

No new commands, no schema changes, no new templates. Pure documentation gap-fill. Skill functionality identical to 1.5.2.

### Migration from 1.5.2

None needed. SETUP.md describes existing prerequisites that were always there but undocumented. Users who already have Claude in Chrome / Playwright / ffmpeg installed don't need to do anything.

For users who tried v1.5.2 walkthrough/record and got confused by missing tools: read SETUP.md and install per Path 1/2/3 as needed.

## [1.5.2] - 2026-04-28

Plugin format compliance — restructure to follow official Claude Code plugin marketplace specification. Required to fix `/plugin marketplace add dangtran1003/docsmith-v2` failing with "Marketplace file not found".

### Changed (BREAKING for direct-clone installs only)

- **Repository restructured** to plugin marketplace layout:
  ```
  docsmith-v2/                       (was: skill files at root)
  ├── .claude-plugin/                ← new
  │   ├── marketplace.json           ← new (catalog)
  │   └── plugin.json                ← new (manifest, replaces root plugin.json)
  ├── skills/
  │   └── docsmith/                  ← new (skill content moved here)
  │       ├── SKILL.md
  │       ├── deploy-reference.md
  │       ├── intake-reference.md
  │       ├── translate-reference.md
  │       ├── process-reference.md
  │       ├── tools-reference.md
  │       ├── presets/
  │       └── templates/
  └── (top-level docs unchanged: README, CHANGELOG, etc.)
  ```
- **Old root `plugin.json` removed** — replaced by `.claude-plugin/plugin.json` with the same purpose. Marketplace integration was broken without this restructure.
- **Reference docs and templates moved into `skills/docsmith/`** — they're referenced from SKILL.md by relative path, must be siblings.

### Added

- `.claude-plugin/marketplace.json` — marketplace catalog with one plugin entry pointing to `./` (the same repo)
- `.claude-plugin/plugin.json` — official plugin manifest format
- README "Update" section showing how to update for each install method
- PUBLISHING.md rewritten with new install commands using marketplace + alternative manual install paths

### Fixed

- `/plugin marketplace add dangtran1003/docsmith-v2` now works (was failing pre-1.5.2 because no `.claude-plugin/marketplace.json`)
- Install command syntax corrected: `/plugin install docsmith@dangtran1003-docsmith-v2` (the `@<marketplace-name>` form was missing in v1.5.1 docs)

### Migration

For users who installed v1.5.x via `/plugin marketplace add` (impossible — install was broken until 1.5.2):
- N/A; v1.5.2 is the first version that actually works as a marketplace plugin.

For users who direct-cloned v1.5.x into `~/.claude/skills/docsmith/`:
1. Pull latest: `cd ~/.claude/skills/docsmith && git pull`
2. Reinstall from new path:
   ```bash
   rm -rf ~/.claude/skills/docsmith
   git clone https://github.com/dangtran1003/docsmith-v2.git ~/repos/docsmith-v2
   ln -s ~/repos/docsmith-v2/skills/docsmith ~/.claude/skills/docsmith
   ```
   Or copy: `cp -r ~/repos/docsmith-v2/skills/docsmith ~/.claude/skills/`

For users who didn't install yet: just use `/plugin marketplace add dangtran1003/docsmith-v2` directly.

### Why this is a patch (not minor)

No new commands, no new features, no new templates, no schema changes. Pure restructuring required by the platform. Plugin functionality identical to 1.5.1.

## [1.5.1] - 2026-04-26

Patch release — bug fixes for path conflicts + comprehensive intake usage guide. No new commands or breaking changes.

### Fixed

- **Issue 1: `<feature>` path ambiguity resolved.** Drafts go to `documentation/drafts/<source-locale>/<module.folder>/<doc>.md`. `<module.folder>` defaults to `module.slug`. Documented explicitly in SKILL.md § "Path rules" and intake-reference.md § "Path mapping".
- **Issue 2: in-place mode collision detection.** `init` now pre-checks for existing `documentation/` content, recognizes when running inside a Docusaurus repo (suggests in-place mode), refuses to scaffold silently next to user content. New `--in-place` flag for explicit opt-in.
- **Issue 3: re-run protocol on `module` create.** Already worked, but now documented explicitly in SKILL.md and INTAKE_GUIDE.
- **Issue 4: per-module run state.** `documentation/.run-state/<module>.yaml` (one file per module) replaces single `documentation/.run-state.yaml`. Fixes loss of state when running multiple modules sequentially through different pause gates.
- **Issue 5: deployments folder moved inside workspace.** `documentation/deployments/` instead of root-level `deployments/`. docsmith's footprint at the project root is now exactly one folder: `documentation/`. Reduces collision risk in in-place mode.
- **Issue 6: glossary path explicit in intake.** Project intake template now has a § Languages > Glossary files subsection clarifying that AI looks for `documentation/standards/glossary.<locale>.yaml` at a fixed path. Auto-created (empty) by `init` when target languages are set.
- **`.gitignore` smart append.** `init` no longer overwrites existing `.gitignore`. It appends a `# BEGIN docsmith ... # END docsmith` block, idempotent on re-run.

### Added

- **`INTAKE_GUIDE.md` and `INTAKE_GUIDE.vi.md`** — comprehensive practical guide for BAs filling intake forms. Covers: what each section means, how AI uses each field, common patterns (single-feature, in-place, multilingual, Notion sources, post-UI-change updates, source-change updates), error recovery.
- **README link** to INTAKE_GUIDE in How it works section.

### Changed

- **File organization in SKILL.md** updated to reflect new paths (`documentation/deployments/`, per-module `.run-state/<module>.yaml`).
- **Path scoping rules** updated: writes only inside `documentation/` (workspace, includes deployments + run-state) and `deploy.target_path` (deploy command only).
- **`init` behavior** documented with pre-checks and `--in-place` flag.
- **Project intake template** § Languages now has glossary path note + glossary_required option.

### Migration from 1.5.0

Existing v1.5.0 workspaces (if any beta users):

1. Move `deployments/` (root level) into `documentation/deployments/`: `mv deployments documentation/deployments`
2. Move `.run-state.yaml` into per-module files: `mkdir documentation/.run-state && mv documentation/.run-state.yaml documentation/.run-state/<module>.yaml` (rename per the module the state was for)
3. Update `.gitignore`: replace any standalone `documentation/.run-state.yaml` entry with `documentation/.run-state/`

Or simpler: fresh `init` if you don't have unsaved progress.

For v1.4.x users following `--upgrade-from-1.4`: the upgrade automatically applies the new layout.

## [1.5.0] - 2026-04-26

Lean refactor + intake-driven config. Largest release in the v1.x line. Originally planned as v1.5 + v1.6 combined.

### Added

- **Markdown intake forms** — `documentation/intake/project.md` and `documentation/intake/modules/<n>.md` replace `.docsmithrc.yaml` as primary config. BA-friendly: checkboxes, fillable backtick fields, no YAML syntax.
- **Layered config** — defaults → project intake → module intake → CLI flags. Higher layers override lower for the same field; sources are cumulative.
- **`module` command** — manage per-feature module intakes. Sub-commands: `<n>`, `--from`, `list`, `archive`, `unarchive`. Updates project.md modules list automatically.
- **`fetch` command** — pull external knowledge sources into local cache. Types: Notion, GitHub, Google Drive, URL, local file. Auth via env var references (never plaintext).
- **`run` command** — orchestrated pipeline that auto-chains stages with configurable pause gate (default `after-draft`). Saves state to `.run-state.yaml`.
- **`continue` command** — resume `run` from saved state.
- **`update` command** — detect external source changes via cheap metadata calls (Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime). Propose targeted draft re-runs.
- **`intake-help` command** — print field reference for intake forms with validation rules and defaults.
- **`sources.lock.yaml`** — auto-managed lock file tracking fetched state per source (versions, hashes, cache paths).
- **External source cache** — `documentation/.cache/sources/` (gitignored) holds fetched content for offline runs.
- **`intake-reference.md`** — comprehensive reference for intake parsing, layered config resolution, validation, fetch/update workflow.
- **3 new templates**: `PROJECT_INTAKE_TEMPLATE.md`, `MODULE_INTAKE_TEMPLATE.md`, `SOURCES_LOCK_TEMPLATE.md`.

### Changed

- **`init` command** — scaffolds intake forms instead of writing yaml. New flag `--upgrade-from-1.4` reads existing `.docsmithrc.yaml` and pre-fills `project.md`.
- **`voice` command** — Quick mode default (1 file: voice-chart.md). `--full` flag opt-in for legacy 3-file output (UX patterns + scorecard).
- **`translate` command** — Default review mode is now `batch` (whole-file diff) instead of `per-block`. `--per-block` flag for opt-in safer mode.
- **Default `behavior.on_existing`** — kept as `prompt` (safer for new users). Power users can switch via intake `Auto-run behavior`.
- **Process flow** — now starts at intake-fill rather than interactive prompts. Stages still individually invokable; `run` chains them.
- **20 commands** total. v1.4.0 had 21 commands (with overlap). Lean cuts: removed `start`, `validate`, `test`, `tech-review`, `peer-review` as separate commands (merged into others or replaced by intake/run). Added: `module`, `fetch`, `run`, `continue`, `update`, `intake-help`.

### Removed

- **`start` command** — duplicated `init`. Use `init` to scaffold; use `run` to execute pipeline.
- **`validate` command** — replaced by `walkthrough --check` (already does the same job).
- **`test` command** — folded into `walkthrough` (test cases auto-built from drafts).
- **`peer-review` and `tech-review` commands** — these are inherently human steps. Now part of the natural pause-and-review flow between `run` and `continue`. No separate command needed.
- **`review-plan` command** — removed; review happens inline with `--pause-at after-plan`.
- **`incorporate` command** — folded into `edit --from-review`.
- **`sitemap` command** — output of `plan` (combined into one stage).
- **5 templates removed** as redundant or no longer needed: `MERGE_DECISION_TEMPLATE.md`, `TRANSLATION_DECISIONS_TEMPLATE.md`, `UX_CONTENT_SCORECARD_TEMPLATE.md` (use `voice --full` if needed), `UX_TEXT_PATTERNS_TEMPLATE.md` (use `voice --full`), `TRACEABILITY_MATRIX_TEMPLATE.md` (inlined into documentation-plan), `WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md` (folded into test-case template).
- **2 reference docs removed**: `subprocess-010a.md` (folded into `process-reference.md`), `update-reference.md` (folded into SKILL.md re-run protocol section).

### Deferred / not in 1.5.0

- **Visual regression** in walkthrough (pixel-diff between captured and previous screenshots): future, no version assigned. Was originally on v1.5 plan but de-prioritized after user feedback that re-run safety + intake matter more.
- **`migrate` command** for config schema changes: removed from roadmap. Since docsmith has no production users yet, breaking config changes are handled in CHANGELOG migration notes only.
- **`adopt` command** (convert existing Docusaurus docs into workspace): future. v1.5.0 has `init --upgrade-from-1.4` for migrating from prior docsmith versions, but not for converting existing untracked docs.
- **`health` command** (one-shot wrapper of verify + drift + compare): future. Use `/docsmith verify` and `/docsmith update` separately for now.
- **Translation drift tracking** (`translate --check` mode): future. Workaround: re-run `translate` in Update mode.
- **`<!-- translation-locked -->` markers**: future.
- **Per-locale image namespacing** for products with localized UI screenshots: future.
- **Per-doc source provenance tracking** in `update` (which doc uses which source for surgical updates): future. Currently `update` re-evaluates all docs in a module on any source change.

### Migration from 1.4.x

For existing v1.4.x workspaces:

1. Pull v1.5.0
2. Run `/docsmith init --upgrade-from-1.4` — reads `.docsmithrc.yaml` and pre-fills `documentation/intake/project.md`
3. Edit `project.md` to fill remaining fields (audience, sources, voice details that weren't in yaml)
4. Run `/docsmith module <n>` for each feature area (`instances`, `storage`, etc.)
5. Edit each module file
6. Run `/docsmith run` to verify everything resolves correctly

Old yaml is read for backward compat but deprecated. v1.6 will remove yaml support.

For NEW projects: just `/docsmith init` directly produces the new structure.

### Why this large release

User feedback during v1.4.0 design: "22 commands too many, intake configuration unfriendly to BAs". Lean refactor addresses both:
- Commands cut/merged from 21 to 20 (number similar but functional overlap removed)
- YAML replaced with markdown forms
- New `run`/`continue`/`update` automate the boring parts

Combined with v1.6's planned translate command (already shipped in v1.4.0), the v1.x line is feature-complete enough for early production trial.

## [1.4.0] - 2026-04-26

Multi-locale translation lands. Originally planned as v1.6 but promoted because translation gates the "done" definition for any multi-locale project.

### Added

- **`translate` command** — AI translation from `locales.source` to each entry in `locales.targets`. Per-block review gate by default; `--auto-approve` flag for speed. See [translate-reference.md](translate-reference.md).
- **Per-block review gate** — for each translatable block (heading, paragraph, list, table cell, alt text, link text, frontmatter title), AI proposes translation. User decides: `y` approve / `e` edit / `s` skip (keep source) / `n` remove / `a` approve all remaining / `q` quit.
- **Glossary support** — optional per-locale `documentation/standards/glossary.<locale>.yaml` with longest-match-wins lookup, case sensitivity, context disambiguation, UI label preservation. Built iteratively from per-block review corrections. See [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml).
- **Translation metadata** in frontmatter — `translated_from`, `translated_at`, `source_hash`, `glossary_version`, `translation_status`. Forward-compatible with v1.6.x drift tracking.
- **Block-level preserve rules** — code blocks, inline code, file paths, URLs, frontmatter `id`/`slug`, image src paths, video markers, MDX component tags are NEVER translated.
- **Re-run protocol integration** — translate honors the 4-option gate. Update mode preserves manually-edited translations when source unchanged; only proposes changes for modified/new/removed source blocks.
- **Translation completeness check in `deploy`** — when `locales.targets` non-empty, deploy verifies each target has translated drafts for all source files. Missing translations emit a warning per locale.
- **`--locale <locale>` flag** for `translate` and `walkthrough` — scope to single target locale.
- **Process Flow update** — `translate` is positioned after `incorporate` and before `categorize`/`deploy`. Required for multi-locale projects; no-op for single-locale.
- 2 new templates: `GLOSSARY_TEMPLATE.yaml`, `TRANSLATION_DECISIONS_TEMPLATE.md`. New: `translate-reference.md`.

### Changed

- **`init` command** — for each target locale, scaffolds empty `glossary.<locale>.yaml` from template (replaces v1.2.x README placeholders). Target draft folders remain empty for `translate` to populate.
- **`deploy` command workflow** — adds translation completeness check as step 1 before detection. Warns and lists incomplete locales but does not block.
- **`.docsmithrc.yaml` schema** — added `translate:` block (`enabled`, `glossary_required`, `default_review_mode`, `preserve_frontmatter_fields`, `translate_frontmatter_fields`, `warn_on_deploy_if_incomplete`).
- **Docusaurus preset** — comment clarifies non-source-locale drafts come from `translate` command, not direct authoring.
- **File Organization** — `glossary.<locale>.yaml` listed under `standards/`; target locale draft folders contain real `.md` files (not README placeholders) once `translate` has run.

### Deferred to future versions

- **Translation drift tracking** (`translate --check` mode listing source-changed sections without prompting): v1.6.x roadmap. Workaround: re-run `translate` in Update mode.
- **`<!-- translation-locked -->` markers** for protecting blocks from re-translation: v1.6.x roadmap.
- **Per-locale image namespacing** for products with localized UI screenshots: v1.5+ roadmap.
- **Voice chart per locale** (`voice-chart.<locale>.md`) for tone consistency in translations: v1.5+ roadmap.
- **Visual regression in walkthrough** (pixel-diff): v1.5+ roadmap.
- **`migrate` command** for config schema changes: v1.5+ roadmap if schema changes.
- **`adopt` command** + **`health` command**: v1.5+ roadmap (unchanged).

### Migration from 1.3.x

No breaking changes for single-locale projects. For multi-locale projects:

1. Pull v1.4.0
2. (Optional) Create `documentation/standards/glossary.<locale>.yaml` from [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml) for each target locale
3. Replace v1.2.x README placeholders in target draft folders with actual translations: `/docsmith translate <product>`
4. Per-block review gate ensures no garbage gets into target drafts
5. Add `translate` to your CI pipeline if applicable
6. Future deploys verify translation completeness before applying

If you previously hand-translated files into `drafts/<target-locale>/`, those files are detected by `translate` re-run protocol and treated as existing — Update mode preserves them as KB.

### Roadmap revision

Numbering compressed: what was planned as v1.4 + v1.5 + v1.6 is now redistributed:

- **v1.4.0** (this release): translation (was v1.6)
- **v1.5.0** (next): visual regression + migrate command + per-locale image namespacing + voice chart per locale (was v1.4 + parts of v1.5)
- **v1.6.0** (later): translation drift tracking + lock markers + adopt command + health command (deferred items from v1.4 + originals from v1.5/v1.6)

## [1.3.0] - 2026-04-26

Re-run safety + drift detection + delete propagation. Doc CRUD becomes deterministic.

### Added

- **Re-run protocol**: every command checks output existence before writing. 4-option gate when exists: Update / Overwrite / Side-by-side / Cancel. Default `prompt`; configurable via `behavior.on_existing` in `.docsmithrc.yaml`.
- **KB inheritance**: in Update mode, AI reads existing artifact in full as canonical knowledge base. Generates 4-kind delta proposal (NEW / UPDATE / REMOVE / KEEP). User approves per-item. Untouched content (KEEP) is NEVER regenerated — preserves team's manual edits.
- **Per-artifact merge logic**: detailed rules per artifact type (audience, plan, sitemap, voice, drafts, test cases) in [update-reference.md](update-reference.md).
- **Walkthrough 3-phase pipeline**:
  - Phase A (`--check`): VERIFY only — read drafts, run assertions, output drift report. Read-only, fast (~30s for moderate doc set).
  - Gate: user reviews drift report, sets per-item decisions in `decisions.yaml` (auto-fix / manual-fix / product-bug / skip).
  - Phase B (`--apply`): UPDATE drafts per decisions.
  - Phase C: CAPTURE screenshots.
  - Default mode (no flags): runs A → interactive gate → B → C in sequence.
  - `--skip-drift` flag: Phase C only (backward compatible with 1.2.x usage).
  - `--auto-apply-high-confidence` flag: skip gate, auto-apply HIGH confidence fixes only.
- **Product-bug tracking** (optional, non-blocking): drift items can be marked `product-bug` (doc is correct, UI has regression). Tracked in `walkthrough/active-product-bugs.yaml` across runs. Auto-resolved when UI matches doc again.
- **Delete propagation**: `deploy` always lists "Orphan files in target" in plan output. With `--sync-deletes` flag, executes deletes after copies (with backup to `deployments/<ts>/deleted/`). Default: never delete (safe).
- **Drift report template** ([templates/DRIFT_REPORT_TEMPLATE.md](templates/DRIFT_REPORT_TEMPLATE.md)): format spec, confidence rubric, decision values.
- **Merge decision template** ([templates/MERGE_DECISION_TEMPLATE.md](templates/MERGE_DECISION_TEMPLATE.md)): on-disk format for re-run protocol audit.
- **Update reference doc** ([update-reference.md](update-reference.md)): full re-run + KB inheritance + drift + delete propagation logic.
- **Archive folder**: `documentation/archive/<timestamp>/` for re-run backups.
- **Drift folder**: `documentation/walkthrough/drift/<timestamp>/` for drift detection runs.

### Changed

- **`draft` / `plan` / `sitemap` / `voice`**: now subject to re-run protocol gate when output already exists. Update mode reads existing as KB and proposes deltas only.
- **`walkthrough`**: refactored from single-pass to 3-phase pipeline. Default mode unchanged externally (still works without flags), but internally adds drift verification before fix-and-capture.
- **`deploy`**: plan output always includes "Orphan files in target" section. New `--sync-deletes` flag opts into actual deletion.
- **`.docsmithrc.yaml` schema**: added `behavior.on_existing`, `behavior.drift_default_action`, `behavior.side_by_side_suffix_style`, `deploy.sync_deletes`, `deploy.audit_retention_days`.

### Deferred to future versions

- **Visual regression** in walkthrough (pixel-diff between captured screenshots and previous): on v1.4 roadmap.
- **`migrate` command** for `.docsmithrc.yaml` schema changes between major versions: on v1.4 roadmap if needed.
- **`health` command** as a one-shot wrapper of verify + drift + compare: v1.5 roadmap.
- **Translation** (auto-translate from source locale): v1.6 roadmap (unchanged).
- **`adopt` command** (convert existing Docusaurus docs into workspace): v1.5 roadmap (unchanged).

### Migration from 1.2.x

No breaking changes. New behavior is opt-in via flags:

- All v1.2.x command invocations continue to work without modification
- First re-run of any command on existing output will trigger the new gate; choose Update mode to merge with existing
- `walkthrough --skip-drift` mimics v1.2.x behavior exactly if you want to defer adopting drift detection

To adopt fully:

1. Pull v1.3.0 (`git pull` or `/plugin marketplace update`)
2. Optionally edit `.docsmithrc.yaml` to add `behavior:` block (see [.docsmithrc.example.yaml](.docsmithrc.example.yaml))
3. Run `/docsmith walkthrough <product> --check` after your next product release to see drift detection in action
4. Use `/docsmith deploy <product> --sync-deletes --dry-run` after deleting any drafts to preview cleanup

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
