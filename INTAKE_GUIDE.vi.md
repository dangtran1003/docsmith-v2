# Hướng dẫn điền intake — `project.md` và `module.md`

Hướng dẫn dành cho BA và content owner điền form intake của docsmith. Nếu bạn chưa từng đụng vào file YAML, hướng dẫn này dành cho bạn.

> 🇬🇧 English version: [INTAKE_GUIDE.md](INTAKE_GUIDE.md)

## Bạn cần điền gì

Hai file markdown, đều nằm trong `documentation/intake/`:

```
documentation/intake/
├── project.md                  ← Điền 1 lần cho cả project
└── modules/
    ├── instances.md            ← 1 file mỗi module/feature area
    └── storage.md
```

`project.md` được tạo khi chạy `/docsmith init`. File module được tạo khi chạy `/docsmith module <tên>`.

## Cách edit (3 quy tắc đơn giản)

### Quy tắc 1: Điền vào backtick

Bất cứ thứ gì nằm trong dấu backtick `như thế này` là chỗ bạn điền value. Thay text mặc định bằng giá trị thực của bạn.

**Trước:**
```markdown
- Product slug: `your-product-slug`
```

**Sau:**
```markdown
- Product slug: `mycloud`
```

Đừng xóa backticks. Chỉ thay text bên trong.

### Quy tắc 2: Tick checkbox

Checkbox nhìn như `[ ]` (rỗng) hoặc `[x]` (đã tick). Tick bằng cách thay khoảng trắng bằng `x`.

**Trước:**
```markdown
- [ ] Standalone (không deploy)
- [ ] Docusaurus
```

**Sau:**
```markdown
- [ ] Standalone (không deploy)
- [x] Docusaurus
```

Đa số group bạn tick đúng 1 option. Một vài group (như target languages) cho phép tick nhiều — form sẽ ghi rõ.

### Quy tắc 3: Thêm hoặc xóa section lặp lại

Một số section là template lặp lại. Tìm `### Source 1`, `### Source 2`, `#### Feature 1`, vv. Để thêm mới, copy cả block và tăng số. Để xóa, xóa cả block.

**Đừng lo về section "trống"** — nếu mọi backtick đều empty và mọi checkbox đều không tick trong block Source/Feature, AI sẽ ignore.

### Quy tắc 4: Skip section "Advanced" (v1.5.6+)

Section bắt đầu bằng `<details>` và label "Advanced" mặc định **collapse**. Nếu thấy chúng trong editor như header có thể click — đúng rồi. **Đa số project không cần expand cái nào.** Skill dùng default value bên trong.

Chỉ mở khi:
- Có lý do cụ thể cần customize (regulated industry cần formal voice, module mobile cần aspect ratio khác, vv.)
- Default hiện tại không hợp sau lần chạy đầu

Cho **project đầu tiên**, chỉ điền:
- Section 1 (Product) — BẮT BUỘC
- Section 2 (Audience > Primary persona) — BẮT BUỘC
- Section 3 (Languages > Source và Target) — BẮT BUỘC
- Section 4 (Deploy > Preset) — BẮT BUỘC
- Section 5 (Credentials env var names) — BẮT BUỘC nếu chạy walkthrough
- Section 6 (Source 1) — Optional nhưng recommend

Skip mọi thứ còn lại. Chạy `/docsmith run`. Nếu output không như ý, mới expand Advanced section liên quan và adjust.

## Mỗi section có ý nghĩa gì (project.md)

### 1. Product

Identity cơ bản của product.

| Field | Là gì | Ví dụ |
|---|---|---|
| Product slug | Identifier ngắn dùng trong URL và image namespace. Lowercase, không space, hyphen OK. | `mycloud`, `acme-app` |
| Product display name | Tên dễ đọc cho title | `MyCloud`, `Acme App` |
| Product URL | URL product chạy live (cho walkthrough automation) | `https://console.mycloud.com` |
| One-line description | Product làm gì, 1 câu | `Cloud platform host Docker apps` |

Slug phải unique giữa các product cùng share Docusaurus target — nó trở thành namespace folder ảnh (`/img/<slug>/...`).

### 2. Audience

Ai đọc doc. AI dùng để set tone, vocabulary, kiến thức nền giả định.

**Primary persona** (bắt buộc):

| Field | Là gì | Ví dụ |
|---|---|---|
| Role / job title | Họ làm việc gì | `DevOps Engineer`, `Marketing Manager`, `Junior Developer` |
| Technical level | Mức thoải mái với doc kỹ thuật | Tick 1 trong Low / Medium / High |
| Primary goal | Họ muốn đạt gì khi đọc doc | `Deploy 1 service lên production trong 30 phút` |

**Secondary personas** là optional. Chỉ thêm khi có audience khác biệt rõ ràng (vd "developers VÀ managers"). Đừng thêm persona chỉ vì có thể — mỗi persona làm content khó viết hơn cho mọi người.

### 3. Languages

Doc viết bằng ngôn ngữ gì, dịch sang ngôn ngữ nào.

- **Source**: ngôn ngữ bạn viết. Tick đúng 1.
- **Target**: ngôn ngữ auto-translate. Tick 0 hoặc nhiều. Để trống hết nếu single-language.

Nếu tick target languages, AI tìm glossary file ở `documentation/standards/glossary.<locale>.yaml` (vd: `glossary.vi.yaml` cho tiếng Việt). Glossary là optional nhưng giúp consistency. Xem [templates/GLOSSARY_TEMPLATE.yaml](templates/GLOSSARY_TEMPLATE.yaml).

### 4. Deploy

Doc đi đâu khi publish.

- **Standalone**: doc nằm trong `documentation/drafts/` và xong. Không host project. Chọn nếu bạn chỉ đang draft và chưa quyết publish ở đâu.
- **Docusaurus**: deploy sync sang Docusaurus repo. Cho biết ở đâu:
  - Relative path: `../mycloud-docusaurus` (folder ngang hàng)
  - Absolute path: `/home/user/sites/mycloud-docs`
  - In-place: `.` (khi bạn chạy docsmith trong chính Docusaurus repo)

**On collision**: khi target đã có file với content khác, deploy nên làm gì? Default `Warn` an toàn nhất.

### 5. Voice and tone

Doc nghe như thế nào. AI dùng trực tiếp khi viết.

| Field | Chọn gì |
|---|---|
| Tone | Casual = tán gẫu; Friendly-professional = mặc định cho đa số product; Technical-direct = không màu mè cho power user; Formal = corporate/regulated |
| Perspective | Second-person ("bạn làm X") = chuẩn cho doc hướng dẫn; First-person plural ("chúng tôi làm X") = collaborative tone (kiểu Stripe); Third-person ("user làm X") = formal/legal |
| Reading level | 6th = ai cũng đọc được; 8th = mặc định; 10th-college = developer/specialist |
| Custom terms to AVOID | Từ/cụm từ AI không nên dùng. Ví dụ: `tận dụng, leveraging, robust` |

Khi nghi ngờ, để defaults (Friendly-professional + Second-person + 8th grade).

### 6. Credentials cho product walkthrough

Walkthrough command login vào product để chụp screenshot. **Đừng bao giờ để password ở đây.** Để TÊN biến môi trường thay vì.

```markdown
- Username env var: `MYCLOUD_TEST_USER`
- Password env var: `MYCLOUD_TEST_PASS`
```

Trước khi chạy walkthrough:

```bash
export MYCLOUD_TEST_USER="qa@mycloud.test"
export MYCLOUD_TEST_PASS="password-thực-tế"
/docsmith run
```

Như vậy bạn có thể commit `project.md` lên git an toàn — nó không bao giờ chứa password thực.

Nếu product có MFA, thêm env var thứ 3. Nếu có note về test account (vd "test account, free tier, no production data"), thêm cho rõ.

### 7. Knowledge sources

Nơi AI fetch content khi draft doc.

Mỗi source, tick type và điền URL/path. Thêm bao nhiêu source tùy bạn có.

**Source types**:

| Type | Là gì | URL/ID format | Auth |
|---|---|---|---|
| Notion page | 1 Notion doc | URL đầy đủ: `https://notion.so/abc123` hoặc chỉ `abc123` | Set env var `NOTION_TOKEN` |
| GitHub repo | Code hoặc doc trên GitHub | `owner/repo` (vd: `mycloud/cloud`) | `GITHUB_TOKEN` cho private; không cần cho public |
| Google Drive doc | 1 Google Doc | File ID từ URL | Env var `GOOGLE_DRIVE_TOKEN` |
| Public URL | Bất kỳ web page nào | URL đầy đủ | Không cần |
| Local file | File trên máy bạn | Relative hoặc absolute path | Không cần |

Cho GitHub đặc biệt, có thể chỉ định subpath:

```markdown
- URL or path or ID: `mycloud/cloud`
- (Trong field Notes hoặc Paths): `api/instances/*.ts, docs/instances/*.md`
```

AI chỉ fetch file match, không phải cả repo.

**Auth env vars**: ý tương tự credentials — chỉ tên, không bao giờ token thực.

### 8. Auto-run behavior

AI pause ở đâu để bạn review trong pipeline.

- **After plan** = pause sớm để validate documentation plan trước khi draft
- **After draft** (mặc định) = pause để review draft trước khi walkthrough chụp screenshot
- **Before deploy** = để mọi thứ generate, chỉ review ở cuối
- **Never** = full auto, chỉ cho re-run trusted sau lần đầu thành công

Cho **lần chạy đầu**, "After draft" an toàn nhất — bạn xem AI viết gì trước khi đi đến product live.

Drift detection trong walkthrough:

- **Prompt per item** (mặc định) = an toàn nhất, chậm nhất
- **Auto-apply HIGH confidence** = nhanh hơn, cần caption discipline tốt

Translation:

- **Per-block** = review từng đoạn (chậm nhưng an toàn)
- **Batch** (mặc định) = review whole-file diff (nhanh hơn, mặc định)
- **Auto-approve** = skip review (chỉ cho glossary trusted)

### 9. Sitemap pattern

Quyết định cấu trúc navigation cho tất cả modules. Pick 1 lần ở project level.

- **Pattern A — Learning path** (mặc định): `Overview → Initial setup → Quick Starts → Tutorials → Reference → Troubleshooting → Glossary`. Tốt cho product kỹ thuật có conceptual depth.
- **Pattern B — Task-first**: `Overview → Quick Starts → Guides → Reference → Troubleshooting`. Tốt cho product trưởng thành mà user biết rõ mình muốn gì.
- **Pattern C — Custom**: bạn tự định thứ tự.

Tại sao quan trọng: đảm bảo tất cả modules trong project có **navigation consistent**. Không có cái này, module A có thể có `Quick Starts` trong khi module B có `Guides`, làm user rối.

Mỗi module sau đó tick section nào include (trong module intake § Sitemap sections). AI follow pattern project khi sắp xếp và warn khi module thiếu section pattern bao gồm.

**Display name overrides**: có thể rename section canonical per project (vd `Quick Starts` → `Hướng dẫn nhanh`). Slugs (folder names) giữ canonical.

### 10. Module intake files

Auto-managed. Đừng edit thủ công. Command `module` update section này khi bạn create/archive module.

### 11. Media policy (v1.5.5+)

Quyết định cách screenshot và video được tạo ra. 2 phần: screenshots và videos. Default work cho 80% project — chỉ override khi có nhu cầu cụ thể.

#### Screenshots

**Density**: bao nhiêu ảnh per doc, theo content type. Defaults:
- Tutorial: 1 / major step (~5-10 / doc)
- How-to: 1 / heading (~3-5 / doc)
- Reference: không có (chỉ text + tables)
- Concept: optional (chỉ khi concept abstract cần illustration)
- Troubleshooting: 1 / error case
- Quickstart: 1 / heading + 1 final state

**Style**: viewport-only (clean, không browser chrome) là default. Options khác cho case cụ thể.

**Aspect ratio**: 16:9 desktop (1280×720) là default. Mobile-portrait cho mobile-first product.

**Per-locale strategy** (khi project multi-language):
- **Source-only** (default): capture 1 lần ở source language UI, reuse cho tất cả translated docs. Rẻ nhất. Translated docs note "Screenshots in English UI".
- **Per-locale**: capture 1 lần per target locale. Cost gấp 3 cho 3 locales nhưng UX native.
- **Hybrid**: source full + selective per-locale cho important touchpoints (login, errors).

#### Videos

**Density**: khi nào cần video.
- Tutorial: required (1 / tutorial, ≤90s)
- How-to: optional (chỉ khi >5 steps, ≤30s)
- Reference: never
- Concept: optional (≤2min cho animation)

**Script files (v1.5.7+)**: mỗi video có script file riêng tại `documentation/scripts/<module>/<id>.md`. Voiceover (nếu có) đọc từ file này. Script multi-locale lưu dưới headings `## en`, `## vi`, `## jp` cùng file. AI generate script ban đầu từ draft của bạn khi chạy `record` lần đầu; bạn review và edit.

**Voiceover strategy** là quyết định lớn nhất:

- **Silent + on-screen captions** (default): không audio. Text overlay mô tả action. Rẻ nhất. Multi-locale chỉ cần text overlay khác. Recommend cho lần đầu.
- **AI synthetic voice per locale**: mỗi locale có voiceover TTS-generated riêng. UX tốt nhất. Tốn TTS API calls. Cần TTS provider configured.
- **Source voice + per-locale subtitles**: 1 audio source, .vtt subtitle per locale. Rẻ, UX khá.
- **Human recorded voiceover**: thu âm thủ công ngoài docsmith. Quality cao nhất. Premium content.
- **Không video gì cả**: skip command `record` hoàn toàn.

**TTS provider** (chỉ khi AI synthetic voice):
- **local-piper** (default): free, offline, quality medium
- **local-coqui**: free, nhiều voice hơn, chậm hơn
- **openai**: paid (~$15/1M chars), wide languages
- **elevenlabs**: paid ($5-99/mo), quality cao nhất
- **google-cloud**: paid (~$4/1M chars), Vietnamese tốt
- **azure-cognitive**: paid, nhiều neural voices

Cho mỗi locale dùng, chỉ định voice ID/name từ catalog của provider.

#### Subtitles

Khi nào generate `.vtt`:
- **Source-only** hoặc **per-locale** subtitles (per-locale là default cho multi-locale)
- **Auto từ script**: work cho Silent và AI voice (deterministic)
- **STT sau recording**: chỉ cho human voiceover (cần Whisper)
- **Manual**: user cung cấp file `.vtt`

Sidecar `.vtt` (default): 1 video file, nhiều subtitle file. Burn-in: video riêng per locale.

#### Cost reality check

Default config (silent + source-only screenshots) cho 3 locales × 5 modules × 30 docs:
- Walkthrough: ~30 min runtime
- Recording: instant (silent)
- Storage: ~65 MB tổng
- TTS cost: $0
- **Thời gian: ~1 giờ**

Premium config (AI voice per locale + per-locale screenshots):
- Walkthrough: ~90 min (3× run cho 3 locales)
- Recording: TTS API calls (~$5-10 lần đầu)
- Storage: ~200 MB
- **Thời gian: ~3 giờ lần đầu, ~30 min update**

Bắt đầu với default. Upgrade strategy sau khi đã ship 1 cycle và biết cái gì matter.

## Mỗi section có ý nghĩa gì (modules/<n>.md)

File module đơn giản hơn — chỉ chứa thứ KHÁC project default.

### 1. Module identity

| Field | Là gì | Ví dụ |
|---|---|---|
| Module slug | Identifier ngắn; match filename | `instances` (file: `instances.md`) |
| Module display name | Tên dễ đọc | `Instances` |
| Priority | 1 (cao nhất) đến 5 (thấp nhất). Dùng order trong sitemap | `1` |
| Folder in target docs | Module này nằm ở đâu trong site đã publish | `instances` (deploy đến `docs/instances/`) |

Folder mặc định = slug. Override chỉ khi muốn URL khác slug.

### 2. Scope

Module này document feature gì.

Cho mỗi feature:

- **Feature name**: gì user sẽ tra cứu. `Tạo instance`, `Auto-scaling`, `IP allocation`
- **Content types**: tick type cần. Tick nhiều = generate nhiều doc. Xem content types bên dưới.

**Content types** (tick 1 hoặc nhiều mỗi feature):

| Type | Khi nào dùng | Output style |
|---|---|---|
| Tutorial | User lần đầu học bằng làm | Hand-holding, mỗi step giải thích, beginner-friendly |
| How-to | User có goal, không học | Task-focused, giả định context |
| Reference | Tra cứu parameters, options, settings | Tables, lists, structured |
| Concept | Hiểu cách thức hoạt động | Prose explanation, có thể có diagram |

**Ví dụ**: feature `Tạo instance` với `tutorial` + `how-to` được tick → AI generate 2 doc:
- `documentation/drafts/en/instances/create-instance-tutorial.md` (tutorial)
- `documentation/drafts/en/instances/create-instance.md` (how-to)

**Out of scope**: liệt kê rõ thứ module này KHÔNG cover. Giúp AI tránh hallucinating tangential content.

### 3. Voice override

Thường để mặc định "Inherit from project". Chỉ override khi module này cần voice khác (vd module admin muốn formal trong khi module user là casual).

### 4. Module-specific knowledge sources

Cùng format với project sources, nhưng chỉ cho module này. Nó THÊM vào project sources — cả hai list đều dùng.

Dùng cho PRD module, design doc, hoặc code area không liên quan module khác.

### 5. Walkthrough setup

Cho module cần test account khác project default, override ở đây. Đa số module inherit.

**Pre-walkthrough setup script**: bash command chạy trước browser automation. Dùng phổ biến:
- Tạo test data: `mycloud volume create --size 10G --name test-vol`
- Seed user account: `mycloud user create --email test@example.com`

**Post-walkthrough teardown**: cleanup sau capture. Không bắt buộc (lần chạy sau có thể re-create), nhưng gọn hơn.

### 6. Special handling

- **Sensitive fields to redact**: tick giúp walkthrough biết blur gì trên screenshot. Thêm visual placeholder trên các vùng đó.
- **Module status**:
  - `Active` (mặc định) — được `/docsmith run` xử lý
  - `Paused` — skip tạm thời; giữ cho sau
  - `Archived` — skip vĩnh viễn; không bị `--sync-deletes` xóa từ target

### 7. Sitemap sections (v1.5.4+)

Tick section type canonical mà module này include. Thứ tự xuất hiện do project pattern quyết định (xem project.md § 9).

| Section type | Khi nào tick |
|---|---|
| `overview` | Luôn (auto-required) |
| `initial-setup` | Module cần setup riêng beyond project-level |
| `quickstarts` | Có content task-focused ngắn (5-10 phút mỗi cái) |
| `tutorials` | Có content step-by-step learning với hand-holding |
| `guides` | Có how-to giả định context (alternative cho quickstarts) |
| `concepts` | Có khái niệm non-obvious cần giải thích |
| `dashboard` | Module có dashboard hoặc report view |
| `reference` | Có parameter tables, schema specs |
| `api-reference` | Module có stable API |
| `glossary` | Term riêng của module |
| `troubleshooting` | Common issues của module này |

**Pick `quickstarts` HOẶC `guides`, đừng cả hai.** Chúng overlap.

**AI suggestions**: khi chạy `/docsmith plan`, AI check mỗi module với project pattern và warn nếu module thiếu section pattern bao gồm. Quyết định per warning — đôi khi 1 section thực sự không áp dụng.

**Display name overrides**: tên tùy chỉnh per module (vd module này hiển thị "Quick Starts" thành "Bắt đầu nhanh" trong nav). Override module level > project level > default.

### 8. Media override (v1.5.5+)

Đa số module inherit media policy từ project intake § 11. Chỉ override khi module này khác biệt cơ bản. Common cases:

- **Module compliance** cần human voiceover thay vì AI voice của project
- **Module admin** có UI khác nhau theo locale (force per-locale screenshots)
- **Module mobile** trong project chủ yếu desktop (aspect ratio khác)
- **Module reference-only** không cần screenshot nào

Mỗi override option có tick "Inherit from project" (default). Chỉ tick override khi module thực sự khác.

## Common patterns

### "Tôi chỉ muốn viết doc cho 1 feature"

1. `/docsmith init` (Standalone preset, không deploy)
2. `/docsmith module myfeature`
3. Điền `documentation/intake/project.md` Product + Audience + Voice (cơ bản thôi)
4. Điền `documentation/intake/modules/myfeature.md` Scope (1 feature, How-to content type)
5. `/docsmith run myfeature`

### "Tôi có site Docusaurus và muốn thêm 1 module"

1. `cd ~/my-docusaurus-site`
2. `/docsmith init --in-place` (auto-detect, suggest in-place)
3. `/docsmith module pricing`
4. Điền intake
5. `/docsmith run pricing`
6. `/docsmith deploy --dry-run` rồi `/docsmith deploy`

### "Tôi muốn doc đa ngôn ngữ (EN + VI + JP)"

1. Trong `project.md` § Languages: source `en`, targets `vi` + `jp`
2. Tùy chọn edit `documentation/standards/glossary.vi.yaml` và `glossary.jp.yaml` (auto-create)
3. `/docsmith run` chạy qua stage translate tự động
4. Translated drafts xuất hiện trong `documentation/drafts/vi/` và `drafts/jp/`

### "Source content nằm trong Notion"

1. Lấy Notion integration token: notion.so/my-integrations → tạo internal integration → copy token
2. Share Notion page với integration đó
3. `export NOTION_TOKEN="secret_..."`
4. Trong intake `Knowledge sources`:
   - Type: tick Notion page
   - URL: `https://notion.so/abc123` (URL đầy đủ hoặc chỉ ID)
   - Auth env var: `NOTION_TOKEN`

### "Tôi muốn update doc sau khi product UI đổi"

1. `/docsmith walkthrough --check` — produce drift report
2. Đọc `documentation/walkthrough/drift/<latest>/drift-report.md`
3. Edit `decisions.yaml` cạnh report (auto-fix / skip / product-bug)
4. `/docsmith walkthrough --apply` để apply decisions và re-capture

### "PRD source trong Notion vừa update"

1. `/docsmith update`
2. AI check tất cả sources, list cái nào đã đổi
3. Confirm để re-fetch và re-evaluate doc bị ảnh hưởng
4. Review proposed deltas (KB inheritance — chỉ section đổi mới được update)

## AI thực sự làm gì với mỗi field?

| Field trong intake | AI dùng ở đâu |
|---|---|
| product.slug | Image namespace `/img/<slug>/`; sources.lock entries |
| product.name | Title trong frontmatter; intro lines |
| audience.tech_level | Vocabulary choice, kiến thức nền giả định |
| audience.primary_goal | Tutorial intro, "bạn sẽ học cách..." statement |
| locales.source | Drafts đi đâu: `drafts/<source>/...` |
| locales.targets | Translate chạy ngôn ngữ gì |
| voice.tone | Sentence style, contractions, formality |
| voice.perspective | "bạn" vs "chúng tôi" vs "user" xuyên suốt |
| voice.reading_level | Word choice, sentence length |
| voice.terms_to_avoid | Negative constraint khi draft |
| deploy.preset | Preset deploy logic nào activate |
| deploy.target_path | File đi đâu khi `deploy` |
| sources | Fetch lúc run; content available cho AI làm KB lúc draft |
| credentials.*_env | AI đọc env var khi launch browser cho walkthrough |
| auto_run.pause_at | Khi nào pipeline interrupt cho review |
| translate.review_mode | per-block vs batch vs auto-approve |
| module.folder | Folder name trong target (vd: `docs/instances/`) |
| features[].content_types | Drive bao nhiêu doc generate per feature |
| out_of_scope | Negative constraint — đừng generate topic này |
| status (module) | `run` xử lý module này không |

## Nếu tôi mắc lỗi thì sao?

- **Empty critical field** → `/docsmith run` dừng với list rõ chỗ cần fix
- **Tick checkbox sai** → chạy lại với file đã sửa; re-run protocol kick in để merge sạch
- **Source URL sai** → `/docsmith fetch` báo lỗi với URL fail; sửa và retry
- **Env var chưa set** → Command dừng, in env var nào cần set
- **Format error** (xóa backticks vô tình) → `/docsmith intake-help` flag issue

Bạn luôn có thể chạy `/docsmith intake-help` xem field reference, hoặc `/docsmith intake-help <section>` xem 1 section.

## Sau khi run đầu tiên thành công

Bạn sẽ có:
- Audience profile generate từ input
- Documentation plan với sitemap
- Voice chart trong project standards
- Drafts trong `documentation/drafts/<locale>/<module>/`
- Screenshots capture vào `documentation/images/<module>/`
- Translated drafts (nếu set targets)
- Audit trail trong `documentation/deployments/<ts>/`

Edit draft trực tiếp nếu cần (re-run protocol giữ edit qua run kế tiếp). Sau đó:

```bash
/docsmith deploy --dry-run    # preview gì sẽ copy đến host
/docsmith deploy              # apply
cd ../my-docusaurus-site && git diff && git add . && git commit && git push
```

Xong. Site đã live.
