# How docsmith works

The full operating model of docsmith — every command, every file, the lifecycle from intake through deploy. Read this when you need to understand WHY the skill is structured as it is, not just HOW to run a command.

> 🇻🇳 Bản tiếng Việt: [HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md)

> Last updated for v1.5.14. For just running commands, see [README.md](README.md). For filling intake forms, see [INTAKE_GUIDE.md](INTAKE_GUIDE.md).

## TL;DR

You fill out 2 markdown forms. AI generates docs through a 7-stage pipeline (audience → plan → voice → draft → edit → walkthrough → record → translate). Optional: deploy to Docusaurus. Everything is reproducible: re-running any stage detects existing output and either updates incrementally (preserving your edits) or asks before overwriting.

The skill is **prompt-based, not code-based**. Bug fixes are prose edits to SKILL.md, not code patches.

---

## 1. The two-layer config model

docsmith reads configuration from two markdown intake files (since v1.5.0):

```
documentation/intake/
├── project.md                  # Layer 1 — applies to ALL modules
└── modules/
    ├── instances.md            # Layer 2 — per-module override
    ├── storage.md
    └── billing.md
```

**Project intake** (`project.md`) covers:
- Product identity (slug, name, URL, description)
- Audience (personas, technical level)
- Languages (source + targets)
- Deploy preset (standalone or Docusaurus + path)
- Voice and tone (defaults work for ~80% of cases)
- Knowledge sources (Notion / GitHub / GDrive / URL / file)
- Credentials env vars (for walkthrough)
- Auto-run behavior (where to pause for review)
- Sitemap pattern (Pattern A / B / C — added v1.5.4)
- Media policy (screenshot density, voiceover strategy, TTS provider — added v1.5.5)

**Module intake** (`modules/<name>.md`) covers:
- Module identity (slug, display name, folder)
- Scope (features, content types per feature)
- Per-module overrides (voice, sources, credentials, sitemap sections, media)

**Resolution order** when AI runs:
1. Module intake fields (if set)
2. Fall back to project intake
3. Fall back to template default

Sources are CUMULATIVE: project sources + module sources both used.

### Why two layers (and not just one big intake or one per module)?

- One big intake → repeated config per module, drift over time
- One per module → no place for project-wide settings (deploy target, locales, audience)
- Two layers → DRY at project level, override at module level when needed

### `.docsmithrc.yaml` is deprecated

v1.4.x used a YAML config. v1.5.0+ uses markdown intake forms instead. The YAML still parses for backward compat (with `init --upgrade-from-1.4`) but will be removed in v1.6+.

---

## 2. The 20 commands and where they live in the pipeline

```
init ─→ module ─→ [fill intake] ─→ run ──┬─→ continue ─→ deploy
                                          │
                                          ▼
                                    intake-help
                                    fetch
                                    (any individual stage)
```

Sequential pipeline (auto-chained by `run`):

| # | Command | What it does |
|---|---|---|
| 1 | `audience` | Generate audience profile from intake |
| 2 | `plan` | Generate documentation plan + sitemap |
| 3 | `voice` | Generate voice chart |
| 4 | `draft` | Draft docs in `documentation/drafts/<source>/` |
| 5 | `edit` | 5-pass self-review of drafts |
| 6 | `walkthrough` (alias `wt`) | Verify against live product, capture screenshots |
| 7 | `record` (alias `rec`) | Record short tutorial videos (silent or voiced) |
| 8 | `translate` (alias `tr`) | Translate drafts + scripts to target locales |

Setup / lifecycle commands:

| Command | Purpose |
|---|---|
| `init` | Scaffold workspace + project intake |
| `module` | Create / list / archive a module intake |
| `intake-help` | Print intake field reference |
| `fetch` | Pull external sources into local cache |
| `run` | Orchestrated pipeline; pauses at configured gate |
| `continue` | Resume `run` after pause |
| `update` | Detect source/module changes; propose updates |
| `verify` (alias `vf`) | 11-check audit |
| `categorize` (alias `cat`) | Generate `_category_.json` for Docusaurus |
| `deploy` (alias `dep`) | Sync workspace to target |
| `publish` (alias `pub`) | Final checklist (human step — git commit + push) |

Most users only run: `init`, `module`, `run`, `deploy`. Everything else is for fine-grained control or recovery.

---

## 3. What `/docsmith init` does

`init` runs once per project. It:

1. **Detect host context**:
   - `docusaurus.config.{js,ts,mjs}` present → suggest `docusaurus` preset (with in-place option)
   - `package.json` present → confirm intent before scaffolding
   - Empty directory → fine
   - Non-empty unrelated content → STOP, ask user to confirm or move

2. **Create directory structure**:
   ```
   documentation/
   ├── intake/
   │   ├── project.md              # the form you fill
   │   ├── modules/                # empty initially
   │   └── sources.lock.yaml       # auto-managed, empty initially
   ├── plan/                       # populated by `plan` command
   ├── standards/                  # populated by `voice` and `translate`
   ├── drafts/                     # populated by `draft`
   ├── walkthrough/                # populated by `wt`
   ├── images/                     # populated by `wt`
   ├── scripts/                    # v1.5.7+: populated by `record`
   ├── videos/                     # populated by `rec`
   ├── deployments/                # populated by `deploy`
   ├── archive/                    # backups from re-run protocol
   ├── .cache/sources/             # gitignored, fetched source content
   └── .run-state/                 # gitignored, per-module run state
   ```

3. **Append to `.gitignore`** (at project root, idempotent block between `# BEGIN docsmith` and `# END docsmith` markers):
   ```
   documentation/.cache/
   documentation/videos/raw/
   documentation/.run-state/
   ```

4. **Pre-fill `project.md`** with detected values (slug from package.json, deploy target from inspected Docusaurus path, etc.)

5. **Print next-step hint**: "Edit documentation/intake/project.md, then run `/docsmith module <n>` for each feature area."

### Variants

- `init --in-place` — explicitly request in-place mode (when running inside a Docusaurus repo)
- `init --upgrade-from-1.4` — read `.docsmithrc.yaml` and pre-fill `project.md` (deprecated path)
- `init --reformat-intake` — re-render existing intake with current template (preserves filled values)
- `init --from-source <path-or-url>` (v1.5.9+) — AI auto-fills intake from BA doc / PRD / existing docs
- `init --resume` (v1.5.10+) — retry parallel sub-agents from last partial-completion run

---

## 4. The intake form workflow

### Manual fill (traditional)

```bash
/docsmith init                   # scaffolds project.md
/docsmith module instances       # scaffolds modules/instances.md
# (open project.md, fill in checkboxes and backtick fields)
# (open modules/instances.md, fill scope)
/docsmith run                    # AI uses intake to drive pipeline
```

Each field has an inline `>` hint explaining what to put. For first project, fill ONLY:
- project.md sections 1-6 (Product, Audience, Languages, Deploy, Credentials, Source 1)
- modules/<n>.md sections 1-2 (Identity, Scope)

Skip Advanced sections (collapsed by default since v1.5.6) — defaults work.

### AI auto-fill from source (v1.5.9+)

```bash
/docsmith init --from-source documentation/sources/ba-doc.md
# AI reads BA doc, infers fields, asks 5-10 questions, writes project.md
# Marks uncertain fields with "← AI guess" for you to verify

/docsmith module instances --from-source documentation/sources/ba-doc.md
# AI extracts features for "instances" module from BA doc
```

Source can be:
- Local file (markdown, plain text)
- Notion URL or page ID
- GitHub `owner/repo` or specific file path
- Google Drive file ID
- Public URL

Multiple sources comma-separated.

AI categorizes inferences:
- **Fact** — direct quote from source (no marker)
- **Guess** — inferred from context (`← AI guess, please verify`)
- **Default** — no source data (`← default applied`)
- **Asked** — answered during interactive Q&A

Inference report at `documentation/intake/.inference/<ts>-<scope>.md` documents every inference with source quote and reasoning.

### Re-run with `--from-source`

When BA doc updates:

```bash
/docsmith init --from-source path/to/ba-doc.md
```

AI:
1. Re-fetches source
2. Per-field hash check: BA-edited fields preserved; AI-inferred fields can be re-inferred
3. Shows diff before applying
4. Re-run protocol gate (Update / Overwrite / Skip)

---

## 5. The pipeline (`/docsmith run`)

`run` orchestrates all 8 stages. Default pause gate: **after draft** (review drafts before walkthrough captures screenshots from live product).

```
[1] audience
       │
       ▼
[2] plan ─→ documentation/plan/{audience-profile.md, documentation-plan.md, sitemap.md}
       │
       ▼
[3] voice ─→ documentation/standards/voice-chart.md
       │
       ▼
[4] draft ─→ documentation/drafts/<source-locale>/<module>/<doc>.md
       │
       ▼
[5] edit (5-pass self-review)
       │
       ▼
   ┌─── PAUSE GATE: review drafts ───┐
   │                                  │
   │  /docsmith continue             │
   │                                  │
   ▼                                  ▼
[6] walkthrough ─→ images/<module>/   ─→ drift reports
       │
       ▼
[7] record ─→ scripts/<module>/<id>.md (v1.5.7+)
              voiceover/<id>.<locale>.mp3
              videos/<id>.mp4
              subtitles/<id>.<locale>.vtt
       │
       ▼
[8] translate ─→ drafts/<target-locale>/<module>/<doc>.md
                 scripts/<module>/<id>.md (## <locale> sections)
       │
       ▼
   verify (optional, recommended before deploy)
       │
       ▼
   deploy
```

### Pause gates (set in project intake § 6 advanced)

- **After plan** (before draft) — review documentation plan first
- **After draft** (before walkthrough) — DEFAULT, safest for first run
- **Before deploy** — let everything generate, only review deploy plan
- **Never** — full auto, only for trusted re-runs

### What each stage does

#### Audience
Generates `documentation/plan/audience-profile.md` from intake — narrative of personas, technical level, primary goals, motivations.

#### Plan
Generates:
- `documentation/plan/documentation-plan.md` — module breakdown, doc list per module
- `documentation/plan/sitemap.md` — full nav structure following project pattern (A/B/C)

`plan` checks each module against project sitemap pattern; warns if module is missing a section the pattern includes.

#### Voice
Generates `documentation/standards/voice-chart.md` — concrete examples of how the chosen tone/perspective sounds in this domain. Used as reference during draft.

#### Draft
Drafts each doc in `documentation/drafts/<source-locale>/<module>/`. Uses:
- Intake fields (audience, scope, voice)
- Sources (Notion / GitHub / etc fetched via `fetch`)
- Voice chart for tone consistency

In Update mode (existing draft present), reads current draft as KB and proposes deltas only — your manual edits preserved.

#### Edit
5-pass self-review of each draft:
1. Voice consistency
2. Reading level match
3. Cross-references between docs
4. Glossary consistency (within source locale)
5. Code block/example correctness

#### Walkthrough
Three-phase pipeline (v1.3.0+):

1. **VERIFY**: read drafts, generate test cases, dry-run against product UI structure
2. **DRIFT GATE**: compare drafts vs live product UI; produce drift report; user decides per item (auto-fix doc / mark as product bug / skip)
3. **APPLY + CAPTURE**: apply user decisions, capture screenshots into `documentation/images/<module>/`

Per-locale strategy from media policy (default: source-only — capture once in source UI, reuse for translated docs).

#### Record (optional)
Scans drafts for `<!-- VIDEO id: <id> -->` markers (v1.5.7+ simplified syntax). For each marker:

1. Look up `documentation/scripts/<module>/<id>.md` (auto-generated from draft prose if missing)
2. User reviews/edits script
3. Per voiceover strategy:
   - **Silent** (default): on-screen captions from script, no audio
   - **AI synthetic voice**: TTS produces audio per locale (using TTS provider from intake — local-piper default)
   - **Source voice + per-locale subtitles**: 1 audio + N .vtt files
   - **Human recorded**: BA places audio manually; record only does screen capture
4. Encode video; produce subtitles if enabled

#### Translate
Processes BOTH drafts AND scripts (v1.5.7+):
- Drafts: `drafts/<source>/...` → `drafts/<target>/...`
- Scripts: `scripts/<module>/<id>.md` # Source script → ## `<locale>` sections in same file

Per-block (review each block) or batch (whole-file diff) review mode. Glossary at `documentation/standards/glossary.<locale>.yaml` enforces terminology.

---

## 6. The `deploy` command

Copy/sync workspace to host project (Docusaurus repo) with transforms.

### Preset: standalone

No deploy. Workspace IS the artifact. Skip this command.

### Preset: docusaurus

Maps:
- `documentation/drafts/<source-locale>/<module>/<doc>.md` → `<target>/docs/<module>/<doc>.md`
- `documentation/drafts/<target-locale>/<module>/<doc>.md` → `<target>/i18n/<locale>/.../current/<module>/<doc>.md`
- `documentation/images/<module>/<asset>.png` → `<target>/static/img/<product.slug>/<module>/<asset>.png`
- `documentation/videos/<id>.mp4` → `<target>/static/videos/<product.slug>/<id>.mp4`

Always start with `--dry-run`:

```bash
/docsmith deploy --dry-run                  # preview file changes
/docsmith deploy                            # apply
cd ../my-docusaurus-site && git diff && git commit
```

### Transforms applied

- Frontmatter injection (slug, sidebar_position from sitemap)
- Image namespacing (`/img/<product.slug>/...` to avoid collisions with other Docusaurus content)
- MDX escape (`<`, `>`, `{` in prose escaped to prevent JSX parsing)
- Category generation (`_category_.json` per directory; respects sitemap pattern order)

### In-place mode

When deploy target = `.` (running docsmith inside the Docusaurus repo):
- No copy; transforms applied IN PLACE
- Drafts in `documentation/` stay as authoring source
- Final files at `docs/` (Docusaurus convention) are deploy targets

### Audit trail

Every deploy produces `documentation/deployments/<ts>-<target>/`:
- `manifest.yaml` — what was copied where
- `diff.md` — files added/modified/deleted

Retained for git history. Recoverable.

---

## 7. The `update` command (v1.5.10 expanded)

`update` is the "what changed?" inspector. Three layers:

### Layer 1: Content drift (always checked)

For each registered source in `sources.lock.yaml`:
- Cheap metadata check (Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime)
- If unchanged → skip
- If changed → flag affected drafts (uses module sources to determine which docs)

### Layer 2: Module diff (v1.5.10+)

Compares modules in source structure vs modules in workspace:

| Bucket | Action options |
|---|---|
| **In source AND workspace** | Continue |
| **In source, NOT in workspace** (new) | Create intake (with `--from-source` auto-fill) |
| **In workspace, NOT in source** (orphan) | Archive (mark `status: archived`) |

### Layer 3: Scope drift (v1.5.10+)

For modules in both source and workspace, compare features list:
- Module intake says: `[create instance, edit instance]`
- Source mentions: `[create instance, edit instance, auto-scaling]`
- → Scope drift detected; offer to add `auto-scaling` to module intake

### Interactive prompt

User chooses actions per bucket. AI applies. Update inference report at `documentation/intake/.inference/<ts>-update.md`.

### Multi-module performance

When ≥5 modules need processing (new + scope drift combined), AI MAY use Task tool to spawn parallel sub-agents (v1.5.10+ Cấp 1 guidance). Sequential fallback always valid. See [intake-reference.md § 9.9](skills/docsmith/intake-reference.md).

---

## 8. Multi-locale translation

Set in project intake § 3:
- Source language (the language you draft in)
- Target languages (translate to)

After `draft` produces source-locale docs, `translate` processes:

```
documentation/drafts/en/instances/create.md     # source
        ↓
documentation/drafts/vi/instances/create.md     # AI translation
documentation/drafts/jp/instances/create.md
```

Per-block review (each markdown block reviewed individually) is safest. Batch (whole-file diff) is faster — default for v1.5.0+. Auto-approve only after building trusted glossary.

### Glossary

Auto-created at `documentation/standards/glossary.<locale>.yaml`:

```yaml
- term_en: instance
  term_vi: máy ảo
  notes: prefer over "VM" for consistency
- term_en: auto-scaling
  term_vi: tự động mở rộng
  notes: keep "auto-scaling" in code blocks
```

Translate command applies glossary before sending blocks to AI. Manual additions persist.

### Script translation (v1.5.7+)

Video scripts at `documentation/scripts/<module>/<id>.md` have:
```markdown
# Source script (en)
Welcome to MyCloud...

## vi
Chào mừng đến MyCloud...

## jp
MyCloudへようこそ...
```

Translate command processes scripts alongside drafts. Same review mode, same glossary.

---

## 9. Re-running commands safely (re-run protocol)

Every command checks if its output exists before writing. If it does, you get a 4-option gate:

1. **Update** (default) — existing content read as KB, AI proposes deltas only. Manual edits preserved.
2. **Overwrite** — discard existing, regenerate. Backup to `documentation/archive/<ts>/`.
3. **Side-by-side** — keep both, new file gets `-v2` suffix.
4. **Cancel** — abort.

Update mode uses **KB inheritance** (v1.3.0+):
- Existing draft read as authoritative content
- AI compares with current intake/sources
- Proposes only the deltas (new section here, removed section there, refined wording)
- User reviews delta, accepts or skips
- Result: incremental refinement, not full regeneration

This applies to drafts, plans, voice charts, glossaries, every generated artifact.

### Drift detection in walkthrough

Walkthrough has its own version of re-run protocol focused on UI changes:
- Compare draft instructions vs live product UI
- For each mismatch, classify: HIGH / MEDIUM / LOW confidence in fix
- HIGH: doc says "Click Submit" but button is "Save" → auto-fix doc (if `--auto-apply-high-confidence`)
- MEDIUM: doc says "Settings tab" but UI has "Preferences tab" — usually a label rename → prompt user
- LOW: doc describes flow that no longer exists — likely product bug; mark for product team

Drift reports at `documentation/walkthrough/drift/<ts>/`. User decisions in `decisions.yaml` per drift report.

### Source change detection

`update` checks if external sources (Notion / GitHub / GDoc) have new content. Cheap metadata calls — no full re-fetch unless something changed. When change detected, propagates to affected drafts via Update mode.

---

## 10. Mental model: skill = prompt, not program

docsmith is a Claude skill. The "code" is markdown:

- `SKILL.md` — main skill spec, primary prompt
- `templates/*.md` — output templates AI uses
- `*-reference.md` — deep technical reference for AI to consult
- `presets/*.yaml` — preset configs (standalone, docusaurus)

When you run `/docsmith init`, Claude:
1. Loads SKILL.md
2. Reads relevant template (PROJECT_INTAKE_TEMPLATE.md for init)
3. Follows the steps described in SKILL.md `### init` section
4. Produces output

There's no compiled binary, no daemon, no state machine. AI follows the spec each time.

### Implications

- **Bug fixes** are prose edits in `SKILL.md` or reference docs, not code patches. Bump version, push.
- **New features** are new sections / templates / flags described in spec. AI follows updated spec next run.
- **Behavior is deterministic** if spec is precise, indeterministic if spec is vague. Quality of skill = quality of prose.
- **No silent failures**: if AI can't follow a step, it asks you. The whole skill is "AI does X if Y, asks if Z".

This is why docsmith has accumulated to ~9000 lines of spec across 41 files. Each behavior is documented, not coded.

---

## 11. Daily workflow cheat-sheet

```bash
# One-time setup
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2

# First-time: project from BA doc
mkdir my-docs && cd my-docs
/docsmith init --from-source path/to/ba-doc.md
# AI fills project.md, asks 5-10 questions, detects modules
# Review project.md (look for "← AI guess" markers)

# Or first-time: manual
mkdir my-docs && cd my-docs
/docsmith init
/docsmith module instances
# Fill project.md and modules/instances.md (each field has inline hint)

# Run pipeline
/docsmith run                       # pauses after draft (default)
# Review drafts in documentation/drafts/<source>/<module>/
/docsmith continue                  # walkthrough → record → translate

# Verify before deploy
/docsmith verify                    # 11-check audit

# Deploy
/docsmith deploy --dry-run          # preview
/docsmith deploy                    # apply to Docusaurus repo
cd ../my-docusaurus-site
git diff && git add . && git commit -m "Update docs" && git push

# Iterate after BA doc changes
/docsmith update                    # detects content drift + module diff + scope drift
# Review proposed changes; AI applies selected
/docsmith continue                  # re-walk + re-translate as needed

# Iterate after product UI change
/docsmith walkthrough --check       # detect drift only
# Review documentation/walkthrough/drift/<ts>/
/docsmith walkthrough --apply       # capture new screenshots, apply decisions

# Re-record video after script change
/docsmith record --re-record <id>
```

---

## 12. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "Required field missing in project.md" | Empty backtick or unticked critical checkbox | Run `/docsmith intake-help` for field reference; fill listed |
| Walkthrough hangs at login | Test account env var unset or wrong | `echo $MYPRODUCT_TEST_USER` to verify; re-export |
| Walkthrough captures wrong UI | Per-locale strategy mismatch | In project intake media policy, check locale strategy |
| Translated docs have inconsistent terms | Glossary incomplete or not applied | Edit `documentation/standards/glossary.<locale>.yaml` and re-run translate |
| Deploy refuses to write | File collision detected | Use `--dry-run` to inspect; resolve manually OR use `--force` |
| Sitemap shows wrong order | Sitemap pattern mismatch with module sections | Run `/docsmith plan --migrate-sitemap` |
| Video has no voice but should | Voiceover strategy = silent OR TTS provider env var unset | Check project intake § Media policy; expand Advanced if collapsed |
| Source not refreshing on update | Source unchanged since last fetch | `/docsmith fetch <source-name> --force` |
| `--from-source` auto-fill missed a feature | Source heading not detected | Edit module intake manually; add feature; or rerun with clearer source |
| Parallel sub-agents reported partial | Network or parsing error during one sub-agent | `/docsmith init --from-source --resume` retries failed only |

For deeper issues:
- `/docsmith intake-help` — field reference
- `documentation/intake/.inference/<latest>.md` — what AI inferred
- `documentation/walkthrough/drift/<latest>/` — UI mismatch reports
- `documentation/.run-state/<module>.yaml` — pipeline state per module
- `documentation/deployments/<latest>/diff.md` — what last deploy did
