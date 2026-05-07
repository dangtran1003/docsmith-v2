# Comparison: v1.1.0 vs v1.5.14

Self-review comparing the first release (v1.1.0) with the current release (v1.5.14). Purpose: check whether the skill has changed too much, drifted from original goals, or grown in scope appropriately.

> 🇻🇳 Bản tiếng Việt: [COMPARISON.vi.md](COMPARISON.vi.md)

## TL;DR

**Volume**: significantly larger (files +95%, spec lines +210%, commands +11% but functionality very different).

**Spirit**: preserved — still PRC-010 process, still AI-driven, still focused on doc quality.

**Direction**: shifted from "skill that writes docs" to "skill that automates the full doc lifecycle with intelligent intake from BA sources". Shift was intentional and user-directed across multiple turns.

**Self-assessment**: changed A LOT but did NOT drift. Each version solved a specific user-stated pain point. The skill is now production-ready in spec but **has never been tested on a real project**. Strong recommendation: stop and test before building further.

---

## Snapshot of both versions

|                          | v1.1.0 (initial)        | v1.5.14 (current)                                            | Delta    |
| ------------------------ | ----------------------- | ------------------------------------------------------------ | -------- |
| **Release date**         | 2026-04-26              | 2026-04-29                                                   | 3 days   |
| **Total files**          | 21                      | 41                                                           | +95%     |
| **Templates**            | 12                      | 16                                                           | +33%     |
| **Reference docs**       | 2 (process, tools)      | 5 (+ deploy, intake, translate)                              | +150%    |
| **Top-level guides**     | 1 (README)              | 7 (README + HOW_IT_WORKS + INTAKE_GUIDE + SETUP + COMPARISON, all en/vi) | +600% |
| **Commands**             | 18                      | 20                                                           | +11%     |
| **SKILL.md length**      | 298 lines               | ~570 lines                                                   | +91%     |
| **Total spec lines**     | ~3,500                  | ~9,000                                                       | +157%    |
| **Plugin format**        | Plain skill folder      | Claude Code plugin marketplace                               | New      |
| **Multi-locale**         | No                      | Yes (translate command, glossary)                            | New      |
| **Source-driven intake** | No (manual fill only)   | Yes (`init --from-source`)                                   | New      |
| **Sitemap consistency**  | Free-form               | 3 patterns (Learning / Task-first / Custom)                  | New      |
| **Media policy**         | Caption rules only      | Full (density, voiceover, TTS, subtitles)                    | New      |
| **Real-world tested?**   | No                      | No                                                           | Same    |

---

## What stayed the same (the spirit)

The original v1.1.0 design choices that survived all 16 versions:

1. **PRC-010 process** — audience → plan → voice → draft → edit → walkthrough → record. Still the backbone.
2. **AI-as-implementor, prompt-as-spec** — skill is markdown files Claude reads, not compiled code.
3. **Workspace-first** — everything happens in `documentation/`, can be standalone or feed Docusaurus.
4. **Quality bar** — caption discipline, voice charts, drift detection. All preserved.
5. **One source of truth per artifact** — drafts, plans, sitemaps each have one canonical location.
6. **No silent failures** — when AI can't follow a step, it asks the user. Hasn't changed.
7. **Re-runnable** — every command checks output existence and gates re-run. Strengthened in v1.3+ but the principle was already there.

---

## What changed (and why)

### Phase 1: First releases (v1.1.0 → v1.2.x)

**v1.1.0 (initial)**: bare skill. Caption rules, walkthrough basics, manual init/run. No deploy, no multi-locale, no presets.

**v1.2.0**: added `init`, `deploy`, `categorize` commands. YAML config (`.docsmithrc.yaml`). Standalone vs Docusaurus presets. Image namespacing.

**v1.2.1**: HOW_IT_WORKS.md guide added (494 lines).

*Drift assessment*: scope expanded but stayed within "doc skill" bounds.

### Phase 2: Hardening (v1.3.0 → v1.4.0)

**v1.3.0**: re-run protocol formalized. KB inheritance for drafts. 3-phase walkthrough (VERIFY → drift gate → APPLY). Drift reports. Per-doc product bug tracking.

**v1.4.0**: multi-locale translation. Per-block AI translation with user review. Glossary at `standards/glossary.<locale>.yaml`. Translation decisions audit.

*Drift assessment*: still focused on doc quality. Re-run protocol was a real-world need (regenerating drafts losing edits = bad). Translation was clearly scoped.

### Phase 3: Big refactor (v1.5.0 → v1.5.4)

**v1.5.0** (LEAN refactor): replaced `.docsmithrc.yaml` with markdown intake forms (project.md + modules/<n>.md). Added external sources (Notion / GitHub / GDrive / URL / file). Run state tracking. 6 new commands (`module`, `fetch`, `run`, `continue`, `update`, `intake-help`). Removed 8 commands.

**v1.5.1**: fixed 6 path conflict issues. INTAKE_GUIDE.md added (516 lines en + vi).

**v1.5.2**: Claude Code plugin marketplace format compliance. Restructured to `.claude-plugin/marketplace.json` + `skills/docsmith/`.

**v1.5.3**: SETUP.md added (~700 lines en + vi) covering Claude in Chrome, Playwright MCP, ffmpeg, env vars, token acquisition for each source type.

**v1.5.4**: sitemap consistency. 11 canonical section types, 3 patterns (Learning / Task-first / Custom). Per-module section selection. AI warns when modules lag pattern.

*Drift assessment*: this is where scope started feeling big. But each addition was triggered by a specific user pain (e.g., user showed two screenshots of inconsistent sitemaps from same project).

### Phase 4: UX polish (v1.5.5 → v1.5.8)

**v1.5.5**: media policy template (~600 lines). Screenshot density rules per content type, voiceover strategy (silent / AI / human), 6 TTS providers, subtitle generation, cost transparency.

**v1.5.6**: collapsed Advanced sections in intake forms (HTML `<details>`). BAs see ~80 lines essentials top-level, 354 total. Pure UX patch.

**v1.5.7**: per-video script files at `documentation/scripts/<module>/<id>.md`. Multi-locale scripts in same file (`## en`, `## vi`, `## jp` headings). VIDEO marker simplified. Translate processes scripts alongside drafts.

**v1.5.8**: docs refresh. Inline `>` hints in every intake field. INTAKE_GUIDE rewritten 516→258 lines (templates self-document now). README refreshed for v1.5.7.

*Drift assessment*: UX patches, not feature creep. Each addressed the "BA has to flip between 3 docs" problem.

### Phase 5: Source-driven intake (v1.5.9 → v1.5.10)

**v1.5.9**: `init --from-source` and `module --from-source`. AI reads BA doc / PRD / existing docs and infers fields. 4-tier confidence model (Fact / Guess / Default / Asked). Interactive Q&A for environment-specific fields. Inference report with audit trail.

**v1.5.10**: missing module detection in `update`. 3-layer change report (content drift + module diff + scope drift). Parallel sub-agent guidance (Cấp 1 — "MAY", not "MUST"). `--resume` flag for partial failures.

*Drift assessment*: solves the biggest manual pain (BA gõ tay 354 dòng). Real value. Parallel guidance is documentation-only — no overpromise.

---

## Where do we stand now?

### Strengths

- **Complete pipeline**: intake → fetch → audience → plan → voice → draft → edit → walkthrough → record → translate → deploy
- **Multi-locale first-class**: glossary, per-block review, per-locale media optional
- **Source-driven intake**: BA doesn't re-type info from BA doc
- **Re-run safety**: every command preserves manual edits; archives backups
- **Audit trail**: deployments, drift reports, inference reports, run state — all kept
- **Plugin marketplace**: installable via `/plugin marketplace add`

### Weaknesses

- **Spec is large** (~9000 lines across 41 files)
- **Real-world tested: never** (16 versions, 0 production runs)
- **AI behavior is heuristic** — `--from-source` inference, sitemap pattern detection, scope drift comparison all work via prose-described heuristics. Real failures only surface in real use.
- **Performance unknowns** — parallel sub-agents document MAY, don't guarantee speedup. Walkthrough on 30-module project: untested.
- **Editing flow assumes Markdown editor with `<details>` rendering** — plain text editor users see raw HTML markup
- **No CI/CD integration** — no GitHub Action template, no auto-deploy on merge

### Risks

- **Spec drift**: each new version adds ~100-300 lines. At 9000 already, cognitive load on AI grows. Spec quality may degrade (consistency between sections, conflicting rules) without test data to validate.
- **AI may not follow spec faithfully** — Claude has finite attention. 9000 lines is a lot to consult. Real test will show how well skill scales.
- **Maintenance burden** — bug fixes require finding the right section, editing carefully, ensuring no contradictions. Larger spec → harder.

---

## What I'd do differently if starting over

If I were starting v2 with all current learnings:

1. **Start with `--from-source` first** — manual intake fill was added before AI auto-fill. In hindsight, AI auto-fill should be primary path; manual fill is the fallback. This would reduce intake form complexity (forms are now self-documenting; AI fills most of them).

2. **Skip multi-version YAML config era** — v1.4.x used `.docsmithrc.yaml`. v1.5.0 replaced with markdown. Backward compat (`--upgrade-from-1.4`) still parses YAML. If starting fresh, no YAML phase.

3. **Test after v1.3** — by v1.3 the skeleton was in place. Should have stopped, tested 1 real project, then built features informed by real findings instead of speculative needs.

4. **Smaller media policy** — v1.5.5 added 600 lines covering 6 TTS providers, multi-locale strategies, subtitle generation. In retrospect, "silent video" default plus a single TTS option (local-piper) covers 90% of cases. Other strategies could be Roadmap items.

5. **Defer sitemap patterns to user feedback** — v1.5.4 added 3 patterns + 11 section types because user showed inconsistent sitemaps. Real fix: validate against pattern after first project. v1.5.4 implemented preemptively.

---

## Recommendation for the user

You've built v1.5.10 = a comprehensive doc automation skill. **Stop.** Test 1 real project end-to-end:

```bash
# 30-minute smoke test
mkdir test-project && cd test-project
echo "# MyApp\nA todo app for teams." > ba-doc.md
/docsmith init --from-source ba-doc.md
# Note: AI questions, awkward steps, surprising defaults
/docsmith module todo --from-source ba-doc.md
/docsmith run todo
# Note: drafts produced, walkthrough behavior
```

Real-world findings will guide v1.6+. Without test data, more spec just compounds untested assumptions.

---

## Honest assessment of skill development methodology

This conversation built v1.5.10 across 16 releases in 3 calendar days. Each version was:

✅ User-driven (responding to specific pain or question)
✅ Internally consistent (no contradicting features)
✅ Backward-compatible (re-run protocol covers migrations)

But also:

❌ Untested in production
❌ Heuristic-heavy (much of `--from-source`, sitemap detection, scope drift relies on AI heuristics that may not generalize)
❌ Cumulatively complex (~9000 lines spec is hard for any single person — or AI — to hold in mind)

The skill is in the unusual position of being **designed thoroughly but not validated at all**. Recommended next action is universally: validate.

---

## Conclusion

The skill changed substantially (95% more files, 157% more spec) but did not drift in spirit. Each change addressed a specific user-stated need.

The risk now is not "this skill is wrong"; it's "this skill is large and untested." Test before adding more.
