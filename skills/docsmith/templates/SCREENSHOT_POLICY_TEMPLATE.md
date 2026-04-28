# Screenshot Policy: [Product/Project Name]

This file is the source of truth for how screenshots are authored, captured, and named in this project. It is read by the `draft` and `walkthrough` commands.

## 1. When to include a screenshot

Include a screenshot when **at least one** of the following is true:

- **UI element has to be located.** A screenshot saves the user from a sentence like "the small icon in the top right corner with three dots".
- **State changes visibly.** A modal opens, a form is submitted, an item moves to a new column. Show before and/or after when the change is the point.
- **Confirmation of success.** End of a procedure. The user needs to see what "done" looks like.
- **Layout matters.** Dashboards, list views with key columns, complex forms where the structure of the page is itself part of the explanation.

Do NOT include a screenshot when:

- The content is a definition, concept, or list of properties (use a table or diagram instead)
- The content is command-line input/output (use a code block — searchable, copy-pasteable, accessible)
- The screenshot would only show generic chrome (browser tabs, window decorations, OS taskbar)
- The state shown is identical to a screenshot two paragraphs above

## 2. Density rule (procedure-style docs)

Every procedure must have, at minimum:

- One screenshot at the **starting state**
- One screenshot after **each step that visibly changes the UI**
- One screenshot at the **success state** at the end

A procedure with 5 steps and 3 visible UI changes gets ~5 screenshots, not 1.

## 3. Caption writing rules

The caption is the **contract** between the doc and the screenshot. The walkthrough uses the caption to match content. Vague captions cause wrong screenshots.

### Rule A: Describe what is on screen, not what the user does

Captions are not action labels. They describe the visual state at the moment of capture.

| Bad (action) | Good (state) |
|---|---|
| `![Click the Create button](...)` | `![Compute Instances list with empty state and Create button visible at top right](...)` |
| `![Filling out the form](...)` | `![Create Instance form with Name field set to "demo-vm" and Ubuntu 22.04 selected](...)` |
| `![Submit the form](...)` | `![Create Instance form fully filled, Submit button enabled](...)` |

### Rule B: Be specific about data and state

A list view can have many states. Always say which one.

| Vague | Specific |
|---|---|
| `![Instance list](...)` | `![Instance list with 3 running instances sorted by creation date](...)` |
| `![Settings page](...)` | `![Settings > Network tab with "Auto-assign IP" enabled](...)` |
| `![Dashboard](...)` | `![Dashboard showing 0 alerts and 5 active resources](...)` |

### Rule C: Reference UI by label, not by appearance

Labels are stable across themes; colors and positions are not.

| Bad | Good |
|---|---|
| `![The blue button at the top](...)` | `![Page header with "Create Instance" button](...)` |
| `![Icon in the corner](...)` | `![Page header with Help icon (question mark)](...)` |

### Rule D: Place the placeholder AFTER the step it illustrates

The walkthrough builds context from the lines above the placeholder. Putting the screenshot before the step it shows leaves the placeholder context-free.

```markdown
3. Click **Create Instance**.

   ![Compute Instances list with Create Instance button highlighted](https://placehold.co/600x400)

4. In the form, enter the instance name and select an image.

   ![Create Instance form with Name and Image fields filled](https://placehold.co/600x400)
```

## 4. File naming conventions

- Lowercase, hyphen-separated, no spaces
- Named for **what the asset is**, not what it looks like (UI changes; meaning doesn't)
- Folder by feature, not by doc

```
docs/images/
├── instances/
│   ├── instances-list-empty.png
│   ├── instance-create-form-empty.png
│   ├── instance-create-form-filled.png
│   └── instance-create-success.png
├── storage/
│   └── ...
```

Bad: `screenshot-of-blue-button.png`, `step-3.png`, `image1.png`
Good: `instance-create-form-filled.png`, `network-settings-firewall-rules.png`

## 5. State that cannot be reproduced

If a screenshot requires data state that the AI cannot create reliably (a long-running process, specific historical data, real customer records), **keep the placeholder** in the doc and document the reason in the walkthrough execution record. Do not delete the placeholder; do not capture an inaccurate substitute.

## 6. Pre-walkthrough caption review

Before the `walkthrough` command captures any screenshot, it presents the **screenshot capture plan** for human review. For each entry, verify the caption passes Rules A–C above. Captions that fail are flagged. Fix them in the draft, then re-run.

## 7. When in doubt

A good caption is one where someone reading **only the caption** (no surrounding text) could roughly imagine what the screenshot shows. If you cannot, neither can the AI capturing it.
