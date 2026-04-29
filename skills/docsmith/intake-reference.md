# Intake Reference

Detailed logic for the intake system introduced in v1.5.0: project-level + module-level intake markdown forms, layered config resolution, source fetch/update, and the `run` / `continue` orchestration commands.

SKILL.md references this file. Read alongside [translate-reference.md](translate-reference.md), [deploy-reference.md](deploy-reference.md), and [update-reference.md](update-reference.md).

## Why intake exists

Before v1.5.0, every command asked the user for context interactively. This works for one-off use but doesn't scale:

- BAs who don't know YAML can't fill `.docsmithrc.yaml` confidently
- Re-running a command repeats the same questions
- No single place to see what AI knows about the project
- Hard to share context across modules without copy-paste

v1.5.0 introduces **two markdown intake files** that BAs can fill comfortably (checkboxes, fillable backtick fields), and an **AI parser** that reads them deterministically. Project intake is global; module intake is per-feature-area.

## File layout

```
documentation/
├── intake/
│   ├── project.md                    # Layer 1 — global config
│   ├── modules/
│   │   ├── instances.md              # Layer 2 — per-module override
│   │   └── storage.md
│   └── sources.lock.yaml             # Auto-managed — fetch state
├── .cache/
│   └── sources/                      # Gitignored — fetched content cache
│       ├── notion-abc123.md
│       └── github-mycloud-cloud-arch/
└── ... (drafts/, plan/, walkthrough/, etc.)
```

`project.md` is created by `init`. Module files are created by `module <name>`. The lock file is created on first `fetch` or `run`.

---

## 1. Markdown form parsing

Forms use 3 input primitives that AI parses deterministically:

### Backtick fields

```markdown
- Product slug: `mycloud`
```

Regex: `^- (?:[*\d.\s]*)?(.+?):\s*\`(.*)\`\s*$`

Extracted: key = `Product slug`, value = `mycloud`.

If the value is the placeholder default (e.g., `your-product-slug`, `Product Name Here`, `e.g., DevOps Engineer`), treat as empty.

### Checkboxes

```markdown
- [x] Docusaurus
- [ ] Standalone
```

Regex per line: `^\s*-\s*\[([ x])\]\s*(.+?)$`

In a single-select group (group of consecutive `- [ ]` lines under one section), exactly one MUST be checked. If zero or multiple checked → validation error.

In a multi-select group (e.g., target locales), zero or more may be checked.

The header preceding the group disambiguates single vs multi-select. Forms in this skill use natural-language hints: "Source language" implies single, "Target languages (translate to)" implies multi. Module/feature templates make the type explicit in template comments.

### Repeating sections

```markdown
### Source 1

- Type:
  - [ ] Notion page
  - [x] GitHub repo
- URL or path or ID: `mycloud/cloud`

### Source 2

- Type:
  - [x] Notion page
- URL or path or ID: `https://notion.so/abc123`
```

Pattern: `### Source N` (or `#### Feature N`) where N is a positive integer. AI iterates through these blocks in order, stops when it hits a section that has only empty backticks AND zero ticked checkboxes (treats as "user copy-pasted the template but didn't fill it").

### What AI does NOT parse

- Free prose between fields (treated as comments, ignored unless inside an `auto-update` region)
- HTML comments `<!-- ... -->` (treated as instructions to the BA, ignored by parser)
- Code fences (preserved verbatim if a field type is "script", e.g. pre-walkthrough setup)

### `<details>` / `<summary>` blocks (v1.5.6+)

Project and module intake templates use HTML `<details>` blocks to collapse advanced sections. Render behavior:

- GitHub / VS Code preview: `<summary>` text shown; content collapsed, expandable on click
- Plain markdown viewer: all content shown
- Cursor / GitHub.dev: collapsed by default

AI parser treats `<details>` blocks as transparent:

1. Strip `<details>` opening tag and matching `</details>` closing tag
2. Strip `<summary>...</summary>` line entirely
3. Parse content inside as if no wrapper existed

Result: BA sees ~100 lines top-level (essential sections); ~250 lines total exists in file (advanced sections collapsed). AI sees full content for parsing.

When BA expands a `<details>` block and edits a field, AI picks up the edit normally. When BA leaves it collapsed, AI uses defaults documented in the section.

Example:

```markdown
## 4. Deploy (*)

Preset:
- [x] Standalone
- [ ] Docusaurus

<details>
<summary><b>Advanced — collision behavior</b> (using defaults)</summary>

When target file exists with different content:
- [x] Warn (default)
- [ ] Skip
- [ ] Overwrite
- [ ] Prompt

</details>
```

AI parses:
- `deploy.preset = standalone` (from top section)
- `deploy.on_collision = warn` (from inside `<details>`)
- BA didn't expand the details block; default tick `Warn` is taken

## 2. Layered config resolution

When a command runs `/docsmith run instances`, AI resolves config in this order:

```
1. Defaults (from preset standalone or docusaurus)
2. Project intake (documentation/intake/project.md)
3. Module intake (documentation/intake/modules/instances.md)
4. Command-line flags (highest priority)
```

Higher layers override lower for the SAME key. Sources are CUMULATIVE (project + module sources are concatenated, not replaced).

### Override examples

```
project.md:        voice.tone = friendly-professional
module.md:         voice.tone = technical-direct
                   ↓
resolved:          voice.tone = technical-direct
```

```
project.md:        sources = [notion-prd, github-arch]
module.md:         sources = [notion-instances-prd]
                   ↓
resolved:          sources = [notion-prd, github-arch, notion-instances-prd]
```

```
project.md:        deploy.preset = docusaurus
                   deploy.target_path = ../mycloud-docusaurus
module.md:         (not specified)
                   ↓
resolved:          deploy.preset = docusaurus
                   deploy.target_path = ../mycloud-docusaurus
```

### Resolved config snapshot

For audit, AI writes the resolved config to `documentation/deployments/<ts>/resolved-config.yaml` on every `run`. This lets you reconstruct what AI saw when generating any output.

### Path mapping: how module fields become folder paths

When `module.folder = instances` (defaulting from `module.slug = instances`), this single value drives multiple paths:

| Artifact                  | Path                                                                |
| ------------------------- | ------------------------------------------------------------------- |
| Module intake             | `documentation/intake/modules/instances.md`                         |
| Source-locale drafts      | `documentation/drafts/<source-locale>/instances/<doc>.md`           |
| Translated drafts         | `documentation/drafts/<target-locale>/instances/<doc>.md`           |
| Screenshot folder         | `documentation/images/instances/<asset>.png`                        |
| Test cases                | `documentation/walkthrough/test-cases/instances/<test>.md`          |
| Drift reports             | `documentation/walkthrough/drift/<ts>/instances-*.md`               |
| Run state                 | `documentation/.run-state/instances.yaml`                           |
| Deploy target (Docusaurus)| `<deploy_target>/docs/instances/<doc>.md`                           |
| Deploy target images      | `<deploy_target>/static/img/<product.slug>/instances/<asset>.png`   |
| Deploy target i18n        | `<deploy_target>/i18n/<locale>/.../current/instances/<doc>.md`      |

**Each module produces multiple docs** (tutorial, how-to, reference, concept). The `Features to document` section in module intake drives how many. Example for `instances` module with two features:

- `Create instance` → `tutorial`, `how-to` content types → produces 2 docs
- `Auto-scaling` → `how-to`, `reference` content types → produces 2 docs

Final draft tree:
```
documentation/drafts/en/instances/
├── create-instance-tutorial.md
├── create-instance.md          (how-to)
├── auto-scaling.md             (how-to)
└── auto-scaling-reference.md
```

AI names files based on feature + content type, normalizing to kebab-case.

## 3. Validation (hybrid strictness)

Validation runs at the start of every command that consumes intake. Two levels:

### Critical fields → STOP

Missing → command halts with a list of fixes needed:

```
Cannot run /docsmith run: required intake fields missing.

Project intake (documentation/intake/project.md):
  ✗ Section 1 "Product": product slug is empty (placeholder unchanged)
  ✗ Section 4 "Deploy": no preset selected
  ✗ Section 6 "Credentials": username env var empty (required for walkthrough)

Module intake (documentation/intake/modules/instances.md):
  ✗ Section 2 "Scope": no features added (need at least one)
  ✗ Section 2 "Scope > Feature 1": no content type checked

Fix these and re-run, or run /docsmith intake-help for field reference.
```

Critical fields per intake type:

**project.md**:
- Section 1 (Product): slug, name
- Section 2 (Audience): primary persona role, technical level, primary goal
- Section 3 (Languages): source language exactly one ticked
- Section 4 (Deploy): preset exactly one ticked
- Section 6 (Credentials): username and password env vars (only if walkthrough is in pipeline)

**module.md**:
- Section 1 (Module identity): slug, display name, folder
- Section 2 (Scope): at least one feature with at least one content type ticked

### Nice-to-have fields → DEFAULT

Missing → AI applies a default and prints a note:

```
Note: voice.tone not specified in project.md. Using default: "friendly-professional".
Note: voice.perspective not specified. Using default: "second-person".
Note: reading_level not specified. Using default: "8th grade".
Note: 0 secondary personas defined. Proceeding with primary only.
```

Defaults table:

| Field                          | Default                       |
| ------------------------------ | ----------------------------- |
| `voice.tone`                   | `friendly-professional`       |
| `voice.perspective`            | `second-person`               |
| `voice.reading_level`          | `8th grade`                   |
| `voice.terms_to_avoid`         | (empty list)                  |
| `secondary_personas`           | (empty list)                  |
| `deploy.on_collision`          | `warn`                        |
| `deploy.sync_deletes`          | `false`                       |
| `auto_run.pause_at`            | `after-draft`                 |
| `auto_run.drift_action`        | `prompt`                      |
| `translate.review_mode`        | `batch`                       |
| `behavior.on_existing`         | `prompt`                      |

### `intake-help` command

```bash
/docsmith intake-help            # Show all intake fields and meanings
/docsmith intake-help <section>  # Show a specific section
```

Prints a reference of every field, what it does, validation rules, and default value. Useful when BA doesn't know what a field means.

---

## 4. External source fetching

Sources declared in intake forms are fetched by `/docsmith fetch` (manual) or automatically by `/docsmith run` / `/docsmith update`.

### Source types

| Type     | Required env var       | Auth method                             | Fetch what                          |
| -------- | ---------------------- | --------------------------------------- | ----------------------------------- |
| `notion` | `NOTION_TOKEN`         | Internal integration token              | Page content as markdown            |
| `github` | `GITHUB_TOKEN` (private) | PAT or app token (public: no auth)    | Files at paths, current commit SHA  |
| `gdrive` | `GOOGLE_DRIVE_TOKEN` or `GOOGLE_APPLICATION_CREDENTIALS` | OAuth2 token or service account | Doc content exported as markdown |
| `url`    | (none)                 | (none — public only)                    | HTML/markdown at URL                |
| `file`   | (none)                 | (none — local filesystem)               | File content                        |

### Fetch workflow

```
/docsmith fetch                  # Fetch all sources for project + all modules
/docsmith fetch --module instances
/docsmith fetch --source notion-abc123
```

For each source:
1. Look up auth env var (if needed); error if not set
2. Call API/HTTP/filesystem to fetch metadata + content
3. Compute content hash (SHA-256)
4. Write content to `.cache/sources/<source-id>.md` (or directory for multi-file sources)
5. Update entry in `sources.lock.yaml`

### Per-type fetch details

**Notion**:
- Use Notion API `pages/{id}` + `blocks/{id}/children` recursively
- Convert to markdown via standard converter (preserve headings, lists, code blocks, tables)
- Cache result; record `version` (Notion's edit version) and `last_edited_time`

**GitHub**:
- For paths with globs (`api/instances/*.ts`): use `repos/{owner}/{repo}/git/trees/{branch}?recursive=true`, filter by glob
- Fetch each matching file via `repos/{owner}/{repo}/contents/{path}`
- Cache as directory: `.cache/sources/<source-id>/<path>`
- Record `commit_sha` of latest commit on default branch (or specified branch)
- Per-file hashes for granular change detection

**Google Drive**:
- Use Drive API `files.export` with `mimeType=text/markdown` (or `text/plain` if doc has tables/images that don't export cleanly to MD)
- Record `revisionId` and `modifiedTime`

**URL**:
- HTTP GET with `User-Agent: docsmith/1.5.0`
- Save response body
- Record `ETag` and `Last-Modified` headers if present
- Content-Type matters: HTML → strip to readable text; markdown → save as-is

**File**:
- `cat` the file; record `mtime` and `size`
- No cache (read live each time it's needed)

### Auth env var resolution

Auth env vars are NEVER stored in intake or lock files. Only the env var NAME is. At fetch time:

```
1. Read source entry, get auth_env field
2. os.environ.get(auth_env) → token
3. If empty/missing → fail with clear error: "Set $NOTION_TOKEN before fetching"
```

Tokens never appear in logs, audit trails, or commits.

---

## 5. The `update` command — change detection

```bash
/docsmith update                 # Check all sources across all modules
/docsmith update <module>        # One module
```

Workflow:

1. Read `sources.lock.yaml` to get last-known state
2. For each source, do a CHEAP metadata check:
   - Notion: `GET /pages/{id}` → check `last_edited_time` vs locked
   - GitHub: `GET /repos/{owner}/{repo}/commits?path=...&per_page=1` → check latest SHA vs locked
   - GDrive: `files.get?fields=modifiedTime,headRevisionId` → check vs locked
   - URL: `HEAD` request → check `ETag` / `Last-Modified` vs locked
   - File: `stat` → check `mtime` vs locked
3. Build a change report:
   ```
   Module: instances
   
   Source changes detected:
     ~ notion://abc123 "Instances PRD"
       Version: v47 → v52 (5 edits since last sync 2 days ago)
     
     ~ github://mycloud/cloud api/instances/*.ts
       New commits: 3 (deadbeef → cafe1234)
       Files changed: create.ts, list.ts
     
     = gdrive://xyz789 "UX research notes" (no change)
     
     = file://../product-docs/feature-list.md (no change)
   
   Affected docs (will re-evaluate):
     documentation/drafts/en/instances/create.md
     documentation/drafts/en/instances/list.md
     documentation/drafts/en/instances/auto-scaling.md  (uses overall PRD context)
   
   Continue? [y/n/show-diff]
   ```
4. If user confirms:
   - Full-fetch changed sources; update cache and lock entry
   - Re-run `draft` in Update mode for affected docs (KB inheritance — read existing, propose deltas based on new source content)
   - Re-run `wt --check` to detect doc-vs-product drift on top of source-vs-doc drift
5. Print summary and what to do next

### Tracking which docs are affected by which sources

In v1.5.0 minimal: AI assumes ALL docs in a module are potentially affected by ANY source change in that module. Conservative but correct.

A future enhancement (v1.6+) is to track per-doc source provenance — "doc X was generated using sources A, B" — for surgical re-evaluation.

---

## 6. The `run` and `continue` commands

### `run` — orchestrated pipeline

```bash
/docsmith run                    # Run for all active modules
/docsmith run <module>           # One module
/docsmith run --pause-at <gate>  # Override pause_at from intake
```

Workflow:
1. Validate project + module intake (stop on critical errors)
2. Resolve config (layered)
3. Fetch sources (calls `fetch` internally; uses lock if recent enough)
4. Run pipeline stages in sequence:
   ```
   audience  →  plan  →  voice  →  draft  →  edit  →  walkthrough  →  record  →  translate
   ```
5. PAUSE at the configured gate (default: `after-draft`)
6. Save state to `documentation/.run-state.yaml`:
   ```yaml
   started_at: 2026-04-26T15:30:00Z
   module: instances
   completed_stages: [audience, plan, voice, draft, edit]
   paused_at: after-draft
   next_stage: walkthrough
   resolved_config_path: documentation/deployments/2026-04-26-153000/resolved-config.yaml
   ```
7. Print resume instructions: `Run /docsmith continue when ready`

### `continue` — resume after gate

```bash
/docsmith continue
/docsmith continue <module>
```

Workflow:
1. Read `.run-state.yaml`
2. If state stale (>7 days old): warn user, suggest re-validation before continuing
3. Resume from `next_stage`
4. Run remaining stages until next pause or end of pipeline
5. Update state on completion or new pause

### Pause gates

| Gate              | When pause occurs                      | What user does                          |
| ----------------- | -------------------------------------- | --------------------------------------- |
| `after-plan`      | After `plan` writes plan + sitemap     | Review plan, edit if needed             |
| `after-draft`     | After `edit` finishes                  | Review drafts, edit; default            |
| `before-walkthrough` | Before browser opens                | Confirm test account ready              |
| `after-walkthrough` | After capture, before record/translate | Review captured screenshots             |
| `before-deploy`   | After translate, before deploy plan    | Review final state                      |
| `never`           | Never pause                            | Full auto; risky for first runs         |

User can also Ctrl-C any time; `run` saves state and prints resume instructions.

### State invalidation

If user runs an individual command (e.g. `/docsmith draft instances`) outside of `run`, state may go stale. State file tracks last-modified time; `continue` checks if any draft has been modified after `state.completed_stages` includes `draft` — if yes, suggests re-running from `walkthrough` to verify.

---

## 7. The `module` command

```bash
/docsmith module <name>          # Create a new module intake
/docsmith module <name> --from <existing>  # Clone from existing module
/docsmith module list            # List modules and their status
/docsmith module archive <name>  # Mark module as archived
```

`/docsmith module instances` does:
1. Check `documentation/intake/modules/instances.md` doesn't already exist (re-run protocol if it does)
2. Copy `templates/MODULE_INTAKE_TEMPLATE.md` to that path
3. Pre-fill the module name and slug from the command argument
4. Update project.md `Module intake files` section with the new entry
5. Print: "Module 'instances' created. Edit documentation/intake/modules/instances.md and run /docsmith run instances when ready."

`--from <existing>` copies field values from another module file (useful when modules share structure: same sources, same voice override).

`module list` reads the project.md modules list and prints status:
```
Modules registered (3):
  ✓ instances     active     last run 2026-04-25 (2 days ago)
  ⏸ storage       paused     last run 2026-03-15 (40 days ago)
  ✗ networking    archived   last run 2026-01-10 (deploy --sync-deletes will not delete)
```

`module archive <name>` flips the module's "Module status" checkbox to Archived in its intake file. Archived modules:
- Skipped by `/docsmith run` (no implicit processing)
- Not deleted from target on `/docsmith deploy --sync-deletes` (orphan filter excludes them)
- Source entries kept in `sources.lock.yaml` for reference

---

## 8. Validation field reference (concrete)

Critical (CRT) and nice-to-have (NTH) fields:

### project.md

| Section | Field                         | Critical | Default if missing             |
| ------- | ----------------------------- | -------- | ------------------------------ |
| 1       | product.slug                  | CRT      | (stop)                         |
| 1       | product.name                  | CRT      | (stop)                         |
| 1       | product.url                   | NTH      | empty (walkthrough won't auto-login if also empty) |
| 2       | audience.primary.role         | CRT      | (stop)                         |
| 2       | audience.primary.tech_level   | CRT      | (stop)                         |
| 2       | audience.primary.goal         | CRT      | (stop)                         |
| 2       | audience.secondary            | NTH      | (empty list)                   |
| 3       | locales.source                | CRT      | (stop)                         |
| 3       | locales.targets               | NTH      | (empty)                        |
| 4       | deploy.preset                 | CRT      | (stop)                         |
| 4       | deploy.target_path            | CRT (if Docusaurus) | (stop)              |
| 4       | deploy.on_collision           | NTH      | `warn`                         |
| 5       | voice.tone                    | NTH      | `friendly-professional`        |
| 5       | voice.perspective             | NTH      | `second-person`                |
| 5       | voice.reading_level           | NTH      | `8th grade`                    |
| 5       | voice.terms_to_avoid          | NTH      | (empty)                        |
| 6       | credentials.username_env      | CRT (if walkthrough in pipeline) | (stop)        |
| 6       | credentials.password_env      | CRT (if walkthrough in pipeline) | (stop)        |
| 7       | sources                       | NTH      | (empty — AI works from intake alone) |
| 8       | auto_run.pause_at             | NTH      | `after-draft`                  |
| 8       | auto_run.drift_action         | NTH      | `prompt`                       |
| 8       | translate.review_mode         | NTH      | `batch`                        |

### modules/<n>.md

| Section | Field                    | Critical | Default if missing             |
| ------- | ------------------------ | -------- | ------------------------------ |
| 1       | module.slug              | CRT      | (stop)                         |
| 1       | module.display_name      | CRT      | (stop)                         |
| 1       | module.priority          | NTH      | `3`                            |
| 1       | module.folder            | CRT      | (default to module.slug)       |
| 2       | features                 | CRT (≥1) | (stop)                         |
| 2       | features[N].name         | CRT      | (stop)                         |
| 2       | features[N].content_types | CRT (≥1 ticked) | (stop)                  |
| 2       | out_of_scope             | NTH      | (empty)                        |
| 3       | voice override           | NTH      | (inherit from project)         |
| 4       | sources                  | NTH      | (empty — adds to project sources) |
| 5       | walkthrough.test_account | NTH      | (inherit)                      |
| 5       | walkthrough.setup_script | NTH      | (none)                         |
| 6       | redact.fields            | NTH      | (none)                         |
| 6       | module.status            | NTH      | `Active`                       |

---

## 9. Migration from v1.4.x

If you have an existing v1.4.x workspace:

1. Run `/docsmith init --upgrade-from-1.4` (new flag in v1.5.0)
2. AI:
   - Reads existing `.docsmithrc.yaml`
   - Generates `documentation/intake/project.md` pre-filled with values from yaml
   - Keeps `.docsmithrc.yaml` for backward-compat reading (deprecated, will be removed in v1.6)
3. Manually create modules: `/docsmith module <existing-feature-area>` for each feature in your old plan
4. Edit each module file to refine scope
5. Run `/docsmith run` to verify everything resolves correctly

If you have NO existing workspace, just run `/docsmith init` normally.

---

## 10. Known limitations (v1.5.0)

1. **No conditional field display** — all sections always shown. BA fills 50 fields even if 30 don't apply. Future: `--minimal` template variant for simple projects.
2. **Source fetching is synchronous** — large repos can be slow. No parallel fetching in 1.5.0. Future: parallel fetcher with progress bar.
3. **Notion conversion** uses standard markdown — complex Notion features (databases, embeds) become inline references, not first-class. Future: better databases handling.
4. **Per-doc source provenance not tracked** — `update` re-evaluates ALL docs in a module when ANY source changes. Conservative. Future: track which doc used which source for surgical updates.
5. **No incremental fetch on cache hit** — `fetch` always re-downloads even if cache is fresh. Future: respect `Cache-Control` / `ETag`.
6. **GDrive token refresh not handled** — long-lived OAuth2 tokens may expire. User must refresh manually. Future: refresh flow integrated.
7. **Source content cache is gitignored** — meaning across machines, every dev re-fetches. Future: optional shared cache via S3 or similar.
