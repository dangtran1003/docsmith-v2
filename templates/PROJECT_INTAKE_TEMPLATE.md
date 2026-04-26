<!--
  PROJECT INTAKE — Source of truth for the whole documentation project.

  This is a fillable form. Edit by:
   - Ticking checkboxes:    [ ] → [x]
   - Filling backtick fields:  `value here`
   - Adding/removing source sections at the bottom.

  Save and run /docsmith run to start.

  Field references:
   - REQUIRED fields are marked with (*). Missing → /docsmith run will stop.
   - Backtick values must be inside the `...` markers.
   - Checkbox: tick exactly one in single-select groups; multiple in multi-select.

  Layered config:
   - This file applies to ALL modules in the project.
   - Module-level intake at documentation/intake/modules/<name>.md overrides
     specific fields per module. See /docsmith module <name>.
-->

# Project Intake — `Product Name Here`

## 1. Product (*)

- Product slug (lowercase, kebab-case, globally unique): `your-product-slug`
- Product display name: `Your Product`
- Product URL (for walkthrough): `https://console.example.com`
- One-line description: `What does this product do?`

## 2. Audience (*)

### Primary persona

- Role / job title: `e.g., DevOps Engineer`
- Technical level:
  - [ ] Low (general users, non-technical)
  - [ ] Medium (semi-technical, comfortable with web apps)
  - [ ] High (developers, sysadmins, command-line users)
- Primary goal (what they want to achieve with the product): `Why do they use this?`

### Secondary personas (optional)

<!-- Copy this block to add more personas. -->

- Role: ``
- Technical level:
  - [ ] Low
  - [ ] Medium
  - [ ] High
- Primary goal: ``

## 3. Languages

Source language (the language you draft in):
- [ ] English (en)
- [ ] Vietnamese (vi)
- [ ] Japanese (jp)
- [ ] Other: ``

Target languages (translate to — leave all unchecked for single-locale):
- [ ] None (single-locale project)
- [ ] English (en)
- [ ] Vietnamese (vi)
- [ ] Japanese (jp)
- [ ] Other: ``

## 4. Deploy (*)

Preset:
- [ ] Standalone (no deploy target — workspace IS the artifact)
- [ ] Docusaurus

If Docusaurus, target path (relative to project root or absolute): `../path-to-docusaurus`

On collision (when target file differs from workspace):
- [ ] Warn (default — block deploy, ask user to resolve)
- [ ] Skip (keep target as-is)
- [ ] Overwrite (replace target)
- [ ] Prompt (ask per file)

## 5. Voice and tone

Tone:
- [ ] Casual (like talking to a friend)
- [ ] Friendly-professional (default)
- [ ] Technical-direct (concise, no fluff)
- [ ] Formal (corporate, regulated industries)

Perspective:
- [ ] Second-person ("you create an instance")
- [ ] First-person plural ("we create an instance")
- [ ] Third-person ("the user creates an instance")

Reading level (target):
- [ ] 6th grade (broadest audience)
- [ ] 8th grade (default for technical docs)
- [ ] 10th grade
- [ ] College

Custom terms to AVOID (comma-separated): `e.g., utilize, leverage, synergize`

## 6. Credentials for product walkthrough

<!--
  Walkthrough automation needs to log into the product. Provide ENV VAR NAMES
  here, not actual passwords. Set the env vars before running /docsmith wt.
-->

- Username env var: `MYPRODUCT_TEST_USER`
- Password env var: `MYPRODUCT_TEST_PASS`
- (Optional) Multi-factor auth env var: ``
- (Optional) Test account note: `e.g., free tier, no production data`

## 7. Knowledge sources

<!--
  AI fetches content from these sources to inform drafts. Copy a "Source N"
  block to add more.

  Auth env vars are required for private sources:
   - Notion:        NOTION_TOKEN
   - GitHub private: GITHUB_TOKEN
   - Google Drive:   GOOGLE_DRIVE_TOKEN  (or GOOGLE_APPLICATION_CREDENTIALS)
   - URL fetch:      no auth (public only)
   - Local file:     no auth (path on disk)
-->

### Source 1

- Type:
  - [ ] Notion page
  - [ ] GitHub repo
  - [ ] Google Drive doc
  - [ ] Public URL
  - [ ] Local file
- URL or path or ID: ``
- Name (for reference): ``
- Auth env var (if private): ``
- Notes: ``

### Source 2

- Type:
  - [ ] Notion page
  - [ ] GitHub repo
  - [ ] Google Drive doc
  - [ ] Public URL
  - [ ] Local file
- URL or path or ID: ``
- Name: ``
- Auth env var (if private): ``
- Notes: ``

<!--
  Add more sources by copying the "Source N" block above and incrementing N.
-->

## 8. Auto-run behavior

When you run `/docsmith run`, where should AI pause for human review?

- [ ] After plan (before draft) — review the documentation plan first
- [ ] After draft (before walkthrough) — default; review drafts before product verification
- [ ] Before deploy — let everything generate, only review the deploy plan
- [ ] Never (full auto) — risky; only for trusted re-runs after first success

Drift detection action (when product UI changes):
- [ ] Prompt per item (default — safest)
- [ ] Auto-apply HIGH confidence fixes (faster, requires good caption discipline)

Translation review mode (only matters if target languages set above):
- [ ] Per-block (review each block individually — slowest, safest)
- [ ] Batch (review whole-file diff — default, faster)
- [ ] Auto-approve (no review — only for trusted glossary)

## 9. Module intake files (auto-managed)

Modules registered (created via `/docsmith module <name>`):

<!-- AI updates this list automatically when you run /docsmith module. -->

<!-- BEGIN MODULES LIST -->

(none yet)

<!-- END MODULES LIST -->

---

<!--
  Validation: when you run /docsmith run, AI checks:
  - All (*) sections have content
  - Backticks have non-default values
  - At least one source-type checkbox is ticked per Source N block (or no Source N)
  - Deploy preset is selected
  - Source language is selected

  Missing critical fields → AI stops with a list. Missing nice-to-have → defaults applied.
-->
