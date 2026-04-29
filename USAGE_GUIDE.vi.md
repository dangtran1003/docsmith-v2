# Hướng dẫn sử dụng docsmith — từ cơ bản đến nâng cao

Tài liệu này hướng dẫn dùng skill `docsmith` qua từng kịch bản thực tế: từ project đầu tiên (5 phút) đến các tính năng nâng cao như multi-locale, video voiceover, drift detection, deploy Docusaurus.

> **Đọc trước:** [SETUP.vi.md](SETUP.vi.md) (cài browser tool, env vars, ffmpeg/TTS) và [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md) (cách điền intake form).
> **Tham khảo nội bộ:** [HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md) (operating model), [CHANGELOG.md](CHANGELOG.md) (lịch sử version).

---

## Mục lục

- [Phần 1 — Cơ bản: Project đầu tiên](#phần-1--cơ-bản-project-đầu-tiên)
- [Phần 2 — Trung cấp: Quy trình thực tế](#phần-2--trung-cấp-quy-trình-thực-tế)
- [Phần 3 — Nâng cao: Tinh chỉnh sâu](#phần-3--nâng-cao-tinh-chỉnh-sâu)
- [Phần 4 — Bảo trì & cập nhật](#phần-4--bảo-trì--cập-nhật)
- [Phần 5 — Xử lý sự cố thường gặp](#phần-5--xử-lý-sự-cố-thường-gặp)
- [Phần 6 — Cheat sheet lệnh](#phần-6--cheat-sheet-lệnh)

---

## Phần 1 — Cơ bản: Project đầu tiên

Mục tiêu: tạo tài liệu cho 1 module, 1 ngôn ngữ, không walkthrough/video. Khoảng 10 phút.

### 1.1. Cài skill (một lần / máy)

```bash
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2
```

Hoặc clone trực tiếp: xem [README.md](README.md) § Install.

### 1.2. Init workspace

```bash
mkdir my-docs && cd my-docs
/docsmith init
```

Kết quả: tạo `documentation/intake/project.md` + folder structure. Footprint duy nhất ở root: thư mục `documentation/`.

### 1.3. Tạo module

```bash
/docsmith module pricing
```

Tạo `documentation/intake/modules/pricing.md`.

### 1.4. Điền intake (chỉ phần essentials)

Mở `documentation/intake/project.md` và `documentation/intake/modules/pricing.md`. **Bỏ qua mọi section "Advanced — ..." (đang collapsed)**. Chỉ điền:

- **Project intake**: Product info, Audience (1 persona chính), Languages (`source: en`, không target → 1 locale duy nhất), Knowledge sources (ít nhất 1 nguồn — file local hoặc URL công khai là dễ nhất)
- **Module intake**: Module identity, Scope (1-3 features cần document)

Xem [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md) § "Quy tắc 4: Bỏ qua Advanced" để biết bộ tối thiểu.

### 1.5. Chạy pipeline (skip walkthrough/record)

```bash
/docsmith run pricing --pause-at after-draft
```

AI sẽ chạy tự động: validate intake → fetch sources → audience → plan → voice → draft → edit, sau đó **dừng** ở after-draft.

### 1.6. Review draft

Mở `documentation/drafts/en/pricing/`, đọc lại nội dung AI viết. Sửa thủ công nếu cần.

### 1.7. Verify

```bash
/docsmith verify pricing
```

Chạy 11 check (placeholders, voice, links, ...). Sửa lỗi nếu có.

**Tới đây bạn đã có 1 bộ docs hoàn chỉnh.** Nếu chỉ cần markdown để paste vào nơi khác, dừng lại. Nếu muốn deploy → đi tiếp Phần 2.

---

## Phần 2 — Trung cấp: Quy trình thực tế

Bao gồm: nhiều module, nhiều nguồn knowledge, multi-locale, walkthrough capture screenshot, deploy Docusaurus.

### 2.1. Multi-module project

```bash
/docsmith module instances
/docsmith module storage
/docsmith module pricing
/docsmith module list                    # liệt kê + status
/docsmith module archive old-feature     # tạm bỏ qua trong run
```

Mỗi module có intake riêng. Chạy `/docsmith run` (không argument) sẽ chạy tất cả module active.

Tip: dùng `--from` để clone setup từ module có sẵn (sources, voice override):

```bash
/docsmith module billing --from pricing
```

### 2.2. Knowledge sources từ nhiều nơi

Trong `project.md` § Knowledge sources, tick các loại nguồn cần dùng:

- **Notion**: cần `NOTION_TOKEN` env var (xem [SETUP.vi.md](SETUP.vi.md) § "Environment variables")
- **GitHub** (private repo): `GITHUB_TOKEN`
- **Google Drive**: `GOOGLE_DRIVE_*` qua MCP
- **URL công khai**: không cần token
- **File local**: đường dẫn tương đối từ project

Mỗi source có 1 ID (vd `notion-pricing-spec`). AI fetch về `documentation/.cache/sources/<id>.md` (gitignored) và ghi metadata vào `sources.lock.yaml`.

```bash
/docsmith fetch                          # fetch tất cả
/docsmith fetch --source notion-pricing  # fetch 1 source
```

`run` tự gọi `fetch` ở stage đầu, không cần chạy tay trừ khi muốn force refresh.

### 2.3. Multi-locale

Trong `project.md` § Languages:

```markdown
- Source locale: `en`
- Target locales:
  - [x] vi
  - [x] jp
```

Sau khi xong draft, pipeline `run` sẽ tự gọi `translate` để dịch sang `vi` và `jp`. Mặc định review mode = `batch` (xem cả file diff). Dùng `--per-block` để review từng block (an toàn hơn, chậm hơn).

```bash
/docsmith translate                       # dịch tất cả module, tất cả target
/docsmith translate pricing --locale vi   # 1 module + 1 locale
/docsmith translate --auto-approve        # CI/bulk, skip review (dùng cẩn thận)
```

**Glossary** (khuyến nghị): tạo `documentation/standards/glossary.vi.yaml` để cố định thuật ngữ. Build dần qua các lần review correction. Xem `templates/GLOSSARY_TEMPLATE.yaml`.

### 2.4. Walkthrough — verify draft với product thật

**Yêu cầu**: cần browser automation (Claude in Chrome extension HOẶC Playwright MCP) + test account credentials. Xem [SETUP.vi.md](SETUP.vi.md).

Pipeline 3 phase:

```bash
/docsmith walkthrough --check            # Phase A: VERIFY (read-only, ~30s)
                                          # Tạo drift report
# Edit documentation/walkthrough/drift/<ts>/decisions.yaml
# Đánh dấu mỗi item: auto-fix / manual-fix / product-bug / skip
/docsmith walkthrough --apply            # Phase B + C: apply + capture screenshots
```

Hoặc 1 lệnh full:

```bash
/docsmith walkthrough                    # A → gate → B → C
```

Screenshot lưu ở `documentation/images/<module>/`. Markdown trong draft có placeholder `![caption](placehold.co/...)`, sau capture sẽ thay bằng đường dẫn thật.

**Drift report**: liệt kê chỗ doc sai khác UI (button đổi tên, screen mới, ...). Item đánh dấu `product-bug` track trong `walkthrough/active-product-bugs.yaml` và auto-resolve khi UI khớp lại.

### 2.5. Deploy lên Docusaurus

```bash
/docsmith deploy --dry-run               # Preview: in plan, không write
/docsmith deploy                          # Apply
cd ../my-docusaurus-site
git diff && git add . && git commit -m "Update docs" && git push
```

`deploy` làm:
- Inject frontmatter Docusaurus (id, title, sidebar_position)
- Namespace image paths (`/img/<slug>/`)
- Escape MDX (curly braces, JSX-like syntax)
- Generate `_category_.json` (nếu preset = docusaurus)
- Tạo audit trail ở `documentation/deployments/<ts>-<target>/`

Flag hữu ích:
- `--target <path>` — override target_path 1 lần
- `--locale vi` — chỉ deploy 1 locale
- `--sync-deletes` — propagate xóa file (mặc định chỉ report orphan)
- `--force` — bỏ qua conflict warning

### 2.6. In-place mode (đã có Docusaurus)

```bash
cd ~/my-docusaurus-site
/docsmith init --in-place                # Detect docusaurus.config, target = .
/docsmith module pricing
# fill intakes
/docsmith run pricing
/docsmith deploy
```

Workspace `documentation/` và target `docs/`, `i18n/`, `static/` là siblings trong cùng repo Docusaurus.

---

## Phần 3 — Nâng cao: Tinh chỉnh sâu

### 3.1. Sitemap pattern (consistency cross-module)

3 pattern có sẵn (project intake § Sitemap pattern):

- **Pattern A — Learning path** (default): `overview → initial-setup → quickstarts → tutorials → guides → concepts → reference → api-reference → troubleshooting → glossary`
- **Pattern B — Task-first**: `overview → quickstarts → guides → concepts → reference → troubleshooting → glossary`
- **Pattern C — Custom**: tự define order

Mỗi module tick các canonical sections nó include (module intake § Sitemap sections). `plan` warn nếu module thiếu section mà pattern bao gồm.

Override display name (project hoặc module level) để đổi tên hiển thị mà giữ slug canonical:

```markdown
Display name overrides:
- guides: How-To Guides
- api-reference: API Docs
```

Migrate workspace cũ (v1.5.3-): `/docsmith plan --migrate-sitemap`.

### 3.2. Voice & UX patterns

```bash
/docsmith voice                           # Quick mode (default): voice chart
/docsmith voice --full                    # + UX text patterns + scorecard
```

Voice chart sinh từ project intake § Voice and tone. Module có thể override (module intake § Voice override).

5 pass của `edit` đối chiếu draft với voice chart (voice match score). `verify` check #3 enforce score ≥ threshold.

### 3.3. Media policy (screenshot + video)

Project intake § 11 Media policy điều khiển:

- **Screenshot density** per content type (tutorial: cao, reference: thấp, ...)
- **Style** (browser chrome on/off, annotations, blur)
- **Aspect ratio** (16:9, 4:3, mobile)
- **Per-locale strategy**: `source-only` (1 screenshot dùng chung — default), `per-locale` (capture mỗi locale UI riêng)
- **Video density**, length cap, voiceover strategy
- **TTS provider**: local-piper (default, $0), ElevenLabs, OpenAI, Google, Azure, Coqui

Module có thể override (module intake § 8 Media override). Xem `templates/MEDIA_POLICY_TEMPLATE.md`.

`verify` check #11 enforce media compliance.

### 3.4. Video tutorial + voiceover (v1.5.7)

Trong draft, đánh marker:

```markdown
<!-- VIDEO id: instance-create-tour -->
```

Khi chạy `/docsmith record`:

1. AI scan markers → tìm script file `documentation/scripts/<module>/<id>.md`
2. Nếu chưa có → AI sinh script ban đầu từ context xung quanh, **pause** để bạn review
3. Bạn edit script (frontmatter: duration_target, voiceover_strategy, voiceover_provider; body: `# Source script`)
4. Confirm → TTS sinh audio per locale (skip nếu silent), capture screen, encode video
5. Marker được thay bằng `<video>` embed; giữ marker comment để re-record

Translate script: `/docsmith translate` xử lý cả script files (không chỉ draft). Translation lưu trong cùng file dưới heading `## vi`, `## jp`, ...

Flags hữu ích:
- `--check` — validate tất cả marker có script tương ứng
- `--re-record <id>` — force re-record 1 video
- `--migrate-scripts` — extract inline scripts từ draft cũ (v1.5.6-) ra file riêng

Xem `templates/VIDEO_SCRIPT_TEMPLATE.md`.

### 3.5. Glossary management

Tạo `documentation/standards/glossary.vi.yaml` (đường dẫn cố định, không config được):

```yaml
- term: instance
  translation: phiên bản máy
  notes: dùng "phiên bản máy" thay vì "instance" trong tài liệu vận hành
- term: bucket
  translation: bucket
  notes: giữ nguyên (đã quen thuộc với devops VN)
```

Build dần khi review translate. `verify` check #4 enforce glossary consistency.

### 3.6. Re-run protocol & KB inheritance

Mọi command sinh output đều check tồn tại trước. Nếu có:

1. **Update** (recommended) — đọc file cũ làm KB, propose delta (NEW / UPDATE / REMOVE / KEEP). Phần không đổi giữ nguyên verbatim → **không mất manual edit**.
2. **Overwrite** — discard, archive vào `archive/<ts>/`, sinh mới. Cần xác nhận "OVERWRITE".
3. **Side-by-side** — giữ cả 2, file mới có suffix `-v2`.
4. **Cancel** — abort.

Default = `prompt`. Có thể đặt `auto: update` trong project intake § Auto-run behavior để tự chọn Update.

### 3.7. Drift detection & product-bug tracking

```bash
/docsmith walkthrough --check            # Verify-only, fast
```

Sinh `drift/<ts>/decisions.yaml`. Mỗi item:

```yaml
- type: button-renamed
  doc: drafts/en/instances/create.md:42
  ui: "Launch instance" → "Create instance"
  decision: auto-fix         # hoặc: manual-fix, product-bug, skip
  confidence: HIGH
```

`--auto-apply-high-confidence` skip gate, apply HIGH-confidence fixes.

Item `product-bug` track trong `walkthrough/active-product-bugs.yaml` cross-run, auto-resolve khi UI khớp lại doc.

### 3.8. Source change detection

```bash
/docsmith update                          # check tất cả module
/docsmith update pricing                  # 1 module
/docsmith update --auto-apply             # CI mode
```

Workflow:
1. Đọc `sources.lock.yaml`
2. Cheap metadata check: Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime
3. Build change report
4. Confirm → full-fetch source thay đổi → re-run `draft` Update mode → re-run `wt --check`
5. Update lock

Đây là cách tự động phát hiện khi spec/source thay đổi và đề xuất re-draft đúng phần ảnh hưởng.

### 3.9. Verify đầy đủ (11 checks)

```bash
/docsmith verify
/docsmith verify pricing                  # 1 module
/docsmith verify --locale vi              # locale dịch
/docsmith verify "drafts/en/**/api-*.md"  # glob scope
```

11 check: placeholders, broken images, voice score, glossary consistency, code blocks unchanged across locale, frontmatter, internal links, cross-refs, sitemap match, sitemap consistency, media compliance.

### 3.10. Custom auto-run gate

Project intake § Auto-run behavior:

```markdown
- Pause at: `after-plan` / `after-draft` (default) / `before-walkthrough` / `after-walkthrough` / `before-deploy` / `never`
- On existing output: `prompt` (default) / `update` / `overwrite` / `side-by-side`
```

Set `pause-at: never` + `on existing: update` cho CI pipeline tự động.

---

## Phần 4 — Bảo trì & cập nhật

### 4.1. Khi spec thay đổi

```bash
/docsmith update                          # propose targeted re-drafts
# Review proposal → confirm
```

Update mode dùng KB inheritance: phần không đổi giữ nguyên, AI chỉ sinh delta.

### 4.2. Khi UI thay đổi (drift)

```bash
/docsmith walkthrough --check
# Edit decisions.yaml
/docsmith walkthrough --apply
```

### 4.3. Khi thêm locale mới

1. Edit `project.md` § Languages → tick locale mới
2. `/docsmith translate --locale <new>`
3. `/docsmith deploy --locale <new>`

Source locale draft không bị ảnh hưởng. Glossary `<new>.yaml` build dần qua review.

### 4.4. Khi đổi voice / audience

```bash
/docsmith voice                           # Update mode → propose delta
/docsmith edit --from-review feedback.md  # apply reviewer feedback có annotation
```

`edit --from-review` đọc file markdown có annotation `// FEEDBACK: ...` hoặc block comment, apply đúng chỗ.

### 4.5. Khi archive feature cũ

```bash
/docsmith module archive old-feature
```

Module bị skip trong `run`. Draft + intake giữ lại (không xóa). Unarchive khi cần dùng lại.

### 4.6. Khi upgrade docsmith version

```bash
/plugin marketplace update dangtran1003-docsmith-v2
```

Đọc [CHANGELOG.md](CHANGELOG.md) cho breaking change. Một số version có flag migrate (vd `plan --migrate-sitemap` cho v1.5.4, `record --migrate-scripts` cho v1.5.7).

---

## Phần 5 — Xử lý sự cố thường gặp

| Triệu chứng | Nguyên nhân | Cách xử lý |
| --- | --- | --- |
| `init` báo "documentation/ already exists" | Folder cũ không phải docsmith | Move/backup, hoặc `--force` (cần confirm) |
| `fetch` báo `NOTION_TOKEN not set` | Thiếu env var | Set env var theo [SETUP.vi.md](SETUP.vi.md) |
| `walkthrough` báo "no browser tool available" | Chưa cài Claude in Chrome / Playwright MCP | [SETUP.vi.md](SETUP.vi.md) § "Path 1 / Path 2" |
| `record` báo "ffmpeg not found" | Chưa cài ffmpeg | [SETUP.vi.md](SETUP.vi.md) § "Path 3" |
| `translate` không dịch glossary đúng | Chưa có `glossary.<locale>.yaml` | Tạo file theo template |
| `deploy` báo "translation incomplete" | Có locale chưa dịch xong | Chạy `/docsmith translate --locale <x>` trước |
| Re-run mất manual edit | Đã chọn Overwrite thay vì Update | Phục hồi từ `archive/<ts>/`, lần sau chọn Update |
| Sitemap inconsistent across modules | Module thiếu canonical section | Sửa module intake § Sitemap sections, re-run `plan` |
| `update` không detect change | Source backend không expose metadata | Force refresh: `fetch --source <id>` |
| Drift report rỗng nhưng UI rõ ràng đã đổi | Walkthrough cache cũ | `walkthrough --check` lại; nếu vẫn rỗng, kiểm tra credentials/login |

### 5.1. Khi pipeline bị stuck

```bash
/docsmith continue                        # Resume từ .run-state/<module>.yaml
```

Nếu state >7 ngày hoặc workspace bị edit ngoài docsmith, sẽ warn trước khi resume. Có thể xóa `.run-state/<module>.yaml` để bắt đầu lại module đó.

### 5.2. Khi muốn rollback deploy

Mỗi deploy lưu audit ở `documentation/deployments/<ts>-<target>/` (manifest, target-config snapshot, diff, pre-deploy hashes). Restore thủ công từ snapshot, hoặc revert commit ở target repo.

### 5.3. Path scoping reject

Nếu thấy "write outside allowed roots": kiểm tra `target_path` trong project intake. Mọi write phải nằm trong `documentation/` hoặc `deploy.target_path`.

---

## Phần 6 — Cheat sheet lệnh

| Lệnh | Alias | Khi dùng |
| --- | --- | --- |
| `init` | `i` | Lần đầu, scaffold workspace |
| `module <n>` | `m` | Mỗi feature area / nhóm doc |
| `module list` | | Xem trạng thái module |
| `module archive <n>` | | Tạm bỏ qua module |
| `intake-help` | `ih` | Xem nghĩa từng field intake |
| `fetch` | `f` | Force refresh source (thường tự chạy) |
| `run [module]` | `r` | Pipeline đầy đủ, pause ở `after-draft` |
| `continue` | `c` | Resume sau pause |
| `audience` | | Stage rời (debug / iterate) |
| `plan` | `pl` | Stage rời. `--migrate-sitemap` cho upgrade |
| `voice` | `vc` | Stage rời. `--full` để gen UX patterns |
| `draft` | `dr` | Stage rời |
| `edit` | `ed` | Stage rời. `--from-review <file>` apply feedback |
| `walkthrough` | `wt` | Verify với product. `--check`, `--apply`, `--skip-drift` |
| `record` | `rec` | Tutorial video. `--check`, `--re-record <id>`, `--migrate-scripts` |
| `translate [module]` | `tr` | Đa ngôn ngữ. `--locale`, `--per-block`, `--auto-approve` |
| `verify [glob]` | `vf` | 11-check audit. `--locale` |
| `update [module]` | `up` | Detect source change. `--auto-apply` |
| `categorize` | `cat` | Sinh `_category_.json` (Docusaurus) |
| `deploy` | `dep` | Sync sang target. `--dry-run`, `--target`, `--locale`, `--sync-deletes`, `--force` |
| `publish` | `pub` | Checklist git commit/push |
| `help` | `h` | Bảng lệnh |

### Pattern thường dùng

```bash
# Project mới → deploy
/docsmith init && /docsmith module <n>
# fill intake
/docsmith run --pause-at after-draft
# review
/docsmith continue
/docsmith deploy --dry-run && /docsmith deploy

# Maintain
/docsmith update                          # source change
/docsmith walkthrough --check             # UI drift
/docsmith verify                          # health check

# Single module iteration
/docsmith run <module> --from draft       # skip planning, redraft only
/docsmith edit <module>
/docsmith walkthrough <module>

# Đa locale từ đầu
# (đặt locales.targets trong project.md trước)
/docsmith run                             # tự gọi translate
/docsmith translate --per-block           # nếu cần review chặt
```

---

## Tham khảo thêm

- [README.md](README.md) — overview + quick start
- [SETUP.vi.md](SETUP.vi.md) — prerequisites (browser, env vars, ffmpeg, TTS)
- [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md) — chi tiết từng field intake form
- [HOW_IT_WORKS.vi.md](HOW_IT_WORKS.vi.md) — operating model & internals
- [CHANGELOG.md](CHANGELOG.md) — lịch sử version
- [PUBLISHING.md](PUBLISHING.md) — publish skill lên marketplace của riêng bạn
