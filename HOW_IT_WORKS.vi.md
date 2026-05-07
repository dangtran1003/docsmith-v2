# docsmith hoạt động thế nào

Mô hình vận hành đầy đủ của docsmith — mỗi command, mỗi file, lifecycle từ intake đến deploy. Đọc khi bạn cần hiểu TẠI SAO skill được tổ chức như vậy, không chỉ cách chạy command.

> 🇬🇧 English version: [HOW_IT_WORKS.md](HOW_IT_WORKS.md)

> Cập nhật cho v1.5.14. Để chạy command thông thường, xem [README.md](README.md). Để điền intake forms, xem [INTAKE_GUIDE.vi.md](INTAKE_GUIDE.vi.md).

## TL;DR

Bạn điền 2 form markdown. AI gen docs qua pipeline 7 stage (audience → plan → voice → draft → edit → walkthrough → record → translate). Tùy chọn: deploy lên Docusaurus. Tất cả reproducible: re-run bất kỳ stage nào → detect output đã có và update incrementally (giữ lại edit của bạn) hoặc hỏi trước khi overwrite.

Skill là **prompt-based, không phải code-based**. Bug fix là sửa prose trong SKILL.md, không phải code patches.

---

## 1. Mô hình config 2-layer

docsmith đọc config từ 2 file markdown intake (từ v1.5.0):

```
documentation/intake/
├── project.md                  # Layer 1 — áp dụng cho TẤT CẢ modules
└── modules/
    ├── instances.md            # Layer 2 — override per module
    ├── storage.md
    └── billing.md
```

**Project intake** (`project.md`) bao gồm:
- Product identity (slug, name, URL, description)
- Audience (personas, technical level)
- Languages (source + targets)
- Deploy preset (standalone hoặc Docusaurus + path)
- Voice và tone (defaults work cho ~80% case)
- Knowledge sources (Notion / GitHub / GDrive / URL / file)
- Credentials env vars (cho walkthrough)
- Auto-run behavior (pause review ở đâu)
- Sitemap pattern (Pattern A / B / C — thêm v1.5.4)
- Media policy (screenshot density, voiceover strategy, TTS provider — thêm v1.5.5)

**Module intake** (`modules/<name>.md`) bao gồm:
- Module identity (slug, display name, folder)
- Scope (features, content types per feature)
- Per-module overrides (voice, sources, credentials, sitemap sections, media)

**Resolution order** khi AI chạy:
1. Module intake fields (nếu set)
2. Fall back về project intake
3. Fall back về template default

Sources được CỘNG DỒN: project sources + module sources cùng dùng.

### Tại sao 2 layer (không phải 1 file lớn hoặc 1 file/module)?

- 1 file lớn → config lặp lại per module, drift theo thời gian
- 1 file/module → không có chỗ cho settings project-wide (deploy target, locales, audience)
- 2 layer → DRY ở project level, override ở module level khi cần

### `.docsmithrc.yaml` đã deprecated

v1.4.x dùng YAML config. v1.5.0+ dùng markdown intake forms thay thế. YAML vẫn parse được cho backward compat (với `init --upgrade-from-1.4`) nhưng sẽ bị remove ở v1.6+.

---

## 2. 20 commands và vị trí trong pipeline

```
init ─→ module ─→ [fill intake] ─→ run ──┬─→ continue ─→ deploy
                                          │
                                          ▼
                                    intake-help
                                    fetch
                                    (any individual stage)
```

Sequential pipeline (auto-chained bởi `run`):

| # | Command | Làm gì |
|---|---|---|
| 1 | `audience` | Generate audience profile từ intake |
| 2 | `plan` | Generate documentation plan + sitemap |
| 3 | `voice` | Generate voice chart |
| 4 | `draft` | Draft docs trong `documentation/drafts/<source>/` |
| 5 | `edit` | 5-pass self-review của drafts |
| 6 | `walkthrough` (alias `wt`) | Verify với product live, capture screenshots |
| 7 | `record` (alias `rec`) | Quay video tutorial ngắn (silent hoặc voiced) |
| 8 | `translate` (alias `tr`) | Translate drafts + scripts sang target locales |

Setup / lifecycle commands:

| Command | Mục đích |
|---|---|
| `init` | Scaffold workspace + project intake |
| `module` | Tạo / list / archive module intake |
| `intake-help` | In intake field reference |
| `fetch` | Pull external sources về local cache |
| `run` | Pipeline orchestrated; pause ở gate đã config |
| `continue` | Resume `run` sau pause |
| `update` | Detect source/module changes; propose updates |
| `verify` (alias `vf`) | 11-check audit |
| `categorize` (alias `cat`) | Generate `_category_.json` cho Docusaurus |
| `deploy` (alias `dep`) | Sync workspace sang target |
| `publish` (alias `pub`) | Final checklist (human step — git commit + push) |

Đa số user chỉ chạy: `init`, `module`, `run`, `deploy`. Còn lại cho fine-grained control hoặc recovery.

---

## 3. `/docsmith init` làm gì

`init` chạy 1 lần per project. Nó:

1. **Detect host context**:
   - Có `docusaurus.config.{js,ts,mjs}` → suggest `docusaurus` preset (kèm in-place option)
   - Có `package.json` → confirm intent trước khi scaffold
   - Empty directory → fine
   - Non-empty unrelated content → STOP, hỏi user confirm hoặc move

2. **Tạo directory structure**:
   ```
   documentation/
   ├── intake/
   │   ├── project.md              # form bạn điền
   │   ├── modules/                # rỗng ban đầu
   │   └── sources.lock.yaml       # auto-managed, rỗng ban đầu
   ├── plan/                       # populated bởi `plan`
   ├── standards/                  # populated bởi `voice` và `translate`
   ├── drafts/                     # populated bởi `draft`
   ├── walkthrough/                # populated bởi `wt`
   ├── images/                     # populated bởi `wt`
   ├── scripts/                    # v1.5.7+: populated bởi `record`
   ├── videos/                     # populated bởi `rec`
   ├── deployments/                # populated bởi `deploy`
   ├── archive/                    # backups từ re-run protocol
   ├── .cache/sources/             # gitignored, fetched source content
   └── .run-state/                 # gitignored, per-module run state
   ```

3. **Append vào `.gitignore`** (ở project root, idempotent block giữa markers `# BEGIN docsmith` và `# END docsmith`):
   ```
   documentation/.cache/
   documentation/videos/raw/
   documentation/.run-state/
   ```

4. **Pre-fill `project.md`** với detected values (slug từ package.json, deploy target từ inspected Docusaurus path, vv.)

5. **In hint bước tiếp**: "Edit documentation/intake/project.md, sau đó run `/docsmith module <n>` cho mỗi feature area."

### Variants

- `init --in-place` — explicit request in-place mode (khi chạy bên trong Docusaurus repo)
- `init --upgrade-from-1.4` — đọc `.docsmithrc.yaml` và pre-fill `project.md` (deprecated path)
- `init --reformat-intake` — re-render existing intake với current template (giữ filled values)
- `init --from-source <path-or-url>` (v1.5.9+) — AI auto-fills intake từ BA doc / PRD / docs hiện có
- `init --resume` (v1.5.10+) — retry parallel sub-agents từ partial-completion run trước

---

## 4. Workflow điền intake form

### Manual fill (truyền thống)

```bash
/docsmith init                   # scaffolds project.md
/docsmith module instances       # scaffolds modules/instances.md
# (mở project.md, điền checkboxes và backtick fields)
# (mở modules/instances.md, điền scope)
/docsmith run                    # AI dùng intake drive pipeline
```

Mỗi field có hint inline `>` giải thích phải điền gì. Cho project đầu, CHỈ điền:
- project.md sections 1-6 (Product, Audience, Languages, Deploy, Credentials, Source 1)
- modules/<n>.md sections 1-2 (Identity, Scope)

Skip Advanced sections (mặc định collapsed từ v1.5.6) — defaults work.

### AI auto-fill từ source (v1.5.9+)

```bash
/docsmith init --from-source documentation/sources/ba-doc.md
# AI đọc BA doc, infer fields, hỏi 5-10 câu, viết project.md
# Mark uncertain fields với "← AI guess" để bạn verify

/docsmith module instances --from-source documentation/sources/ba-doc.md
# AI extract features cho module "instances" từ BA doc
```

Source có thể:
- File local (markdown, plain text)
- Notion URL hoặc page ID
- GitHub `owner/repo` hoặc đường dẫn file cụ thể
- Google Drive file ID
- Public URL

Nhiều sources cách nhau bằng dấu phẩy.

AI categorize inferences:
- **Fact** — direct quote từ source (no marker)
- **Guess** — inferred từ context (`← AI guess, please verify`)
- **Default** — không có source data (`← default applied`)
- **Asked** — trả lời trong interactive Q&A

Inference report tại `documentation/intake/.inference/<ts>-<scope>.md` document mỗi inference với source quote và reasoning.

### Re-run với `--from-source`

Khi BA doc update:

```bash
/docsmith init --from-source path/to/ba-doc.md
```

AI:
1. Re-fetch source
2. Per-field hash check: BA-edited fields preserved; AI-inferred fields có thể re-infer
3. Show diff trước khi apply
4. Re-run protocol gate (Update / Overwrite / Skip)

---

## 5. Pipeline (`/docsmith run`)

`run` orchestrate cả 8 stages. Default pause gate: **after draft** (review drafts trước khi walkthrough chụp screenshots từ product live).

```
[1] audience
       │
       ▼
[2] plan ─→ documentation/plan/{audience-profile.md, documentation-plan.md, sitemap.md}
       │
       ▼
[3] voice ─→ documentation/standards/voice-chart.md
       │
       ▼
[4] draft ─→ documentation/drafts/<source-locale>/<module>/<doc>.md
       │
       ▼
[5] edit (5-pass self-review)
       │
       ▼
   ┌─── PAUSE GATE: review drafts ───┐
   │                                  │
   │  /docsmith continue             │
   │                                  │
   ▼                                  ▼
[6] walkthrough ─→ images/<module>/   ─→ drift reports
       │
       ▼
[7] record ─→ scripts/<module>/<id>.md (v1.5.7+)
              voiceover/<id>.<locale>.mp3
              videos/<id>.mp4
              subtitles/<id>.<locale>.vtt
       │
       ▼
[8] translate ─→ drafts/<target-locale>/<module>/<doc>.md
                 scripts/<module>/<id>.md (## <locale> sections)
       │
       ▼
   verify (optional, recommend trước deploy)
       │
       ▼
   deploy
```

### Pause gates (set trong project intake § 6 advanced)

- **After plan** (trước draft) — review documentation plan trước
- **After draft** (trước walkthrough) — DEFAULT, an toàn nhất cho run đầu
- **Before deploy** — để mọi thứ generate, chỉ review deploy plan
- **Never** — full auto, chỉ cho re-runs trusted

### Mỗi stage làm gì

#### Audience
Generate `documentation/plan/audience-profile.md` từ intake — narrative về personas, technical level, primary goals, motivations.

#### Plan
Generate:
- `documentation/plan/documentation-plan.md` — module breakdown, doc list per module
- `documentation/plan/sitemap.md` — full nav structure follow project pattern (A/B/C)

`plan` check mỗi module với project sitemap pattern; warn nếu module thiếu section pattern bao gồm.

#### Voice
Generate `documentation/standards/voice-chart.md` — concrete examples về tone/perspective đã chọn nghe thế nào trong domain này. Dùng làm reference khi draft.

#### Draft
Draft mỗi doc trong `documentation/drafts/<source-locale>/<module>/`. Dùng:
- Intake fields (audience, scope, voice)
- Sources (Notion / GitHub / etc fetched qua `fetch`)
- Voice chart cho tone consistency

Trong Update mode (existing draft đã có), đọc current draft như KB và propose deltas only — manual edits của bạn được giữ.

#### Edit
5-pass self-review của mỗi draft:
1. Voice consistency
2. Reading level match
3. Cross-references giữa docs
4. Glossary consistency (trong source locale)
5. Code block/example correctness

#### Walkthrough
Three-phase pipeline (v1.3.0+):

1. **VERIFY**: đọc drafts, generate test cases, dry-run với product UI structure
2. **DRIFT GATE**: so sánh drafts vs live product UI; produce drift report; user decide per item (auto-fix doc / mark là product bug / skip)
3. **APPLY + CAPTURE**: apply user decisions, capture screenshots vào `documentation/images/<module>/`

Per-locale strategy từ media policy (default: source-only — capture 1 lần ở source UI, reuse cho translated docs).

#### Record (optional)
Scan drafts cho `<!-- VIDEO id: <id> -->` markers (v1.5.7+ simplified syntax). Mỗi marker:

1. Tìm `documentation/scripts/<module>/<id>.md` (auto-gen từ draft prose nếu thiếu)
2. User review/edit script
3. Theo voiceover strategy:
   - **Silent** (default): on-screen captions từ script, không audio
   - **AI synthetic voice**: TTS produce audio per locale (dùng TTS provider từ intake — local-piper default)
   - **Source voice + per-locale subtitles**: 1 audio + N file .vtt
   - **Human recorded**: BA đặt audio thủ công; record chỉ làm screen capture
4. Encode video; produce subtitles nếu enabled

#### Translate
Process CẢ drafts AND scripts (v1.5.7+):
- Drafts: `drafts/<source>/...` → `drafts/<target>/...`
- Scripts: `scripts/<module>/<id>.md` # Source script → ## `<locale>` sections trong cùng file

Per-block (review từng block) hoặc batch (whole-file diff) review mode. Glossary tại `documentation/standards/glossary.<locale>.yaml` enforce terminology.

---

## 6. Command `deploy`

Copy/sync workspace sang host project (Docusaurus repo) với transforms.

### Preset: standalone

Không deploy. Workspace IS artifact. Skip command này.

### Preset: docusaurus

Maps:
- `documentation/drafts/<source-locale>/<module>/<doc>.md` → `<target>/docs/<module>/<doc>.md`
- `documentation/drafts/<target-locale>/<module>/<doc>.md` → `<target>/i18n/<locale>/.../current/<module>/<doc>.md`
- `documentation/images/<module>/<asset>.png` → `<target>/static/img/<product.slug>/<module>/<asset>.png`
- `documentation/videos/<id>.mp4` → `<target>/static/videos/<product.slug>/<id>.mp4`

Luôn bắt đầu với `--dry-run`:

```bash
/docsmith deploy --dry-run                  # preview file changes
/docsmith deploy                            # apply
cd ../my-docusaurus-site && git diff && git commit
```

### Transforms applied

- Frontmatter injection (slug, sidebar_position từ sitemap)
- Image namespacing (`/img/<product.slug>/...` để tránh collision với content Docusaurus khác)
- MDX escape (`<`, `>`, `{` trong prose escape để tránh JSX parsing)
- Category generation (`_category_.json` per directory; respect sitemap pattern order)

### In-place mode

Khi deploy target = `.` (chạy docsmith bên trong Docusaurus repo):
- Không copy; transforms apply IN PLACE
- Drafts trong `documentation/` giữ làm authoring source
- File cuối tại `docs/` (Docusaurus convention) là deploy targets

### Audit trail

Mỗi deploy produce `documentation/deployments/<ts>-<target>/`:
- `manifest.yaml` — copy gì đi đâu
- `diff.md` — files added/modified/deleted

Giữ cho git history. Recoverable.

---

## 7. Command `update` (v1.5.10 mở rộng)

`update` là inspector "đã đổi gì?". 3 layers:

### Layer 1: Content drift (luôn check)

Cho mỗi source registered trong `sources.lock.yaml`:
- Cheap metadata check (Notion edit time, GitHub commit SHA, GDrive revision, URL ETag, file mtime)
- Nếu unchanged → skip
- Nếu changed → flag affected drafts (dùng module sources xác định docs nào)

### Layer 2: Module diff (v1.5.10+)

So sánh modules trong source structure vs modules trong workspace:

| Bucket | Action options |
|---|---|
| **Trong source AND workspace** | Continue |
| **Trong source, KHÔNG có workspace** (mới) | Tạo intake (với `--from-source` auto-fill) |
| **Trong workspace, KHÔNG có source** (orphan) | Archive (mark `status: archived`) |

### Layer 3: Scope drift (v1.5.10+)

Cho modules trong cả source và workspace, so sánh features list:
- Module intake nói: `[create instance, edit instance]`
- Source mention: `[create instance, edit instance, auto-scaling]`
- → Scope drift detected; offer thêm `auto-scaling` vào module intake

### Interactive prompt

User chọn actions per bucket. AI apply. Update inference report tại `documentation/intake/.inference/<ts>-update.md`.

### Multi-module performance

Khi ≥5 modules cần process (new + scope drift cộng lại), AI MAY dùng Task tool spawn parallel sub-agents (v1.5.10+ Cấp 1 guidance). Sequential fallback luôn valid. Xem [intake-reference.md § 9.9](skills/docsmith/intake-reference.md).

---

## 8. Multi-locale translation

Set trong project intake § 3:
- Source language (ngôn ngữ bạn draft)
- Target languages (translate sang)

Sau khi `draft` produce source-locale docs, `translate` process:

```
documentation/drafts/en/instances/create.md     # source
        ↓
documentation/drafts/vi/instances/create.md     # AI translation
documentation/drafts/jp/instances/create.md
```

Per-block review (mỗi markdown block review riêng) là an toàn nhất. Batch (whole-file diff) nhanh hơn — default từ v1.5.0+. Auto-approve chỉ sau khi build glossary trusted.

### Glossary

Auto-create tại `documentation/standards/glossary.<locale>.yaml`:

```yaml
- term_en: instance
  term_vi: máy ảo
  notes: ưu tiên hơn "VM" cho consistency
- term_en: auto-scaling
  term_vi: tự động mở rộng
  notes: giữ "auto-scaling" trong code blocks
```

Translate command apply glossary trước khi gửi blocks tới AI. Manual additions persist.

### Script translation (v1.5.7+)

Video scripts tại `documentation/scripts/<module>/<id>.md` có:
```markdown
# Source script (en)
Welcome to MyCloud...

## vi
Chào mừng đến MyCloud...

## jp
MyCloudへようこそ...
```

Translate command process scripts cùng drafts. Cùng review mode, cùng glossary.

---

## 9. Re-run commands an toàn (re-run protocol)

Mỗi command check nếu output đã tồn tại trước khi write. Nếu có, được 4-option gate:

1. **Update** (default) — existing content đọc làm KB, AI propose deltas only. Manual edits preserved.
2. **Overwrite** — discard existing, regenerate. Backup vào `documentation/archive/<ts>/`.
3. **Side-by-side** — giữ cả 2, file mới có suffix `-v2`.
4. **Cancel** — abort.

Update mode dùng **KB inheritance** (v1.3.0+):
- Existing draft đọc làm authoritative content
- AI so sánh với current intake/sources
- Propose chỉ deltas (section mới đây, section bỏ đó, refined wording)
- User review delta, accept hoặc skip
- Result: incremental refinement, không full regeneration

Apply cho drafts, plans, voice charts, glossaries, mọi generated artifact.

### Drift detection trong walkthrough

Walkthrough có version riêng của re-run protocol focus vào UI changes:
- So sánh draft instructions vs live product UI
- Cho mỗi mismatch, classify: HIGH / MEDIUM / LOW confidence trong fix
- HIGH: doc nói "Click Submit" nhưng button là "Save" → auto-fix doc (nếu `--auto-apply-high-confidence`)
- MEDIUM: doc nói "Settings tab" nhưng UI có "Preferences tab" — thường là rename label → prompt user
- LOW: doc mô tả flow không tồn tại nữa — likely product bug; mark cho product team

Drift reports tại `documentation/walkthrough/drift/<ts>/`. User decisions trong `decisions.yaml` per drift report.

### Source change detection

`update` check nếu external sources (Notion / GitHub / GDoc) có content mới. Cheap metadata calls — không full re-fetch trừ khi thực sự đổi. Khi detect change, propagate sang affected drafts qua Update mode.

---

## 10. Mental model: skill = prompt, không phải program

docsmith là Claude skill. "Code" là markdown:

- `SKILL.md` — main skill spec, primary prompt
- `templates/*.md` — output templates AI dùng
- `*-reference.md` — deep technical reference cho AI consult
- `presets/*.yaml` — preset configs (standalone, docusaurus)

Khi bạn chạy `/docsmith init`, Claude:
1. Load SKILL.md
2. Đọc template liên quan (PROJECT_INTAKE_TEMPLATE.md cho init)
3. Follow steps mô tả trong SKILL.md `### init` section
4. Produce output

Không có compiled binary, không daemon, không state machine. AI follow spec mỗi lần.

### Implications

- **Bug fixes** là sửa prose trong `SKILL.md` hoặc reference docs, không phải code patches. Bump version, push.
- **New features** là sections / templates / flags mới mô tả trong spec. AI follow spec updated lần next run.
- **Behavior deterministic** nếu spec precise, indeterministic nếu spec vague. Quality of skill = quality of prose.
- **No silent failures**: nếu AI không follow được step, hỏi bạn. Cả skill là "AI làm X nếu Y, hỏi nếu Z".

Đó là lý do docsmith tích lũy đến ~9000 dòng spec across 41 files. Mỗi behavior được documented, không coded.

---

## 11. Cheat-sheet workflow hàng ngày

```bash
# Setup 1 lần
/plugin marketplace add dangtran1003/docsmith-v2
/plugin install docsmith@dangtran1003-docsmith-v2

# Lần đầu: project từ BA doc
mkdir my-docs && cd my-docs
/docsmith init --from-source path/to/ba-doc.md
# AI fill project.md, hỏi 5-10 câu, detect modules
# Review project.md (tìm marker "← AI guess")

# Hoặc lần đầu: manual
mkdir my-docs && cd my-docs
/docsmith init
/docsmith module instances
# Điền project.md và modules/instances.md (mỗi field có hint inline)

# Run pipeline
/docsmith run                       # pause sau draft (default)
# Review drafts trong documentation/drafts/<source>/<module>/
/docsmith continue                  # walkthrough → record → translate

# Verify trước deploy
/docsmith verify                    # 11-check audit

# Deploy
/docsmith deploy --dry-run          # preview
/docsmith deploy                    # apply sang Docusaurus repo
cd ../my-docusaurus-site
git diff && git add . && git commit -m "Update docs" && git push

# Iterate sau khi BA doc đổi
/docsmith update                    # detect content drift + module diff + scope drift
# Review proposed changes; AI apply selected
/docsmith continue                  # re-walk + re-translate khi cần

# Iterate sau khi product UI đổi
/docsmith walkthrough --check       # chỉ detect drift
# Review documentation/walkthrough/drift/<ts>/
/docsmith walkthrough --apply       # capture screenshots mới, apply decisions

# Re-record video sau khi script đổi
/docsmith record --re-record <id>
```

---

## 12. Troubleshooting

| Triệu chứng | Nguyên nhân có thể | Fix |
|---|---|---|
| "Required field missing in project.md" | Backtick rỗng hoặc critical checkbox chưa tick | Run `/docsmith intake-help` xem field reference; điền listed |
| Walkthrough hang ở login | Test account env var unset hoặc sai | `echo $MYPRODUCT_TEST_USER` verify; re-export |
| Walkthrough capture wrong UI | Per-locale strategy mismatch | Trong project intake media policy, check locale strategy |
| Translated docs có terms inconsistent | Glossary incomplete hoặc không apply | Edit `documentation/standards/glossary.<locale>.yaml` và re-run translate |
| Deploy refuse write | File collision detected | Dùng `--dry-run` inspect; resolve manual HOẶC `--force` |
| Sitemap show wrong order | Sitemap pattern mismatch với module sections | Run `/docsmith plan --migrate-sitemap` |
| Video không có voice nhưng cần | Voiceover strategy = silent HOẶC TTS provider env var unset | Check project intake § Media policy; expand Advanced nếu collapsed |
| Source không refresh trên update | Source unchanged từ last fetch | `/docsmith fetch <source-name> --force` |
| `--from-source` auto-fill miss feature | Source heading không detected | Edit module intake manually; thêm feature; hoặc rerun với source rõ hơn |
| Parallel sub-agents reported partial | Network hoặc parsing error trong 1 sub-agent | `/docsmith init --from-source --resume` retry chỉ failed |

Cho deeper issues:
- `/docsmith intake-help` — field reference
- `documentation/intake/.inference/<latest>.md` — AI infer gì
- `documentation/walkthrough/drift/<latest>/` — UI mismatch reports
- `documentation/.run-state/<module>.yaml` — pipeline state per module
- `documentation/deployments/<latest>/diff.md` — last deploy đã làm gì
