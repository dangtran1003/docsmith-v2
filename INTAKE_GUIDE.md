# Intake Guide

Practical guide to filling docsmith's intake forms. The forms themselves have inline hints — this guide is for context, common patterns, and decisions you'll need to make.

> 🇻🇳 Bản tiếng Việt: [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md)

## What you fill out

Two markdown files, both inside `documentation/intake/`:

```
documentation/intake/
├── project.md                  # Fill once per project
└── modules/
    ├── instances.md            # One file per feature area
    └── storage.md
```

`project.md` is created by `/docsmith init`. Module files are created by `/docsmith module <name>`.

Each field in these files has a `>` hint right below it explaining what to put. **You don't need to keep this guide open while filling — the templates explain themselves.**

This guide is for: deciding patterns, understanding tradeoffs, troubleshooting.

## Three editing rules

### 1. Fill backtick fields

```markdown
- Product slug: `your-product-slug`
```

becomes

```markdown
- Product slug: `mycloud`
```

Don't remove the backticks. Replace text between them.

### 2. Tick checkboxes

```markdown
- [ ] Standalone
- [ ] Docusaurus
```

becomes

```markdown
- [ ] Standalone
- [x] Docusaurus
```

Most groups: tick exactly one. A few (like target languages): tick multiple.

### 3. Skip Advanced sections

Sections wrapped in `<details>` are collapsed by default. Most projects don't need to expand any of them. Open only if you have a specific reason (regulated industry needs formal voice, compliance module needs human voiceover, etc.).

## Minimum-fill checklist for first project

Open `project.md` and fill ONLY:
- § 1 Product (slug, name, URL, description)
- § 2 Audience (primary persona only)
- § 3 Languages (source language; target languages if any)
- § 4 Deploy (preset; target path if Docusaurus)
- § 5 Credentials (env var names — only if you'll use walkthrough)
- § 6 Knowledge sources > Source 1 (skip if AI should draft from intake info alone)

Open each `modules/<name>.md` and fill ONLY:
- § 1 Module identity (slug, display name, folder)
- § 2 Scope (at least 1 feature with at least 1 content type ticked)

That's it. Run `/docsmith run`. If output isn't what you want, expand the relevant Advanced section and tweak.

## Common patterns

### "I want quick docs for one feature"

1. `/docsmith init` (Standalone preset, no deploy target)
2. `/docsmith module myfeature`
3. Fill `project.md` § 1, 2, 3 (en source, no targets)
4. Fill `modules/myfeature.md` § 1, 2 (1 feature, How-to ticked)
5. `/docsmith run myfeature`

5 minutes from zero to drafts.

### "I have a Docusaurus site and want to add docs"

1. `cd ~/my-docusaurus-site`
2. `/docsmith init --in-place` (auto-detects, suggests in-place mode)
3. `/docsmith module pricing`
4. Fill intakes (in-place mode = `target_path: .`)
5. `/docsmith run pricing`
6. `/docsmith deploy --dry-run` then `/docsmith deploy`
7. `git add . && git commit -m "Add pricing docs" && git push`

### "I need multilingual docs (EN + VI + JP)"

1. In `project.md` § 3: source `en`, targets `vi` + `jp` ticked
2. (Optional) Edit `documentation/standards/glossary.vi.yaml` and `glossary.jp.yaml` (auto-created) to add domain terms
3. `/docsmith run` — pipeline runs through translate stage automatically
4. Translated drafts appear in `documentation/drafts/vi/` and `drafts/jp/`

Defaults: batch review (whole-file diff), no glossary required, source-only screenshots.

### "Source content lives in Notion"

1. Get Notion integration token: notion.so/my-integrations → create internal integration → copy token starting with `secret_`
2. Share each Notion page docsmith should read with that integration
3. `export NOTION_TOKEN="secret_..."` (or add to `.env`)
4. In `project.md` § 6 Source 1:
   - Type: tick Notion page
   - URL: `https://notion.so/abc123` (full URL or just ID)
   - Auth env var: `NOTION_TOKEN`
5. `/docsmith run`

### "I want to update docs after the product UI changed"

1. `/docsmith walkthrough --check` — produces drift report
2. Read `documentation/walkthrough/drift/<latest>/drift-report.md`
3. Edit `decisions.yaml` next to it (auto-fix / skip / mark as product bug)
4. `/docsmith walkthrough --apply` to apply decisions and re-capture screenshots

### "Source PRD in Notion was updated"

1. `/docsmith update`
2. AI checks all sources via cheap metadata calls, lists what changed
3. Confirm to re-fetch and re-evaluate affected docs
4. Review proposed deltas (Update mode: only changed sections proposed)

### "I need video tutorials with Vietnamese voiceover"

1. In `project.md` § 6 expand "Advanced — media policy"
2. Voiceover strategy: tick "AI synthetic voice per locale"
3. TTS provider: tick "google-cloud" (best Vietnamese support) or "local-piper" (free)
4. Voice ID per locale: fill from provider's catalog (e.g., `vi-VN-Wavenet-A` for Google Cloud)
5. Auth env var: set `GOOGLE_APPLICATION_CREDENTIALS` to service account JSON path
6. Add `<!-- VIDEO id: my-tour -->` markers in your drafts where videos should go
7. `/docsmith record` — AI generates script per locale, you review, then TTS produces audio per locale, video is encoded

See [SETUP.md § 6](SETUP.md) for TTS provider install.

## Decision matrix

### Voiceover strategy (project intake § 6 advanced — media policy)

| You want | Pick |
|---|---|
| Just get docs done; cheap; no audio complexity | Silent + on-screen captions (default) |
| Multi-locale, native UX, willing to budget $5-10/cycle | AI synthetic voice per locale |
| Tight budget but multi-locale OK with subtitles | Source voice + per-locale subtitles |
| Premium quality, regulated content, marketing video | Human recorded voiceover |

### Per-locale screenshots (project intake § 6 advanced)

| You want | Pick |
|---|---|
| Cheapest; single source-locale capture; note "EN UI" in translated docs | Source-only (default) |
| Native experience; UI text varies per locale; willing to triple capture time | Per-locale |
| Best of both; capture key flows per locale, reuse generic | Hybrid |

### TTS provider (only if voiceover = AI synthetic voice)

| You want | Pick |
|---|---|
| Free, offline, no API keys, decent quality | local-piper (default) |
| Free, offline, more voice options | local-coqui |
| High quality, wide languages, simple API | openai (~$15/1M chars) |
| Highest quality, marketing-grade | elevenlabs ($5-99/mo) |
| Best Vietnamese / Asian languages | google-cloud (~$4/1M chars) |
| Many neural voices, Azure already in stack | azure-cognitive |

## What does AI actually do with each field?

| Field | Where AI uses it |
|---|---|
| product.slug | Image namespace `/img/<slug>/`; sources.lock entries; deploy paths |
| product.name | Title in frontmatter; landing intro lines |
| audience.tech_level | Vocabulary choices, assumed prior knowledge level |
| audience.primary_goal | Tutorial intros ("you'll learn how to..."), how-to motivation lines |
| locales.source | Where drafts go: `drafts/<source>/...` |
| locales.targets | What languages translate command produces |
| voice.tone | Sentence style, contractions, formality |
| voice.perspective | "you" vs "we" vs "the user" throughout |
| voice.reading_level | Word choice, sentence length |
| voice.terms_to_avoid | Negative constraint during drafting |
| deploy.preset | Which preset's deploy logic activates (standalone or docusaurus) |
| deploy.target_path | Where files go on `deploy` |
| sources | Fetched at run time; available to AI as KB during drafting |
| credentials.*_env | AI reads env var when launching browser for walkthrough |
| auto_run.pause_at | When pipeline interrupts for review |
| translate.review_mode | per-block vs batch vs auto-approve |
| sitemap.pattern | Section ordering across all modules (consistency) |
| media.screenshot_density | How many screenshots per doc by content type |
| media.voiceover_strategy | Whether videos have voice; how multi-locale handled |
| media.tts_provider | Which TTS service generates voice audio |
| module.folder | Folder name in target deploy (e.g., `docs/instances/`) |
| features[].content_types | Drives how many docs generated per feature |
| out_of_scope | Negative constraint — AI won't generate these topics |
| status (module) | Whether `run` processes this module or skips |

## What if I make a mistake?

| Mistake | What happens | Fix |
|---|---|---|
| Empty critical field | `/docsmith run` stops with clear list | Fill listed fields, re-run |
| Wrong checkbox tick | Output doesn't match expectation | Edit, re-run; re-run protocol merges cleanly |
| Bad source URL | `/docsmith fetch` errors with the URL | Fix URL, retry |
| Unset env var | Command stops, prints which env var | `export VAR=value`, retry |
| Deleted backticks accidentally | `/docsmith intake-help` flags issue | Restore backticks |
| Missing TTS provider env var | `record` warns before generating | Set env var or switch to local-piper |

You can always run `/docsmith intake-help` for the full field reference, or `/docsmith intake-help <section>` for one section.

## After your first successful run

```
documentation/
├── plan/audience-profile.md          # AI-generated from your intake
├── plan/documentation-plan.md
├── plan/sitemap.md
├── standards/voice-chart.md
├── drafts/<locale>/<module>/         # The docs themselves
├── images/<module>/                  # Screenshots from walkthrough
├── scripts/<module>/                 # Video scripts (if record used)
├── videos/                           # Final videos
├── walkthrough/                      # Drift reports, test cases
└── deployments/<ts>/                 # Audit trail per deploy
```

Edit any draft directly if needed (re-run protocol preserves edits on next pipeline run). Then deploy and commit.

## When to expand Advanced sections

Most projects never need to. But here are the signals:

| Signal | Expand |
|---|---|
| First run: drafts feel too formal/casual | § 6 Advanced — voice and tone |
| Multi-locale: terminology inconsistent in translations | Edit `documentation/standards/glossary.<locale>.yaml` |
| Multi-locale: UI text appears in screenshots | § 6 Advanced — media policy → per-locale screenshots |
| Source content too vague, AI hallucinates | Add more Source N blocks (Advanced) |
| Reviewing every block too slow | § 6 Advanced — auto-run → translate review = batch |
| Module needs different voice than rest | module.md Advanced — voice override |
| Compliance module needs human voiceover | module.md Advanced — media override |
| Pipeline fails partway, want to start over | Use `/docsmith init --force` (after backup) |

## Tips for first-time success

1. **Start small**. One module, one feature, How-to content type only. Run, review, then expand scope.
2. **Don't expand Advanced sections on first try**. Defaults are tuned for "good enough" first output.
3. **Set the pause gate to "After draft"** (default). You see what AI wrote before walkthrough captures screenshots.
4. **Test with a real test account, not your own**. Walkthrough automation is fast and might do unexpected things; isolate it.
5. **Run `verify` before deploy**. Catches placeholders, broken links, voice inconsistency.
6. **Commit `documentation/` to git**. The audit trail (deployments/, archive/, drift/) is small and worth keeping.
7. **Don't commit `.cache/` or `videos/raw/`**. Init adds these to .gitignore automatically.
