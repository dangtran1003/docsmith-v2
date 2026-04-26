# Translation Decisions Template

Records per-block decisions made during interactive `translate` review. Saved to `documentation/archive/<timestamp>/translation-decisions-<locale>.yaml`.

## Format

```yaml
generated: 2026-04-26T11:50:32Z
command: translate
source_locale: en
target_locale: vi
source_file: documentation/drafts/en/instances/create.md
target_file: documentation/drafts/vi/instances/create.md
existing_target: false                # was target_file present before this run?
glossary_version: "2026-04-26"
glossary_terms_applied: 14            # how many glossary substitutions
mode: per-block                        # per-block | auto-approve
docsmith_version: "1.4.0"

blocks:
  - id: block-001
    type: frontmatter
    source_lines: [1, 5]
    source: |
      ---
      id: instances/create
      title: Create an instance
      sidebar_position: 2
      ---
    translation: |
      ---
      id: instances/create
      title: Tạo instance
      sidebar_position: 2
      ---
    decision: approve
    decided_at: 2026-04-26T11:50:35Z
    glossary_hits:
      - term: "instance"
        rule: keep_verbatim
      - term: "Create"
        rule: imperative_context
        substituted_with: "Tạo"

  - id: block-002
    type: heading
    level: 1
    source_lines: [7, 7]
    source: "# Create an instance"
    translation: "# Tạo một instance"
    decision: approve

  - id: block-003
    type: paragraph
    source_lines: [9, 11]
    source: |
      An instance is a virtual machine that runs in the cloud. You can
      start, stop, and configure instances through the Compute service.
    translation: |
      Instance là một máy ảo chạy trên đám mây. Bạn có thể khởi động,
      dừng, và cấu hình instance qua dịch vụ Compute.
    decision: edit
    user_edit: |
      Instance là một máy ảo chạy trên đám mây. Bạn có thể khởi động,
      dừng và cấu hình các instance thông qua dịch vụ Compute.
    decided_at: 2026-04-26T11:51:02Z
    note: "User adjusted: 'và' before 'cấu hình' (no comma); 'thông qua' instead of 'qua'"

  - id: block-004
    type: code_block
    source_lines: [13, 17]
    source: |
      ```bash
      mycloud instance create --name demo-vm --image ubuntu-22.04
      ```
    translation: same      # code blocks always preserve verbatim
    decision: implicit-approve
    decided_at: 2026-04-26T11:51:02Z
    note: "Auto-preserved (code block)"

  - id: block-005
    type: list_item
    source_lines: [21, 21]
    source: "- **Name**: instance name (required)"
    translation: "- **Name**: tên instance (bắt buộc)"
    decision: approve
    glossary_hits:
      - term: "instance"
        rule: keep_verbatim
    note: "'Name' kept as UI label per glossary"

  - id: block-006
    type: image
    source_lines: [25, 25]
    source: "![Compute Instances list with empty state](/images/instances/list-empty.png)"
    translation: "![Danh sách Compute Instances ở trạng thái trống](/images/instances/list-empty.png)"
    decision: approve
    note: "Alt text translated; src preserved"

  - id: block-007
    type: video_marker
    source_lines: [27, 35]
    source: |
      <!-- VIDEO
      id: instance-create-tour
      duration: 75s
      ...
      -->
    translation: same      # video markers always preserve verbatim
    decision: implicit-approve
    note: "Auto-preserved (video marker — not user-visible)"

  - id: block-008
    type: paragraph
    source_lines: [37, 39]
    source: "(some block...)"
    translation: "(some translation...)"
    decision: skip
    decided_at: 2026-04-26T11:52:18Z
    note: "User chose to keep source text in target file (not yet ready to translate)"

  - id: block-009
    type: paragraph
    source_lines: [41, 43]
    source: "(some block...)"
    decision: remove
    decided_at: 2026-04-26T11:52:30Z
    note: "User omitted block from target — source-only content"

summary:
  total_blocks: 23
  approved_as_is: 16
  edited: 4
  skipped_kept_source: 2
  removed: 1
  glossary_terms_applied: 14
  unique_glossary_terms: 6

# Linked artifacts
artifacts:
  source_snapshot: archive/2026-04-26-115032/instances-create.en.md
  target_backup: null            # null = no existing target file (first translate)
  target_after: documentation/drafts/vi/instances/create.md
  diff_vs_target_before: null
```

## Decision values

| Value              | Meaning                                                        |
| ------------------ | -------------------------------------------------------------- |
| `approve`          | User pressed `y` — applied AI translation as-is                |
| `edit`             | User pressed `e` — provided their own translation              |
| `skip`             | User pressed `s` — kept SOURCE text in target file             |
| `remove`           | User pressed `n` — block omitted from target file              |
| `implicit-approve` | No prompt shown — block is in preserve-verbatim category       |
| `auto-approve`     | `--auto-approve` flag mode — applied without user gate         |

## Why this audit trail matters

1. **Reproducibility**: re-running translate uses these decisions to skip blocks already approved
2. **Quality monitoring**: high `edited` count signals AI translation quality issues — surface terms to add to glossary
3. **Translator handoff**: when a human translator takes over, they can see what AI proposed and what was rejected
4. **Compliance**: for regulated docs, audit trail shows human review happened
