# Comparison: v1.1.0 vs v1.5.1

A self-review comparing the first release (v1.1.0) with the current release (v1.5.1) of docsmith. Purpose: check whether the skill has changed too much, drifted from original goals, or grown in scope appropriately.

> 🇻🇳 Bản tiếng Việt: [COMPARISON.vi.md](COMPARISON.vi.md)

## TL;DR

**Volume**: significantly larger (files +52%, spec lines roughly +200%, command count similar but functionality very different).

**Spirit**: preserved — still PRC-010 process, still AI-driven, still focused on doc quality.

**Direction**: shifted from "skill that writes docs" to "skill that automates the full doc lifecycle with clear BA input". Shift was intentional and user-directed across multiple turns.

**Self-assessment**: changed A LOT but did NOT drift. Each version solved a specific user-stated pain point. If it now feels bloated, recommend stopping and testing on a real project before building further.

---

## Snapshot of both versions

|                          | v1.1.0 (initial)        | v1.5.1 (current)                                          | Delta    |
| ------------------------ | ----------------------- | --------------------------------------------------------- | -------- |
| **Release date**         | Apr 26, 2026            | Apr 26, 2026 (same day)                                   | —        |
| **Total files**          | 21                      | 32                                                        | +52%     |
| **Templates**            | 12                      | 14                                                        | +17%     |
| **Reference docs**       | 2 (process, tools)      | 5 (+ deploy, intake, translate)                           | +150%    |
| **Guides**               | 1 (README)              | 4 (README + HOW_IT_WORKS + INTAKE_GUIDE en/vi)            | +300%    |
| **Commands**             | 18                      | 20                                                        | +11%     |
| **Functional commands**  | 16 doing actual work    | 18 doing actual work                                      | +13%     |
| **SKILL.md length**      | 298 lines               | 461 lines                                                 | +55%     |
| **Multi-locale support** | No                      | Yes (translate command, glossary, locales config)         | +∞       |
| **External sources**     | No                      | Yes (Notion / GitHub / GDrive / URL / file)               | +∞       |
| **Deploy automation**    | No                      | Yes (Docusaurus preset, dry-run, sync-deletes)            | +∞       |
| **Re-run safety**        | No                      | 4-option gate, KB inheritance                             | +∞       |
| **Drift detection**      | No                      | walkthrough 3-phase with drift report                     | +∞       |

---

## Commands: kept, renamed, removed, added

### Kept across versions (15 commands)

`help`, `audience`, `plan`, `voice`, `draft`, `edit`, `walkthrough`, `record`, `verify`, `publish` — core list largely intact.

### Renamed or merged (8 commands)

| v1.1.0          | v1.5.1                              | Reason                                          |
| --------------- | ----------------------------------- | ----------------------------------------------- |
| `start`         | merged into `init`                  | Duplicated functionality                        |
| `validate`      | merged into `walkthrough --check`   | Same verification job                           |
| `test`          | merged into `walkthrough` (auto)    | Rarely run standalone                           |
| `peer-review`   | removed (gate in `run` instead)     | It's a human gate, doesn't need a command       |
| `tech-review`   | removed (gate in `run` instead)     | It's a human gate, doesn't need a command       |
| `review-plan`   | removed (`run --pause-at after-plan`) | Inline gate replaces it                       |
| `incorporate`   | merged into `edit --from-review`    | Both apply feedback                             |
| `sitemap`       | merged into `plan` (output)         | Sitemap is plan's output                        |

### Added (10 commands not in v1.1.0)

| Command         | Purpose                                                            |
| --------------- | ------------------------------------------------------------------ |
| `init`          | Setup workspace + intake form scaffold                             |
| `module`        | Manage per-feature module intakes                                  |
| `intake-help`   | Print field reference for intake forms                             |
| `fetch`         | Pull external sources                                              |
| `run`           | Orchestrated pipeline with pause gate                              |
| `continue`      | Resume after gate                                                  |
| `update`        | Detect external source changes                                     |
| `translate`     | Multi-locale translation                                           |
| `categorize`    | Docusaurus category file generator                                 |
| `deploy`        | Sync workspace to host project                                     |

**Command bottom line**: count went up slightly (18 → 20), but functionality changed dramatically. v1.1.0 was "a writing tool"; v1.5.1 is "a doc lifecycle automation tool with external integrations and deploy automation".

---

## Workspace structure: fundamental change

### v1.1.0 (initial — "docs for documentation")

```
docs/                              ← workspace named "docs"
├── plan/
├── standards/
├── drafts/                        ← flat (no locales)
├── walkthrough/
├── images/
└── videos/
```

Characteristics: flat, simple, no config, no audit trail, no deploy target.

### v1.5.1 (current)

```
documentation/                     ← renamed; deploy target uses "docs"
├── intake/                        ← NEW: BA-friendly config forms
│   ├── project.md
│   ├── modules/
│   │   └── instances.md
│   └── sources.lock.yaml          ← NEW: external source state
├── plan/
├── standards/
│   └── glossary.<locale>.yaml     ← NEW: translation glossary
├── drafts/<locale>/<module>/      ← NEW: locale-aware, module-aware
├── walkthrough/
│   ├── drift/<ts>/                ← NEW: drift detection runs
│   └── active-product-bugs.yaml   ← NEW: cross-run product bug tracker
├── archive/<ts>/                  ← NEW: re-run protocol backups
├── images/<module>/
├── videos/
├── deployments/<ts>-<target>/     ← NEW: deploy audit trail
├── .cache/sources/                ← NEW: external source content cache
└── .run-state/<module>.yaml       ← NEW: orchestration state per module
```

Characteristics: hierarchical (locale + module), config-driven, full audit trail, deploy automation, integration with external systems.

**Assessment**: this is a BIG change. Workspace went from "holds outputs" to "holds config + state + audit + cache + outputs". Driven by emerging needs: BAs needed config, run/continue needed state, re-run safety needed audit, external sources needed cache.

---

## Versioning timeline — what each release solved

| Version | Date | Trigger | What was added |
|---------|------|---------|----------------|
| v1.1.0 | Apr 26 | Caption rules for screenshots, video markers | `record` command, 3 templates (screenshot policy, video marker, video plan) |
| v1.2.0 | Apr 26 | Docusaurus integration concern | `init`, `deploy`, `categorize` commands; deploy preset; `.docsmithrc.yaml` |
| v1.2.1 | Apr 26 | Need accessible guide | HOW_IT_WORKS.md (en + vi) |
| v1.3.0 | Apr 26 | Self-check after product UI changes | Re-run protocol, walkthrough drift gate, KB inheritance, delete propagation |
| v1.4.0 | Apr 26 | "Must translate before deploy" | `translate` command, glossary, multi-locale flow |
| v1.5.0 | Apr 26 | "22 commands too many; BAs hate YAML" | Markdown intake forms, layered config, `run`/`continue`/`update`/`module`/`fetch` commands, lean refactor (cut 8 old commands) |
| v1.5.1 | Apr 26 | Audit found 6 path conflicts | Bug fixes + INTAKE_GUIDE practical guide |

**Observation**: every version solved a specific stated pain point. No version was built for imagined users. But the dense release pace (7 versions in one day) accumulated spec quickly.

---

## How much did things change from v1.1.0 → v1.5.1?

### Did we change a lot?

**Yes. Specifically:**

- **Workspace folder rename**: `docs/` → `documentation/`
- **New config approach**: `.docsmithrc.yaml` (v1.2) → `intake/project.md + modules/<n>.md` (v1.5)
- **Workflow shift**: individual commands → `run` orchestrator
- **Deploy automation**: added entirely
- **Multi-locale**: added entirely
- **External integrations**: added entirely (Notion / GitHub / GDrive)
- **Re-run safety**: added entirely

### Did we drift from original goals?

**No. Original goals are still respected:**

| v1.1.0 goal | v1.5.1 |
|---|---|
| PRC-010 process | ✅ Still followed |
| AI-driven authoring | ✅ Still AI |
| 5-pass self-review | ✅ Still present (`edit`) |
| Walkthrough verification | ✅ Still present, stronger (3-phase) |
| Caption-driven screenshots | ✅ Still present |
| Voice consistency | ✅ Still present |
| BA-friendly | ⚠️ NEW (intake forms) — aligns with original spirit |

### Is overhead too high for new users?

**Probably yes.** A new user running v1.5.1 first time will face:
- Concepts to grasp: project intake, module intake, sources, locales, presets
- Two MD files to fill (project.md + at least one module file)
- Env vars to set for credentials and sources
- 6+ checkboxes to tick and 10+ backtick fields to fill before `run` works

Compare to v1.1.0: just `/docsmith start` and answer interactive questions.

**Trade-off**: v1.5.1 takes 15-30 minutes setup first time but is reproducible and faster on re-run. v1.1.0 was faster setup but required repeat input every time.

---

## Risks I see

### 1. Not tested on a real project yet

7 versions in one day, no real data. Each feature is theoretically sound but:
- Does AI parse MD forms deterministically? (untested)
- Do Notion/GDrive source fetches work with real tokens? (untested)
- Is `run` orchestration reliable end-to-end? (untested)
- Does KB inheritance in Update mode actually preserve manual edits? (untested)

### 2. Spec may be too detailed

5 reference docs (process, tools, deploy, intake, translate). Total spec ~3000+ lines. AI loading skill must read SKILL.md + relevant references. High token cost per run.

### 3. Feature count grew faster than integration testing

Each new feature (deploy, translate, drift, intake, run, update) depends on others. Bugs in one place can cascade. No integration tests yet.

### 4. Onboarding path is unclear

4 guides (README, HOW_IT_WORKS, INTAKE_GUIDE, CHANGELOG) totaling 2000+ lines. Which should a new user read first? Currently directing is rather generic.

---

## Overall assessment

### You're heading the right direction if

- Project will be used by a real team, BAs need to fill config (intake forms address this)
- You have an existing Docusaurus repo (deploy automation addresses this)
- Multiple languages (translate addresses this)
- Source content lives in Notion / GitHub (fetch addresses this)
- Multiple modules/features needing separate tracking (module intakes address this)

### You're over-engineering if

- Solo project, one module, 5-10 doc pages (v1.1.0 was enough)
- Source content lives only in your head (no fetch needed)
- One language (no translate needed)
- No Docusaurus yet (standalone preset is fine)
- Docs done once, never updated (no re-run safety, drift detection, update command needed)

---

## Recommendations

**If untested**: STOP building. Test v1.5.1 on a small real project. 30 minutes - half day. Output is a list of "where AI got it wrong, where it felt awkward, what wasn't necessary". This is the only meaningful feedback after 7 versions.

**If feeling bloated**: consider maintaining 2 branches:
- `lean` branch — pinned at v1.1.0 or v1.2.x for small projects
- `main` branch — v1.5+ for enterprise projects with Docusaurus + multi-locale

**If happy with the direction**: keep v1.5.1 as final for v1.x. Test for 2-4 weeks. v1.6 only if specific pain points emerge from real usage.

---

## Concrete numbers (technical detail)

```
v1.1.0 → v1.5.1 file changes:

ADDED (16):
  .docsmithrc.example.yaml (deprecated since 1.5.0, will be removed 1.6)
  HOW_IT_WORKS.md, HOW_IT_WORKS.vi.md
  INTAKE_GUIDE.md, INTAKE_GUIDE.vi.md
  deploy-reference.md
  intake-reference.md
  translate-reference.md
  presets/docusaurus.yaml, presets/standalone.yaml
  templates/CATEGORY_FILE_TEMPLATE.md
  templates/DRIFT_REPORT_TEMPLATE.md
  templates/GLOSSARY_TEMPLATE.yaml
  templates/MODULE_INTAKE_TEMPLATE.md
  templates/PROJECT_INTAKE_TEMPLATE.md
  templates/SOURCES_LOCK_TEMPLATE.md

REMOVED (5):
  subprocess-010a.md (folded into process-reference)
  templates/TRACEABILITY_MATRIX_TEMPLATE.md (inlined)
  templates/UX_CONTENT_SCORECARD_TEMPLATE.md (now opt-in via voice --full)
  templates/UX_TEXT_PATTERNS_TEMPLATE.md (now opt-in)
  templates/WALKTHROUGH_TEST_EXECUTION_TEMPLATE.md (folded into test-case)

MODIFIED (significantly): all other files

Net file change: +11 (21 → 32)
```
