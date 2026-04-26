# Cách docsmith hoạt động

Tài liệu này giải thích cơ chế vận hành của skill: cái gì chạy khi nào, đọc gì, ghi đi đâu, các command kết nối với nhau ra sao. Đọc một lần trước khi dùng docsmith trên dự án thật.

> 🇬🇧 English version: [HOW_IT_WORKS.md](HOW_IT_WORKS.md)

## TL;DR

Docsmith là một **Claude Code skill** (tập hợp instruction dạng markdown cho AI), không phải chương trình đã compile. Bạn chạy command như `/docsmith deploy MyProduct`. Claude đọc instruction trong `SKILL.md`, rồi thực thi các bước bằng tool có sẵn (read/write file, bash...) — tự sinh code on-the-fly khi cần.

Có **19 commands** xếp thành pipeline. Mỗi command đọc `.docsmithrc.yaml` (config dự án của bạn), chỉ ghi trong các path đã scope (workspace + deploy target), và để lại audit trail.

---

## 1. Mô hình 2 lớp

```
┌──────────────────────────────────────────────────────────────────┐
│  Lớp 1: docsmith skill (repo này)                                │
│  Instruction markdown để Claude làm theo                         │
│  ─────────────────────────────────────                           │
│  SKILL.md ............ catalog command + spec từng command       │
│  deploy-reference.md . logic deploy chi tiết                     │
│  process-reference.md  chi tiết quy trình PRC-010                │
│  presets/*.yaml ...... mapping deploy (standalone, docusaurus)   │
│  templates/*.md ...... format output Claude điền vào             │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │ Claude đọc instruction,
                              │ dùng tool (bash, view, etc.),
                              │ ghi vào project của bạn
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  Lớp 2: project của bạn (nơi chạy /docsmith)                     │
│  ─────────────────────────────────────                           │
│  .docsmithrc.yaml .... config (do /docsmith init tạo)            │
│  documentation/ ...... workspace (drafts, plans, walkthrough)    │
│  deployments/ ........ audit trail của các lần deploy            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │ /docsmith deploy
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  Lớp 3: deploy target (optional — Docusaurus / host khác)        │
│  ─────────────────────────────────────                           │
│  docs/ ............... file markdown (có frontmatter)            │
│  i18n/<locale>/...... markdown theo từng locale                  │
│  static/img/<slug>/.. ảnh, có namespace theo product slug        │
└──────────────────────────────────────────────────────────────────┘
```

Lớp 1 không thay đổi khi bạn dùng (skill read-only).
Lớp 2 do `/docsmith init` tạo và là nơi đa số command ghi vào.
Lớp 3 là site doc đã publish, chỉ được ghi bởi `/docsmith deploy`.

---

## 2. 19 commands và vị trí trong pipeline

```
SETUP             init  ──▶ tạo .docsmithrc.yaml + workspace
                    │
                    ▼
PLAN          audience ──▶ AUDIENCE PROFILE
                  plan  ──▶ DOC PLAN + TRACEABILITY MATRIX
            review-plan ──▶ (cổng phê duyệt — Human)
                sitemap ──▶ SITEMAP
                  voice ──▶ VOICE CHART + UX PATTERNS + SCORECARD
                    │
                    ▼
AUTHOR            draft ──▶ documentation/drafts/<source-locale>/*.md
                            (với placeholder ảnh + video markers)
                   edit ──▶ tự review 5 lượt
                    │
                    ▼
VERIFY      walkthrough ──▶ thay placeholder bằng ảnh thật,
                            chạy test cases trên product live
                 record ──▶ quay video tutorial ngắn (optional)
               validate ──▶ chỉ chạy test cases (không chụp ảnh)
                   test ──▶ tạo test cases từ doc có sẵn
                 verify ──▶ audit tổng hợp 10 checks
                    │
                    ▼
REVIEW     peer-review  ──▶ review từ đồng nghiệp
           tech-review  ──▶ tech review (optional)
            incorporate ──▶ AI áp dụng feedback
                    │
                    ▼
DEPLOY      categorize  ──▶ tạo _category_.json (Docusaurus)
                 deploy ──▶ copy workspace sang host project
                            kèm frontmatter, rewrite ảnh, escape MDX
                    │
                    ▼
PUBLISH        publish  ──▶ Human: git diff / commit / push trên target
```

Command chạy 1 lần: `init`.
Command chạy nhiều lần khi doc evolve: `draft`, `edit`, `walkthrough`, `record`, `verify`, `deploy`.
Command chạy khi tổ chức lại: `sitemap`, `categorize`.

---

## 3. Lệnh `/docsmith init` làm gì

Chạy đầu tiên, ở thư mục bạn muốn lưu doc.

```bash
mkdir my-product-docs && cd my-product-docs
/docsmith init
```

Câu hỏi tương tác (Claude hỏi bạn):

1. **Product slug** — tên ngắn kebab-case (VD: `mycloud`). Trở thành namespace ảnh `/img/mycloud/...` khi deploy. Phải duy nhất toàn cục giữa các product cùng share một Docusaurus target.
2. **Tên hiển thị** — dùng trong title (VD: "MyCloud").
3. **Source locale** — ngôn ngữ bạn draft (mặc định `en`).
4. **Target locales** — các ngôn ngữ đích, cách nhau bằng dấu phẩy (VD: `vi,jp`). Optional. Cấu trúc thư mục được tạo; auto-translate sẽ có ở v1.6.
5. **Deploy preset** — `standalone` (mặc định, không có host) hoặc `docusaurus`.
6. **Deploy target path** — chỉ khi chọn `docusaurus`. Đường dẫn host project (`../my-docusaurus-site`) hoặc `.` cho in-place mode.

Nếu preset là `docusaurus`, Claude **inspect** target trước khi viết config:
- Đọc `<target>/CLAUDE.md` nếu có
- Parse `<target>/docusaurus.config.{ts,js,mjs}` để lấy `i18n`, `docs.path`, `staticDirectories`
- Confirm path đã detect với bạn

Sau khi confirm, Claude ghi:

```
.docsmithrc.yaml                 # config của bạn
documentation/
├── plan/                         # rỗng, dành cho command plan
├── standards/                    # rỗng, dành cho command voice
├── drafts/<source-locale>/       # rỗng, draft sẽ ghi vào đây
├── drafts/<each-target-locale>/
│   └── README.md                 # giải thích: rỗng cho đến v1.6 auto-translate
├── walkthrough/
│   ├── test-cases/
│   ├── video-plan/
│   └── executions/
├── images/                       # walkthrough ghi vào đây
└── videos/
    ├── raw/                      # raw recordings (gitignored)
    └── (mp4 output)
deployments/                      # audit trail, rỗng
```

Sau bước này, mọi command khác đọc `.docsmithrc.yaml` đầu tiên để biết ghi đi đâu.

---

## 4. `.docsmithrc.yaml` — nguồn sự thật duy nhất

Mọi command đọc file này trước khi làm bất cứ việc gì. Nó scope path và ngăn ghi nhầm ra ngoài project.

Các field quan trọng (schema đầy đủ trong [.docsmithrc.example.yaml](.docsmithrc.example.yaml)):

```yaml
product:
  slug: mycloud                 # namespace ảnh
  name: "MyCloud"

locales:
  source: en                    # ngôn ngữ draft
  targets: [vi, jp]             # scaffolding cho translation (chưa auto-translate)

paths:
  workspace: documentation/     # nơi command được phép ghi
  deployments: deployments/     # vị trí audit trail

deploy:
  default_target: ../my-site    # nơi deploy copy đến (rỗng = standalone)
  preset: docusaurus            # standalone | docusaurus
  on_collision: warn            # warn | skip | overwrite | prompt

docusaurus:                     # chỉ dùng khi preset = docusaurus
  docs_path: docs
  i18n_path: i18n
  static_path: static
  image_subpath: img/{product.slug}
  inject_frontmatter: true
  generate_categories: true
  generate_sidebars: false      # mặc định an toàn — không override sidebars.js của bạn
```

**Path scoping**: mọi command validate path tuyệt đối nằm trong `paths.workspace` HOẶC `deploy.default_target` trước khi ghi. Reject nếu nằm ngoài. Đây là safety net chính chống bug AI ghi vào nơi không liên quan trong host project.

---

## 5. Vòng lặp authoring (draft → walkthrough → record)

Khi plan xong, bạn lặp:

```
┌─────────────────────────────────────────────────────────┐
│  1. /docsmith draft MyProduct                           │
│     AI viết:                                            │
│       documentation/drafts/en/getting-started.md        │
│       documentation/drafts/en/instances/create.md       │
│     Với placeholders:                                   │
│       ![Caption text](https://placehold.co/600x400)     │
│       <!-- VIDEO id: instance-tour ... -->              │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  2. /docsmith edit MyProduct                            │
│     AI chạy 5 lượt tự review (rõ ràng, voice,           │
│     scorecard, completeness, accuracy)                  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  3. /docsmith walkthrough MyProduct                     │
│     AI:                                                 │
│       a. Build capture plan từ tất cả placeholders      │
│       b. SHOW PLAN cho bạn review caption               │
│       c. Chạy test cases trên product live              │
│       d. Chụp screenshots thật                          │
│       e. Replace URL placehold.co bằng path thật        │
│          /images/instances/create-form-filled.png       │
│       f. Fix mọi doc bug do test case phát hiện         │
│       g. Ghi log execution → walkthrough/executions/    │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼ (optional)
┌─────────────────────────────────────────────────────────┐
│  4. /docsmith record MyProduct                          │
│     AI:                                                 │
│       a. Scan các marker <!-- VIDEO ... -->             │
│       b. Validate field bắt buộc của mỗi marker         │
│       c. Record screen → transcode → mp4                │
│       d. Replace marker bằng <video> tag,               │
│          giữ marker dưới dạng comment để re-record sau  │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  5. /docsmith verify MyProduct                          │
│     AI chạy đủ 10 checks:                               │
│       không còn placeholder, link OK, voice score đạt,  │
│       test cases pass, etc.                             │
│     Báo issue, không tự fix                             │
└─────────────────────────────────────────────────────────┘
```

Nếu `verify` fail → fix → chạy lại từ bước fail (thường là `walkthrough` hoặc `record`).

### Image flow trong vòng lặp

| Stage          | Markdown chứa                                        | File trên đĩa                                              |
| -------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| Sau `draft`    | `![Form filled](https://placehold.co/600x400)`       | (chưa có file)                                             |
| Sau `walkthrough` | `![Form filled](/images/instances/create-form-filled.png)` | `documentation/images/instances/create-form-filled.png`    |
| Sau `deploy`   | `![Form filled](/img/mycloud/instances/create-form-filled.png)` | `<target>/static/img/mycloud/instances/create-form-filled.png` |

Slug `mycloud` namespace được thêm vào ở thời điểm deploy, không phải lúc capture. Điều này giữ workspace của bạn portable nếu cần deploy sang nhiều target khác nhau.

---

## 6. Command `deploy` — chuyển từ workspace sang host project

Đây là chỗ thú vị nhất. Đọc [deploy-reference.md](deploy-reference.md) để biết logic đầy đủ; phần dưới là model người dùng.

### Luôn bắt đầu bằng dry-run

```bash
/docsmith deploy MyProduct --dry-run
```

Dry-run chạy detect + plan rồi in ra những gì sẽ xảy ra:

```
Target: ../mycloud-docusaurus (docusaurus 3.x)
Detected config:
  docs_path: docs
  i18n_path: i18n
  static_path: static
  default_locale: en
  locales: [en, vi]

Plan summary:
  43 files to create
  8 files to update
  12 files unchanged (skip)
  2 conflicts (BLOCKING)

Conflicts:
  ! docs/getting-started.md
    Reason: target hash differs from workspace hash
    Resolve: re-run with --force, or set on_collision: overwrite

New files:
  + docs/getting-started.md
  + static/img/mycloud/instances/create-form-filled.png
  ...
```

Bạn review plan, fix conflict, rồi chạy lại không có `--dry-run` để apply.

### Deploy thực sự transform gì

Cho mỗi file markdown, deploy áp dụng theo thứ tự:

1. **Inject frontmatter** nếu thiếu — `id`, `title`, `sidebar_position`. Không overwrite field bạn đã viết sẵn.
2. **Rewrite image refs** — `/images/<feature>/<asset>` → `/img/<product.slug>/<feature>/<asset>`.
3. **Rewrite video refs** — `/videos/<id>.mp4` → `/videos/<product.slug>/<id>.mp4`.
4. **Escape MDX specials** — `{` và `}` trong prose body thành `\{` và `\}`. Code blocks và frontmatter giữ nguyên.

Sau đó copy:
- File markdown sang `<target>/docs/...` (source locale) hoặc `<target>/i18n/<locale>/.../current/...` (target locales)
- Ảnh sang `<target>/static/img/<slug>/...`
- Video sang `<target>/static/videos/<slug>/...`

### Audit trail

Mỗi lần deploy tạo `deployments/<timestamp>-<target>/` với:

```
manifest.yaml          # planned + actual results
target-config.yaml     # detected target config (cache cho deploy lần sau)
diff.md                # tóm tắt human-readable
pre-deploy-state.txt   # hashes file ở target TRƯỚC deploy (cho rollback thủ công)
```

Rollback thủ công: dùng các path trong `pre-deploy-state.txt` để restore từ git. Docsmith không tự rollback — git trên target project là safety net chính.

### In-place mode

Nếu workspace docsmith nằm trong chính Docusaurus project:

```yaml
# .docsmithrc.yaml ở root Docusaurus repo
deploy:
  default_target: .
  preset: docusaurus
```

Khi đó `documentation/` và `docs/` là sibling trong cùng 1 repo. `deploy` vẫn tạo audit trail và áp dụng transform; chỉ khác là source và target nằm trên cùng 1 đĩa.

---

## 7. Categorize — sidebars Docusaurus

Chạy sau deploy (hoặc tự động trong deploy nếu `generate_categories: true`):

```bash
/docsmith categorize MyProduct
```

Nó làm gì:

1. Đọc `documentation/plan/sitemap.md`
2. Walk thư mục target `docs/`
3. Với mỗi folder có trong sitemap → ghi `_category_.json`:
   ```json
   {
     "label": "Getting Started",
     "position": 1,
     "link": { "type": "generated-index" }
   }
   ```
4. Với folder KHÔNG có trong sitemap → list trong report, không động vào

Title normalization (consistent giữa các sibling):
- Acronym viết hoa: `API`, `CLI`, `JSON`, `URL`, `HTTP`, `SSH`...
- Mạo từ viết thường giữa câu: `a`, `an`, `the`, `to`, `of`, `for`...
- Từ đầu và cuối luôn viết hoa

Ẩn folder không documented là một **limitation đã biết** — Docusaurus không có cơ chế "hide" first-class. Các option được ghi trong [deploy-reference.md](deploy-reference.md): exclude trong `docusaurus.config`, custom sidebars, hoặc move folder ra khỏi `docs/`.

---

## 8. Locales hiện tại (1.2.0) và tương lai (1.6)

Chạy được trong 1.2.0:

- Cấu trúc thư mục: `documentation/drafts/<locale>/` cho mỗi locale
- Deploy map mỗi locale sang đúng path Docusaurus i18n
- Bạn có thể tự tạo translation trong `drafts/vi/`, `drafts/jp/` và deploy sẽ pick up

Chưa có:

- AI auto-translate. Command `translate` ở v1.6 roadmap.
- Translation drift tracking (khi EN update, biết file VI nào đã stale)

Đến trước v1.6, workflow multi-locale như sau:

```bash
# Draft EN
/docsmith draft MyProduct
# documentation/drafts/en/foo.md được tạo

# Tự tạo translation VI
cp documentation/drafts/en/foo.md documentation/drafts/vi/foo.md
# Edit drafts/vi/foo.md thủ công hoặc bằng tool khác

# Deploy cả hai
/docsmith deploy MyProduct
# Copy drafts/en/* → docs/, drafts/vi/* → i18n/vi/.../current/
```

---

## 9. Re-run command an toàn (1.3.0+)

Docsmith không cho phép silent overwrite. Mọi command check xem output đã tồn tại chưa, nếu có sẽ show gate 4 lựa chọn:

| Lựa chọn | Khi nào dùng |
|---|---|
| **Update** (khuyên dùng) | Content cũ đã có edit từ team đáng giữ. AI đọc cái cũ làm KB, propose delta, apply từng item. |
| **Overwrite** | Cái cũ đã quá outdated, muốn viết mới. File gốc archive vào `documentation/archive/<timestamp>/`. |
| **Side-by-side** | So sánh hoặc rewrite lớn. File mới có suffix `-v2`. |
| **Cancel** | Hủy. |

**Quy tắc Update mode**: AI đọc content cũ là **canonical** và chỉ propose delta (NEW / UPDATE / REMOVE / KEEP). Section không thay đổi được giữ nguyên verbatim. Đây là quan trọng — không có quy tắc này thì mỗi re-run sẽ silently mất edit thủ công của team.

**Drift detection** (riêng walkthrough) là pipeline 3 phase:

```
A: VERIFY  /docsmith wt --check         → drift-report.md (read-only)
   ↓
   GATE    Edit decisions.yaml          (auto-fix / manual-fix / product-bug / skip)
   ↓
B: APPLY   /docsmith wt --apply         → update drafts theo decisions
   ↓
C: CAPTURE                              → screenshots
```

Item marked `product-bug` (doc đúng, UI bị regression) được track trong `walkthrough/active-product-bugs.yaml` qua nhiều run. Không bị re-flag cho đến khi UI khớp lại với doc, lúc đó tự auto-resolve.

**Delete propagation**: khi bạn xóa 1 draft, `deploy` report orphan file ở target nhưng mặc định KHÔNG xóa. Dùng `--sync-deletes` để thực sự cleanup:

```bash
/docsmith deploy MyProduct --sync-deletes --dry-run    # preview xóa
/docsmith deploy MyProduct --sync-deletes              # apply
```

File ở target bị xóa được backup vào `deployments/<ts>/deleted/` để audit.

Logic đầy đủ và quy tắc merge per-artifact: xem [update-reference.md](update-reference.md).

---

## 10. Mental model: skill = prompt, không phải program

Đây là điều quan trọng nhất cần internalize.

Plugin truyền thống có code đã compile. Khi gọi function, behavior deterministic. Bug → patch code → re-test.

Skill docsmith có **instruction tiếng Anh dạng markdown**. Khi bạn chạy `/docsmith deploy`, Claude:

1. Đọc `SKILL.md` và `deploy-reference.md` vào context
2. Đọc `.docsmithrc.yaml` của bạn
3. **Sinh code on-the-fly** (Python, bash, regex) để làm việc
4. Thực thi code đó qua tool (`bash_tool`, `view`, `create_file`, `str_replace`)

Hệ quả:

- **Không reproducible 100%**. Cùng command 2 lần có thể chọn path hơi khác hoặc format output khác.
- **Luôn dry-run trước** với `deploy`. Plan là cách bạn check an toàn.
- **Audit trail quan trọng hơn bình thường**. `deployments/<ts>/` là cách reconstruct lại đã làm gì.
- **Đừng để AI tự commit**. Bạn review `git diff` trên target và tự commit.
- **Fix bug** = sửa prose trong `SKILL.md` hoặc `deploy-reference.md`, không phải patch code. Bump version, push.
- **Không có unit test**. Cách verify skill đúng là chạy thật trên project thật và xem output.

Trade-off này (linh hoạt thay cho deterministic) chính là điểm cốt lõi của format skill.

---

## 11. Cheat-sheet workflow hằng ngày

```bash
# Setup 1 lần
mkdir mycloud-docs && cd mycloud-docs
/docsmith init

# Draft lần đầu (product mới)
/docsmith audience MyCloud
/docsmith plan MyCloud
/docsmith review-plan MyCloud      # human approval
/docsmith sitemap MyCloud
/docsmith voice MyCloud
/docsmith draft MyCloud
/docsmith edit MyCloud
/docsmith wt MyCloud               # walkthrough — chụp screenshot
/docsmith rec MyCloud              # optional — record video
/docsmith verify MyCloud
/docsmith deploy MyCloud --dry-run # preview
/docsmith deploy MyCloud           # apply
cd ../mycloud-docusaurus
git diff && git add . && git commit -m "Update docs" && git push

# Iterate trên 1 doc sau khi product UI thay đổi
/docsmith wt MyCloud documentation/drafts/en/instances/create.md
/docsmith verify MyCloud documentation/drafts/en/instances/create.md
/docsmith deploy MyCloud --dry-run
/docsmith deploy MyCloud

# Re-record video sau khi UI đổi
# (Edit marker trong doc nếu cần, hoặc cứ chạy lại)
/docsmith rec MyCloud
/docsmith deploy MyCloud --dry-run
```

---

## 12. Khi có vấn đề, xem ở đâu

| Triệu chứng                                          | Xem                                                                          |
| ---------------------------------------------------- | ---------------------------------------------------------------------------- |
| Claude không biết command                            | Verify skill đã cài: `ls ~/.claude/skills/docsmith/SKILL.md`                  |
| Walkthrough chụp sai file                            | Caption rules trong [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) — review caption |
| Deploy ghi vào path lạ                               | `.docsmithrc.yaml` paths, đặc biệt `paths.workspace` và `deploy.default_target` |
| Ảnh broken trên site Docusaurus                      | Check `static/img/<slug>/...` tồn tại; check ref trong `docs/*.md` khớp       |
| MDX build fail vì ký tự `{`                          | Deploy chưa escape — chạy lại, hoặc thêm escape thủ công                     |
| `_category_.json` sai title                          | Title trong sitemap (categorize dùng sitemap làm nguồn sự thật)              |
| Không biết deploy lần trước đã làm gì                | `deployments/<latest-timestamp>/manifest.yaml` và `diff.md`                  |
| Muốn rollback một lần deploy                         | Dùng git trên target project (preferred) hoặc `pre-deploy-state.txt`         |
| Field trong `.docsmithrc.yaml` không hiểu            | [.docsmithrc.example.yaml](.docsmithrc.example.yaml) có schema đầy đủ kèm comment |
| Câu hỏi về logic deploy                              | [deploy-reference.md](deploy-reference.md)                                   |
| Câu hỏi về quy trình / tác nghiệp                    | [process-reference.md](process-reference.md), [subprocess-010a.md](subprocess-010a.md) |

---

## 13. Đọc tiếp gì

- [.docsmithrc.example.yaml](.docsmithrc.example.yaml) — schema config có comment
- [SKILL.md](SKILL.md) — command reference đầy đủ (đây là cái Claude đọc)
- [deploy-reference.md](deploy-reference.md) — logic deploy chi tiết
- [templates/SCREENSHOT_POLICY_TEMPLATE.md](templates/SCREENSHOT_POLICY_TEMPLATE.md) — caption rules
- [templates/VIDEO_MARKER_TEMPLATE.md](templates/VIDEO_MARKER_TEMPLATE.md) — cú pháp video marker
- [CHANGELOG.md](CHANGELOG.md) — lịch sử version + migration notes
- [README.md](README.md) — install + quick start

Nếu bạn dùng docsmith trên dự án thật, đọc tối thiểu mục 1–4.
