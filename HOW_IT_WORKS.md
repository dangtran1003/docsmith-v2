# How docsmith works

This document explains the operating model of the skill: what runs when, what reads what, what writes where, and how the commands chain together. Read this once before using docsmith on a real project.

> 🇻🇳 Bản tiếng Việt: [HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md)

## TL;DR

Docsmith is a Claude Code **skill** (markdown instructions for an AI), not a compiled program. You run commands like `/docsmith deploy MyProduct`. Claude reads the instructions in `SKILL.md`, then executes the steps using its own tools (file read/write, bash, etc.) — generating any code it needs on the fly.

There are **19 commands** organized into a pipeline. Each command reads from `.docsmithrc.yaml` (your project config), writes only inside scoped paths (workspace + deploy target), and produces an audit trail.

---

## 1. The two-layer model

```
┌──────────────────────────────────────────────────────────────────┐
│  Layer 1: docsmith skill (this repo)                             │
│  Markdown instructions Claude follows                            │
│  ─────────────────────────────────────                           │
│  SKILL.md ............ command catalog + per-command spec        │
│  deploy-reference.md . detailed deploy logic                     │
│  process-reference.md  PRC-010 process detail                    │
│  presets/*.yaml ...... deploy mappings (standalone, docusaurus)  │
│  templates/*.md ...... output formats Claude fills in            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │ Claude reads instructions,
                              │ uses its own tools (bash, view, etc.),
                              │ writes into your project
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  Layer 2: your project (where you run /docsmith)                 │
│  ─────────────────────────────────────                           │
│  .docsmithrc.yaml .... config (created by /docsmith init)        │
│  documentation/ ...... workspace (drafts, plans, walkthrough)    │
│  deployments/ ........ audit trail of past deploys               │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │ /docsmith deploy
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  Layer 3: deploy target (optional — Docusaurus / other host)     │
│  ─────────────────────────────────────                           │
│  docs/ ............... markdown files (with frontmatter)         │
│  i18n/<locale>/...... per-locale markdown                        │
│  static/img/<slug>/.. images, namespaced by product slug         │
└──────────────────────────────────────────────────────────────────┘
```

Layer 1 never changes when you use it (read-only skill).
Layer 2 is what `/docsmith init` creates and what most commands write to.
Layer 3 is your published docs site, written only by `/docsmith deploy`.

---

## 2. The 19 commands and where they live in the pipeline

```
SETUP             init  ──▶ creates .docsmithrc.yaml + workspace
                    │
                    ▼
PLAN          audience ──▶ AUDIENCE PROFILE
                  plan  ──▶ DOC PLAN + TRACEABILITY MATRIX
            review-plan ──▶ (human approval gate)
                sitemap ──▶ SITEMAP
                  voice ──▶ VOICE CHART + UX PATTERNS + SCORECARD
                    │
                    ▼
AUTHOR            draft ──▶ documentation/drafts/<source-locale>/*.md
                            (with placeholder screenshots + video markers)
                   edit ──▶ 5-pass self-review
                    │
                    ▼
VERIFY      walkthrough ──▶ replaces placeholders with real screenshots,
                            runs test cases against live product
                 record ──▶ records short tutorial videos (optional)
               validate ──▶ runs only the test cases (no screenshots)
                   test ──▶ creates test cases from existing docs
                 verify ──▶ comprehensive 10-check audit
                    │
                    ▼
REVIEW     peer-review  ──▶ human review
           tech-review  ──▶ optional engineering review
            incorporate ──▶ AI applies feedback
                    │
                    ▼
DEPLOY      categorize  ──▶ generates _category_.json (Docusaurus)
                 deploy ──▶ copies workspace to host project
                            with frontmatter, image rewrites, MDX escape
                    │
                    ▼
PUBLISH        publish  ──▶ human: git diff / commit / push on target
```

Commands you run once: `init`.
Commands you run as the doc evolves: `draft`, `edit`, `walkthrough`, `record`, `verify`, `deploy`.
Commands you run when reorganizing: `sitemap`, `categorize`.

---

## 3. What `/docsmith init` does

Run this first, in the directory where you want your docs to live.

```bash
mkdir my-product-docs && cd my-product-docs
/docsmith init
```

Interactive prompts (Claude asks you):

1. **Product slug** — short kebab-case name (e.g. `mycloud`). Becomes the image namespace `/img/mycloud/...` on deploy. Must be globally unique across products that share a Docusaurus target.
2. **Product display name** — used in titles (e.g. "MyCloud").
3. **Source locale** — language you draft in (default `en`).
4. **Target locales** — translation targets, comma-separated (e.g. `vi,jp`). Optional. Folder structure is created; auto-translation comes in v1.6.
5. **Deploy preset** — `standalone` (default, no host) or `docusaurus`.
6. **Deploy target path** — only if `docusaurus`. Path to host project (`../my-docusaurus-site`) or `.` for in-place mode.

If preset is `docusaurus`, Claude **inspects** the target before writing your config:
- Reads `<target>/CLAUDE.md` if present
- Parses `<target>/docusaurus.config.{ts,js,mjs}` for `i18n`, `docs.path`, `staticDirectories`
- Confirms detected paths with you

After confirmation, Claude writes:

```
.docsmithrc.yaml                 # your config
documentation/
├── plan/                         # empty, for plan command
├── standards/                    # empty, for voice command
├── drafts/<source-locale>/       # empty, draft command writes here
├── drafts/<each-target-locale>/
│   └── README.md                 # explains: empty until v1.6 auto-translation
├── walkthrough/
│   ├── test-cases/
│   ├── video-plan/
│   └── executions/
├── images/                       # walkthrough writes here
└── videos/
    ├── raw/                      # raw recordings (gitignored)
    └── (mp4 output)
deployments/                      # audit trail, empty
```

After this, every other command reads `.docsmithrc.yaml` first to know where to write.

---

## 4. The `.docsmithrc.yaml` config — source of truth

Every command reads this file before doing anything. It scopes paths and prevents accidental writes outside your project.

Key fields (full schema in [.docsmithrc.example.yaml](.docsmithrc.example.yaml)):

```yaml
product:
  slug: mycloud                 # image namespace
  name: "MyCloud"

locales:
  source: en                    # draft language
  targets: [vi, jp]             # translation scaffolding (no auto-translate yet)

paths:
  workspace: documentation/     # where commands can write
  deployments: deployments/     # audit trail location

deploy:
  default_target: ../my-site    # where deploy copies to (empty = standalone)
  preset: docusaurus            # standalone | docusaurus
  on_collision: warn            # warn | skip | overwrite | prompt

docusaurus:                     # only used if preset = docusaurus
  docs_path: docs
  i18n_path: i18n
  static_path: static
  image_subpath: img/{product.slug}
  inject_frontmatter: true
  generate_categories: true
  generate_sidebars: false      # safer default — don't override user's sidebars.js
```

**Path scoping**: every command validates the absolute target path is inside `paths.workspace` OR `deploy.default_target` before writing. Reject otherwise. This is the safety net against AI bugs writing into unrelated parts of the host project.

---

## 5. Authoring loop (draft → walkthrough → record)

Once your plan is done, you iterate:

```
┌─────────────────────────────────────────────────────────┐
│  1. /docsmith draft MyProduct                           │
│     AI writes:                                          │
│       documentation/drafts/en/getting-started.md        │
│       documentation/drafts/en/instances/create.md       │
│     With placeholders:                                  │
│       ![Caption text](https://placehold.co/600x400)     │
│       <!-- VIDEO id: instance-tour ... -->              │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  2. /docsmith edit MyProduct                            │
│     AI runs 5 self-review passes (clarity, voice,       │
│     scorecard, completeness, accuracy)                  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  3. /docsmith walkthrough MyProduct                     │
│     AI:                                                 │
│       a. Builds capture plan from all placeholders      │
│       b. SHOWS PLAN to you for caption review           │
│       c. Runs test cases against live product           │
│       d. Captures real screenshots                      │
│       e. Replaces placehold.co URLs with real paths     │
│          /images/instances/create-form-filled.png       │
│       f. Fixes any doc bugs found by test cases         │
│       g. Records execution → walkthrough/executions/    │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼ (optional)
┌─────────────────────────────────────────────────────────┐
│  4. /docsmith record MyProduct                          │
│     AI:                                                 │
│       a. Scans for <!-- VIDEO ... --> markers           │
│       b. Validates each marker's required fields        │
│       c. Records screen → transcodes → mp4              │
│       d. Replaces marker with <video> tag,              │
│          preserves marker as comment for re-record      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  5. /docsmith verify MyProduct                          │
│     AI runs all 10 checks:                              │
│       no placeholders left, all links work, voice score │
│       passes, test cases pass, etc.                     │
│     Reports issues, doesn't auto-fix                    │
└─────────────────────────────────────────────────────────┘
```

If `verify` fails → fix → re-run from the failing step (usually `walkthrough` or `record`).

### Image flow inside the loop

| Stage          | Markdown contains                                    | File on disk                                               |
| -------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| After `draft`  | `![Form filled](https://placehold.co/600x400)`       | (no file)                                                  |
| After `walkthrough` | `![Form filled](/images/instances/create-form-filled.png)` | `documentation/images/instances/create-form-filled.png`    |
| After `deploy` | `![Form filled](/img/mycloud/instances/create-form-filled.png)` | `<target>/static/img/mycloud/instances/create-form-filled.png` |

The slug `mycloud` namespace is added at deploy time, not at capture time. This keeps your workspace portable across multiple deploy targets if needed.

---

## 6. The `deploy` command — moving from workspace to host project

This is where things get interesting. Read [deploy-reference.md](deploy-reference.md) for the full logic; here's the user-facing model.

### Always start with dry-run

```bash
/docsmith deploy MyProduct --dry-run
```

Dry-run goes through detect + plan and prints what would happen:

```
Target: ../mycloud-docusaurus (docusaurus 3.x)
Detected config:
  docs_path: docs
  i18n_path: i18n
  static_path: static
  default_locale: en
  locales: [en, vi]

Plan summary:
  43 files to create
  8 files to update
  12 files unchanged (skip)
  2 conflicts (BLOCKING)

Conflicts:
  ! docs/getting-started.md
    Reason: target hash differs from workspace hash
    Resolve: re-run with --force, or set on_collision: overwrite

New files:
  + docs/getting-started.md
  + static/img/mycloud/instances/create-form-filled.png
  ...
```

You review the plan, fix any conflicts, then run again without `--dry-run` to apply.

### What deploy actually transforms

Per markdown file, deploy applies in order:

1. **Inject frontmatter** if missing — `id`, `title`, `sidebar_position`. Won't overwrite fields you already wrote.
2. **Rewrite image refs** — `/images/<feature>/<asset>` → `/img/<product.slug>/<feature>/<asset>`.
3. **Rewrite video refs** — `/videos/<id>.mp4` → `/videos/<product.slug>/<id>.mp4`.
4. **Escape MDX specials** — `{` and `}` in body prose become `\{` and `\}`. Code blocks and frontmatter are left alone.

Then it copies:
- Markdown files to `<target>/docs/...` (source locale) or `<target>/i18n/<locale>/.../current/...` (target locales)
- Images to `<target>/static/img/<slug>/...`
- Videos to `<target>/static/videos/<slug>/...`

### Audit trail

Every deploy creates `deployments/<timestamp>-<target>/` with:

```
manifest.yaml          # what was planned + actual results
target-config.yaml     # detected target config (cached for next deploy)
diff.md                # human-readable summary
pre-deploy-state.txt   # target file hashes BEFORE deploy (for manual rollback)
```

Manual rollback: copy `pre-deploy-state.txt` paths back from git. Docsmith doesn't auto-rollback — git on the target project is the proper safety net.

### In-place mode

If your docsmith workspace lives inside the Docusaurus project itself:

```yaml
# .docsmithrc.yaml in the Docusaurus repo root
deploy:
  default_target: .
  preset: docusaurus
```

Then `documentation/` and `docs/` are siblings in one repo. `deploy` still produces audit trail and applies transforms; only the copy step has source and target on the same disk.

---

## 7. Categorize — Docusaurus sidebars

Run after deploy (or as part of deploy if `generate_categories: true`):

```bash
/docsmith categorize MyProduct
```

What it does:

1. Reads `documentation/plan/sitemap.md`
2. Walks the target `docs/` folder
3. For each folder that appears in sitemap → writes `_category_.json`:
   ```json
   {
     "label": "Getting Started",
     "position": 1,
     "link": { "type": "generated-index" }
   }
   ```
4. For each folder NOT in sitemap → lists in report, leaves alone

Title normalization (consistent across siblings):
- Acronyms uppercase: `API`, `CLI`, `JSON`, `URL`, `HTTP`, `SSH`, etc.
- Articles lowercase mid-title: `a`, `an`, `the`, `to`, `of`, `for`, etc.
- First and last word always capitalized

Hiding undocumented folders is a **known limitation** — Docusaurus has no first-class "hide" mechanism. Options documented in [deploy-reference.md](deploy-reference.md): exclude in `docusaurus.config`, custom sidebars, or move folder out of `docs/`.

---

## 8. Multi-locale translation (1.4.0+)

What works in 1.4.0:

- Folder structure: `documentation/drafts/<locale>/` per locale
- **`translate` command** — AI translates from source to each `locales.targets`
- **Per-block review gate** — for each translatable block, AI proposes translation, you approve/edit/skip
- **Glossary** — optional `documentation/standards/glossary.<locale>.yaml` enforces consistent terminology
- **Translation completeness check in `deploy`** — warns when target locales incomplete
- **Re-run safe** — Update mode preserves manually-edited translations when source unchanged

Multi-locale workflow:

```bash
# 1. Draft in source locale (en)
/docsmith draft MyProduct
/docsmith edit MyProduct
/docsmith walkthrough MyProduct
/docsmith record MyProduct                  # optional
/docsmith verify MyProduct

# 2. Translate to all target locales
/docsmith translate MyProduct
# For each block, AI proposes translation; you approve/edit/skip
# Decisions saved to documentation/archive/<ts>/translation-decisions-<locale>.yaml

# 3. (Optional) Verify translated drafts against localized product UI
/docsmith walkthrough MyProduct --locale vi --check

# 4. Deploy all locales together
/docsmith categorize MyProduct
/docsmith deploy MyProduct --dry-run        # warns if any locale incomplete
/docsmith deploy MyProduct
```

What does NOT work yet (deferred):

- **Translation drift tracking** when source updates after translation. Workaround: re-run `translate` in Update mode — it compares blocks via similarity and proposes UPDATE only for changed source. First-class `translate --check` mode is on v1.6.x roadmap.
- **`<!-- translation-locked -->` markers** for protecting blocks from re-translation. v1.6.x.
- **Per-locale image namespacing** for products with localized UI screenshots. v1.5+.
- **Voice chart per locale** for tone consistency in translations. v1.5+.

The "done" definition for multi-locale projects:

```
Source drafts complete  ✓
        ↓
walkthrough/record done ✓
        ↓
incorporate done        ✓
        ↓
translate done          ✓  ← required step
        ↓
categorize/deploy ready ✓
```

Skipping `translate` and going straight to `deploy` produces a Docusaurus site with broken locale switcher (target folders empty). v1.4.0 deploys warn explicitly when this would happen.

---

## 9. Re-running commands safely (1.3.0+)

Docsmith doesn't trust silent overwrites. Every command checks if its output already exists, and if so, presents a 4-option gate:

| Option | When to use |
|---|---|
| **Update** (recommended) | Existing content has team edits worth preserving. AI reads existing as KB, proposes deltas, applies per item. |
| **Overwrite** | Existing is stale; want fresh. Original archived to `documentation/archive/<timestamp>/`. |
| **Side-by-side** | Comparison or major rewrite. New file gets `-v2` suffix. |
| **Cancel** | Abort. |

**Update mode rule**: AI reads existing content as **canonical** and proposes only deltas (NEW / UPDATE / REMOVE / KEEP). Untouched sections are preserved verbatim. This is critical — without this rule, every re-run would silently lose team's manual edits.

**Drift detection** (walkthrough specifically) is a 3-phase pipeline:

```
A: VERIFY  /docsmith wt --check         → drift-report.md (read-only)
   ↓
   GATE    Edit decisions.yaml          (auto-fix / manual-fix / product-bug / skip)
   ↓
B: APPLY   /docsmith wt --apply         → update drafts per decisions
   ↓
C: CAPTURE                              → screenshots
```

Items marked `product-bug` (doc is correct, UI has regression) are tracked in `walkthrough/active-product-bugs.yaml` across runs. They're not re-flagged until UI matches doc again, then auto-resolved.

**Delete propagation**: when you delete a draft, `deploy` reports orphan files in target but doesn't delete by default. Use `--sync-deletes` to actually clean up:

```bash
/docsmith deploy MyProduct --sync-deletes --dry-run    # preview deletions
/docsmith deploy MyProduct --sync-deletes              # apply
```

Deleted target files are backed up to `deployments/<ts>/deleted/` for audit.

For full logic and per-artifact merge rules, see [update-reference.md](update-reference.md).

---

## 10. Mental model: skill = prompt, not program

This is the most important thing to internalize.

A traditional plugin has compiled code. When you call a function, deterministic behavior. Bug → patch code → re-test.

A docsmith skill has **markdown instructions in English**. When you run `/docsmith deploy`, Claude:

1. Reads `SKILL.md` and `deploy-reference.md` into context
2. Reads your `.docsmithrc.yaml`
3. **Generates code on the fly** (Python, bash, regex) to do the work
4. Executes that code via its tools (`bash_tool`, `view`, `create_file`, `str_replace`)

Implications:

- **Not 100% reproducible**. Same command twice may pick slightly different paths or output formats.
- **Always dry-run first** for `deploy`. The plan is your safety check.
- **Audit trail matters more than usual**. `deployments/<ts>/` is how you reconstruct what happened.
- **Never let AI commit**. You review `git diff` on the target and commit yourself.
- **Bug fixes** are prose edits in `SKILL.md` or `deploy-reference.md`, not code patches. Bump version, push.
- **There are no unit tests**. The way to verify a skill works is to run it on a real project and check the output.

This trade-off (flexibility over determinism) is the whole point of the skill format.

---

## 11. Daily workflow cheat-sheet

```bash
# One-time setup
mkdir mycloud-docs && cd mycloud-docs
/docsmith init

# First-time draft (fresh product)
/docsmith audience MyCloud
/docsmith plan MyCloud
/docsmith review-plan MyCloud      # human approval
/docsmith sitemap MyCloud
/docsmith voice MyCloud
/docsmith draft MyCloud
/docsmith edit MyCloud
/docsmith wt MyCloud               # walkthrough — capture screenshots
/docsmith rec MyCloud              # optional — record videos
/docsmith verify MyCloud
/docsmith deploy MyCloud --dry-run # preview
/docsmith deploy MyCloud           # apply
cd ../mycloud-docusaurus
git diff && git add . && git commit -m "Update docs" && git push

# Iteration on a single doc after product UI change
/docsmith wt MyCloud documentation/drafts/en/instances/create.md
/docsmith verify MyCloud documentation/drafts/en/instances/create.md
/docsmith deploy MyCloud --dry-run
/docsmith deploy MyCloud

# Re-record a video after UI change
# (Edit the marker in the doc if needed, or just re-run)
/docsmith rec MyCloud
/docsmith deploy MyCloud --dry-run
```

---

## 12. Where to look when something breaks

| Symptom                                            | Look at                                                                      |
| -------------------------------------------------- | ---------------------------------------------------------------------------- |
| Claude doesn't know a command exists               | Verify skill is installed: `ls ~/.claude/skills/docsmith/SKILL.md`           |
| Wrong files captured in walkthrough                | Caption rules in [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) — review captions |
| Deploy writes to unexpected paths                  | `.docsmithrc.yaml` paths, especially `paths.workspace` and `deploy.default_target` |
| Image broken on Docusaurus site                    | Check `static/img/<slug>/...` exists; check ref in `docs/*.md` matches        |
| MDX build fails on `{` characters                  | Deploy didn't escape — re-run, or add manual escape                          |
| `_category_.json` wrong title                      | Sitemap title (categorize uses sitemap as source of truth)                   |
| Don't know what last deploy did                    | `deployments/<latest-timestamp>/manifest.yaml` and `diff.md`                 |
| Want to roll back a deploy                         | Use git on target project (preferred) or `pre-deploy-state.txt`              |
| Fields in `.docsmithrc.yaml` not understood        | [.docsmithrc.example.yaml](.docsmithrc.example.yaml) has full schema with comments |
| Deploy logic question                              | [deploy-reference.md](deploy-reference.md)                                   |
| Process / authorial question                       | [process-reference.md](process-reference.md), [subprocess-010a.md](subprocess-010a.md) |

---

## 13. What to read next

- [.docsmithrc.example.yaml](.docsmithrc.example.yaml) — annotated config schema
- [SKILL.md](SKILL.md) — full command reference (this is what Claude reads)
- [deploy-reference.md](deploy-reference.md) — deploy logic in depth
- [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) — caption rules
- [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md) — video marker syntax
- [CHANGELOG.md](CHANGELOG.md) — version history with migration notes
- [README.md](README.md) — install + quick start

If you're going to use docsmith on a real project, read items 1–4 minimum.
