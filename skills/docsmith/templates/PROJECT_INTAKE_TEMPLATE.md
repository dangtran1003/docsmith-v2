<!--
  PROJECT INTAKE — Source of truth for the whole documentation project.

  How to fill:
   - Tick checkboxes:    [ ] → [x]
   - Fill backtick fields: `value here`
   - Sections marked "Advanced — using defaults" are COLLAPSED by default.
     Most projects skip these. Expand only if you need to customize.

  Save and run /docsmith run to start.

  Field references:
   - REQUIRED fields are marked with (*). Missing → /docsmith run will stop.
   - Backtick values must be inside the `...` markers.
   - Checkbox: tick exactly one in single-select groups; multiple in multi-select.
-->

# Project Intake — `Product Name Here`

## 1. Product (*)

- Product slug (lowercase, kebab-case): `your-product-slug`
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
- Primary goal: `Why do they use this?`

<details>
<summary><b>Advanced — secondary personas</b> (skip if only one audience)</summary>

Add additional personas if you have clearly different audience segments. Don't add them just because you can — every persona makes content harder to write for everyone.

### Secondary persona 1

- Role: ``
- Technical level:
  - [ ] Low
  - [ ] Medium
  - [ ] High
- Primary goal: ``

</details>

## 3. Languages (*)

Source language (the language you draft in):
- [ ] English (en)
- [ ] Vietnamese (vi)
- [ ] Japanese (jp)
- [ ] Other: ``

Target languages (translate to — leave all unchecked for single-locale):
- [ ] None (single-locale project)
- [ ] Vietnamese (vi)
- [ ] Japanese (jp)
- [ ] Other: ``

<details>
<summary><b>Advanced — glossary settings</b> (using defaults)</summary>

Glossary location is fixed: `documentation/standards/glossary.<locale>.yaml`. Auto-created when you add target languages.

Glossary required for translation:
- [x] No, run translation even without glossary (default — AI uses general knowledge)
- [ ] Yes, fail translate if glossary missing for a target language

</details>

## 4. Deploy (*)

Preset:
- [ ] Standalone (no deploy target — workspace IS the artifact)
- [ ] Docusaurus

If Docusaurus, target path: `../path-to-docusaurus`

<details>
<summary><b>Advanced — collision behavior</b> (using defaults)</summary>

When target file exists with different content:
- [x] Warn (default — block deploy, ask user to resolve)
- [ ] Skip (keep target as-is)
- [ ] Overwrite (replace target)
- [ ] Prompt (ask per file)

</details>

<details>
<summary><b>Advanced — voice and tone</b> (using defaults)</summary>

The defaults below produce friendly-professional, second-person, 8th-grade reading level — works for ~80% of technical products. Override only if you have a specific style (e.g., regulated industry needs formal).

Tone:
- [ ] Casual
- [x] Friendly-professional (default)
- [ ] Technical-direct
- [ ] Formal

Perspective:
- [x] Second-person — "you create an instance" (default)
- [ ] First-person plural — "we create an instance"
- [ ] Third-person — "the user creates an instance"

Reading level:
- [ ] 6th grade
- [x] 8th grade (default)
- [ ] 10th grade
- [ ] College

Custom terms to AVOID: ``  (e.g., utilize, leverage)

</details>

## 5. Credentials for product walkthrough

<!--
  Required only if you'll use /docsmith walkthrough or /docsmith record.
  ENV VAR NAMES only — never actual passwords.
-->

- Username env var: `MYPRODUCT_TEST_USER`
- Password env var: `MYPRODUCT_TEST_PASS`

<details>
<summary><b>Advanced — MFA / SSO</b> (skip for plain login)</summary>

- Multi-factor auth env var: ``
- Test account note: `e.g., free tier, no production data`

</details>

## 6. Knowledge sources

<!--
  AI fetches content from these sources to inform drafts.
  Skip entirely if AI should draft from scratch using only intake info.
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

<details>
<summary><b>Advanced — additional sources</b> (skip if you only have 1 source)</summary>

Copy "Source 1" block below for each additional source. Increment N.

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

### Source 3

- Type:
  - [ ] Notion page
  - [ ] GitHub repo
  - [ ] Google Drive doc
  - [ ] Public URL
  - [ ] Local file
- URL or path or ID: ``
- Name: ``
- Auth env var (if private): ``

</details>

<details>
<summary><b>Advanced — auto-run behavior</b> (using defaults)</summary>

Where AI pauses for human review during `/docsmith run`:
- [ ] After plan (before draft)
- [x] After draft (before walkthrough) — default; safest for first run
- [ ] Before deploy
- [ ] Never (full auto)

Drift detection action:
- [x] Prompt per item (default — safest)
- [ ] Auto-apply HIGH confidence fixes

Translation review mode:
- [ ] Per-block (review each block individually)
- [x] Batch (review whole-file diff — default)
- [ ] Auto-approve

</details>

<details>
<summary><b>Advanced — sitemap pattern</b> (using Pattern A default)</summary>

Pattern A (Learning path) default works for most technical products.

Sitemap pattern:
- [x] Pattern A — Learning path (default — overview → setup → quickstarts → tutorials → reference → troubleshooting → glossary)
- [ ] Pattern B — Task-first (overview → quickstarts → guides → reference → troubleshooting)
- [ ] Pattern C — Custom (you define the order below)

If Pattern C, list section types in order:
```
overview
initial-setup
quickstarts
tutorials
reference
troubleshooting
glossary
```

### Section display name overrides (optional)

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

</details>

<details>
<summary><b>Advanced — media policy</b> (using defaults — silent video, source-only screenshots)</summary>

Default config: capture screenshots once in source language UI (reused for translated docs), generate silent videos with on-screen captions. **Cost: $0 TTS, ~1 hour for 30 docs × 3 locales.** Override below for premium output (~$5-10 TTS cost, ~3 hours).

### Screenshots

Density per content type (override per module if needed):
- Tutorial: `1 per major step` (default)
- How-to: `1 per heading` (default)
- Reference: `none` (default)
- Concept: `optional` (default)
- Troubleshooting: `1 per error case` (default)
- Quickstart: `1 per heading + 1 final state` (default)

Style:
- [x] viewport-only — clean, no browser chrome (default)
- [ ] full-window — includes URL bar, tabs
- [ ] cropped-element — focus on relevant element + 20px padding
- [ ] annotated — overlay arrows / numbered callouts (manual edit needed)

Aspect ratio:
- [x] 16:9 desktop / 1280×720 (default)
- [ ] 4:3 legacy / 1024×768
- [ ] mobile portrait / 375×812
- [ ] square / 1080×1080
- [ ] custom: ``

Per-locale strategy:
- [x] Source-only — capture in source UI, reuse for translated docs (default — cheapest)
- [ ] Per-locale — re-capture for each target locale
- [ ] Hybrid — source full + selective per-locale for important touchpoints

### Videos

Density per content type:
- Tutorial: `required, ≤ 90s` (default)
- How-to: `optional if >5 steps, ≤ 30s` (default)
- Reference: `never` (default)
- Concept: `optional, ≤ 2min` (default)
- Troubleshooting: `never` (default)
- Quickstart: `optional, ≤ 60s` (default)

Voiceover strategy:
- [x] Silent + on-screen captions (default — cheapest, no TTS needed)
- [ ] AI synthetic voice per locale
- [ ] Source voice + per-locale subtitles
- [ ] Human recorded voiceover
- [ ] No video at all (skip `record` command entirely)

#### TTS provider (only if "AI synthetic voice")

- [x] local-piper — free, offline (default)
- [ ] local-coqui — free, more voices
- [ ] openai — paid (~$15/1M chars)
- [ ] elevenlabs — paid ($5-99/mo), highest quality
- [ ] google-cloud — paid (~$4/1M chars), good Vietnamese
- [ ] azure-cognitive — paid

Voice ID/name per locale (only if AI voice):
- en: ``
- vi: ``
- jp: ``

Auth env var (only if non-local TTS):
- TTS provider env var: ``

### Subtitles / captions

Generate `.vtt`:
- [ ] No subtitles
- [ ] Source-only `.vtt`
- [x] Per-locale `.vtt` (default if multi-locale)

Generation method:
- [x] Auto from script (default)
- [ ] STT after recording (needs Whisper)
- [ ] Manual user-provided `.vtt`

Caption packaging:
- [x] Sidecar `.vtt` files (default)
- [ ] Burned into video

</details>

## 7. Module intake files (auto-managed)

Modules registered (created via `/docsmith module <name>`):

<!-- BEGIN MODULES LIST -->

(none yet)

<!-- END MODULES LIST -->

---

<!--
  Validation: when /docsmith run triggers, AI checks:
  - Required fields (Product, Audience primary, Languages source, Deploy preset)
  - Credentials env vars (if walkthrough in pipeline)
  - At least one source-type ticked per Source N block (or no Source N)
  - Advanced sections: defaults applied if not customized

  Missing critical → AI stops with a list. Missing nice-to-have → defaults applied silently.
-->
