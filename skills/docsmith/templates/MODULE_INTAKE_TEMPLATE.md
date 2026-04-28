<!--
  MODULE INTAKE — Per-module override of project-level config.

  This file is for ONE module/feature area within the project. It overrides
  fields from documentation/intake/project.md when the same field is set here.

  Save and run /docsmith run <module-name> to generate docs for this module.

  Layered config rules:
   - Anything NOT specified here inherits from project.md
   - Anything specified here OVERRIDES project.md for this module
   - Sources are CUMULATIVE: project sources + module sources both used
-->

# Module Intake — `Module Name Here`

## 1. Module identity (*)

- Module slug (lowercase, kebab-case): `your-module-slug`
- Module display name: `Your Module Name`
- Priority (1 = highest, 5 = lowest): `1`
- Folder in target docs (relative to docs root): `your-module-slug`

## 2. Scope (*)

### Features to document

<!-- Copy "Feature N" block to add more. -->

#### Feature 1

- Feature name: `e.g., Create instance`
- Content types needed:
  - [ ] Tutorial (learning-focused, hand-holding)
  - [ ] How-to (task-focused, assumes context)
  - [ ] Reference (lookup tables, parameter lists)
  - [ ] Concept (explanation of how something works)
- Priority within module: `1`

#### Feature 2

- Feature name: ``
- Content types needed:
  - [ ] Tutorial
  - [ ] How-to
  - [ ] Reference
  - [ ] Concept
- Priority: ``

### Out of scope (explicitly NOT documented in this module)

`e.g., Internal admin features, deprecated v1 APIs`

## 3. Voice override (optional)

<!--
  Leave blank to inherit project-level voice settings.
  Tick to override for this module only.
-->

Override tone:
- [ ] Inherit from project (default)
- [ ] Casual
- [ ] Friendly-professional
- [ ] Technical-direct
- [ ] Formal

Override perspective:
- [ ] Inherit from project (default)
- [ ] Second-person
- [ ] First-person plural
- [ ] Third-person

## 4. Module-specific knowledge sources

<!--
  Sources here are ADDED to project-level sources for this module.
  Useful when a module has its own PRD, design doc, or code area.
-->

### Source 1

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

### Source 2

- Type:
  - [ ] Notion page
  - [ ] GitHub repo
  - [ ] Google Drive doc
  - [ ] Public URL
  - [ ] Local file
- URL or path or ID: ``
- Name: ``
- Auth env var: ``
- Notes: ``

## 5. Walkthrough setup

Test account: inherit from project intake (default)
- [ ] Yes, inherit
- [ ] No, override (specify env vars):
  - Username env var: ``
  - Password env var: ``

Pre-walkthrough setup script (optional, runs before browser session):

```bash
# Example: create test data
# mycloud instance create --name test-vm --image ubuntu-22.04
```

Post-walkthrough teardown (optional):

```bash
# Example: clean up
# mycloud instance delete test-vm
```

## 6. Special handling

Sensitive fields to redact in screenshots:
- [ ] None
- [ ] User email addresses
- [ ] API keys / tokens (any visible)
- [ ] Account IDs
- [ ] Custom: ``

Module status:
- [ ] Active (will be processed by /docsmith run)
- [ ] Paused (skip in /docsmith run, kept for later)
- [ ] Archived (do not process; do not delete from target on deploy --sync-deletes)

## 7. Sitemap sections (v1.5.4+)

<!--
  Tick which canonical section types this module includes. AI will follow the
  project's sitemap pattern when ordering them. See:
  templates/SITEMAP_PATTERNS_TEMPLATE.md for what each section type means.

  When you run /docsmith plan, AI checks this against the project pattern and
  warns about missing sections (e.g., if project pattern A includes
  'troubleshooting' but you didn't tick it here, AI suggests adding it).
-->

For this module, include:

- [ ] overview          (always required — auto-ticked even if you forget)
- [ ] initial-setup     (skip if module needs no specific setup beyond project-level)
- [ ] quickstarts       (group of short task-focused docs, 5-10 mins each)
- [ ] tutorials         (group of step-by-step learning docs with hand-holding)
- [ ] guides            (group of how-tos; alternative to quickstarts — pick one based on project pattern)
- [ ] concepts          (explanation docs for non-obvious concepts)
- [ ] dashboard         (only if module has a dedicated dashboard or report view)
- [ ] reference         (parameter tables, schema specs, lookup docs)
- [ ] api-reference     (only if module has a stable API)
- [ ] glossary          (term definitions specific to this module)
- [ ] troubleshooting   (common issues and fixes for this module)

### Display name overrides for this module (optional)

<!--
  Override how section names appear on the module's nav. Leave empty to use
  project-level default (which itself defaults to canonical names).
-->

- overview: ``
- initial-setup: ``
- quickstarts: ``
- tutorials: ``
- guides: ``
- concepts: ``
- dashboard: ``
- reference: ``
- api-reference: ``
- glossary: ``
- troubleshooting: ``

---

<!--
  Validation when /docsmith run <module> triggers:
  - Module slug, display name, folder set
  - At least one feature with at least one content type checked
  - At least one persona either inherited from project or specified here
  - At least one sitemap section ticked (overview is auto-added if missing)
  - Sections ticked are valid canonical types (per SITEMAP_PATTERNS_TEMPLATE)

  Missing critical → stop. Missing nice-to-have → AI infers from project intake or applies defaults.
  Sitemap inconsistencies (e.g., missing section that project pattern includes) → WARN, not stop.
-->
