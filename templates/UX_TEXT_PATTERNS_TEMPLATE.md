# UX Text Patterns Guide

> Part of [PRC-010A: UX Content Standards Subprocess](prc-010a-ux-content-standards-subprocess.md)
> Based on *Strategic Writing for UX* (Podmajersky, 2019), Chapter 4

## Product Information

| Field | Value |
|-------|-------|
| **Product Name** | [Product name] |
| **Voice Chart Reference** | [Link to completed Voice Chart] |
| **Date Created** | [YYYY-MM-DD] |
| **Approved By** | [Name/role] |

## Pattern Applicability

Mark each pattern as Applicable or N/A. Provide justification for N/A patterns.

### Documentation Patterns

These patterns govern how the documentation itself is written. Applicable to all documentation projects.

| Pattern | Applicable? | Justification (if N/A) |
|---------|:-----------:|------------------------|
| Page Titles | [ ] Yes / [ ] N/A | |
| Section Headings | [ ] Yes / [ ] N/A | |
| Introductions | [ ] Yes / [ ] N/A | |
| Procedures | [ ] Yes / [ ] N/A | |
| Code Examples | [ ] Yes / [ ] N/A | |
| Callouts | [ ] Yes / [ ] N/A | |
| Cross-References | [ ] Yes / [ ] N/A | |
| Screenshots | [ ] Yes / [ ] N/A | |
| UI References | [ ] Yes / [ ] N/A | |
| API References | [ ] Yes / [ ] N/A | |
| Error Documentation | [ ] Yes / [ ] N/A | |

### UI Patterns (Optional)

These patterns govern in-product text. Only applicable when the project includes authoring UI copy.

| Pattern | Applicable? | Justification (if N/A) |
|---------|:-----------:|------------------------|
| Buttons & Links | [ ] Yes / [ ] N/A | |
| Empty States | [ ] Yes / [ ] N/A | |
| Labels | [ ] Yes / [ ] N/A | |
| Controls | [ ] Yes / [ ] N/A | |
| Text Input Fields | [ ] Yes / [ ] N/A | |
| Transitional Text | [ ] Yes / [ ] N/A | |
| Confirmation Messages | [ ] Yes / [ ] N/A | |
| Notifications | [ ] Yes / [ ] N/A | |

---

## Documentation Patterns

### Page Titles

The title at the top of each document. Must tell the reader what the page is about at a glance.

| Rule | Standard |
|------|----------|
| **Capitalization** | [Sentence case / Title Case] |
| **Max length** | [e.g., 60 characters] |
| **Form** | [Imperative for guides ("Track click stats") / Noun phrase for concepts ("How links work")] |
| **Punctuation** | [No periods / No trailing colons] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Section Headings

Headings within a document that break content into scannable sections.

| Rule | Standard |
|------|----------|
| **Capitalization** | [Sentence case / Title Case] |
| **Max depth** | [e.g., H2-H3 only, avoid H4+] |
| **Form** | [Noun phrase / Gerund / Imperative — be consistent within a doc] |
| **Numbering** | [Never number headings / Number only in tutorials] |
| **Parallel structure** | [All sibling headings use the same grammatical form] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Introductions

The opening paragraph of each document or major section. Sets context and tells the reader what to expect.

| Rule | Standard |
|------|----------|
| **Max sentences** | [e.g., 2-3 sentences] |
| **Opening** | [Lead with what the user can do / Lead with what this page covers] |
| **Person** | [Second person "you" / Impersonal] |
| **Include prerequisites?** | [Yes: state them upfront / No: separate section] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Procedures

Step-by-step instructions that guide the reader through a task.

| Rule | Standard |
|------|----------|
| **Format** | [Numbered list, one action per step] |
| **Step opening** | [Start with a verb: "Click", "Enter", "Select"] |
| **Expected result** | [Include after the action: "The dashboard opens." / Omit if obvious] |
| **Verification** | [End procedures with a verification step: "You should see..."] |
| **Max steps** | [e.g., 10 steps before splitting into sub-procedures] |
| **Screenshots** | [Include placeholder after key steps / Only for complex steps] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Code Examples

Code blocks showing API requests, terminal commands, or configuration.

| Rule | Standard |
|------|----------|
| **Language tag** | [Always specify: ```bash, ```json, ```python] |
| **Runnable** | [Examples should be copy-pasteable / Pseudocode is OK] |
| **Placeholders** | [Format: `YOUR_VALUE` in SCREAMING_SNAKE_CASE / `<value>` in angle brackets] |
| **Secrets** | [Never use realistic tokens. Use `YOUR_ACCESS_TOKEN` or `YOUR_API_KEY`] |
| **Output** | [Show expected response after request / Separate block or inline comment] |
| **Max lines** | [e.g., 20 lines per block before splitting] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Callouts

Admonitions that highlight important, cautionary, or supplementary information.

| Rule | Standard |
|------|----------|
| **Types** | [Note (tip/info), Caution (unexpected consequence), Warning (danger/data loss)] |
| **Frequency** | [Max 1-2 per page section; use sparingly] |
| **Format** | [Blockquote with emoji / Custom admonition syntax / Bold prefix] |
| **Length** | [1-2 sentences max] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Cross-References

Links to other documents within the documentation set.

| Rule | Standard |
|------|----------|
| **Format** | [Inline link with descriptive text: "See [How links work](path)" / Not bare URLs] |
| **Placement** | [Inline where relevant + "Additional resources" section at end] |
| **Wording** | ["See [Page title]" / "Learn more in [Page title]" — be consistent] |
| **Avoid** | [Circular references / "Click here" / Linking the same page more than once per section] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Screenshots

Images showing the product UI to support written instructions.

| Rule | Standard |
|------|----------|
| **Placement** | [After the step they illustrate] |
| **Caption** | [Always include: `![Caption describing what the screenshot shows](path)`] |
| **Placeholders** | [During drafting, use `![Caption](https://placehold.co/600x400)`] |
| **Format** | [PNG for static / GIF for multi-step flows] |
| **Annotations** | [Use arrows or highlights to draw attention / Keep clean without annotations] |
| **Naming** | [Descriptive filename: `create-link-form.png`, not `screenshot1.png`] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### UI References

How to refer to buttons, fields, menus, and other UI elements in documentation text.

| Rule | Standard |
|------|----------|
| **Formatting** | [Bold: **Create your link** / Bold for buttons, plain for pages] |
| **Match UI exactly** | [Use the exact label from the product — never paraphrase] |
| **Navigation paths** | [Arrow separator: **Settings → Developer settings**] |
| **Keyboard shortcuts** | [Format: `Ctrl+C` / **Ctrl+C** — be consistent] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### API References

Patterns for documenting API endpoints, parameters, and responses.

| Rule | Standard |
|------|----------|
| **Endpoint format** | [HTTP method + path: `GET /v4/links/{id}`] |
| **Parameters** | [Table with: Name, Type, Required, Description] |
| **Request example** | [Full curl command with placeholder token] |
| **Response example** | [Complete JSON with realistic but fictional data] |
| **Status codes** | [Table with: Code, Meaning, Description] |
| **Base URL** | [State once at the top, use relative paths in endpoints] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

### Error Documentation

How to document error messages and troubleshooting steps.

| Rule | Standard |
|------|----------|
| **Format** | [Symptom → Cause → Fix] |
| **Tone** | [Calm, solution-focused. Never blame the reader.] |
| **Symptom wording** | [Describe what the user sees: "Link not redirecting" not "Error 404"] |
| **Fix specificity** | [Actionable: "Regenerate your API key from **Settings → Developer settings**"] |
| **API errors** | [Include error code, message, and common cause] |

**Do:**
- [Example]

**Don't:**
- [Example]

---

## UI Patterns (Optional)

> Complete this section only if the project includes authoring in-product UI text.

### Buttons & Links

| Rule | Standard |
|------|----------|
| **Max words** | [e.g., 3 words] |
| **Verb form** | [Imperative: "Save", "Create" / Descriptive: "Next", "Done"] |
| **Capitalization** | [Sentence case / Title Case] |
| **Specificity** | [Specific: "Save changes" / Generic: "Submit"] |

---

### Empty States

| Rule | Standard |
|------|----------|
| **Tone** | [Encouraging / Neutral] |
| **Include CTA?** | [Yes / No] |
| **Max length** | [e.g., 2 sentences] |

---

### Labels

| Rule | Standard |
|------|----------|
| **Capitalization** | [Sentence case / Title Case] |
| **Max words** | [e.g., 3 words] |
| **Abbreviations** | [Allowed / Spell out] |

---

### Controls

| Rule | Standard |
|------|----------|
| **Toggle format** | [e.g., "Enable notifications"] |
| **Checkbox format** | [e.g., "Send me updates"] |
| **Capitalization** | [Sentence case] |

---

### Text Input Fields

| Rule | Standard |
|------|----------|
| **Placeholder text** | [Example format / Hint / Leave empty] |
| **Helper text** | [When to include] |
| **Error text position** | [Below field / Inline / Tooltip] |

---

### Transitional Text

| Rule | Standard |
|------|----------|
| **Verb tense** | [Present progressive: "Saving..." / Past: "Saved"] |
| **Ellipsis** | [Use trailing ellipsis / No ellipsis] |

---

### Confirmation Messages

| Rule | Standard |
|------|----------|
| **Tone** | [Positive / Neutral] |
| **Include next step?** | [Yes / No] |
| **Max length** | [e.g., 1 sentence] |

---

### Notifications

| Rule | Standard |
|------|----------|
| **Urgency levels** | [Info / Warning / Critical] |
| **Action required?** | [Always / Informational OK] |
| **Max length** | [e.g., Title: 40 chars, Body: 100 chars] |

---

## Cross-Pattern Rules

Rules that apply across all patterns:

| Rule | Standard |
|------|----------|
| **Abbreviations** | [Spell out on first use per document, then abbreviate] |
| **Numbers** | [Spell out 1-9, numerals for 10+ / Always numerals in technical context] |
| **Dates** | [ISO 8601 in API: `2026-12-31T23:59:59Z`. Human-readable in docs: "December 31, 2026"] |
| **Times** | [Relative when possible: "within 5 minutes". 24-hour in API.] |
| **URLs** | [Inline code in API context. Link text in docs: [Dashboard](url)] |
| **Product name** | [Exact casing and formatting rules] |
| **Oxford comma** | [Yes / No] |
| **Contractions** | [Allowed / Not allowed / Context-dependent] |
