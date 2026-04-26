# Video Marker Template

This file documents the convention for embedding video specifications in draft documentation. The `record` command consumes these markers, produces video files, and replaces the markers with embed code.

## When to include a video

Videos are **optional**. They supplement screenshots and text, never replace them. Use a video only when one of the following applies:

- **Quick Start tour**: A 60–90 second overview at the top of a Getting Started page that shows what the product looks like end-to-end.
- **Complex procedure with motion**: Drag-and-drop, resizing, animation, or multi-page flow where stills don't convey the dynamics. Place the video at the top of the procedure section, before the text steps.
- **End-to-end tutorial summary**: A single video at the end of a tutorial that replays the whole journey.

Do NOT use videos for:

- Concept or explanation pages (use diagrams)
- Reference pages (API docs, settings tables)
- Troubleshooting (text is searchable, videos aren't)
- Anything a screenshot can already show

## Marker syntax

Insert this HTML comment block at the position where the video should appear in the rendered doc:

```html
<!-- VIDEO
id: <unique-kebab-case-id>
duration: <seconds>s
type: <tour | procedure | tutorial-summary>
start: <description of the starting screen and state>
actions:
  - <action 1, including any data to enter>
  - <action 2>
  - <action 3>
end: <description of the final screen and state to capture before stopping recording>
highlight: <optional — UI elements to visually emphasize, comma-separated>
pacing: <optional — slow on X, fast on Y>
caption: <one-line title shown to users below the embedded video>
-->
```

## Field reference

| Field        | Required | Purpose                                                                                                                          |
| ------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `id`         | Yes      | Stable identifier. Used as filename (`videos/<id>.mp4`) and re-record key. Lowercase, kebab-case, descriptive of the flow.       |
| `duration`   | Yes      | Target length. Hard cap: 120 seconds. Most videos should be 30–90s.                                                              |
| `type`       | Yes      | One of: `tour` (Quick Start), `procedure` (single how-to), `tutorial-summary` (end of tutorial).                                 |
| `start`      | Yes      | Where to begin recording. Must describe both the page AND the state (e.g. empty list, populated list, specific tab open).        |
| `actions`    | Yes      | Ordered list of actions. Each action references UI by label and includes any data to enter. One action per line.                 |
| `end`        | Yes      | The visible state at which recording stops. Like `start`, must describe page + state.                                            |
| `highlight`  | No       | UI elements to visually emphasize during the relevant action (e.g. cursor halo, outline). Comma-separated.                       |
| `pacing`     | No       | Hints to slow down (form filling, decisions) or speed up (loading waits, repetitive steps).                                      |
| `caption`    | No       | One-line title shown beneath the video in the rendered doc. Not used for capture — purely user-facing.                           |

## Examples

### Good — Quick Start tour

```html
<!-- VIDEO
id: instance-quickstart-tour
duration: 75s
type: tour
start: Compute > Instances list page, empty state, no instances created
actions:
  - Click "Create Instance" button in page header
  - In form, enter name "demo-vm"
  - Select Ubuntu 22.04 from Image dropdown
  - Select s1.medium from Flavor dropdown
  - Click "Create" at bottom of form
  - Wait for provisioning indicator to complete
end: Instance list with one running instance named "demo-vm", status "Running"
highlight: Create Instance button at start, status badge at end
pacing: Slow on form filling, fast on provisioning wait
caption: Creating your first instance in under 90 seconds
-->
```

This marker is good because:
- Every action is unambiguous and includes the data to enter
- Start and end describe both page and state
- Pacing prevents a 30-second wait from inflating duration
- The `id` describes the flow, not the file

### Bad — vague and underspecified

```html
<!-- VIDEO
id: video1
duration: 60s
start: Instance page
actions:
  - Create instance
  - Configure it
  - Submit
end: Done
-->
```

Why it fails:
- `id: video1` — meaningless when there are 10 such markers
- `start: Instance page` — which page? what state?
- Actions don't say what data to enter or which UI elements to use
- `end: Done` — done what? what's on screen?
- No `type`, no `pacing`, will run uneven
- The AI has to guess; the video will not be reproducible

## After recording

The `record` command replaces the marker with a video embed but **preserves the original marker as a hidden HTML comment** beside the embed. This lets re-record locate which video corresponds to which marker without re-parsing the doc.

```html
<video src="videos/instance-quickstart-tour.mp4" controls></video>
<!-- VIDEO_RECORDED
id: instance-quickstart-tour
last_recorded: 2026-04-26
ui_version: console-v3.2
source_marker: |
  (full original marker preserved here)
-->
```

When the product UI changes and you need to re-record, edit the original marker (now stored under `source_marker`), bump `ui_version`, and re-run `record`. It will find the marker via the preserved comment.

## Caption rules

The caption rules from [SCREENSHOT_POLICY_TEMPLATE.md](SCREENSHOT_POLICY_TEMPLATE.md) apply equally to video `start`, `actions`, `end`:

- Describe what is on screen, not what the user is doing — for `start` and `end`
- Be specific about data and state
- Reference UI by label, not appearance

For `actions`, use imperative phrasing with explicit UI labels and data values: "Enter 'demo-vm' in the Name field" — not "fill out the form".
