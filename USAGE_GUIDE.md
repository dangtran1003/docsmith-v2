# Usage Guide — from basic to advanced

A workflow-focused guide to using the `docsmith` skill, walking through real scenarios from your first 10-minute project to advanced features (multi-locale, video voiceover, drift detection, Docusaurus deploy).

> **Read first:** [SETUP.md](SETUP.md) (browser tool, env vars, ffmpeg/TTS) and [INTAKE_GUIDE.md](INTAKE_GUIDE.md) (how to fill intake forms).
> **Internal references:** [HOW_IT_WORKS.md](HOW_IT_WORKS.md) (operating model), [CHANGELOG.md](CHANGELOG.md) (version history).

---

## Table of contents

- [Part 1 — Basic: your first project](#part-1--basic-your-first-project)
- [Part 2 — Intermediate: real-world workflow](#part-2--intermediate-real-world-workflow)
- [Part 3 — Advanced: deep customization](#part-3--advanced-deep-customization)
- [Part 4 — Maintenance & updates](#part-4--maintenance--updates)
- [Part 5 — Troubleshooting](#part-5--troubleshooting)
- [Part 6 — Command cheat sheet](#part-6--command-cheat-sheet)

---

## Part 1 — Basic: your first project

Goal: produce documentation for one module, one locale, no walkthrough/video. ~10 minutes.

### 1.1. Install the skill (once per machine)

```bash
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2
```

Or clone directly: see [README.md](README.md) § Install.

### 1.2. Init workspace

```bash
mkdir my-docs && cd my-docs
/docsmith init
```

Result: creates `documentation/intake/project.md` and folder structure. Footprint at the project root is exactly one folder: `documentation/`.

### 1.3. Create a module

```bash
/docsmith module pricing
```

Creates `documentation/intake/modules/pricing.md`.

### 1.4. Fill the intake (essentials only)

Open `documentation/intake/project.md` and `documentation/intake/modules/pricing.md`. **Skip every "Advanced — ..." section (collapsed by default).** Fill only:

- **Project intake**: Product info, Audience (one primary persona), Languages (`source: en`, no targets → single-locale), Knowledge sources (at least one — a local file or public URL is easiest)
- **Module intake**: Module identity, Scope (1-3 features to document)

See [INTAKE_GUIDE.md](INTAKE_GUIDE.md) § "Rule 4: skip Advanced sections" for the minimum-fill checklist.

### 1.5. Run the pipeline (skip walkthrough/record)

```bash
/docsmith run pricing --pause-at after-draft
```

The AI auto-chains: validate intake → fetch sources → audience → plan → voice → draft → edit, then **pauses** at after-draft.

### 1.6. Review draft

Open `documentation/drafts/en/pricing/`, read what the AI wrote. Edit by hand if needed.

### 1.7. Verify

```bash
/docsmith verify pricing
```

Runs 11 checks (placeholders, voice, links, ...). Fix any issues.

**At this point you have a complete docs set.** If you only need markdown to paste elsewhere, stop here. To deploy → continue to Part 2.

---

## Part 2 — Intermediate: real-world workflow

Covers: multiple modules, multiple knowledge sources, multi-locale, walkthrough screenshot capture, Docusaurus deploy.

### 2.1. Multi-module project

```bash
/docsmith module instances
/docsmith module storage
/docsmith module pricing
/docsmith module list                    # list + status
/docsmith module archive old-feature     # skip in run
```

Each module has its own intake. Running `/docsmith run` (no arg) processes all active modules.

Tip: use `--from` to clone setup from an existing module (sources, voice override):

```bash
/docsmith module billing --from pricing
```

### 2.2. Knowledge sources from various places

In `project.md` § Knowledge sources, tick the source types you need:

- **Notion**: requires `NOTION_TOKEN` env var (see [SETUP.md](SETUP.md) § "Environment variables")
- **GitHub** (private repo): `GITHUB_TOKEN`
- **Google Drive**: `GOOGLE_DRIVE_*` via MCP
- **Public URL**: no token needed
- **Local file**: relative path from project

Each source has an ID (e.g., `notion-pricing-spec`). The AI fetches it to `documentation/.cache/sources/<id>.md` (gitignored) and records metadata in `sources.lock.yaml`.

```bash
/docsmith fetch                          # fetch all
/docsmith fetch --source notion-pricing  # fetch one
```

`run` calls `fetch` automatically in the first stage; you don't need to call it manually unless forcing a refresh.

### 2.3. Multi-locale

In `project.md` § Languages:

```markdown
- Source locale: `en`
- Target locales:
  - [x] vi
  - [x] jp
```

After draft, the `run` pipeline auto-calls `translate` for `vi` and `jp`. Default review mode is `batch` (whole-file diff). Use `--per-block` for per-block review (safer, slower).

```bash
/docsmith translate                       # all modules, all targets
/docsmith translate pricing --locale vi   # one module + one locale
/docsmith translate --auto-approve        # CI/bulk, skip review (use carefully)
```

**Glossary** (recommended): create `documentation/standards/glossary.vi.yaml` to lock terminology. Build incrementally from review corrections. See `templates/GLOSSARY_TEMPLATE.yaml`.

### 2.4. Walkthrough — verify drafts against the live product

**Requires**: browser automation (Claude in Chrome extension OR Playwright MCP) + test account credentials. See [SETUP.md](SETUP.md).

Three-phase pipeline:

```bash
/docsmith walkthrough --check            # Phase A: VERIFY (read-only, ~30s)
                                          # Produces drift report
# Edit documentation/walkthrough/drift/<ts>/decisions.yaml
# Mark each item: auto-fix / manual-fix / product-bug / skip
/docsmith walkthrough --apply            # Phase B + C: apply + capture screenshots
```

Or run the whole thing:

```bash
/docsmith walkthrough                    # A → gate → B → C
```

Screenshots land in `documentation/images/<module>/`. Drafts use `placehold.co` placeholders that get replaced after capture.

**Drift report**: lists places where docs disagree with the UI (renamed buttons, new screens, ...). Items marked `product-bug` are tracked in `walkthrough/active-product-bugs.yaml` across runs and auto-resolve when the UI matches the doc again.

### 2.5. Deploy to Docusaurus

```bash
/docsmith deploy --dry-run               # Preview: print plan, no writes
/docsmith deploy                          # Apply
cd ../my-docusaurus-site
git diff && git add . && git commit -m "Update docs" && git push
```

`deploy` does:
- Inject Docusaurus frontmatter (id, title, sidebar_position)
- Namespace image paths (`/img/<slug>/`)
- Escape MDX (curly braces, JSX-like syntax)
- Generate `_category_.json` (if preset = docusaurus)
- Create audit trail at `documentation/deployments/<ts>-<target>/`

Useful flags:
- `--target <path>` — override target_path for this run
- `--locale vi` — single locale only
- `--sync-deletes` — propagate deletions (default: report orphans only)
- `--force` — ignore conflict warnings

### 2.6. In-place mode (existing Docusaurus)

```bash
cd ~/my-docusaurus-site
/docsmith init --in-place                # Detects docusaurus.config, target = .
/docsmith module pricing
# fill intakes
/docsmith run pricing
/docsmith deploy
```

Workspace `documentation/` and target `docs/`, `i18n/`, `static/` are siblings inside the same Docusaurus repo.

---

## Part 3 — Advanced: deep customization

### 3.1. Sitemap pattern (cross-module consistency)

Three built-in patterns (project intake § Sitemap pattern):

- **Pattern A — Learning path** (default): `overview → initial-setup → quickstarts → tutorials → guides → concepts → reference → api-reference → troubleshooting → glossary`
- **Pattern B — Task-first**: `overview → quickstarts → guides → concepts → reference → troubleshooting → glossary`
- **Pattern C — Custom**: define your own order

Each module ticks which canonical sections it includes (module intake § Sitemap sections). `plan` warns if a module is missing a section the project pattern includes.

Override display names (project or module level) to rename without changing canonical slugs:

```markdown
Display name overrides:
- guides: How-To Guides
- api-reference: API Docs
```

Migrate older workspaces (v1.5.3-): `/docsmith plan --migrate-sitemap`.

### 3.2. Voice & UX patterns

```bash
/docsmith voice                           # Quick mode (default): voice chart
/docsmith voice --full                    # + UX text patterns + scorecard
```

Voice chart derives from project intake § Voice and tone. Modules can override (module intake § Voice override).

The 5-pass `edit` measures drafts against the voice chart (voice match score). `verify` check #3 enforces score ≥ threshold.

### 3.3. Media policy (screenshots + video)

Project intake § 11 Media policy controls:

- **Screenshot density** per content type (tutorial: high, reference: low, ...)
- **Style** (browser chrome, annotations, blur)
- **Aspect ratio** (16:9, 4:3, mobile)
- **Per-locale strategy**: `source-only` (one screenshot reused — default), `per-locale` (capture each locale's UI separately)
- **Video density**, length cap, voiceover strategy
- **TTS provider**: local-piper (default, $0), ElevenLabs, OpenAI, Google, Azure, Coqui

Modules can override (module intake § 8 Media override). See `templates/MEDIA_POLICY_TEMPLATE.md`.

`verify` check #11 enforces media compliance.

### 3.4. Tutorial video + voiceover (v1.5.7)

In drafts, place a marker:

```markdown
<!-- VIDEO id: instance-create-tour -->
```

When you run `/docsmith record`:

1. AI scans markers → looks for `documentation/scripts/<module>/<id>.md`
2. If missing → AI generates initial script from surrounding draft, **pauses** for review
3. You edit the script (frontmatter: duration_target, voiceover_strategy, voiceover_provider; body: `# Source script`)
4. Confirm → TTS produces audio per locale (skipped if silent), captures screen, encodes video
5. Marker is replaced with `<video>` embed; original kept as a comment for re-record

Translate scripts: `/docsmith translate` processes script files alongside drafts. Translations live in the same file under `## vi`, `## jp`, ... headings.

Useful flags:
- `--check` — validate every marker has a script
- `--re-record <id>` — force one video to be re-recorded
- `--migrate-scripts` — extract inline scripts from older drafts (v1.5.6-) into separate files

See `templates/VIDEO_SCRIPT_TEMPLATE.md`.

### 3.5. Glossary management

Create `documentation/standards/glossary.vi.yaml` (path is fixed, not configurable):

```yaml
- term: instance
  translation: phiên bản máy
  notes: prefer "phiên bản máy" over "instance" in ops docs
- term: bucket
  translation: bucket
  notes: keep as-is (familiar to VN devops)
```

Build incrementally during translate review. `verify` check #4 enforces glossary consistency.

### 3.6. Re-run protocol & KB inheritance

Every command that writes output checks for existence first. If found:

1. **Update** (recommended) — read existing as KB, propose deltas (NEW / UPDATE / REMOVE / KEEP). Untouched parts kept verbatim → **manual edits never lost**.
2. **Overwrite** — discard, archive to `archive/<ts>/`, regenerate. Requires explicit "OVERWRITE" confirmation.
3. **Side-by-side** — keep both, new file gets `-v2` suffix.
4. **Cancel** — abort.

Default = `prompt`. Set `auto: update` in project intake § Auto-run behavior to default to Update.

### 3.7. Drift detection & product-bug tracking

```bash
/docsmith walkthrough --check            # Verify-only, fast
```

Produces `drift/<ts>/decisions.yaml`. Each item:

```yaml
- type: button-renamed
  doc: drafts/en/instances/create.md:42
  ui: "Launch instance" → "Create instance"
  decision: auto-fix         # or: manual-fix, product-bug, skip
  confidence: HIGH
```

`--auto-apply-high-confidence` skips the gate and auto-applies HIGH-confidence fixes.

`product-bug` items are tracked in `walkthrough/active-product-bugs.yaml` across runs and auto-resolve when the UI matches the doc again.

### 3.8. Source change detection

```bash
/docsmith update                          # check all modules
/docsmith update pricing                  # one module
/docsmith update --auto-apply             # CI mode
```

Workflow:
1. Read `sources.lock.yaml`
2. Cheap metadata check: Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime
3. Build change report
4. Confirm → full-fetch changed sources → re-run `draft` in Update mode → re-run `wt --check`
5. Update lock

This is how you automatically detect spec/source changes and propose targeted re-drafts.

### 3.9. Full verify (11 checks)

```bash
/docsmith verify
/docsmith verify pricing                  # one module
/docsmith verify --locale vi              # translated locale
/docsmith verify "drafts/en/**/api-*.md"  # glob scope
```

11 checks: placeholders, broken images, voice score, glossary consistency, code blocks unchanged across locales, frontmatter, internal links, cross-refs, sitemap-draft match, sitemap consistency, media compliance.

### 3.10. Custom auto-run gate

Project intake § Auto-run behavior:

```markdown
- Pause at: `after-plan` / `after-draft` (default) / `before-walkthrough` / `after-walkthrough` / `before-deploy` / `never`
- On existing output: `prompt` (default) / `update` / `overwrite` / `side-by-side`
```

Set `pause-at: never` + `on existing: update` for an automated CI pipeline.

---

## Part 4 — Maintenance & updates

### 4.1. When the spec changes

```bash
/docsmith update                          # propose targeted re-drafts
# Review proposal → confirm
```

Update mode uses KB inheritance: untouched parts preserved, AI generates only deltas.

### 4.2. When the UI changes (drift)

```bash
/docsmith walkthrough --check
# Edit decisions.yaml
/docsmith walkthrough --apply
```

### 4.3. When you add a new locale

1. Edit `project.md` § Languages → tick the new locale
2. `/docsmith translate --locale <new>`
3. `/docsmith deploy --locale <new>`

Source-locale drafts are unaffected. Glossary `<new>.yaml` builds incrementally during review.

### 4.4. When you change voice / audience

```bash
/docsmith voice                           # Update mode → propose delta
/docsmith edit --from-review feedback.md  # apply annotated reviewer feedback
```

`edit --from-review` reads a markdown file with `// FEEDBACK: ...` annotations or block comments and applies them in place.

### 4.5. When you archive an old feature

```bash
/docsmith module archive old-feature
```

The module is skipped in `run`. Drafts and intake are kept (not deleted). Unarchive when needed.

### 4.6. When you upgrade docsmith

```bash
/plugin marketplace update dangtran1003-docsmith-v2
```

Check [CHANGELOG.md](CHANGELOG.md) for breaking changes. Some versions ship migration flags (e.g., `plan --migrate-sitemap` for v1.5.4, `record --migrate-scripts` for v1.5.7).

---

## Part 5 — Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `init` says "documentation/ already exists" | Existing folder isn't from docsmith | Move/back up, or `--force` (with confirm) |
| `fetch` says `NOTION_TOKEN not set` | Missing env var | Set per [SETUP.md](SETUP.md) |
| `walkthrough` says "no browser tool available" | No Claude in Chrome / Playwright MCP | [SETUP.md](SETUP.md) § "Path 1 / Path 2" |
| `record` says "ffmpeg not found" | ffmpeg not installed | [SETUP.md](SETUP.md) § "Path 3" |
| `translate` doesn't honor glossary | No `glossary.<locale>.yaml` | Create from template |
| `deploy` says "translation incomplete" | Some locale not yet translated | Run `/docsmith translate --locale <x>` first |
| Re-run lost my manual edits | Picked Overwrite instead of Update | Recover from `archive/<ts>/`, choose Update next time |
| Sitemap inconsistent across modules | Module missing canonical section | Edit module intake § Sitemap sections, re-run `plan` |
| `update` doesn't detect a change | Source backend hides metadata | Force refresh: `fetch --source <id>` |
| Drift report empty but UI clearly changed | Stale walkthrough cache | Run `walkthrough --check` again; if still empty, check credentials/login |

### 5.1. When the pipeline gets stuck

```bash
/docsmith continue                        # Resume from .run-state/<module>.yaml
```

If state is >7 days old or workspace was edited outside docsmith, you'll get a warning before resuming. You can delete `.run-state/<module>.yaml` to start that module fresh.

### 5.2. Rolling back a deploy

Each deploy stores an audit at `documentation/deployments/<ts>-<target>/` (manifest, target-config snapshot, diff, pre-deploy hashes). Restore manually from the snapshot, or revert the commit on the target repo.

### 5.3. Path scoping rejection

If you see "write outside allowed roots": check `target_path` in the project intake. All writes must land inside `documentation/` or `deploy.target_path`.

---

## Part 6 — Command cheat sheet

| Command | Alias | When to use |
| --- | --- | --- |
| `init` | `i` | First time, scaffold workspace |
| `module <n>` | `m` | Per feature area / doc cluster |
| `module list` | | Module status |
| `module archive <n>` | | Skip in runs |
| `intake-help` | `ih` | See what each intake field means |
| `fetch` | `f` | Force-refresh sources (usually auto) |
| `run [module]` | `r` | Full pipeline, pauses `after-draft` |
| `continue` | `c` | Resume after pause |
| `audience` | | Standalone stage (debug / iterate) |
| `plan` | `pl` | Standalone. `--migrate-sitemap` for upgrades |
| `voice` | `vc` | Standalone. `--full` for UX patterns |
| `draft` | `dr` | Standalone |
| `edit` | `ed` | Standalone. `--from-review <file>` applies feedback |
| `walkthrough` | `wt` | Verify against product. `--check`, `--apply`, `--skip-drift` |
| `record` | `rec` | Tutorial videos. `--check`, `--re-record <id>`, `--migrate-scripts` |
| `translate [module]` | `tr` | Multi-locale. `--locale`, `--per-block`, `--auto-approve` |
| `verify [glob]` | `vf` | 11-check audit. `--locale` |
| `update [module]` | `up` | Detect source change. `--auto-apply` |
| `categorize` | `cat` | Generate `_category_.json` (Docusaurus) |
| `deploy` | `dep` | Sync to target. `--dry-run`, `--target`, `--locale`, `--sync-deletes`, `--force` |
| `publish` | `pub` | Git commit/push checklist |
| `help` | `h` | Command table |

### Common patterns

```bash
# New project → deploy
/docsmith init && /docsmith module <n>
# fill intake
/docsmith run --pause-at after-draft
# review
/docsmith continue
/docsmith deploy --dry-run && /docsmith deploy

# Maintain
/docsmith update                          # source change
/docsmith walkthrough --check             # UI drift
/docsmith verify                          # health check

# Single-module iteration
/docsmith run <module> --from draft       # skip planning, redraft only
/docsmith edit <module>
/docsmith walkthrough <module>

# Multi-locale from day 1
# (set locales.targets in project.md first)
/docsmith run                             # auto-calls translate
/docsmith translate --per-block           # if you want stricter review
```

---

## See also

- [README.md](README.md) — overview + quick start
- [SETUP.md](SETUP.md) — prerequisites (browser, env vars, ffmpeg, TTS)
- [INTAKE_GUIDE.md](INTAKE_GUIDE.md) — every intake field explained
- [HOW_IT_WORKS.md](HOW_IT_WORKS.md) — operating model & internals
- [CHANGELOG.md](CHANGELOG.md) — version history
- [PUBLISHING.md](PUBLISHING.md) — publish the skill to your own marketplace
