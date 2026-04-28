# Intake Guide — How to fill `project.md` and `module.md`

This is a practical guide for BAs and content owners filling out docsmith's intake forms. If you've never touched a YAML file in your life, this guide is for you.

> 🇻🇳 Bản tiếng Việt: [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md)

## What you fill out

Two markdown files, both inside `documentation/intake/`:

```
documentation/intake/
├── project.md                  ← Fill this once per project
└── modules/
    ├── instances.md            ← One file per module/feature area
    └── storage.md
```

`project.md` is created by `/docsmith init`. Module files are created by `/docsmith module <n>`.

## How to edit (3 simple rules)

### Rule 1: Fill backtick fields

Anything inside backticks `like this` is a value you fill in. Replace placeholder text with your actual value.

**Before:**
```markdown
- Product slug: `your-product-slug`
```

**After:**
```markdown
- Product slug: `mycloud`
```

Don't remove the backticks. Just replace the text inside them.

### Rule 2: Tick checkboxes

Checkboxes look like `[ ]` (empty) or `[x]` (ticked). Tick by changing the space to `x`.

**Before:**
```markdown
- [ ] Standalone (no deploy target)
- [ ] Docusaurus
```

**After:**
```markdown
- [ ] Standalone (no deploy target)
- [x] Docusaurus
```

For most groups you tick exactly one option. A few groups (like target languages) let you tick multiple — the form tells you which.

### Rule 3: Add or remove repeating sections

Some sections are templates you can copy. Look for `### Source 1`, `### Source 2`, `#### Feature 1`, etc. To add a new one, copy the whole block and increment the number. To remove, delete the whole block.

**Don't worry about leaving "blank" sections** — if every backtick is empty and every checkbox is unticked in a Source/Feature block, AI ignores it.

## What each section means (project.md)

### 1. Product

The basic identity of your product.

| Field | What it is | Example |
|---|---|---|
| Product slug | Short identifier used in URLs and image namespaces. Lowercase, no spaces, hyphens OK. | `mycloud`, `acme-app` |
| Product display name | Human-readable name for titles | `MyCloud`, `Acme App` |
| Product URL | Where the live product runs (for walkthrough automation) | `https://console.mycloud.com` |
| One-line description | What the product does, in one sentence | `Cloud platform for hosting Docker apps` |

The slug must be unique across all products that share the same Docusaurus target — it becomes the image folder namespace (`/img/<slug>/...`).

### 2. Audience

Who reads the docs. AI uses this to set tone, vocabulary, and assumed prior knowledge.

**Primary persona** (required):

| Field | What it is | Example |
|---|---|---|
| Role / job title | What they do for work | `DevOps Engineer`, `Marketing Manager`, `Junior Developer` |
| Technical level | How comfortable they are with technical docs | Tick one of Low / Medium / High |
| Primary goal | What they want to achieve when reading docs | `Deploy a service to production within 30 minutes` |

**Secondary personas** are optional. Add only if you have clearly different audiences (e.g., "developers AND managers"). Don't add personas just because you can — every persona makes content harder to write for everyone.

### 3. Languages

What language the docs are written in, and what to translate to.

- **Source**: the language you write in. Tick exactly one.
- **Target**: the languages to auto-translate to. Tick zero or more. Leave all unticked for single-language projects.

If you tick target languages, AI looks for a glossary file at `documentation/standards/glossary.<locale>.yaml` (e.g., `glossary.vi.yaml` for Vietnamese). The glossary is optional but helps consistency. See [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml).

### 4. Deploy

Where the docs go when published.

- **Standalone**: docs live in `documentation/drafts/` and that's it. No host project. Choose this if you're just drafting and haven't decided where to publish.
- **Docusaurus**: deploy syncs to a Docusaurus repo. Tell us where:
  - Relative path: `../mycloud-docusaurus` (sibling folder)
  - Absolute path: `/home/user/sites/mycloud-docs`
  - In-place: `.` (when you run docsmith inside your Docusaurus repo)

**On collision**: when the target already has a file with different content, what should deploy do? Default `Warn` is safest.

### 5. Voice and tone

How the docs sound. AI uses this directly when writing.

| Field | What to choose |
|---|---|
| Tone | Casual = chatty; Friendly-professional = default for most products; Technical-direct = no fluff for power users; Formal = corporate/regulated |
| Perspective | Second-person ("you do X") = standard for instructional docs; First-person plural ("we do X") = collaborative tone (think Stripe); Third-person ("the user does X") = formal/legal |
| Reading level | 6th = accessible to all; 8th = default; 10th-college = developer/specialist content |
| Custom terms to AVOID | Words/phrases AI shouldn't use. Example: `utilize, leverage, robust, seamless` |

When in doubt, leave defaults (Friendly-professional + Second-person + 8th grade).

### 6. Credentials for product walkthrough

The walkthrough command logs into your product to capture screenshots. **Never put passwords here.** Put environment variable NAMES instead.

```markdown
- Username env var: `MYCLOUD_TEST_USER`
- Password env var: `MYCLOUD_TEST_PASS`
```

Then before running walkthrough:

```bash
export MYCLOUD_TEST_USER="qa@mycloud.test"
export MYCLOUD_TEST_PASS="your-actual-password"
/docsmith run
```

This way you can commit `project.md` to git safely — it never contains the actual password.

If your product has multi-factor auth, add a third env var. If you have a dedicated test account note (e.g., "test account, free tier, no production data"), add it for clarity.

### 7. Knowledge sources

Where AI fetches content from when drafting docs.

For each source, tick its type and fill in the URL/path. Add as many sources as you have.

**Source types**:

| Type | What it is | URL/ID format | Auth |
|---|---|---|---|
| Notion page | A Notion doc | Full URL: `https://notion.so/abc123` or just `abc123` | Set `NOTION_TOKEN` env var |
| GitHub repo | Code or docs in GitHub | `owner/repo` (e.g., `mycloud/cloud`) | `GITHUB_TOKEN` for private; none for public |
| Google Drive doc | A Google Doc | File ID from URL | `GOOGLE_DRIVE_TOKEN` env var |
| Public URL | Any web page | Full URL | None |
| Local file | File on your computer | Relative or absolute path | None |

For GitHub specifically, you can specify subpaths:

```markdown
- URL or path or ID: `mycloud/cloud`
- (Then in Notes or Paths field): `api/instances/*.ts, docs/instances/*.md`
```

AI fetches only matching files, not the whole repo.

**Auth env vars**: same idea as credentials — name only, never the token itself.

### 8. Auto-run behavior

Where AI pauses for your review during the pipeline.

- **After plan** = pause early to validate documentation plan before drafting
- **After draft** (default) = pause to review drafts before walkthrough captures screenshots
- **Before deploy** = let everything generate, only review at the end
- **Never** = full auto, only for trusted re-runs after first success

For your **first run**, "After draft" is the safest — you see what AI wrote before it goes to the live product.

For drift detection during walkthrough:

- **Prompt per item** (default) = safest, slowest
- **Auto-apply HIGH confidence** = faster, requires good caption discipline

For translation:

- **Per-block** = review every paragraph individually (slow but safe)
- **Batch** (default) = review whole-file diff (faster, default)
- **Auto-approve** = skip review (only for trusted glossaries)

### 9. Sitemap pattern

Determines the navigation structure across all modules. Pick once at project level.

- **Pattern A — Learning path** (default): `Overview → Initial setup → Quick Starts → Tutorials → Reference → Troubleshooting → Glossary`. Best for technical products with conceptual depth.
- **Pattern B — Task-first**: `Overview → Quick Starts → Guides → Reference → Troubleshooting`. Best for mature products where users come knowing what they want.
- **Pattern C — Custom**: you define order.

Why this matters: ensures all modules in your project have **consistent navigation**. Without this, module A might have `Quick Starts` while module B has `Guides`, confusing users.

Each module then ticks which sections it includes (in module intake § Sitemap sections). AI follows the project pattern when ordering them and warns when a module is missing a section the pattern includes.

**Display name overrides**: you can rename canonical sections per project (e.g., `Quick Starts` → `Hands-on guides`). Slugs (folder names) stay canonical.

### 10. Module intake files

Auto-managed. Don't edit by hand. The `module` command updates this section when you create/archive modules.

## What each section means (modules/<n>.md)

Module files are simpler — they only contain things that DIFFER from the project default.

### 1. Module identity

| Field | What it is | Example |
|---|---|---|
| Module slug | Short identifier; matches the filename | `instances` (file: `instances.md`) |
| Module display name | Human-readable | `Instances` |
| Priority | 1 (highest) to 5 (lowest). Used for ordering in sitemap | `1` |
| Folder in target docs | Where in the published site this lives | `instances` (deploys to `docs/instances/`) |

Folder defaults to slug. Override only if you want different URL than slug.

### 2. Scope

What features this module documents.

For each feature:

- **Feature name**: what users will look up. `Create instance`, `Auto-scaling`, `IP allocation`
- **Content types**: tick which types you need. Multiple ticks = multiple docs generated. See content types below.

**Content types** (tick one or more per feature):

| Type | When to use | Output style |
|---|---|---|
| Tutorial | First-time users learning by doing | Hand-holding, every step explained, beginner-friendly |
| How-to | Users with goals, not learning | Task-focused, assumes context |
| Reference | Looking up parameters, options, settings | Tables, lists, structured |
| Concept | Understanding how something works | Prose explanation, may have diagrams |

**Example**: feature `Create instance` with `tutorial` + `how-to` ticked → AI generates 2 docs:
- `documentation/drafts/en/instances/create-instance-tutorial.md` (tutorial)
- `documentation/drafts/en/instances/create-instance.md` (how-to)

**Out of scope**: explicitly list things this module does NOT cover. Helps AI avoid hallucinating tangential content.

### 3. Voice override

Usually leave all "Inherit from project" ticked. Only override when this module needs a different voice (e.g., admin module wants formal tone while user module is casual).

### 4. Module-specific knowledge sources

Same format as project sources, but only for this module. These ADD to project sources — both lists are used.

Use this for module-specific PRDs, design docs, or code areas not relevant to other modules.

### 5. Walkthrough setup

For modules requiring different test accounts than the project default, override here. Most modules inherit.

**Pre-walkthrough setup script**: bash commands run before browser automation. Common uses:
- Create test data: `mycloud volume create --size 10G --name test-vol`
- Seed user accounts: `mycloud user create --email test@example.com`

**Post-walkthrough teardown**: cleanup after capture. Not strictly required (next run can re-create), but tidier.

### 6. Special handling

- **Sensitive fields to redact**: ticks help walkthrough know what to blur in screenshots. Adds visual placeholder over those areas.
- **Module status**:
  - `Active` (default) — processed by `/docsmith run`
  - `Paused` — skipped temporarily; kept for later
  - `Archived` — skipped permanently; not deleted from target by `--sync-deletes`

### 7. Sitemap sections (v1.5.4+)

Tick which canonical section types this module includes. The order in which they appear is determined by the project pattern (see project.md § 9).

| Section type | When to tick |
|---|---|
| `overview` | Always (auto-required) |
| `initial-setup` | Module needs specific setup beyond project-level |
| `quickstarts` | Have short task-focused content (5-10 mins each) |
| `tutorials` | Have step-by-step learning content with hand-holding |
| `guides` | Have how-tos that assume context (alternative to quickstarts) |
| `concepts` | Have non-obvious concepts to explain |
| `dashboard` | Module has a dashboard or report view |
| `reference` | Have parameter tables, schema specs |
| `api-reference` | Module has stable API |
| `glossary` | Module-specific terms |
| `troubleshooting` | Common issues for this module |

**Pick `quickstarts` OR `guides`, not both.** They overlap.

**AI suggestions**: when you run `/docsmith plan`, AI checks each module against the project pattern and warns if a module is missing a section the pattern includes. Decide per warning — sometimes a section legitimately doesn't apply.

**Display name overrides**: optional per-module names (e.g., this module's "Quick Starts" displays as "Get started fast" in nav). Override at module level overrides project level overrides default.

## Common patterns

### "I just want to write docs for one feature"

1. `/docsmith init` (Standalone preset, no deploy)
2. `/docsmith module myfeature`
3. Fill `documentation/intake/project.md` Product + Audience + Voice (basics only)
4. Fill `documentation/intake/modules/myfeature.md` Scope (1 feature, How-to content type)
5. `/docsmith run myfeature`

### "I have a Docusaurus site and want to add a module"

1. `cd ~/my-docusaurus-site`
2. `/docsmith init --in-place` (auto-detects, suggests in-place)
3. `/docsmith module pricing`
4. Fill intakes
5. `/docsmith run pricing`
6. `/docsmith deploy --dry-run` then `/docsmith deploy`

### "I want multilingual docs (EN + VI + JP)"

1. In `project.md` § Languages: source `en`, targets `vi` + `jp`
2. Optionally edit `documentation/standards/glossary.vi.yaml` and `glossary.jp.yaml` (auto-created)
3. `/docsmith run` runs through translate stage automatically
4. Translated drafts appear in `documentation/drafts/vi/` and `drafts/jp/`

### "Source content lives in Notion"

1. Get Notion integration token: notion.so/my-integrations → create internal integration → copy token
2. Share the Notion page with that integration
3. `export NOTION_TOKEN="secret_..."`
4. In intake `Knowledge sources`:
   - Type: tick Notion page
   - URL: `https://notion.so/abc123` (full URL or just the ID)
   - Auth env var: `NOTION_TOKEN`

### "I want to update docs after the product UI changed"

1. `/docsmith walkthrough --check` — produces drift report
2. Read `documentation/walkthrough/drift/<latest>/drift-report.md`
3. Edit `decisions.yaml` next to the report (auto-fix / skip / product-bug)
4. `/docsmith walkthrough --apply` to apply decisions and re-capture

### "Source PRD in Notion was updated"

1. `/docsmith update`
2. AI checks all sources, lists what changed
3. Confirm to re-fetch and re-evaluate affected docs
4. Review proposed deltas (KB inheritance — only changed sections updated)

## What does AI actually do with each field?

| Field in intake | Where AI uses it |
|---|---|
| product.slug | Image namespace `/img/<slug>/`; sources.lock entries |
| product.name | Title in frontmatter; intro lines |
| audience.tech_level | Vocabulary choices, assumed prior knowledge |
| audience.primary_goal | Tutorial intro, "you'll learn how to..." statements |
| locales.source | Where drafts go: `drafts/<source>/...` |
| locales.targets | What languages translate runs |
| voice.tone | Sentence style, contractions, formality |
| voice.perspective | "you" vs "we" vs "the user" throughout |
| voice.reading_level | Word choice, sentence length |
| voice.terms_to_avoid | Negative constraint during drafting |
| deploy.preset | Which preset's deploy logic activates |
| deploy.target_path | Where files go on `deploy` |
| sources | Fetched at run time; content available to AI as KB during draft |
| credentials.*_env | AI reads env var when launching browser for walkthrough |
| auto_run.pause_at | When pipeline interrupts for review |
| translate.review_mode | per-block vs batch vs auto-approve |
| module.folder | Folder name in target (e.g., `docs/instances/`) |
| features[].content_types | Drives how many docs generated per feature |
| out_of_scope | Negative constraint — don't generate these topics |
| status (module) | Whether `run` processes this module |

## What if I make a mistake?

- **Empty critical field** → `/docsmith run` stops with a clear list of fixes needed
- **Wrong checkbox tick** → run again with corrected file; re-run protocol kicks in to merge cleanly
- **Bad source URL** → `/docsmith fetch` errors with the URL that failed; fix and retry
- **Unset env var** → Command stops, prints which env var to set
- **Format error** (deleted backticks accidentally) → `/docsmith intake-help` flags the issue

You can always run `/docsmith intake-help` to see field reference, or `/docsmith intake-help <section>` for one section.

## After your first successful run

You'll have:
- Audience profile generated from your input
- Documentation plan with sitemap
- Voice chart in your project's standards
- Drafts in `documentation/drafts/<locale>/<module>/`
- Screenshots captured into `documentation/images/<module>/`
- Translated drafts (if you set targets)
- Audit trail in `documentation/deployments/<ts>/`

Edit any draft directly if needed (re-run protocol preserves edits on next run). Then:

```bash
/docsmith deploy --dry-run    # preview what will be copied to host
/docsmith deploy              # apply
cd ../my-docusaurus-site && git diff && git add . && git commit && git push
```

Done. The site is live.
