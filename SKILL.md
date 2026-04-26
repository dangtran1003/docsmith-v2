---
name: docsmith
description: "Build documentation from filled markdown intake forms. AI fetches knowledge from Notion/GitHub/GDrive/URL/files, drafts content, captures screenshots via browser walkthrough, translates to multiple locales, and deploys to Docusaurus. Layered intake (project + module). Re-run safe; source change detection via lock file. Commands: init, module, fetch, run, continue, audience, plan, voice, draft, edit, walkthrough, record, translate, verify, update, categorize, deploy, publish, intake-help, help."
---

# docsmith

Documentation skill following the PRC-010 process. Standalone-first workspace; deploys to Docusaurus or other targets via preset. v1.5.0 introduces a **layered intake system** so BAs can configure docs by filling markdown forms (no YAML) — and an `update` command that detects when external knowledge sources change.

Reading order:

- This file (catalog of commands and high-level rules)
- [intake-reference.md](intake-reference.md) — intake forms, layered config, `run`/`continue`/`update`/`fetch`/`module` commands, source fetching
- [process-reference.md](process-reference.md) — PRC-010 process detail
- [deploy-reference.md](deploy-reference.md) — deploy and categorize logic
- [translate-reference.md](translate-reference.md) — translation block-level rules and review modes
- [tools-reference.md](tools-reference.md) — browser automation reference

## Commands

20 commands. Most users typically run only `init`, `module`, `run`, and `deploy` for the happy path.

| Command         | Alias | Owner  | Purpose                                                                                                |
| --------------- | ----- | ------ | ------------------------------------------------------------------------------------------------------ |
| `help`          | `h`   | —      | Show this command reference                                                                            |
| `init`          | `i`   | **AI** | Initialize workspace: scaffold `documentation/intake/project.md`, folders, gitignore                    |
| `module`        | `m`   | **AI** | Create / list / archive a module intake at `documentation/intake/modules/<n>.md`                       |
| `intake-help`   | `ih`  | —      | Print intake field reference (what each section/field means, defaults)                                 |
| `fetch`         | `f`   | **AI** | Fetch external sources (Notion/GitHub/GDrive/URL/file). Updates cache + `sources.lock.yaml`.          |
| `run`           | `r`   | **AI** | Orchestrated pipeline: validate intake → fetch → audience → plan → voice → draft → edit → wt → record → translate. Pauses at the configured gate. |
| `continue`      | `c`   | **AI** | Resume `run` after a gate (read `.run-state.yaml`)                                                     |
| `audience`      | —     | **AI** | Generate audience profile from intake (use intake `Audience` section)                                  |
| `plan`          | `pl`  | **AI** | Generate documentation plan + sitemap from intake `Scope`                                              |
| `voice`         | `vc`  | **AI** | Generate voice chart from intake `Voice`. Quick mode by default; `--full` adds UX patterns + scorecard.|
| `draft`         | `dr`  | **AI** | Draft docs in `documentation/drafts/<source-locale>/`. Uses fetched sources as KB.                     |
| `edit`          | `ed`  | **AI** | Self-review (5 passes). With `--from-review`, applies feedback file.                                   |
| `walkthrough`   | `wt`  | **AI** | Verify against live product + capture screenshots. Phases: VERIFY → gate → APPLY → CAPTURE.            |
| `record`        | `rec` | **AI** | Record short tutorial videos from `<!-- VIDEO ... -->` markers (browser screen recording)              |
| `translate`     | `tr`  | **AI** | Translate from source locale to each `locales.targets`. Batch review by default; `--per-block` opt-in. |
| `verify`        | `vf`  | **AI** | 10-check audit (placeholders, voice score, links, glossary consistency, etc.)                          |
| `update`        | `up`  | **AI** | Detect changed external sources; propose draft updates                                                 |
| `categorize`    | `cat` | **AI** | Generate `_category_.json` from sitemap (Docusaurus preset)                                            |
| `deploy`        | `dep` | **AI** | Sync workspace to host project. `--dry-run`, `--target`, `--force`, `--locale`, `--sync-deletes`.      |
| `publish`       | `pub` | Human  | Final checklist: review deploy, `git commit && push` on target                                         |

### Examples

```
/docsmith help
/docsmith init                                       # one-time setup
/docsmith module instances                           # add a module
/docsmith intake-help                                # browse intake field reference

# Edit documentation/intake/project.md and modules/instances.md, then:
/docsmith run                                        # full pipeline, pauses at after-draft
/docsmith continue                                   # resume after review
/docsmith deploy --dry-run
/docsmith deploy

# Update flow when sources change:
/docsmith update                                     # check all modules
/docsmith update instances                           # one module
```

## Process flow

```
init (once) → module <n> (per module) → fill intakes → run → [pause: review] → continue → deploy → publish
                                                   │
                                                   └─ Underlying stages (auto-chained by run):
                                                      audience → plan → voice → draft → edit
                                                      → walkthrough → [record] → translate → categorize
```

For multi-locale projects, `translate` runs as part of the pipeline and is required before `deploy`. Deploy with empty target locale drafts emits a warning.

For maintenance: `/docsmith update` detects external source changes and proposes targeted re-drafts.

## Configuration

v1.5.0 splits config into 3 layers:

1. **Defaults** — from preset (`standalone` or `docusaurus`)
2. **Project intake** — `documentation/intake/project.md` ([template](templates/PROJECT_INTAKE_TEMPLATE.md))
3. **Module intake** — `documentation/intake/modules/<n>.md` ([template](templates/MODULE_INTAKE_TEMPLATE.md))

Higher layers override lower for the same field. Sources are cumulative.

Intake forms are markdown with checkboxes and backtick fields — no YAML to write. AI parses them deterministically. See [intake-reference.md](intake-reference.md) for parsing rules.

A legacy `.docsmithrc.yaml` is still read for backward compat with v1.4.x but deprecated; new projects use intake forms only.

## Path scoping (safety)

Every command validates that absolute write paths fall inside one of:

- `documentation/` (workspace)
- `deploy.target_path` from project intake (or `--target` override)
- `deployments/` (audit trail)

Writes outside these roots are rejected. This is the core safety guarantee.

## Re-run protocol (safety)

Every command that produces output checks for existence first. If output exists:

1. **Update** (recommended for content) — read existing as KB, propose deltas, apply per item. Untouched content preserved verbatim.
2. **Overwrite** — discard existing (archived to `archive/<ts>/`), generate fresh. Requires explicit "OVERWRITE" confirmation.
3. **Side-by-side** — keep both, new file gets suffix `-v2` / `-v3`.
4. **Cancel** — abort.

Default: `prompt` (always ask). Configurable via intake `Auto-run behavior` or legacy `behavior.on_existing`.

**Critical rule for Update mode**: read existing as canonical KB and propose deltas only (NEW / UPDATE / REMOVE / KEEP). Do NOT regenerate untouched sections — that loses team's manual edits.

## Command details

### `help`

Print the command table above. No other action.

### `init` (AI)

**Purpose**: scaffold a docsmith workspace at the current directory. One-time setup.

**Behavior**:

1. Detect host project context: `docusaurus.config.{js,ts,mjs}` present → suggest `docusaurus` preset; otherwise `standalone`
2. Inspect target if Docusaurus: read `<target>/CLAUDE.md`, parse `docusaurus.config` for paths
3. Create directory structure (see § File organization below)
4. Pre-fill `project.md` with detected values (product slug from package.json or directory name; deploy target from inspected Docusaurus path; etc.)
5. Print: "Workspace scaffolded. Edit documentation/intake/project.md, then run `/docsmith module <n>` for each feature area."

**Flags**:
- `--upgrade-from-1.4` — read existing `.docsmithrc.yaml` and pre-fill `project.md` from it; keep yaml for compat (deprecated)
- `--force` — overwrite existing intake files (requires confirm)

### `module` (AI)

**Purpose**: manage module intake files.

**Sub-commands**:

```
/docsmith module <n>                       # create new module
/docsmith module <n> --from <existing>     # clone from another module
/docsmith module list                      # list modules with status
/docsmith module archive <n>               # mark as archived (skip in run)
/docsmith module unarchive <n>             # un-archive
```

**Create behavior**:
1. Re-run protocol check on `documentation/intake/modules/<n>.md`
2. Copy `templates/MODULE_INTAKE_TEMPLATE.md` to that path
3. Pre-fill module slug, display name from argument
4. If `--from <existing>`: copy non-identity fields from `<existing>` (sources, voice override, etc.)
5. Update project.md `Module intake files` section (between BEGIN/END MODULES LIST markers)
6. Print: "Module '<n>' created. Edit and run `/docsmith run <n>` when ready."

### `intake-help` (—)

Print intake field reference. No file changes.

```
/docsmith intake-help                # all fields
/docsmith intake-help product        # one section
/docsmith intake-help sources        # source types and auth
```

For each field: name, what it controls, validation rules, default value.

### `fetch` (AI)

**Purpose**: pull content from external knowledge sources declared in intakes. Manual; usually called automatically by `run` and `update`.

**Flags**:
- (no args) — fetch all sources for project + all active modules
- `--module <n>` — only this module
- `--source <id>` — only this source

**Per-type behavior**: see [intake-reference.md](intake-reference.md) § "External source fetching".

After fetch: writes content to `documentation/.cache/sources/<source-id>.{md,dir}`, updates `sources.lock.yaml`.

### `run` (AI) — orchestrated pipeline

**Requires**: filled project + module intake; env vars set for sources/credentials in scope.

**Workflow**:

1. Validate intakes (see [intake-reference.md](intake-reference.md) § Validation). Stop on critical errors.
2. Resolve layered config; snapshot to `deployments/<ts>/resolved-config.yaml`.
3. Call `fetch` internally (uses lock file if recent).
4. Run pipeline stages in sequence: `audience` → `plan` → `voice` → `draft` → `edit` → `walkthrough` → `record` → `translate`.
5. Pause at the configured gate (default: `after-draft`).
6. Save state to `documentation/.run-state.yaml`; print resume instructions.

**Flags**:
- `[<module>]` — single module instead of all
- `--pause-at <gate>` — override the configured pause point
- `--from <stage>` — start from a specific stage (e.g. `--from walkthrough` to skip planning)

**Pause gates**: `after-plan`, `after-draft` (default), `before-walkthrough`, `after-walkthrough`, `before-deploy`, `never`.

### `continue` (AI)

Resume from `.run-state.yaml`. If state stale (>7 days) or workspace modified outside docsmith, warn before resuming.

### `audience` / `plan` / `voice` / `draft` / `edit`

Individual pipeline stages. Each:

- Reads resolved config from intake
- Re-run protocol applies (if output exists, gate)
- Writes to its conventional output path
- Returns control (does NOT chain to next stage; that's `run`'s job)

Specific behaviors:

**`audience`** — output: `documentation/plan/audience-profile.md` from [templates/AUDIENCE_PROFILE_TEMPLATE.md](templates/AUDIENCE_PROFILE_TEMPLATE.md). Uses intake `Audience` section.

**`plan`** — outputs: `documentation/plan/documentation-plan.md` (from [templates/DOCUMENTATION_PLAN_TEMPLATE.md](templates/DOCUMENTATION_PLAN_TEMPLATE.md)) AND `documentation/plan/sitemap.md`. Uses intake `Scope` (from each module).

**`voice`** — outputs: `documentation/standards/voice-chart.md` from [templates/VOICE_CHART_TEMPLATE.md](templates/VOICE_CHART_TEMPLATE.md). With `--full`, also generates UX text patterns and content scorecard (these were standard in v1.4.x; now opt-in for lean default).

**`draft`** — outputs: `documentation/drafts/<source-locale>/<feature>/<doc>.md`. Reads cached sources from `.cache/sources/`. Uses [templates/CONTENT_TYPE_TEMPLATES.md](templates/CONTENT_TYPE_TEMPLATES.md). Image refs: workspace-absolute paths `/images/<feature>/<asset>.png`. Caption rules per [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md). Video markers per [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md).

**`edit`** — five passes: voice match, UX patterns, clarity, accuracy markers, link/cross-ref. With `--from-review <file>`, applies a reviewer's feedback file (Markdown with `// FEEDBACK:` annotations or block comments).

### `walkthrough` (AI)

Three-phase pipeline:

```
A: VERIFY    /docsmith wt --check         → drift report (read-only)
   ↓
   GATE      Edit decisions.yaml          (auto-fix / manual-fix / product-bug / skip)
   ↓
B: APPLY     /docsmith wt --apply         → update drafts per decisions
   ↓
C: CAPTURE                                 → screenshots, replace placeholders
```

**Default mode** (no flag): runs A → interactive gate → B → C.

**Flags**:
- `--check` — Phase A only; fast (~30s); for drift monitoring
- `--apply` — B + C with existing decisions
- `--skip-drift` — C only; for first runs or known-clean drafts
- `--auto-apply-high-confidence` — skip gate, auto-apply HIGH-confidence fixes
- `--locale <locale>` — verify against a translated locale (requires product UI in that locale)

Drift report format: [templates/DRIFT_REPORT_TEMPLATE.md](templates/DRIFT_REPORT_TEMPLATE.md). Test cases: [templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md](templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md).

Items marked `product-bug` tracked in `walkthrough/active-product-bugs.yaml` across runs; auto-resolved when UI matches doc.

### `record` (AI)

Optional. Scans drafts for `<!-- VIDEO ... -->` markers and records short tutorial videos via browser screen recording. See [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md) for marker syntax and [templates/WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md](templates/WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md) for capture plan.

After recording: replaces marker with `<video>` embed, preserves marker as comment for re-record.

### `translate` (AI)

**Required** when `locales.targets` non-empty. Position: after `edit`, before `deploy`.

**Default review mode**: `batch` (whole-file diff review). User can opt into `--per-block` for safer per-block review.

**Block-level rules** and full workflow: [translate-reference.md](translate-reference.md).

Glossary file (per locale): `documentation/standards/glossary.<locale>.yaml` ([templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml)). Optional but recommended.

**Flags**:
- `[<module>]` — single module
- `--locale <locale>` — single target locale
- `--per-block` — per-block review gate (slower, safer)
- `--auto-approve` — skip review entirely (CI/bulk only)
- `--glob <pattern>` — only matching files

### `verify` (AI)

10 checks across drafts:
1. No `placehold.co` URLs remain
2. No broken image refs
3. Voice consistency score ≥ threshold
4. Glossary consistency (translated docs)
5. Code blocks unchanged across translations
6. Frontmatter complete
7. Links resolve (internal)
8. Cross-references valid
9. Sitemap entries match draft files
10. No orphan test cases

**Flags**:
- `[<doc-glob>]` — scope to specific docs (faster iteration)
- `--locale <locale>` — verify a target locale's drafts

### `update` (AI)

**Purpose**: detect external source changes and propose draft updates without re-fetching everything.

**Workflow**:
1. Read `sources.lock.yaml`
2. For each source, do cheap metadata check (Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime)
3. Build change report
4. User reviews; on confirm:
   - Full-fetch changed sources
   - Re-run `draft` in Update mode for affected docs (KB inheritance)
   - Re-run `wt --check` for new drift
5. Update lock file

See [intake-reference.md](intake-reference.md) § "The `update` command — change detection".

**Flags**:
- (no args) — check all modules
- `<module>` — one module
- `--auto-apply` — fetch and apply changes without prompt (CI use)

### `categorize` (AI) — Docusaurus categories

Generate `_category_.json` from sitemap. Normalize titles (acronyms uppercase: API, CLI, JSON, etc.; articles lowercase mid-title; first/last word capitalized).

Auto-runs on `deploy` if `generate_categories: true` (Docusaurus preset default). Can run standalone for re-categorization.

See [deploy-reference.md](deploy-reference.md) § "Categorize subcommand" and [templates/CATEGORY_FILE_TEMPLATE.md](templates/CATEGORY_FILE_TEMPLATE.md).

### `deploy` (AI)

Copy/sync workspace to host project with transforms (frontmatter injection, image namespacing `/img/<slug>/`, MDX escape, category generation).

**Flags**:
- `--dry-run` — detect + plan, exit without writes
- `--target <path>` — override `deploy.target_path` for this run
- `--force` — override conflicts
- `--locale <locale>` — single locale
- `--sync-deletes` — propagate workspace deletions to target (default: report orphans only)

**Workflow** (full detail: [deploy-reference.md](deploy-reference.md)):
1. Translation completeness check (warn if `locales.targets` has incomplete translations)
2. Detect target context (CLAUDE.md, docusaurus.config.*, folder signals)
3. Plan file actions (create / update / skip / conflict / delete-if-sync)
4. Show plan; if `--dry-run`, exit
5. Apply if no unresolved conflicts; create audit folder under `deployments/`
6. Save manifest, target-config snapshot, diff, pre-deploy hashes

### `publish` (Human)

Final checklist:
1. Review the deploy summary and target diff
2. Coordinate with code/product release if applicable
3. In target project: `git diff` → `git add` → `git commit` → `git push`
4. Trigger documentation platform build if not auto-triggered
5. Verify live site
6. Announce

AI does NOT auto-commit. Git on target is the safety net.

## File organization

```
<project-root>/
├── documentation/
│   ├── intake/
│   │   ├── project.md                  # Layer 1
│   │   ├── modules/
│   │   │   ├── instances.md            # Layer 2
│   │   │   └── storage.md
│   │   └── sources.lock.yaml           # Auto-managed
│   ├── plan/
│   │   ├── audience-profile.md
│   │   ├── documentation-plan.md
│   │   └── sitemap.md
│   ├── standards/
│   │   ├── voice-chart.md
│   │   ├── screenshot-policy.md
│   │   ├── glossary.vi.yaml            # Optional, per locale
│   │   └── glossary.jp.yaml
│   ├── drafts/
│   │   ├── en/                         # Source locale
│   │   │   └── instances/create.md
│   │   └── vi/                         # Translated by `translate`
│   │       └── instances/create.md
│   ├── walkthrough/
│   │   ├── test-cases/
│   │   ├── video-plan/
│   │   ├── executions/
│   │   ├── drift/<ts>/
│   │   └── active-product-bugs.yaml
│   ├── archive/<ts>/                   # Re-run protocol backups
│   ├── images/
│   │   └── instances/create-form-filled.png
│   ├── videos/
│   │   ├── raw/                        # Gitignored
│   │   └── instance-create-tour.mp4
│   ├── .cache/
│   │   └── sources/                    # Gitignored
│   │       ├── notion-abc123.md
│   │       └── github-mycloud-cloud-arch/
│   └── .run-state.yaml                 # Run/continue state
└── deployments/<ts>-<target>/
    ├── manifest.yaml
    ├── target-config.yaml
    ├── diff.md
    ├── resolved-config.yaml
    ├── pre-deploy-state.txt
    └── deleted/                        # Backup of removed target files (--sync-deletes)
```

Standalone preset: workspace IS the publishable artifact, no target.

In-place mode: `deploy.target_path = .` works inside the same project as workspace.
