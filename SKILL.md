---
name: docsmith
description: "Create, verify, translate-scaffold, and deploy documentation following the PRC-010 process. Standalone-first workspace; deploys to Docusaurus or other targets via preset. Use when asked to create, draft, plan, review, deploy, or publish documentation for a product or project. Guides through audience analysis, documentation planning, sitemap creation, UX content standards, drafting, self-review, product walkthrough, tutorial video recording, and deploy to host project. Supports commands: init, start, audience, plan, review-plan, sitemap, voice, draft, edit, walkthrough, record, validate, test, verify, peer-review, tech-review, incorporate, publish, categorize, deploy."
---

# Docsmith — PRC-010: Documentation Creation Process

Standardized process for systematically creating, reviewing, and publishing documentation. Based on _Docs for Developers_ (Bhatti et al., 2021) and _Strategic Writing for UX_ (Podmajersky, 2019).

## Commands

Parse `$ARGUMENTS` to determine which command to run. If no command is given or the command is `help`, show the command reference table below.

| Command       | Alias | Owner  | Description                                                                                                                               |
| ------------- | ----- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `help`        | `h`   | —      | Show this command reference                                                                                                               |
| `init`        | `i`   | **AI** | Initialize a docsmith workspace: create `.docsmithrc.yaml`, `documentation/` folders, locale dirs                                         |
| `start`       | —     | —      | Begin the full process from the first human step (assumes `init` has been run)                                                            |
| `audience`    | `aud` | Human  | Define audience and goals → Audience Profile                                                                                              |
| `plan`        | `pl`  | **AI** | Create documentation plan + traceability matrix                                                                                           |
| `review-plan` | `rp`  | Human  | Review and approve the documentation plan                                                                                                 |
| `sitemap`     | `sm`  | **AI** | Create sitemap (folder structure, navigation, cross-links)                                                                                |
| `voice`       | `vc`  | **AI** | UX content standards (voice chart, text patterns, scorecard)                                                                              |
| `draft`       | `dr`  | **AI** | Draft documentation using content type templates (in `drafts/{source-locale}/`)                                                          |
| `edit`        | `ed`  | **AI** | Self-review edit (5 passes)                                                                                                               |
| `walkthrough` | `wt`  | **AI** | Product walkthrough (browser verification + screenshots)                                                                                  |
| `record`      | `rec` | **AI** | Record short tutorial videos from `<!-- VIDEO ... -->` markers (browser screen recording)                                                 |
| `validate`    | `val` | **AI** | Run walkthrough test cases only (no screenshots, no doc fixes)                                                                            |
| `test`        | `t`   | **AI** | Create or update walkthrough test cases from existing docs                                                                                |
| `verify`      | `vf`  | **AI** | Run all verification checks (edit passes + voice score + link/placeholder audit). Accepts optional doc path or glob to scope to one area. |
| `peer-review` | `pr`  | Human  | Peer review                                                                                                                               |
| `tech-review` | `tr`  | Human  | Technical review (optional)                                                                                                               |
| `incorporate` | `inc` | **AI** | Incorporate review feedback into documents                                                                                                |
| `categorize`  | `cat` | **AI** | Generate `_category_.json` files from sitemap; normalize titles; flag undocumented folders (Docusaurus preset)                            |
| `deploy`      | `dep` | **AI** | Copy/sync workspace into target host project with transforms (frontmatter, image refs, etc.). Supports `--dry-run` and `--target <path>`. |
| `publish`     | `pub` | Human  | Approve and publish (typically `git commit && push` on the deploy target after `deploy`)                                                  |

### Examples

```
/docsmith help
/docsmith init                                  # one-time setup; creates .docsmithrc.yaml
/docsmith start MyProduct
/docsmith plan MyProduct
/docsmith draft MyProduct
/docsmith validate MyProduct
/docsmith test MyProduct
/docsmith wt MyProduct
/docsmith rec MyProduct
/docsmith verify MyProduct
/docsmith verify MyProduct documentation/drafts/en/getting-started.md
/docsmith categorize MyProduct                  # Docusaurus _category_.json files
/docsmith deploy MyProduct --dry-run            # preview deploy
/docsmith deploy MyProduct                      # apply deploy to default_target
/docsmith deploy MyProduct --target ../other-site --dry-run
```

## Process Flow

```
init (AI, once) → audience (Human) → plan (AI) → review-plan (Human) → sitemap (AI) → voice (AI) → draft (AI) → edit (AI) → walkthrough (AI) → [record (AI, optional)] → peer-review (Human) → [tech-review (Human, optional)] → incorporate (AI) → [categorize (AI, Docusaurus preset)] → deploy (AI) → publish (Human)
```

**Decision points:**

- After `review-plan`: Plan approved? No → back to `plan`
- After `peer-review`: Passes review? No → back to `draft`
- After `peer-review`: Technical review needed? Yes → `tech-review` → `incorporate`

## Configuration

Every docsmith command (after `init`) reads `.docsmithrc.yaml` from the current working directory before doing anything. This file scopes paths and prevents writes outside the configured workspace.

If `.docsmithrc.yaml` is missing when a non-`init`/`help` command is invoked, **stop and run `init` first**.

Key fields (full schema in [.docsmithrc.example.yaml](.docsmithrc.example.yaml)):

- `product.slug` — image namespace (e.g. `mycloud` → `/img/mycloud/...`). Globally unique.
- `locales.source` / `locales.targets` — source language and translation targets. v1.2.0 scaffolds folder structure; auto-translation deferred to v1.6.
- `paths.workspace` — where working files live (default `documentation/`).
- `deploy.default_target` — host project path. Empty = standalone.
- `deploy.preset` — `standalone` (default) or `docusaurus`. See [presets/](presets/).

Presets define defaults for `deploy` and `categorize`. Read the relevant preset file (`presets/<preset>.yaml`) at the start of any command that produces deploy-bound output.

## Path Scoping (Safety)

Commands MUST NOT write outside the workspace and configured deploy target. This includes:

- Workspace writes: only inside `paths.workspace` (default `documentation/`)
- Deploy writes: only inside `deploy.default_target` (or `--target <path>` override)
- Audit writes: inside `paths.deployments`
- Reading: anywhere is allowed (e.g., reading host project's CLAUDE.md, docusaurus.config)

Before any write, validate the absolute target path is inside one of the allowed roots. Reject otherwise with a clear error. This is the core safety guarantee for running docsmith inside or alongside an existing project.

## Command Details

### `help`

Display the command reference table above. No other action.

### `init` (AI)

**Purpose**: Initialize a docsmith workspace at the current directory. One-time setup.

**Inputs (interactive)**:
- Product slug (lowercase, kebab-case, globally unique across products sharing a Docusaurus target)
- Product name (display)
- Source locale (default `en`)
- Target locales (optional, comma-separated; e.g. `vi,jp`)
- Deploy preset (`standalone` or `docusaurus`)
- Deploy target path (if Docusaurus): existing project path or `.` for in-place

**Behavior**:

1. If `.docsmithrc.yaml` already exists, abort with message; suggest `init --force` (overwrites only after showing diff and explicit human confirmation)
2. If running in a directory that already contains files, list them and confirm — never silently overwrite anything
3. Detect host project context to suggest preset:
   - `docusaurus.config.{js,ts,mjs}` present → suggest `docusaurus` preset
   - Otherwise → `standalone`
4. If preset is `docusaurus` and a target is provided, run a non-destructive **target inspection**:
   - Read `<target>/CLAUDE.md` if present
   - Parse `<target>/docusaurus.config.*` for `i18n`, `docs`, `static` paths
   - Show user what was detected, confirm before writing config
5. Write `.docsmithrc.yaml` with chosen values (use [.docsmithrc.example.yaml](.docsmithrc.example.yaml) as the template, comment lines explaining each field preserved)
6. Create workspace directories:
   ```
   documentation/
   ├── plan/
   ├── standards/
   ├── drafts/<source-locale>/         # always created
   ├── drafts/<each-target-locale>/    # empty placeholder, README explains
   ├── walkthrough/
   │   ├── test-cases/
   │   ├── video-plan/
   │   └── executions/
   ├── images/
   └── videos/
   deployments/                        # audit trail folder, empty
   ```
7. For each target locale folder, write a `README.md` explaining: this folder is empty until v1.6 auto-translation; until then, copy from `drafts/<source>/` and translate manually
8. Print next-step guidance: usually `/docsmith audience <product>`

**Does NOT**:
- Touch any file outside the chosen workspace
- Run any other docsmith command automatically
- Modify the deploy target — that's `deploy`'s job

### `start`

Begin the process. Explain that the first step (`audience`) is human-owned, show what needs to be done, and provide the audience profile template.

### `audience` (Human)

Provide the user with:

1. Instructions for defining audience and goals (gather existing knowledge, define user goals, identify target users, outline user needs, identify competitors, condense findings into personas and user stories)
2. The template: [templates/AUDIENCE_PROFILE_TEMPLATE.md](templates/AUDIENCE_PROFILE_TEMPLATE.md)
3. Ask user to provide the completed profile or answer questions interactively

### `plan` (AI)

**Requires**: Completed audience profile
Read [process-reference.md](process-reference.md) § STEP-002. Use templates:

- [templates/DOCUMENTATION_PLAN_TEMPLATE.md](templates/DOCUMENTATION_PLAN_TEMPLATE.md)
- [templates/TRACEABILITY_MATRIX_TEMPLATE.md](templates/TRACEABILITY_MATRIX_TEMPLATE.md)

### `review-plan` (Human)

Present the documentation plan for human review. Explain approval criteria:

- Plan covers all critical user journeys
- Content types are appropriate
- Priorities are clear
  Ask if approved or if feedback is needed (→ re-run `plan` with feedback).

### `sitemap` (AI)

**Requires**: Approved documentation plan
Read [process-reference.md](process-reference.md) § STEP-004. Define folder structure, navigation sidebar, cross-links, and reading order.

### `voice` (AI)

**Requires**: Approved documentation plan + audience profile
Read [subprocess-010a.md](subprocess-010a.md). Use templates:

- [templates/VOICE_CHART_TEMPLATE.md](templates/VOICE_CHART_TEMPLATE.md)
- [templates/UX_TEXT_PATTERNS_TEMPLATE.md](templates/UX_TEXT_PATTERNS_TEMPLATE.md)
- [templates/UX_CONTENT_SCORECARD_TEMPLATE.md](templates/UX_CONTENT_SCORECARD_TEMPLATE.md)

Runs the subprocess: define product principles (ask human) → build voice chart → review (ask human) → define UX text patterns → create content scorecard → approve (ask human).

### `draft` (AI)

**Requires**: Approved plan, sitemap, voice chart, UX text patterns, `.docsmithrc.yaml`
Read [process-reference.md](process-reference.md) § STEP-005. Use templates from:

- [templates/CONTENT_TYPE_TEMPLATES.md](templates/CONTENT_TYPE_TEMPLATES.md)

Draft each document in the plan. **Output location**: `documentation/drafts/<locales.source>/<path>.md`. Drafts are written ONLY in the source locale; target locale folders stay empty until v1.6 auto-translation (or until user manually translates).

Use `![Caption](https://placehold.co/600x400)` for screenshot placeholders, following the rules in [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md).

**Image reference style**: use **workspace-absolute** paths in markdown — `![Caption](/images/<feature>/<asset>.png)`. Walkthrough writes captured images to `documentation/images/<feature>/<asset>.png` and updates these refs. The `deploy` command (Docusaurus preset) rewrites `/images/X` → `/img/<product.slug>/X` and copies image files to target's static folder.

**Screenshot density** (mandatory, applies to every procedure-style doc):

- One screenshot at the **starting state** of each procedure
- One screenshot after **each step that changes the UI in a visible way** (modal opens, form filled, item selected, page transition)
- One screenshot at the **success state** at the end

**Caption rules** (critical for screenshots to match the doc):

- Describe **what is on screen**, not what the user is doing. ✗ "Click the Create button" ✓ "Compute Instances list with empty state and Create button visible at top right"
- Be specific about **data and state**. ✗ "Instance list" ✓ "Instance list with 3 running instances"
- Reference UI elements by their **actual label**, not by color or position
- See [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) for full guidelines and examples

For documents that benefit from short tutorial videos (Quick Start tours, complex multi-step procedures with motion, end-to-end tutorials), insert video markers using the convention in [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md). Markers are processed by the `record` command, not `walkthrough`.

### `edit` (AI)

**Requires**: Draft documents
Read [process-reference.md](process-reference.md) § STEP-006. Perform 5 editing passes:

1. Technical accuracy
2. Completeness
3. Structure
4. Clarity and brevity
5. Voice compliance (score with content scorecard, minimum 70% to pass)

### `walkthrough` (AI) — Full walkthrough

**Requires**: Edited documents + live product access
Read [process-reference.md](process-reference.md) § STEP-007 and [tools-reference.md](tools-reference.md). Full workflow:

1. Create test cases from docs → [templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md](templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md)
2. Create screenshot capture plan
3. Execute test cases + capture screenshots in single browser pass
4. Replace all `placehold.co` placeholders with captured images
5. Fix any doc failures found
6. Record results → [templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md](templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md)
7. Verify zero `placehold.co` references remain

**Caption-driven matching**: Screenshot content is matched to the placeholder via (1) the caption text, (2) surrounding section/step context, (3) the ordered capture plan. Vague captions are the #1 source of mismatches — if a caption is missing data state or refers to UI by color/position, capture content may be wrong. Before executing the capture plan, present it for human review and flag any caption that fails the rules in [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md).

**Scope**: Skips `<!-- VIDEO ... -->` markers entirely. Use `record` for those.

### `record` (AI) — Tutorial videos (optional)

**Requires**: Edited documents containing `<!-- VIDEO ... -->` markers + live product access + screen-recording capability in browser automation.

**When to run**: Optional. Only when documents contain video markers. Run after `walkthrough` so that screenshots and caption checks are settled first. Re-run independently when product UI changes invalidate existing videos.

**Video markers**: Authored at the `draft` stage using the convention in [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md). Markers are HTML comment blocks with structured fields (id, duration, start state, actions, end state, highlight, pacing).

**Workflow**:

1. Scan all docs for `<!-- VIDEO ... -->` markers and extract structured fields
2. Build a video capture plan ordered by navigation sequence → [templates/WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md](templates/WALKTHROUGH_VIDEO_PLAN_TEMPLATE.md)
3. **Validate markers**: every marker must have id, start, actions, end. Missing fields → block execution and report
4. **Reuse test cases where applicable**: if a marker covers the same flow as an existing test case, link them so failures are detected in both
5. Execute each marker in browser:
   - Navigate to the start state described in the marker
   - Begin screen recording
   - Perform the actions in order, respecting pacing hints
   - Stop recording when end state is reached
   - Save raw video to `videos/raw/<id>.mov`
   - Transcode to web-friendly `videos/<id>.mp4` (max 720p, ≤ 5 MB target)
6. **Replace markers** in docs: substitute the `<!-- VIDEO ... -->` block with a video embed pointing to `videos/<id>.mp4`. The exact embed format depends on the documentation platform — use a `<video>` tag for plain HTML, or a platform-specific shortcode/component if configured. Keep the original marker as a sibling HTML comment so re-record can find it later.
7. Update execution record with each video's status, duration, file size, last recorded date
8. Verify zero unprocessed `<!-- VIDEO` markers remain (those without recordings)

**Re-record behavior**: When run on docs that already have replaced markers, look for the preserved comment metadata to identify what to re-shoot. Do not re-record videos whose markers and product UI haven't changed (compare against last recorded date + UI version stored in execution record).

**Does NOT**: Capture screenshots, run test cases against screenshots, modify text content of docs.

### `validate` (AI) — Verification only

**Requires**: Existing test cases + live product access
Run existing walkthrough test cases against the live product **without** modifying documentation or capturing screenshots. Purpose: quick re-check after product changes.

1. Read existing test cases from `docs/walkthrough/test-cases/`
2. Execute each test case against the live product using [tools-reference.md](tools-reference.md)
3. Record pass/fail results → [templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md](templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md)
4. Report failures with details (expected vs actual) but do NOT auto-fix docs
5. Recommend next steps for any failures

### `test` (AI) — Create/update test cases

**Requires**: Existing documentation
Extract verifiable claims from documentation and generate test cases. Does NOT execute them.

1. Scan all docs for verifiable claims (UI labels, navigation paths, URLs, procedures, API behaviors)
2. Generate test cases → [templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md](templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md)
3. Generate screenshot capture plan for any `placehold.co` placeholders
4. Save to `docs/walkthrough/test-cases/`
5. Report coverage summary by category

### `verify` (AI) — Comprehensive verification

**Requires**: Draft or edited documents. Optionally: voice chart, UX text patterns, content scorecard.

Run all verification checks on the entire documentation set or a specific subset.

**Scope**: Pass an optional file path or glob pattern after the product name to limit verification to specific docs. If omitted, verifies all docs.

**Checks performed** (in order):

| #   | Check                                                                                            | Action on Failure                     |
| --- | ------------------------------------------------------------------------------------------------ | ------------------------------------- |
| 1   | **Technical accuracy** — instructions produce promised results, jargon explained, names correct  | List issues with file:line references |
| 2   | **Completeness** — no `[TODO]`/`[TBD]` remaining, all environments covered                       | List gaps                             |
| 3   | **Structure** — follows content type template, headers logical, prerequisites/next-steps present | List deviations                       |
| 4   | **Clarity & brevity** — no unnecessary words, consistent terms, no idioms/biased language        | List suggestions                      |
| 5   | **Voice compliance** — score against voice chart using content scorecard (min 70%)               | Report score per doc                  |
| 6   | **Link audit** — all internal cross-references resolve, no broken links                          | List broken links                     |
| 7   | **Placeholder audit** — search for `placehold.co`, `[TODO]`, `[TBD]`, `[PLACEHOLDER]`            | List remaining placeholders           |
| 8   | **Traceability check** — every doc maps to a user story, no orphan docs                          | Report gaps                           |
| 9   | **Sitemap consistency** — docs match sitemap paths, navigation order intact                      | List mismatches                       |
| 10  | **Template compliance** — each doc follows its declared content type template structure          | List missing sections                 |

**Output**: Verification report saved to `docs/walkthrough/verify-report-[YYYY-MM-DD].md`

**Does NOT**: Modify documentation, execute browser-based walkthrough, or capture screenshots.

### `peer-review` (Human)

Provide the user with:

1. What kind of feedback to give (structural, technical, clarity)
2. How to provide feedback
3. After feedback received, ask: approved? → `publish` or `tech-review`. Not approved? → back to `draft`.

### `tech-review` (Human)

Explain when technical review is needed (multi-system integrations, unfamiliar domains, safety-critical). Ask user to provide SME feedback.

### `incorporate` (AI)

**Requires**: Review feedback
Read [process-reference.md](process-reference.md) § STEP-010. Process feedback one reviewer at a time, prioritize what helps the user most for contradictions.

### `categorize` (AI) — Docusaurus categories

**Requires**: `deploy.preset = docusaurus`, sitemap, drafts ready

Generate `_category_.json` files for documented folders, and normalize sitemap titles. Can run standalone or as part of `deploy`. Read [deploy-reference.md](deploy-reference.md) § "Categorize subcommand" for full logic.

**Behavior**:

1. Read `documentation/plan/sitemap.md`
2. Walk target docs folder structure (or workspace drafts if running before deploy)
3. For each folder in sitemap:
   - Generate `_category_.json` with normalized title from sitemap and order from sitemap position
   - Use `link.type: generated-index` so Docusaurus auto-creates a category landing page
4. For folders NOT in sitemap: list them, do not touch — these are "undocumented folders". Output guidance on how to hide (see [deploy-reference.md](deploy-reference.md) § "Limitations")
5. Normalize titles in sitemap itself: title-case rules consistent across siblings (acronyms uppercase, articles lowercase mid-title, first/last word capitalized)
6. Report any sibling-title inconsistencies

**Does NOT**: write to `sidebars.js` unless `generate_sidebars: true` is explicitly set.

### `deploy` (AI) — Sync to host project

**Requires**: `.docsmithrc.yaml` configured with a deploy target, drafts complete, walkthrough done (no `placehold.co` placeholders remain unless `validation.fail_on_placeholder: false`)

Copy/sync workspace artifacts into the configured host project with transforms (frontmatter injection, image ref rewriting, MDX escaping). Read [deploy-reference.md](deploy-reference.md) for full logic.

**Flags**:
- `--dry-run` — detect + plan, print, exit. No writes.
- `--target <path>` — override `deploy.default_target` for this run
- `--force` — override conflicts on files where target differs from workspace
- `--locale <locale>` — deploy only one locale (default: all configured locales with non-empty drafts)

**High-level workflow**:

1. **Detect** target project context (CLAUDE.md, docusaurus.config.*, folder signals). See [deploy-reference.md](deploy-reference.md) § "Detection phase". On first deploy to a target, REQUIRE human confirmation of detected config.
2. **Plan** the file actions (create / update / skip / conflict). See [deploy-reference.md](deploy-reference.md) § "Plan phase".
3. **Show plan** to user. If `--dry-run`, exit here.
4. **Apply** if no unresolved conflicts:
   - Create `deployments/<timestamp>-<target-name>/` audit folder
   - Execute file actions in order
   - For each markdown file: read → transform (frontmatter, image refs, MDX escape) → write
   - For binaries: copy bytes
   - Generate `_category_.json` files via `categorize` if `generate_categories: true`
5. **Save** manifest, target-config snapshot, diff summary, pre-deploy hashes (for manual rollback)
6. **Print** summary

**Path scoping**: validate every target absolute path is inside the configured target root before writing. Reject otherwise. This is the safety net against AI-generated bugs ever writing into unrelated parts of the host project.

**Standalone preset**: deploy is a no-op. Print message that the workspace is the publishable artifact.

**In-place mode**: when `default_target = .`, workspace and target share a project. Mappings still apply (e.g. `documentation/drafts/en/foo.md` → `docs/foo.md`). Audit trail still created.

### `publish` (Human)

The `deploy` command writes to the target project but does NOT commit. Publication is the human step that follows.

Checklist:

1. Final sign-off on content quality (review the deploy summary, eyeball the diff in target project)
2. Coordinate with code/product release if applicable
3. In the target project: `git diff` → `git add` → `git commit` → `git push`
4. Trigger documentation platform build if not auto-triggered
5. Verify live site shows expected content
6. Announce to users

## File Organization

Docsmith uses a **workspace + target** model. The workspace is always present; the target depends on the deploy preset.

### Workspace (always)

Created by `init`. All docsmith commands write here:

```
<project-root>/
├── .docsmithrc.yaml                        # Config — read by every command
├── documentation/                          # Workspace root (paths.workspace)
│   ├── plan/
│   │   ├── audience-profile.md
│   │   ├── documentation-plan.md
│   │   ├── traceability-matrix.md
│   │   └── sitemap.md
│   ├── standards/
│   │   ├── voice-chart.md
│   │   ├── ux-text-patterns.md
│   │   ├── content-scorecard.md
│   │   └── screenshot-policy.md
│   ├── drafts/
│   │   ├── en/                             # Source locale (locales.source)
│   │   │   └── [doc-name].md
│   │   ├── vi/                             # Target locales (locales.targets)
│   │   │   └── README.md                   # Until v1.6 auto-translation
│   │   └── jp/
│   │       └── README.md
│   ├── walkthrough/
│   │   ├── test-cases/
│   │   ├── video-plan/
│   │   └── executions/
│   ├── images/
│   │   └── [feature]/[asset-name].png      # Refs use /images/[feature]/...
│   └── videos/
│       ├── raw/
│       │   └── [video-id].mov              # NOT committed; .gitignore'd
│       └── [video-id].mp4                  # Committed
└── deployments/                            # Audit trail (paths.deployments)
    └── 2026-04-26-103044-mycloud-docusaurus/
        ├── manifest.yaml
        ├── target-config.yaml
        ├── diff.md
        └── pre-deploy-state.txt
```

### Target — Docusaurus preset

After `deploy`, the target project receives:

```
<deploy.default_target>/
├── docs/                                   # docusaurus.docs_path
│   ├── _category_.json                     # Generated by categorize
│   ├── getting-started.md                  # From drafts/en/getting-started.md
│   └── instances/
│       ├── _category_.json
│       └── create.md
├── i18n/                                   # docusaurus.i18n_path (multi-locale)
│   └── vi/
│       └── docusaurus-plugin-content-docs/
│           └── current/
│               ├── getting-started.md      # From drafts/vi/getting-started.md
│               └── instances/create.md
└── static/                                 # docusaurus.static_path
    ├── img/
    │   └── mycloud/                        # docusaurus.image_subpath
    │       └── instances/create-form-filled.png
    └── videos/
        └── mycloud/                        # docusaurus.video_subpath
            └── instance-create-tour.mp4
```

Image refs in deployed markdown are absolute: `/img/mycloud/instances/create-form-filled.png`.

### Target — standalone preset

No deploy. Workspace IS the publishable artifact. Drafts use relative refs `/images/[feature]/[asset].png` from workspace root.

### In-place mode

When `deploy.default_target = .`, workspace and target are the same project. The `documentation/` workspace and the `docs/` Docusaurus folder coexist as siblings. `deploy` still produces audit trail in `deployments/`.
