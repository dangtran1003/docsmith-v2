# Merge Decision Template

Used by Update mode (Mode 1 of the re-run protocol) to record per-item decisions when AI proposes deltas to existing content.

This is **interactive** — typically the AI proposes deltas inline and the user responds turn-by-turn. The format below is the on-disk record kept for audit.

## File location

`documentation/archive/<timestamp>/merge-decisions.yaml`

## Format

```yaml
# Merge decisions for documentation/plan/documentation-plan.md
generated: 2026-04-26T10:30:44Z
command: plan
mode: update
existing_file: documentation/plan/documentation-plan.md
existing_lines: 487
existing_last_modified: 2026-01-15T14:22:11Z

deltas:
  - id: delta-001
    kind: NEW                    # NEW | UPDATE | REMOVE | KEEP
    location: section "4.3 Auto-scaling instances"
    description: |
      Add new section covering auto-scaling group setup, scaling policies,
      health checks. 4 docs proposed.
    proposed_content_summary: |
      4 new doc entries:
      - getting-started/auto-scaling.md (Tutorial)
      - reference/scaling-policies.md (Reference)
      - guides/configure-health-checks.md (How-to)
      - explanation/scaling-architecture.md (Concept)
    decision: apply              # apply | skip | edit | defer
    decision_at: 2026-04-26T10:31:02Z
    decided_by: human

  - id: delta-002
    kind: UPDATE
    location: section "2.1 Instance types"
    description: Add c5.xlarge variant to instance type table
    diff_preview: |
      @@ -45,6 +45,7 @@
        | s1.medium | 2 vCPU | 4 GB | $0.05/hr |
        | s1.large  | 4 vCPU | 8 GB | $0.10/hr |
       +| c5.xlarge | 4 vCPU | 8 GB | $0.12/hr |
        | m1.xlarge | 8 vCPU | 16 GB | $0.18/hr |
    decision: apply
    decision_at: 2026-04-26T10:31:15Z
    decided_by: human

  - id: delta-003
    kind: REMOVE
    location: section "5.7 Legacy m1 instances"
    description: |
      Section may be obsolete. m1 instances marked deprecated in product
      changelog 2026-02. Removing this section would also drop 2 doc entries.
    confidence: LOW
    decision: skip
    decision_at: 2026-04-26T10:31:30Z
    decided_by: human
    notes: "Keep for migration period. Mark as deprecated instead."

  - id: delta-004
    kind: UPDATE
    location: section "5.7 Legacy m1 instances"
    description: |
      User counter-proposed: keep section but add deprecation banner
    diff_preview: |
      @@ -2,1 +2,4 @@
        # Legacy m1 instances
       +
       +> ⚠️ **Deprecated since 2026-02.** Migrate to c5 series. See [migration guide](...)
       +
    decision: apply
    decision_at: 2026-04-26T10:31:30Z
    decided_by: human
    notes: "Replaces delta-003 with milder change"

  - id: delta-005
    kind: KEEP
    location: sections 1, 3, 4.1, 4.2
    description: No changes proposed; existing content remains canonical
    decision: implicit-keep      # always implicit; logged for completeness
    decided_by: ai

summary:
  total_deltas: 5
  applied: 3
  skipped: 1
  edited: 0
  deferred: 0
  kept_implicit: 1

backup:
  original: archive/2026-04-26-103044/documentation-plan.md
  diff: archive/2026-04-26-103044/documentation-plan.diff
  merged: documentation/plan/documentation-plan.md   # written in place
```

## Decision values

| Value     | Meaning                                                          |
| --------- | ---------------------------------------------------------------- |
| `apply`   | Apply this delta to merged output                                |
| `skip`    | Do not apply; existing content stays as-is                       |
| `edit`    | User wants to modify the proposed delta before applying          |
| `defer`   | Decide later — leave delta in pending state                      |
| `implicit-keep` | KEEP-kind deltas; always preserved; logged for record only |

## How AI presents deltas to user

In conversation, the AI shows one delta at a time (or grouped), waits for response. Example:

```
[AI]
Proposed delta-001 (NEW):
  Add section "4.3 Auto-scaling instances" with 4 doc entries:
    - getting-started/auto-scaling.md (Tutorial)
    - reference/scaling-policies.md (Reference)
    - guides/configure-health-checks.md (How-to)
    - explanation/scaling-architecture.md (Concept)

  Apply this? [y/n/e/q/?]

[User]
y

[AI]
Applied. Next: delta-002 (UPDATE)
  Add c5.xlarge row to instance type table at section 2.1.
  ...
```

`y` apply, `n` skip, `e` edit, `q` quit (defer remaining), `?` help.

`a` (apply all remaining) is allowed but discouraged — defeats the purpose of per-item review. AI should warn when user types `a`.

## When AI generates the proposal

Before proposing deltas, AI must:

1. Read the existing file in full
2. Read related context files (audience profile, sitemap, etc. — see [update-reference.md](../update-reference.md) § "Per-artifact merge logic")
3. Identify gap between existing and what user wants now
4. Categorize each gap as NEW / UPDATE / REMOVE
5. Mark unchanged sections as KEEP
6. Order proposals: NEW first, then UPDATE, then REMOVE (so user can build up before subtracting)

AI must NOT propose:
- Cosmetic improvements (re-wording, re-ordering) unless user requested
- Style changes from updated voice rules unless user opts in via `--apply-voice-update` flag
- Cross-file changes unless explicitly requested

## After all decisions

AI writes the merged file in place, archives original, generates diff. User sees:

```
Merge complete:
  3 deltas applied
  1 delta skipped
  1 delta deferred (will re-prompt on next /docsmith plan run)

Original archived: documentation/archive/2026-04-26-103044/documentation-plan.md
Diff: documentation/archive/2026-04-26-103044/documentation-plan.diff
Merged file: documentation/plan/documentation-plan.md (now 502 lines, was 487)
```
