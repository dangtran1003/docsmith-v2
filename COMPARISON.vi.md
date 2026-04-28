# So sánh v1.1.0 vs v1.5.1

So sánh giữa phiên bản đầu tiên (v1.1.0, ngày 26/04/2026) và phiên bản hiện tại (v1.5.1, cùng ngày). Mục đích: review xem có đổi quá nhiều, có giữ được tinh thần ban đầu, hay đã drift xa khỏi mục tiêu gốc.

> 🇬🇧 English version: [COMPARISON.md](COMPARISON.md)

## TL;DR

**Số lượng**: tăng đáng kể (file +52%, dòng spec ước lượng +200%, command số tương đương nhưng functional rất khác).

**Tinh thần**: giữ được — vẫn là PRC-010 process, vẫn AI-driven, vẫn focus chất lượng doc.

**Hướng**: chuyển từ "skill viết doc" sang "skill tự động hóa toàn bộ vòng đời doc với input rõ ràng từ BA". Thay đổi này có chủ ý, do user chỉ đạo qua nhiều turn.

**Đánh giá tự thân**: thay đổi NHIỀU nhưng KHÔNG drift. Mỗi version giải quyết 1 pain point cụ thể user nêu ra. Nếu cảm thấy quá phình to, recommend stop và test trên project thật trước khi build thêm.

---

## Snapshot 2 phiên bản

|                          | v1.1.0 (đầu)              | v1.5.1 (hiện tại)                                          | Thay đổi |
| ------------------------ | ------------------------- | ---------------------------------------------------------- | -------- |
| **Ngày release**         | 26/04/2026                | 26/04/2026 (cùng ngày)                                     | —        |
| **Tổng số file**         | 21                        | 32                                                         | +52%     |
| **Templates**            | 12                        | 14                                                         | +17%     |
| **Reference docs**       | 2 (process, tools)        | 5 (+ deploy, intake, translate)                            | +150%    |
| **Guides**               | 1 (README)                | 4 (README + HOW_IT_WORKS + INTAKE_GUIDE en/vi)             | +300%    |
| **Commands**             | 18                        | 20                                                         | +11%     |
| **Functional commands**  | 16 doing things           | 18 doing things                                            | +13%     |
| **SKILL.md độ dài**      | 298 dòng                  | 461 dòng                                                   | +55%     |
| **Multi-locale support** | Không                     | Có (translate command, glossary, locales config)           | +∞       |
| **External sources**     | Không                     | Có (Notion / GitHub / GDrive / URL / file)                 | +∞       |
| **Deploy automation**    | Không                     | Có (Docusaurus preset, dry-run, sync-deletes)              | +∞       |
| **Re-run safety**        | Không                     | 4-option gate, KB inheritance                              | +∞       |
| **Drift detection**      | Không                     | walkthrough 3-phase với drift report                       | +∞       |

---

## Commands: cái nào giữ, cái nào đổi, cái nào mới

### Giữ nguyên (15 commands có cả 2 versions)

`help`, `audience`, `plan`, `voice`, `draft`, `edit`, `walkthrough`, `record`, `verify`, `publish` — danh sách core không đổi nhiều.

### Đổi tên hoặc gộp (3 commands)

| v1.1.0          | v1.5.1                               | Lý do                                              |
| --------------- | ------------------------------------ | -------------------------------------------------- |
| `start`         | gộp vào `init`                       | Trùng chức năng                                    |
| `validate`      | gộp vào `walkthrough --check`        | Cùng làm chuyện kiểm tra                           |
| `test`          | gộp vào `walkthrough` (auto)         | Hiếm khi chạy độc lập                              |
| `peer-review`   | xoá (gate trong `run` thay thế)      | Là human gate, không cần command                   |
| `tech-review`   | xoá (gate trong `run` thay thế)      | Là human gate, không cần command                   |
| `review-plan`   | xoá (`run --pause-at after-plan`)    | Là human gate, không cần command                   |
| `incorporate`   | gộp vào `edit --from-review`         | Cùng là apply feedback                             |
| `sitemap`       | gộp vào `plan` (output luôn)         | Sitemap là output của plan                         |

### Mới (10 commands không có ở v1.1.0)

| Command         | Mục đích                                                         |
| --------------- | ---------------------------------------------------------------- |
| `init`          | Setup workspace + intake form scaffold                           |
| `module`        | Manage per-feature module intakes                                |
| `intake-help`   | Print field reference cho intake                                 |
| `fetch`         | Pull external sources                                            |
| `run`           | Orchestrated pipeline với pause gate                             |
| `continue`      | Resume sau gate                                                  |
| `update`        | Detect external source changes                                   |
| `translate`     | Multi-locale translation                                         |
| `categorize`    | Docusaurus category file generator                               |
| `deploy`        | Sync workspace sang host project                                 |

**Kết luận command**: số lượng tăng nhẹ (18 → 20), nhưng functional thay đổi lớn. v1.1.0 là "tool viết doc"; v1.5.1 là "tool tự động hóa vòng đời doc với external integrations và deploy automation".

---

## Cấu trúc workspace: thay đổi căn bản

### v1.1.0 (đầu — "docs cho documentation")

```
docs/                              ← workspace tên là "docs"
├── plan/
├── standards/
├── drafts/                        ← flat (chưa có locale)
├── walkthrough/
├── images/
└── videos/
```

Đặc điểm: flat, đơn giản, không có config, không có audit trail, không có deploy target.

### v1.5.1 (hiện tại)

```
documentation/                     ← rename, deploy target dùng "docs"
├── intake/                        ← MỚI: BA-friendly config forms
│   ├── project.md
│   ├── modules/
│   │   └── instances.md
│   └── sources.lock.yaml          ← MỚI: external source state
├── plan/
├── standards/
│   └── glossary.<locale>.yaml     ← MỚI: translation glossary
├── drafts/<locale>/<module>/      ← MỚI: locale-aware, module-aware
├── walkthrough/
│   ├── drift/<ts>/                ← MỚI: drift detection runs
│   └── active-product-bugs.yaml   ← MỚI: cross-run product bug tracker
├── archive/<ts>/                  ← MỚI: re-run protocol backups
├── images/<module>/
├── videos/
├── deployments/<ts>-<target>/     ← MỚI: deploy audit trail
├── .cache/sources/                ← MỚI: external source content cache
└── .run-state/<module>.yaml       ← MỚI: orchestration state per module
```

Đặc điểm: hierarchical (locale + module), config-driven, đầy đủ audit trail, deploy automation, integration với external systems.

**Đánh giá**: đây là thay đổi LỚN. Workspace từ chỗ chứa output → chứa config + state + audit + cache + output. Do nhu cầu phát sinh: cần config (BAs cần), cần state (run/continue), cần audit (re-run safety), cần cache (external sources fetch).

---

## Versioning timeline — mỗi release giải quyết gì

| Version | Ngày | Trigger | Thêm gì |
|---------|------|---------|---------|
| v1.1.0 | 26/04 | Caption rules cho screenshot, video markers | `record` command, 3 templates (screenshot policy, video marker, video plan) |
| v1.2.0 | 26/04 | Vấn đề Docusaurus integration | `init`, `deploy`, `categorize` commands; deploy preset; `.docsmithrc.yaml` |
| v1.2.1 | 26/04 | Cần guide dễ hiểu | HOW_IT_WORKS.md (en + vi) |
| v1.3.0 | 26/04 | Self-check khi product UI đổi | Re-run protocol, walkthrough drift gate, KB inheritance, delete propagation |
| v1.4.0 | 26/04 | "Phải dịch trước khi deploy" | `translate` command, glossary, multi-locale flow |
| v1.5.0 | 26/04 | "22 commands quá nhiều, BA ngại YAML" | Markdown intake forms, layered config, `run`/`continue`/`update`/`module`/`fetch` commands, lean refactor (cắt 8 commands cũ) |
| v1.5.1 | 26/04 | Audit phát hiện 6 path conflicts | Bug fixes + INTAKE_GUIDE practical guide |

**Quan sát**: mỗi version giải quyết 1 pain point cụ thể bạn nêu ra. Không có version nào "build cho user tưởng tượng". Nhưng tốc độ release dày đặc (7 versions/1 ngày) khiến tích lũy spec nhanh chóng.

---

## Mức độ thay đổi từ v1.1.0 → v1.5.1

### Có thay đổi nhiều không?

**Có. Cụ thể:**

- **Workspace folder rename**: `docs/` → `documentation/`
- **Config file mới**: `.docsmithrc.yaml` (v1.2) → `intake/project.md + modules/<n>.md` (v1.5)
- **Workflow chuyển đổi**: từng command riêng → `run` orchestrator
- **Deploy automation**: thêm hoàn toàn
- **Multi-locale**: thêm hoàn toàn
- **External integrations**: thêm hoàn toàn (Notion/GitHub/GDrive)
- **Re-run safety**: thêm hoàn toàn

### Có drift khỏi mục tiêu gốc không?

**Không. Mục tiêu gốc vẫn được tôn trọng:**

| Mục tiêu v1.1.0 | v1.5.1 |
|---|---|
| Process PRC-010 | ✅ Vẫn theo |
| AI-driven authoring | ✅ Vẫn AI |
| Self-review 5-pass | ✅ Vẫn có (`edit` command) |
| Walkthrough verification | ✅ Vẫn có, mạnh hơn (3-phase) |
| Caption-driven screenshot | ✅ Vẫn có |
| Voice consistency | ✅ Vẫn có |
| BA-friendly | ⚠️ Mới thêm (intake forms) — đúng ý gốc |

### Có overhead quá lớn cho user mới không?

**Có thể.** User chạy lần đầu sẽ thấy:
- Phải hiểu khái niệm: project intake, module intake, source, locale, preset
- Phải edit 2 file MD (project.md + 1 module file)
- Phải set env vars cho credentials và sources
- Phải chọn 6+ checkbox và điền 10+ backtick fields trước khi `run` được

So với v1.1.0: chỉ cần `/docsmith start` và trả lời câu hỏi interactive là chạy.

**Trade-off**: v1.5.1 setup mất 15-30 phút lần đầu, nhưng re-run nhanh hơn và reproducible. v1.1.0 setup nhanh nhưng phải repeat input mỗi lần.

---

## Rủi ro mình thấy

### 1. Chưa test trên project thật

7 versions trong 1 ngày, không có data thực. Mỗi feature là theory hợp lý nhưng:
- AI parse MD form có thực sự deterministic? (chưa test)
- Source fetch Notion/GDrive có work với token thực? (chưa test)
- `run` orchestration có reliable end-to-end? (chưa test)
- KB inheritance trong Update mode có thực sự preserve manual edits? (chưa test)

### 2. Spec có thể quá detail

5 reference docs (process, tools, deploy, intake, translate). Tổng spec ~3000+ dòng. AI khi load skill phải đọc cả SKILL.md + reference relevant. Token cost cao mỗi run.

### 3. Số tính năng tăng nhanh hơn integration testing

Mỗi feature mới (deploy, translate, drift, intake, run, update) phụ thuộc lẫn nhau. Bug ở 1 chỗ có thể cascade. Chưa có integration test nào.

### 4. Onboarding path không clear

Có 4 guide (README, HOW_IT_WORKS, INTAKE_GUIDE, CHANGELOG) tổng hơn 2000 dòng. User mới đọc cái nào trước? Hiện chỉ chỉ đường khá generic.

---

## Đánh giá tổng thể

### Bạn đang dò đúng hướng nếu

- Project sắp dùng cho team thật, cần BA điền config (intake forms giải quyết)
- Có Docusaurus repo sẵn (deploy automation giải quyết)
- Có nhiều ngôn ngữ (translate giải quyết)
- Source content nằm Notion/GitHub (fetch giải quyết)
- Có multiple modules/features cần track riêng (module intake giải quyết)

### Bạn đang over-engineering nếu

- Chỉ có 1 người, 1 module, 5-10 trang doc (v1.1.0 đủ rồi)
- Source content chỉ trong đầu bạn (không cần fetch)
- 1 ngôn ngữ (không cần translate)
- Chưa có Docusaurus (standalone preset đủ)
- Doc 1 lần làm xong không update (không cần re-run safety, drift detection, update command)

---

## Recommend

**Nếu chưa test**: STOP build, test v1.5.1 trên 1 project nhỏ thật. 30 phút - 1 buổi. Output là list "chỗ nào AI làm sai, chỗ nào ngại, chỗ nào không cần thiết". Đây là feedback duy nhất có giá trị thực sau 7 versions.

**Nếu cảm thấy phình to**: cân nhắc maintain 2 branches:
- `lean` branch — pin ở v1.1.0 hoặc v1.2.x cho project nhỏ
- `main` branch — v1.5+ cho project enterprise có Docusaurus + multi-locale

**Nếu hài lòng với hướng đi**: giữ v1.5.1 là final cho v1.x. Test 2-4 tuần. v1.6 chỉ làm khi có pain point cụ thể từ usage thực.

---

## Số liệu cụ thể (cho kỹ thuật)

```
v1.1.0 to v1.5.1 file changes:

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
