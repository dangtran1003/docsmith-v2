# So sánh: v1.1.0 vs v1.5.14

Self-review so sánh release đầu tiên (v1.1.0) với release hiện tại (v1.5.14). Mục đích: kiểm tra skill có thay đổi quá nhiều, drift khỏi original goals, hay grow scope hợp lý.

> 🇬🇧 English version: [COMPARISON.md](COMPARISON.md)

## TL;DR

**Volume**: lớn hơn đáng kể (files +95%, spec lines +210%, commands +11% nhưng functionality khác hẳn).

**Spirit**: được giữ — vẫn PRC-010 process, vẫn AI-driven, vẫn focus chất lượng doc.

**Direction**: dịch chuyển từ "skill viết doc" sang "skill automate full doc lifecycle với intelligent intake từ BA sources". Dịch chuyển có chủ đích và do user driven qua nhiều turn.

**Self-assessment**: thay đổi RẤT NHIỀU nhưng KHÔNG drift. Mỗi version solve 1 user-stated pain point cụ thể. Skill giờ production-ready trong spec nhưng **chưa bao giờ test trên project thật**. Recommend mạnh: stop và test trước khi build thêm.

---

## Snapshot 2 versions

|                          | v1.1.0 (initial)        | v1.5.14 (hiện tại)                                            | Delta    |
| ------------------------ | ----------------------- | ------------------------------------------------------------ | -------- |
| **Release date**         | 2026-04-26              | 2026-04-29                                                   | 3 ngày   |
| **Total files**          | 21                      | 41                                                           | +95%     |
| **Templates**            | 12                      | 16                                                           | +33%     |
| **Reference docs**       | 2 (process, tools)      | 5 (+ deploy, intake, translate)                              | +150%    |
| **Top-level guides**     | 1 (README)              | 7 (README + HOW_IT_WORKS + INTAKE_GUIDE + SETUP + COMPARISON, all en/vi) | +600% |
| **Commands**             | 18                      | 20                                                           | +11%     |
| **SKILL.md length**      | 298 lines               | ~570 lines                                                   | +91%     |
| **Total spec lines**     | ~3,500                  | ~9,000                                                       | +157%    |
| **Plugin format**        | Plain skill folder      | Claude Code plugin marketplace                               | New      |
| **Multi-locale**         | Không                   | Có (translate command, glossary)                             | New      |
| **Source-driven intake** | Không (chỉ manual fill) | Có (`init --from-source`)                                    | New      |
| **Sitemap consistency**  | Free-form               | 3 patterns (Learning / Task-first / Custom)                  | New      |
| **Media policy**         | Caption rules only      | Full (density, voiceover, TTS, subtitles)                    | New      |
| **Test thực tế?**        | Không                   | Không                                                        | Same    |

---

## Cái gì giữ nguyên (the spirit)

Original v1.1.0 design choices survived qua 16 versions:

1. **PRC-010 process** — audience → plan → voice → draft → edit → walkthrough → record. Vẫn là backbone.
2. **AI-as-implementor, prompt-as-spec** — skill là markdown files Claude đọc, không phải compiled code.
3. **Workspace-first** — mọi thứ xảy ra trong `documentation/`, có thể standalone hoặc feed Docusaurus.
4. **Quality bar** — caption discipline, voice charts, drift detection. Tất cả preserved.
5. **One source of truth per artifact** — drafts, plans, sitemaps mỗi cái có 1 canonical location.
6. **No silent failures** — khi AI không follow được step, hỏi user. Không thay đổi.
7. **Re-runnable** — mỗi command check output existence và gate re-run. Strengthened ở v1.3+ nhưng principle đã có sẵn.

---

## Cái gì thay đổi (và tại sao)

### Phase 1: Releases đầu (v1.1.0 → v1.2.x)

**v1.1.0 (initial)**: bare skill. Caption rules, walkthrough basics, manual init/run. Không deploy, không multi-locale, không presets.

**v1.2.0**: thêm `init`, `deploy`, `categorize` commands. YAML config (`.docsmithrc.yaml`). Standalone vs Docusaurus presets. Image namespacing.

**v1.2.1**: HOW_IT_WORKS.md guide (494 lines).

*Drift assessment*: scope expanded nhưng vẫn trong "doc skill" bounds.

### Phase 2: Hardening (v1.3.0 → v1.4.0)

**v1.3.0**: re-run protocol formalized. KB inheritance cho drafts. 3-phase walkthrough (VERIFY → drift gate → APPLY). Drift reports. Per-doc product bug tracking.

**v1.4.0**: multi-locale translation. Per-block AI translation với user review. Glossary tại `standards/glossary.<locale>.yaml`. Translation decisions audit.

*Drift assessment*: vẫn focus chất lượng doc. Re-run protocol là real-world need (regen drafts mất edit = bad). Translation rõ scope.

### Phase 3: Big refactor (v1.5.0 → v1.5.4)

**v1.5.0** (LEAN refactor): replace `.docsmithrc.yaml` bằng markdown intake forms (project.md + modules/<n>.md). Thêm external sources (Notion / GitHub / GDrive / URL / file). Run state tracking. 6 commands mới (`module`, `fetch`, `run`, `continue`, `update`, `intake-help`). Bỏ 8 commands.

**v1.5.1**: fix 6 path conflict issues. INTAKE_GUIDE.md (516 lines en + vi).

**v1.5.2**: Claude Code plugin marketplace format compliance. Restructure thành `.claude-plugin/marketplace.json` + `skills/docsmith/`.

**v1.5.3**: SETUP.md (~700 lines en + vi) cover Claude in Chrome, Playwright MCP, ffmpeg, env vars, token acquisition cho mỗi source type.

**v1.5.4**: sitemap consistency. 11 canonical section types, 3 patterns (Learning / Task-first / Custom). Per-module section selection. AI warn khi modules lag pattern.

*Drift assessment*: đây là chỗ scope bắt đầu cảm thấy lớn. Nhưng mỗi addition trigger bởi specific user pain (vd user show 2 screenshots inconsistent sitemaps cùng project).

### Phase 4: UX polish (v1.5.5 → v1.5.8)

**v1.5.5**: media policy template (~600 lines). Screenshot density rules per content type, voiceover strategy (silent / AI / human), 6 TTS providers, subtitle generation, cost transparency.

**v1.5.6**: collapsed Advanced sections trong intake forms (HTML `<details>`). BAs thấy ~80 lines essentials top-level, 354 total. Pure UX patch.

**v1.5.7**: per-video script files tại `documentation/scripts/<module>/<id>.md`. Multi-locale scripts cùng file (`## en`, `## vi`, `## jp` headings). VIDEO marker simplified. Translate process scripts cùng drafts.

**v1.5.8**: docs refresh. Inline `>` hints mỗi intake field. INTAKE_GUIDE rewrite 516→258 lines (templates self-document giờ). README refresh cho v1.5.7.

*Drift assessment*: UX patches, không phải feature creep. Mỗi cái address "BA phải flip giữa 3 docs" problem.

### Phase 5: Source-driven intake (v1.5.9 → v1.5.10)

**v1.5.9**: `init --from-source` và `module --from-source`. AI đọc BA doc / PRD / existing docs và infer fields. 4-tier confidence model (Fact / Guess / Default / Asked). Interactive Q&A cho environment-specific fields. Inference report với audit trail.

**v1.5.10**: missing module detection trong `update`. 3-layer change report (content drift + module diff + scope drift). Parallel sub-agent guidance (Cấp 1 — "MAY", không "MUST"). `--resume` flag cho partial failures.

*Drift assessment*: solve biggest manual pain (BA gõ tay 354 dòng). Real value. Parallel guidance là documentation-only — không overpromise.

---

## Status hiện tại

### Strengths

- **Complete pipeline**: intake → fetch → audience → plan → voice → draft → edit → walkthrough → record → translate → deploy
- **Multi-locale first-class**: glossary, per-block review, per-locale media optional
- **Source-driven intake**: BA không re-type info từ BA doc
- **Re-run safety**: mọi command preserve manual edits; archive backups
- **Audit trail**: deployments, drift reports, inference reports, run state — tất cả kept
- **Plugin marketplace**: install qua `/plugin marketplace add`

### Weaknesses

- **Spec lớn** (~9000 lines across 41 files)
- **Real-world tested: chưa bao giờ** (16 versions, 0 production runs)
- **AI behavior là heuristic** — `--from-source` inference, sitemap pattern detection, scope drift comparison đều work qua prose-described heuristics. Real failures chỉ surface trong real use.
- **Performance unknowns** — parallel sub-agents document MAY, không guarantee speedup. Walkthrough trên 30-module project: untested.
- **Editing flow giả định Markdown editor render `<details>`** — plain text editor users thấy raw HTML markup
- **Không CI/CD integration** — không GitHub Action template, không auto-deploy on merge

### Risks

- **Spec drift**: mỗi new version thêm ~100-300 lines. Tại 9000 đã, cognitive load lên AI tăng. Spec quality có thể degrade (consistency giữa sections, conflicting rules) không có test data validate.
- **AI có thể không follow spec faithfully** — Claude có finite attention. 9000 lines là nhiều để consult. Real test sẽ show skill scale thế nào.
- **Maintenance burden** — bug fixes require tìm right section, edit carefully, ensure no contradictions. Larger spec → harder.

---

## Cái gì tôi sẽ làm khác nếu start over

Nếu start v2 với all current learnings:

1. **Start với `--from-source` đầu** — manual intake fill được thêm trước AI auto-fill. Hindsight: AI auto-fill nên là primary path; manual fill là fallback. Cái này sẽ giảm intake form complexity (forms giờ self-documenting; AI fill đa số chúng).

2. **Skip multi-version YAML config era** — v1.4.x dùng `.docsmithrc.yaml`. v1.5.0 replace bằng markdown. Backward compat (`--upgrade-from-1.4`) vẫn parse YAML. Nếu start fresh, không YAML phase.

3. **Test sau v1.3** — by v1.3 skeleton đã in place. Nên đã stop, test 1 real project, sau đó build features inform bởi real findings thay vì speculative needs.

4. **Smaller media policy** — v1.5.5 thêm 600 lines cover 6 TTS providers, multi-locale strategies, subtitle generation. Hindsight, "silent video" default plus single TTS option (local-piper) cover 90% cases. Other strategies có thể là Roadmap items.

5. **Defer sitemap patterns đến user feedback** — v1.5.4 thêm 3 patterns + 11 section types vì user show inconsistent sitemaps. Real fix: validate against pattern sau first project. v1.5.4 implement preemptively.

---

## Recommendation cho user

Bạn đã build v1.5.10 = comprehensive doc automation skill. **Stop.** Test 1 real project end-to-end:

```bash
# 30-phút smoke test
mkdir test-project && cd test-project
echo "# MyApp\nA todo app for teams." > ba-doc.md
/docsmith init --from-source ba-doc.md
# Note: AI questions, awkward steps, surprising defaults
/docsmith module todo --from-source ba-doc.md
/docsmith run todo
# Note: drafts produced, walkthrough behavior
```

Real-world findings sẽ guide v1.6+. Không có test data, more spec chỉ compound untested assumptions.

---

## Honest assessment của methodology phát triển skill

Conversation này build v1.5.10 across 16 releases trong 3 ngày calendar. Mỗi version:

✅ User-driven (responding to specific pain or question)
✅ Internally consistent (no contradicting features)
✅ Backward-compatible (re-run protocol cover migrations)

Nhưng cũng:

❌ Chưa test trong production
❌ Heuristic-heavy (nhiều của `--from-source`, sitemap detection, scope drift dựa vào AI heuristics có thể không generalize)
❌ Cumulatively complex (~9000 lines spec khó cho bất kỳ single person — hoặc AI — hold in mind)

Skill ở vị trí bất thường: **được design kỹ lưỡng nhưng không validated chút nào**. Recommended next action universally: validate.

---

## Conclusion

Skill thay đổi substantially (95% more files, 157% more spec) nhưng không drift in spirit. Mỗi change address specific user-stated need.

Risk giờ không phải "skill này sai"; là "skill này lớn và untested." Test trước khi thêm.
