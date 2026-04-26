---
name: docsmith
description: "Create documentation following the PRC-010 process. Use when asked to create, draft, plan, review, or publish documentation for a product or project. Guides through audience analysis, documentation planning, sitemap creation, UX content standards, drafting, self-review, and product walkthrough. Supports commands: start, audience, plan, review-plan, sitemap, voice, draft, edit, walkthrough, validate, test, verify, peer-review, tech-review, incorporate, publish."
---

# Docsmith — PRC-010: Documentation Creation Process

Standardized process for systematically creating, reviewing, and publishing documentation. Based on _Docs for Developers_ (Bhatti et al., 2021) and _Strategic Writing for UX_ (Podmajersky, 2019).

## Commands

Parse `$ARGUMENTS` to determine which command to run. If no command is given or the command is `help`, show the command reference table below.

| Command       | Alias | Owner  | Description                                                                                                                               |
| ------------- | ----- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `help`        | `h`   | —      | Show this command reference                                                                                                               |
| `start`       | —     | —      | Begin the full process from the first human step                                                                                          |
| `audience`    | `aud` | Human  | Define audience and goals → Audience Profile                                                                                              |
| `plan`        | `pl`  | **AI** | Create documentation plan + traceability matrix                                                                                           |
| `review-plan` | `rp`  | Human  | Review and approve the documentation plan                                                                                                 |
| `sitemap`     | `sm`  | **AI** | Create sitemap (folder structure, navigation, cross-links)                                                                                |
| `voice`       | `vc`  | **AI** | UX content standards (voice chart, text patterns, scorecard)                                                                              |
| `draft`       | `dr`  | **AI** | Draft documentation using content type templates                                                                                          |
| `edit`        | `ed`  | **AI** | Self-review edit (5 passes)                                                                                                               |
| `walkthrough` | `wt`  | **AI** | Product walkthrough (browser verification + screenshots)                                                                                  |
| `record`      | `rec` | **AI** | Record short tutorial videos from `<!-- VIDEO ... -->` markers (browser screen recording)                                                 |
| `validate`    | `val` | **AI** | Run walkthrough test cases only (no screenshots, no doc fixes)                                                                            |
| `test`        | `t`   | **AI** | Create or update walkthrough test cases from existing docs                                                                                |
| `verify`      | `vf`  | **AI** | Run all verification checks (edit passes + voice score + link/placeholder audit). Accepts optional doc path or glob to scope to one area. |
| `peer-review` | `pr`  | Human  | Peer review                                                                                                                               |
| `tech-review` | `tr`  | Human  | Technical review (optional)                                                                                                               |
| `incorporate` | `inc` | **AI** | Incorporate review feedback into documents                                                                                                |
| `publish`     | `pub` | Human  | Approve and publish                                                                                                                       |

### Examples

```
/docsmith help
/docsmith start MyProduct
/docsmith plan MyProduct
/docsmith draft MyProduct
/docsmith validate MyProduct
/docsmith test MyProduct
/docsmith wt MyProduct
/docsmith rec MyProduct
/docsmith verify MyProduct
/docsmith verify MyProduct docs/drafts/getting-started.md
/docsmith verify MyProduct docs/drafts/api-*
```

## Process Flow

```
audience (Human) → plan (AI) → review-plan (Human) → sitemap (AI) → voice (AI) → draft (AI) → edit (AI) → walkthrough (AI) → [record (AI, optional)] → peer-review (Human) → [tech-review (Human, optional)] → incorporate (AI) → publish (Human)
```

**Decision points:**

- After `review-plan`: Plan approved? No → back to `plan`
- After `peer-review`: Passes review? No → back to `draft`
- After `peer-review`: Technical review needed? Yes → `tech-review` → `incorporate`

## Command Details

### `help`

Display the command reference table above. No other action.

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

**Requires**: Approved plan, sitemap, voice chart, UX text patterns
Read [process-reference.md](process-reference.md) § STEP-005. Use templates from:

- [templates/CONTENT_TYPE_TEMPLATES.md](templates/CONTENT_TYPE_TEMPLATES.md)

Draft each document in the plan. Use `![Caption](https://placehold.co/600x400)` for screenshot placeholders, following the rules in [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md).

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

### `publish` (Human)

Provide the user with a publication checklist:

1. Final sign-off on content quality
2. Coordinate with code/product release if applicable
3. Publish to documentation platform
4. Announce to users

## File Organization

All documentation outputs should be saved under a project-specific directory:

```
docs/
├── plan/
│   ├── audience-profile.md
│   ├── documentation-plan.md
│   ├── traceability-matrix.md
│   └── sitemap.md
├── standards/
│   ├── voice-chart.md
│   ├── ux-text-patterns.md
│   ├── content-scorecard.md
│   └── screenshot-policy.md
├── drafts/
│   └── [doc-name].md
├── walkthrough/
│   ├── test-cases/
│   ├── video-plan/
│   └── executions/
├── images/
│   └── [feature]/[asset-name].png
└── videos/
    ├── raw/
    │   └── [video-id].mov
    └── [video-id].mp4
```
