# Hướng dẫn điền intake

Hướng dẫn thực dụng để điền form intake của docsmith. Form có sẵn hint inline cho mỗi field — guide này dành cho context, common patterns, và quyết định bạn cần làm.

> 🇬🇧 English version: [INTAKE_GUIDE.md](INTAKE_GUIDE.md)

## Bạn cần điền gì

Hai file markdown trong `documentation/intake/`:

```
documentation/intake/
├── project.md                  # Điền 1 lần cho project
└── modules/
    ├── instances.md            # 1 file mỗi feature area
    └── storage.md
```

`project.md` được tạo khi chạy `/docsmith init`. File module được tạo khi chạy `/docsmith module <tên>`.

Mỗi field trong file có hint `>` ngay bên dưới giải thích phải điền gì. **Không cần mở guide này khi điền — template tự giải thích.**

Guide này dành cho: quyết định pattern, hiểu tradeoffs, troubleshoot.

## Ba quy tắc edit

### 1. Điền backtick

```markdown
- Product slug: `your-product-slug`
```

thành

```markdown
- Product slug: `mycloud`
```

Đừng xóa backticks. Thay text bên trong.

### 2. Tick checkbox

```markdown
- [ ] Standalone
- [ ] Docusaurus
```

thành

```markdown
- [ ] Standalone
- [x] Docusaurus
```

Đa số group: tick đúng 1. Vài group (như target languages): tick nhiều.

### 3. Skip section Advanced

Section bọc trong `<details>` mặc định collapse. Đa số project không cần expand cái nào. Mở chỉ khi có lý do cụ thể (regulated industry cần formal voice, compliance module cần human voiceover, vv.).

## Checklist tối thiểu cho project đầu tiên

Mở `project.md` và CHỈ điền:
- § 1 Product (slug, name, URL, description)
- § 2 Audience (chỉ primary persona)
- § 3 Languages (source language; target languages nếu có)
- § 4 Deploy (preset; target path nếu Docusaurus)
- § 5 Credentials (env var names — chỉ nếu dùng walkthrough)
- § 6 Knowledge sources > Source 1 (skip nếu AI tự draft từ intake info)

Mỗi `modules/<name>.md` CHỈ điền:
- § 1 Module identity (slug, display name, folder)
- § 2 Scope (ít nhất 1 feature với ít nhất 1 content type tick)

Hết. Chạy `/docsmith run`. Nếu output không như ý, mới expand Advanced section liên quan và tweak.

## Common patterns

### "Tôi muốn doc nhanh cho 1 feature"

1. `/docsmith init` (Standalone preset, không deploy)
2. `/docsmith module myfeature`
3. Điền `project.md` § 1, 2, 3 (en source, không target)
4. Điền `modules/myfeature.md` § 1, 2 (1 feature, How-to tick)
5. `/docsmith run myfeature`

5 phút từ zero đến drafts.

### "Tôi có site Docusaurus và muốn thêm doc"

1. `cd ~/my-docusaurus-site`
2. `/docsmith init --in-place` (auto-detect, suggest in-place)
3. `/docsmith module pricing`
4. Điền intake (in-place mode = `target_path: .`)
5. `/docsmith run pricing`
6. `/docsmith deploy --dry-run` rồi `/docsmith deploy`
7. `git add . && git commit -m "Add pricing docs" && git push`

### "Tôi cần doc đa ngôn ngữ (EN + VI + JP)"

1. Trong `project.md` § 3: source `en`, targets tick `vi` + `jp`
2. (Tùy chọn) Edit `documentation/standards/glossary.vi.yaml` và `glossary.jp.yaml` (auto-create) thêm domain terms
3. `/docsmith run` — pipeline chạy qua stage translate tự động
4. Translated drafts xuất hiện trong `documentation/drafts/vi/` và `drafts/jp/`

Defaults: batch review (whole-file diff), không cần glossary, source-only screenshots.

### "Source content nằm trong Notion"

1. Lấy Notion integration token: notion.so/my-integrations → tạo internal integration → copy token bắt đầu với `secret_`
2. Share mỗi Notion page docsmith cần đọc với integration đó
3. `export NOTION_TOKEN="secret_..."` (hoặc thêm vào `.env`)
4. Trong `project.md` § 6 Source 1:
   - Type: tick Notion page
   - URL: `https://notion.so/abc123` (URL đầy đủ hoặc chỉ ID)
   - Auth env var: `NOTION_TOKEN`
5. `/docsmith run`

### "Tôi đã có BA doc / PRD; không muốn gõ lại" (v1.5.9+)

1. Lưu BA doc / PRD dưới dạng file local HOẶC có URL (Notion, GitHub, Google Drive)
2. Chạy:
   ```bash
   /docsmith init --from-source path/to/ba-doc.md
   # HOẶC
   /docsmith init --from-source https://notion.so/abc123
   ```
3. AI đọc source, infer đa số field. Hỏi 5-10 câu cho field source không cover (target locales, deploy preset, credentials, voiceover).
4. AI viết `project.md` với markers:
   - Giá trị bình thường = direct fact từ source
   - `← AI guess, please verify` = infer, có thể sai
   - `← default applied` = source không có info, default conservative
5. Mở `project.md`, scan tìm marker `← AI guess`, verify hoặc sửa
6. (Tùy chọn) Chạy `module --from-source` cho mỗi module được detect
7. Xong. `/docsmith run` như bình thường.

Chi tiết: xem inference report tại `documentation/intake/.inference/<timestamp>-project.md` sau khi auto-fill.

### "BA doc đã update; muốn refresh intake" (v1.5.9+)

```bash
/docsmith init --from-source path/to/ba-doc.md
```

(Cùng command như fill lần đầu.) AI:
- Re-fetch source
- So sánh với inference lần trước (per-field hash)
- Chỉ re-infer field BA không edit manually
- Show diff trước khi apply
- Manual edit được giữ nguyên

### "Source có module mới chưa documented" (v1.5.10+)

```bash
/docsmith update
```

AI so sánh source structure với workspace modules. Output:

```
Source có 7 modules. Workspace có 4.

📋 Mới trong source (chưa documented):
  [ ] billing — BA doc § 5
  [ ] audit-logs — BA doc § 6
  [ ] api-keys — BA doc § 7

⚠️ Trong workspace, không có trong source:
  - legacy-vpn — lần cuối trong source 2025-10

Action: 1=tạo mới, 2=archive orphan, 3=cả hai, 4=show diff, 5=skip
```

Bạn chọn action, AI apply. Module mới được auto-fill từ source (giống `module --from-source`). Module orphan được mark `status: archived` — giữ lại cho git history, skip bởi `run`.

Cho project có nhiều modules (>5 mới), AI MAY dùng parallel sub-agents để gen nhanh hơn. Sequential fallback nếu parallel không khả dụng. Xem SKILL.md § `update` chi tiết.

### "Update doc sau khi product UI đổi"

1. `/docsmith walkthrough --check` — produce drift report
2. Đọc `documentation/walkthrough/drift/<latest>/drift-report.md`
3. Edit `decisions.yaml` cạnh đó (auto-fix / skip / mark product bug)
4. `/docsmith walkthrough --apply` để apply decisions và re-capture

### "PRD trong Notion vừa update"

1. `/docsmith update`
2. AI check tất cả sources qua cheap metadata calls, list cái nào đã đổi
3. Confirm để re-fetch và re-evaluate doc bị ảnh hưởng
4. Review proposed deltas (Update mode: chỉ section đổi mới propose)

### "Tôi cần video tutorial với voiceover tiếng Việt"

1. Trong `project.md` § 6 expand "Advanced — media policy"
2. Voiceover strategy: tick "AI synthetic voice per locale"
3. TTS provider: tick "google-cloud" (Vietnamese tốt nhất) hoặc "local-piper" (free)
4. Voice ID per locale: điền từ catalog provider (vd `vi-VN-Wavenet-A` cho Google Cloud)
5. Auth env var: set `GOOGLE_APPLICATION_CREDENTIALS` đến path JSON service account
6. Thêm marker `<!-- VIDEO id: my-tour -->` trong drafts nơi muốn có video
7. `/docsmith record` — AI gen script per locale, bạn review, sau đó TTS produce audio per locale, video encode

Xem [SETUP.md § 6](SETUP.md) để cài TTS provider.

## Decision matrix

### Voiceover strategy (project intake § 6 advanced — media policy)

| Bạn muốn | Pick |
|---|---|
| Chỉ cần xong doc; rẻ; không phức tạp audio | Silent + on-screen captions (default) |
| Multi-locale, native UX, sẵn sàng budget $5-10/cycle | AI synthetic voice per locale |
| Tight budget nhưng multi-locale, OK với subtitle | Source voice + per-locale subtitles |
| Premium quality, content compliance, marketing video | Human recorded voiceover |

### Per-locale screenshots (project intake § 6 advanced)

| Bạn muốn | Pick |
|---|---|
| Rẻ nhất; capture 1 lần ở source UI; note "EN UI" trong translated docs | Source-only (default) |
| Native experience; UI text khác per locale; sẵn sàng tăng 3× thời gian capture | Per-locale |
| Best of both; capture key flows per locale, reuse cho generic | Hybrid |

### TTS provider (chỉ khi voiceover = AI synthetic voice)

| Bạn muốn | Pick |
|---|---|
| Free, offline, không API key, quality khá | local-piper (default) |
| Free, offline, nhiều voice option | local-coqui |
| Quality cao, wide languages, simple API | openai (~$15/1M chars) |
| Quality cao nhất, marketing-grade | elevenlabs ($5-99/mo) |
| Vietnamese / Asian languages tốt nhất | google-cloud (~$4/1M chars) |
| Nhiều neural voice, Azure đã trong stack | azure-cognitive |

## AI thực sự dùng mỗi field như thế nào

| Field | AI dùng ở đâu |
|---|---|
| product.slug | Image namespace `/img/<slug>/`; sources.lock entries; deploy paths |
| product.name | Title trong frontmatter; landing intro lines |
| audience.tech_level | Vocabulary choice, kiến thức nền giả định |
| audience.primary_goal | Tutorial intro ("bạn sẽ học cách..."), how-to motivation lines |
| locales.source | Drafts đi đâu: `drafts/<source>/...` |
| locales.targets | Translate command produce ngôn ngữ gì |
| voice.tone | Sentence style, contractions, formality |
| voice.perspective | "bạn" vs "chúng tôi" vs "user" xuyên suốt |
| voice.reading_level | Word choice, sentence length |
| voice.terms_to_avoid | Negative constraint khi draft |
| deploy.preset | Preset deploy logic nào active |
| deploy.target_path | File đi đâu khi `deploy` |
| sources | Fetch lúc run; AI dùng làm KB lúc draft |
| credentials.*_env | AI đọc env var khi launch browser cho walkthrough |
| auto_run.pause_at | Khi nào pipeline interrupt cho review |
| translate.review_mode | per-block vs batch vs auto-approve |
| sitemap.pattern | Thứ tự section across modules (consistency) |
| media.screenshot_density | Bao nhiêu ảnh per doc theo content type |
| media.voiceover_strategy | Video có voice không; multi-locale handle thế nào |
| media.tts_provider | TTS service nào generate voice audio |
| module.folder | Folder name trong target deploy (vd `docs/instances/`) |
| features[].content_types | Drive bao nhiêu doc generate per feature |
| out_of_scope | Negative constraint — AI sẽ không generate topic này |
| status (module) | `run` xử lý module này hay skip |

## Nếu mắc lỗi thì sao

| Lỗi | Xảy ra gì | Fix |
|---|---|---|
| Critical field rỗng | `/docsmith run` dừng với list rõ | Điền field listed, retry |
| Tick checkbox sai | Output không như expectation | Edit, retry; re-run protocol merge sạch |
| Source URL sai | `/docsmith fetch` báo lỗi với URL | Sửa URL, retry |
| Env var chưa set | Command dừng, in env var nào cần | `export VAR=value`, retry |
| Xóa backtick vô tình | `/docsmith intake-help` flag issue | Restore backtick |
| Thiếu TTS provider env var | `record` warn trước khi generate | Set env var hoặc switch sang local-piper |

Bạn luôn có thể chạy `/docsmith intake-help` xem field reference đầy đủ, hoặc `/docsmith intake-help <section>` xem 1 section.

## Sau khi run đầu tiên thành công

```
documentation/
├── plan/audience-profile.md          # AI-generated từ intake
├── plan/documentation-plan.md
├── plan/sitemap.md
├── standards/voice-chart.md
├── drafts/<locale>/<module>/         # Docs chính
├── images/<module>/                  # Screenshots từ walkthrough
├── scripts/<module>/                 # Video scripts (nếu dùng record)
├── videos/                           # Final videos
├── walkthrough/                      # Drift reports, test cases
└── deployments/<ts>/                 # Audit trail per deploy
```

Edit draft trực tiếp nếu cần (re-run protocol giữ edit qua run kế tiếp). Sau đó deploy và commit.

## Khi nào expand Advanced section

Đa số project không bao giờ cần. Nhưng đây là signals:

| Signal | Expand |
|---|---|
| Run đầu: drafts cảm giác formal/casual quá | § 6 Advanced — voice and tone |
| Multi-locale: terminology không nhất quán trong translation | Edit `documentation/standards/glossary.<locale>.yaml` |
| Multi-locale: UI text xuất hiện trong screenshots | § 6 Advanced — media policy → per-locale screenshots |
| Source content quá mơ hồ, AI hallucinate | Thêm Source N blocks (Advanced) |
| Review từng block quá chậm | § 6 Advanced — auto-run → translate review = batch |
| Module cần voice khác rest | module.md Advanced — voice override |
| Compliance module cần human voiceover | module.md Advanced — media override |
| Pipeline fail giữa chừng, muốn start lại | `/docsmith init --force` (sau khi backup) |

## Tips cho thành công lần đầu

1. **Bắt đầu nhỏ**. 1 module, 1 feature, content type How-to thôi. Run, review, sau đó expand scope.
2. **Đừng expand Advanced section ở lần đầu**. Default đã tune cho "good enough" first output.
3. **Set pause gate "After draft"** (default). Bạn xem AI viết gì trước khi walkthrough chụp.
4. **Test với real test account, không phải account cá nhân**. Walkthrough automation nhanh và có thể làm unexpected things; isolate.
5. **Chạy `verify` trước khi deploy**. Catch placeholders, broken links, voice inconsistency.
6. **Commit `documentation/` lên git**. Audit trail (deployments/, archive/, drift/) nhỏ và đáng giữ.
7. **Đừng commit `.cache/` hay `videos/raw/`**. Init thêm vào .gitignore tự động.
