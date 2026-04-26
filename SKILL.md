---
name: docsmith
description: "Create, verify, translate, update, and deploy documentation following the PRC-010 process. Standalone-first workspace; deploys to Docusaurus or other targets via preset. Re-run safe with update/overwrite/side-by-side gate, KB inheritance, drift detection, multi-locale translation with per-block review, delete propagation. Use when asked to create, draft, plan, review, translate, update, deploy, or publish documentation. Supports commands: init, start, audience, plan, review-plan, sitemap, voice, draft, edit, walkthrough, record, validate, test, verify, peer-review, tech-review, incorporate, translate, publish, categorize, deploy."
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
| `translate`   | `tr`  | **AI** | Translate drafts from source locale to each `locales.targets`. Per-block review gate by default; `--auto-approve` for speed.              |
| `categorize`  | `cat` | **AI** | Generate `_category_.json` files from sitemap; normalize titles; flag undocumented folders (Docusaurus preset)                            |
| `deploy`      | `dep` | **AI** | Copy/sync workspace into target host project with transforms (frontmatter, image refs, etc.). Supports `--dry-run` and `--target <path>`. |
| `publish`     | `pub` | Human  | Approve and publish (typically `git commit && push` on the deploy target after `deploy`)                                                  |

### Examples

```
/docsmith help
/docsmith init                                  # one-time setup; creates .docsmithrc.yaml
/docsmith start MyProduct
/docsmith plan MyProduct                        # 1st run: fresh; 2nd+ run: re-run gate (update/overwrite/side-by-side/cancel)
/docsmith draft MyProduct                       # same gate per file
/docsmith validate MyProduct
/docsmith test MyProduct
/docsmith wt MyProduct                          # default: A → gate → B → C
/docsmith wt MyProduct --check                  # phase A only: drift report, exit
/docsmith wt MyProduct --apply                  # phase B+C: apply existing decisions, capture
/docsmith wt MyProduct --skip-drift             # phase C only: capture without drift detection
/docsmith rec MyProduct
/docsmith verify MyProduct
/docsmith verify MyProduct documentation/drafts/en/getting-started.md
/docsmith translate MyProduct                       # translate to all locales.targets, per-block review
/docsmith translate MyProduct --locale vi           # only Vietnamese
/docsmith translate MyProduct --auto-approve        # skip review gate (fast, less safe)
/docsmith categorize MyProduct                  # Docusaurus _category_.json files
/docsmith deploy MyProduct --dry-run            # preview deploy (always lists orphan target files)
/docsmith deploy MyProduct                      # apply deploy to default_target
/docsmith deploy MyProduct --sync-deletes --dry-run   # preview WITH delete propagation
/docsmith deploy MyProduct --sync-deletes             # delete orphan target files
/docsmith deploy MyProduct --target ../other-site --dry-run
```

## Process Flow

```
init (AI, once) → audience (Human) → plan (AI) → review-plan (Human) → sitemap (AI) → voice (AI) → draft (AI) → edit (AI) → walkthrough (AI) → [record (AI, optional)] → peer-review (Human) → [tech-review (Human, optional)] → incorporate (AI) → [translate (AI, required if locales.targets non-empty)] → [categorize (AI, Docusaurus preset)] → deploy (AI) → publish (Human)
```

When `locales.targets` is empty (single-locale project), `translate` is a no-op. When non-empty, `translate` MUST run before `deploy`. Deploy with missing translations emits a warning and lists incomplete locales.

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
- `deploy.sync_deletes` — propagate workspace deletions to target. Default `false`. Override per-run with `--sync-deletes`. (1.3.0+)
- `behavior.on_existing` — what to do when output already exists. `prompt` (default) / `update` / `overwrite` / `side-by-side`. (1.3.0+)
- `behavior.drift_default_action` — gate behavior in `walkthrough` drift detection. `prompt` (default) / `auto-apply-high-confidence`. (1.3.0+)

Presets define defaults for `deploy` and `categorize`. Read the relevant preset file (`presets/<preset>.yaml`) at the start of any command that produces deploy-bound output.

## Path Scoping (Safety)

Commands MUST NOT write outside the workspace and configured deploy target. This includes:

- Workspace writes: only inside `paths.workspace` (default `documentation/`)
- Deploy writes: only inside `deploy.default_target` (or `--target <path>` override)
- Audit writes: inside `paths.deployments`
- Reading: anywhere is allowed (e.g., reading host project's CLAUDE.md, docusaurus.config)

Before any write, validate the absolute target path is inside one of the allowed roots. Reject otherwise with a clear error. This is the core safety guarantee for running docsmith inside or alongside an existing project.

## Re-run Protocol (Safety)

Every command that produces output MUST check whether the output already exists before writing. If exists, trigger the 4-option gate:

1. **Update** — read existing as KB, propose deltas, apply per-item with user approval. Default for content artifacts.
2. **Overwrite** — discard existing (archived to `documentation/archive/<timestamp>/`), generate fresh. Requires explicit "OVERWRITE" confirmation.
3. **Side-by-side** — keep both, new file gets suffix `-v2`/`-v3`/etc. or `-<date>`. For comparison or major rewrite without commitment.
4. **Cancel** — abort.

**Default**: `prompt` (always ask). Configurable via `behavior.on_existing` in `.docsmithrc.yaml`.

**Critical rule for Update mode**: read existing content as canonical knowledge base. Propose deltas only (NEW / UPDATE / REMOVE / KEEP). Do NOT regenerate untouched sections "for consistency" — that loses team's manual edits. See [update-reference.md](update-reference.md) for full per-artifact merge logic and [templates/MERGE_DECISION_TEMPLATE.md](templates/MERGE_DECISION_TEMPLATE.md) for decision recording format.

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
7. For each target locale: scaffold an empty `documentation/standards/glossary.<locale>.yaml` from [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml) with locale-specific header. Glossary is optional but recommended; commands will run without it. The target draft folders (`drafts/<target-locale>/`) are created empty — `translate` populates them later.
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

**Re-run safety**: If `documentation-plan.md` or `traceability-matrix.md` already exist, trigger 4-option gate. Update mode: read existing as KB, propose deltas. See [update-reference.md](update-reference.md) § "Per-artifact merge logic".

### `review-plan` (Human)

Present the documentation plan for human review. Explain approval criteria:

- Plan covers all critical user journeys
- Content types are appropriate
- Priorities are clear
  Ask if approved or if feedback is needed (→ re-run `plan` with feedback).

### `sitemap` (AI)

**Requires**: Approved documentation plan
Read [process-reference.md](process-reference.md) § STEP-004. Define folder structure, navigation sidebar, cross-links, and reading order.

**Re-run safety**: If `sitemap.md` exists, trigger gate. Update mode preserves existing structure, adds new entries in logical position relative to siblings. See [update-reference.md](update-reference.md).

### `voice` (AI)

**Requires**: Approved documentation plan + audience profile
Read [subprocess-010a.md](subprocess-010a.md). Use templates:

- [templates/VOICE_CHART_TEMPLATE.md](templates/VOICE_CHART_TEMPLATE.md)
- [templates/UX_TEXT_PATTERNS_TEMPLATE.md](templates/UX_TEXT_PATTERNS_TEMPLATE.md)
- [templates/UX_CONTENT_SCORECARD_TEMPLATE.md](templates/UX_CONTENT_SCORECARD_TEMPLATE.md)

Runs the subprocess: define product principles (ask human) → build voice chart → review (ask human) → define UX text patterns → create content scorecard → approve (ask human).

**Re-run safety**: If standards files exist, trigger gate. Voice chart updates are particularly sensitive (team consensus); Update mode proposes additions only, never silent changes to existing rules. See [update-reference.md](update-reference.md).

### `draft` (AI)

**Requires**: Approved plan, sitemap, voice chart, UX text patterns, `.docsmithrc.yaml`
Read [process-reference.md](process-reference.md) § STEP-005. Use templates from:

- [templates/CONTENT_TYPE_TEMPLATES.md](templates/CONTENT_TYPE_TEMPLATES.md)

Draft each document in the plan. **Output location**: `documentation/drafts/<locales.source>/<path>.md`. Drafts are written ONLY in the source locale; target locale folders stay empty until v1.6 auto-translation (or until user manually translates).

**Re-run safety**: Before writing each draft file, check existence. If exists, trigger the 4-option gate (see § "Re-run Protocol" above). In Update mode, AI MUST read existing draft as canonical KB and propose deltas only — do not regenerate untouched sections. See [update-reference.md](update-reference.md) § "Per-artifact merge logic > drafts" for full rules and [templates/MERGE_DECISION_TEMPLATE.md](templates/MERGE_DECISION_TEMPLATE.md) for decision recording format.

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

### `walkthrough` (AI) — Verify, fix drift, capture

**Requires**: Edited documents + live product access
Read [process-reference.md](process-reference.md) § STEP-007, [tools-reference.md](tools-reference.md), and [update-reference.md](update-reference.md) § "Drift detection".

In v1.3.0, walkthrough is a 3-phase pipeline with a user gate between drift detection and apply:

#### Phase A — VERIFY (read-only)

`/docsmith walkthrough <product> --check`

1. Read drafts in scope, build/load test cases → [templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md](templates/WALKTHROUGH_TEST_CASE_TEMPLATE.md)
2. Open browser, run assertions against live product
3. **Do NOT modify drafts. Do NOT capture screenshots.**
4. Output drift report → [templates/DRIFT_REPORT_TEMPLATE.md](templates/DRIFT_REPORT_TEMPLATE.md) at `documentation/walkthrough/drift/<timestamp>/drift-report.md`
5. Pre-populate `decisions.yaml` with default decisions per item (auto-fix for HIGH, prompt for MEDIUM, skip for LOW)
6. Exit

#### Gate — User reviews and decides

User opens `decisions.yaml`, sets per-item decision: `auto-fix` / `manual-fix` (with custom_fix) / `product-bug` / `skip`. Optional shortcut flag `--auto-apply-high-confidence` skips this gate.

#### Phase B — UPDATE DOC (apply decisions)

`/docsmith walkthrough <product> --apply`

1. Load latest drift report + decisions.yaml
2. Apply auto-fix and manual-fix decisions to drafts
3. Skip product-bug and skip decisions
4. Save patches → `drift/<ts>/auto-fixes-applied.diff`
5. Roll updated product-bugs into `walkthrough/active-product-bugs.yaml` (cross-run tracker)

#### Phase C — CAPTURE

After Phase B (or directly when `--skip-drift` is passed):

1. Re-build capture plan from current drafts (now updated)
2. Present plan for human review (caption check per [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md))
3. Capture screenshots, replace `placehold.co` placeholders
4. Record execution → [templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md](templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md)
5. Verify zero `placehold.co` remain

#### Default mode (no flags)

`/docsmith walkthrough <product>` runs A → gate (interactive) → B → C in sequence. Gate prompts user inline rather than requiring a separate `--apply` invocation. Backward compatible with v1.2.x usage.

#### Flags

- `--check` — Phase A only; produce drift report and exit. Fast (~30s for moderate doc set).
- `--apply` — Phase B + C using existing decisions.yaml; skip detection.
- `--skip-drift` — Phase C only; capture without drift detection. For first-time runs or known-clean drafts.
- `--auto-apply-high-confidence` — skip gate; auto-apply HIGH confidence fixes immediately.

#### Caption-driven matching

Screenshot content is matched to placeholder via (1) caption text, (2) surrounding section/step context, (3) ordered capture plan. Vague captions are the #1 source of mismatches — if a caption is missing data state or refers to UI by color/position, capture content may be wrong. Captions that fail rules in [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) are flagged before capture.

#### Scope rules

- Skips `<!-- VIDEO ... -->` markers entirely (use `record`)
- Items in `active-product-bugs.yaml` are skipped from re-flagging until UI matches doc again (then auto-resolved)
- `walkthrough <product> <doc-glob>` scopes to specific docs (faster iteration)

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

### `translate` (AI) — Multi-locale translation

**Requires**: Drafts in source locale ready for review/translation. `locales.targets` non-empty.

**When**: Position is after `incorporate` and before `categorize`/`deploy`. For multi-locale projects, **translate is required** — deploy without complete translations emits a warning per missing locale.

Read [translate-reference.md](translate-reference.md) for full block-level rules, glossary integration, and review gate behavior.

**High-level workflow**:

1. Read `.docsmithrc.yaml` to determine source and target locales
2. Load glossary per target locale from `documentation/standards/glossary.<locale>.yaml` if present (see [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml))
3. For each target locale, for each source draft:
   - Re-run protocol check on target file (Update/Overwrite/Side-by-side/Cancel gate if exists)
   - Block-parse source (frontmatter, headings, prose, lists, tables, code blocks, image refs, video markers, links)
   - For each translatable block: apply glossary, generate AI translation, present per-block review gate
   - User decides per block: `y` approve / `e` edit / `s` skip (keep source) / `n` remove / `a` approve all remaining / `q` quit
   - Reassemble translated file with translation metadata in frontmatter
   - Write to `documentation/drafts/<target-locale>/<path>.md`
4. Save decisions → `documentation/archive/<timestamp>/translation-decisions-<locale>.yaml` (see [templates/TRANSLATION_DECISIONS_TEMPLATE.md](templates/TRANSLATION_DECISIONS_TEMPLATE.md))

**Preserve verbatim** (NEVER translated): code blocks, inline code, file paths, URLs, frontmatter `id`/`slug`, image src paths, video markers, MDX component tags, identifier references in prose.

**Translate**: headings, prose paragraphs, list items, table cells, image alt text, link text, frontmatter `title`/`description`/`keywords`, MDX component title attributes.

**Glossary** is the strongest signal — overrides AI's natural translation. Build iteratively: start without, add terms as you correct AI mistakes during per-block review.

**Flags**:
- `--locale <locale>` — translate only one target locale (default: all configured)
- `--auto-approve` — skip per-block review gate, apply AI translations directly. Marks files `translation_status: auto-approved`. Use for CI / glossary-confident bulk runs only.
- `--glob <pattern>` — translate only matching source files (e.g., `instances/*`)

**Re-run safety** (Update mode): when target file exists, AI compares source blocks against target via similarity matching. Unchanged source blocks → KEEP existing translation (preserve manual edits). Changed source → propose UPDATE with old/new translation side-by-side. Removed source → propose REMOVE. New source → propose NEW.

**Walkthrough on translated docs**: `walkthrough --locale <locale>` runs verification against translated drafts. Requires product UI to be in target locale (most products have a language switcher).

**Limitations** (1.4.0 minimal):
- No drift tracking when source updates after translation (re-run `translate` in Update mode is the workaround; first-class `--check` mode is on 1.6.x roadmap)
- Per-locale image namespacing not first-class (1.5+)
- Voice chart authored in source locale only — translated docs may have tone drift
- LLM cost: each block is one call. Use Update mode for incremental change.

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
- `--sync-deletes` — propagate workspace deletions to target. Without this flag, orphan target files are reported but never deleted. See [update-reference.md](update-reference.md) § "Delete propagation".

**High-level workflow**:

1. **Translation completeness check** (1.4.0+): if `locales.targets` is non-empty, verify each target locale has translated drafts for all source files. Missing translations → warn per locale with list of untranslated files. User can proceed (deploy partial), abort, or run `translate` first.
2. **Detect** target project context (CLAUDE.md, docusaurus.config.*, folder signals). See [deploy-reference.md](deploy-reference.md) § "Detection phase". On first deploy to a target, REQUIRE human confirmation of detected config.
3. **Plan** the file actions (create / update / skip / conflict / delete-if-sync). See [deploy-reference.md](deploy-reference.md) § "Plan phase".
4. **Show plan** to user. Always include "Orphan files in target" section listing files in target with no source in workspace. If `--dry-run`, exit here.
5. **Apply** if no unresolved conflicts:
   - Create `deployments/<timestamp>-<target-name>/` audit folder
   - Execute file actions in order
   - For each markdown file: read → transform (frontmatter, image refs, MDX escape) → write
   - For binaries: copy bytes
   - Generate `_category_.json` files via `categorize` if `generate_categories: true`
   - If `--sync-deletes`: backup orphans to `deployments/<ts>/deleted/`, then delete from target
6. **Save** manifest (including any deletes), target-config snapshot, diff summary, pre-deploy hashes (for manual rollback)
7. **Print** summary

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
│   │   ├── screenshot-policy.md
│   │   ├── glossary.vi.yaml             # 1.4.0+ — per-locale translation glossary
│   │   └── glossary.jp.yaml
│   ├── drafts/
│   │   ├── en/                             # Source locale (locales.source)
│   │   │   └── [doc-name].md               # Authored here
│   │   ├── vi/                             # Target locale (locales.targets)
│   │   │   └── [doc-name].md               # 1.4.0+ — translated by `translate` command
│   │   └── jp/
│   │       └── [doc-name].md
│   ├── walkthrough/
│   │   ├── test-cases/
│   │   ├── video-plan/
│   │   ├── executions/
│   │   ├── drift/                          # 1.3.0+ — drift detection runs
│   │   │   └── 2026-04-26-103044/
│   │   │       ├── drift-report.md
│   │   │       ├── decisions.yaml
│   │   │       └── auto-fixes-applied.diff
│   │   └── active-product-bugs.yaml        # 1.3.0+ — cross-run tracker
│   ├── archive/                            # 1.3.0+ — re-run protocol backups
│   │   └── 2026-04-26-103044/
│   │       ├── [original-filename].md
│   │       └── merge-decisions.yaml
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
        ├── pre-deploy-state.txt
        └── deleted/                        # 1.3.0+ — backup of removed target files
            └── docs/instances/legacy.md      (only when --sync-deletes used)
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
