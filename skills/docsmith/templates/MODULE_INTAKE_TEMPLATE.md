<!--
  MODULE INTAKE — Per-module override of project-level config.

  This file is for ONE module/feature area within the project. It overrides
  project.md when the same field is set here. Anything NOT specified here
  inherits from project.md.

  Sections marked "Advanced" are COLLAPSED by default. Most modules only
  need to fill section 1 (identity) and section 2 (scope). Other sections
  inherit from project automatically.

  Save and run /docsmith run <module-name> to generate docs for this module.
-->

# Module Intake — `Module Name Here`

## 1. Module identity (*)

- Module slug (lowercase, kebab-case): `your-module-slug`
- Module display name: `Your Module Name`
- Folder in target docs: `your-module-slug`  (defaults to slug)

## 2. Scope (*)

### Feature 1

- Feature name: `e.g., Create instance`
- Content types needed:
  - [ ] Tutorial (learning-focused, hand-holding)
  - [ ] How-to (task-focused, assumes context)
  - [ ] Reference (lookup tables, parameter lists)
  - [ ] Concept (explanation of how something works)

<details>
<summary><b>Add more features</b> (skip if only 1 feature)</summary>

Copy the "Feature N" block below for each additional feature. Increment N.

### Feature 2

- Feature name: ``
- Content types needed:
  - [ ] Tutorial
  - [ ] How-to
  - [ ] Reference
  - [ ] Concept

### Feature 3

- Feature name: ``
- Content types needed:
  - [ ] Tutorial
  - [ ] How-to
  - [ ] Reference
  - [ ] Concept

</details>

<details>
<summary><b>Out of scope</b> (optional — explicit "do NOT document" list)</summary>

Things this module explicitly does NOT cover. Helps AI avoid hallucinating tangential content.

`e.g., Internal admin features, deprecated v1 APIs`

</details>

<details>
<summary><b>Advanced — module priority</b> (default: 3)</summary>

Priority within sitemap (1 = highest, 5 = lowest):

`3`

</details>

<details>
<summary><b>Advanced — voice override</b> (using project default)</summary>

Most modules inherit project voice. Override only when this module needs different style (e.g., admin module wants formal while user module is casual).

Override tone:
- [x] Inherit from project (default)
- [ ] Casual
- [ ] Friendly-professional
- [ ] Technical-direct
- [ ] Formal

Override perspective:
- [x] Inherit from project (default)
- [ ] Second-person
- [ ] First-person plural
- [ ] Third-person

</details>

<details>
<summary><b>Advanced — module-specific knowledge sources</b> (skip if none)</summary>

Sources here ADD to project-level sources for this module. Useful when a module has its own PRD, design doc, or code area.

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

</details>

<details>
<summary><b>Advanced — walkthrough setup</b> (using project credentials)</summary>

Most modules use project credentials. Override when this module needs different test account.

Test account:
- [x] Inherit from project (default)
- [ ] Override:
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

</details>

<details>
<summary><b>Advanced — special handling</b> (skip if none)</summary>

Sensitive fields to redact in screenshots:
- [x] None (default)
- [ ] User email addresses
- [ ] API keys / tokens (any visible)
- [ ] Account IDs
- [ ] Custom: ``

Module status:
- [x] Active (default — processed by /docsmith run)
- [ ] Paused (skip in /docsmith run, kept for later)
- [ ] Archived (do not process; do not delete from target on deploy --sync-deletes)

</details>

<details>
<summary><b>Advanced — sitemap sections for this module</b> (using project pattern)</summary>

Project pattern (A/B/C from project.md § 6) determines section ORDER. This list determines which sections this module INCLUDES. AI auto-suggests sections based on your scope above; you can adjust here.

For this module, include:
- [x] overview          (always required — auto-ticked)
- [x] initial-setup     (default — auto-ticked; untick if no specific setup)
- [x] quickstarts       (auto-ticked if any feature has tutorial or how-to)
- [ ] tutorials         (auto-ticked if any feature has tutorial content type)
- [ ] guides            (alternative to quickstarts — pick one based on project pattern)
- [ ] concepts          (only if non-obvious concepts to explain)
- [ ] dashboard         (only if module has dashboard or report view)
- [x] reference         (auto-ticked if any feature has reference content type)
- [ ] api-reference     (only if module has stable API)
- [ ] glossary          (auto-ticked if module has domain-specific terms)
- [x] troubleshooting   (default — auto-ticked)

Display name overrides for this module (rare — usually inherit project):
- overview: ``
- initial-setup: ``
- quickstarts: ``
- tutorials: ``
- guides: ``
- reference: ``
- glossary: ``
- troubleshooting: ``

</details>

<details>
<summary><b>Advanced — media override</b> (using project defaults)</summary>

Most modules inherit project media policy. Override only when this module is fundamentally different (compliance needs human voiceover, mobile module needs different aspect ratio, etc.).

Screenshot density:
- [x] Inherit from project (default)
- [ ] Override per content type:
  - Tutorial: ``
  - How-to: ``
  - Reference: ``

Video density:
- [x] Inherit from project (default)
- [ ] Skip videos for this module entirely
- [ ] Override per content type:
  - Tutorial: ``
  - How-to: ``

Per-locale screenshots (this module only):
- [x] Inherit from project (default)
- [ ] Force per-locale capture (UI varies significantly across locales)
- [ ] Force source-only (UI identical across locales — skip per-locale)

Voiceover (this module only):
- [x] Inherit from project (default)
- [ ] Skip voiceover (silent only, even if project has AI voice)
- [ ] Override TTS voice ID for this module: ``

</details>

---

<!--
  Validation when /docsmith run <module> triggers:
  - Module identity fields filled
  - At least 1 feature with at least 1 content type checked
  - Advanced sections: defaults applied or inherited from project if not customized

  Missing critical → stop. Missing advanced → inherit from project intake.
-->
