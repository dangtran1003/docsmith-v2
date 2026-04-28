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

---

<!--
  Validation when /docsmith run <module> triggers:
  - Module slug, display name, folder set
  - At least one feature with at least one content type checked
  - At least one persona either inherited from project or specified here

  Missing critical → stop. Missing nice-to-have → AI infers from project intake or applies defaults.
-->
