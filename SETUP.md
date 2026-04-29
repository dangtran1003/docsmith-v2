# Setup Guide — Walkthrough & Record Prerequisites

`walkthrough` (verify docs against live product, capture screenshots) and `record` (capture tutorial videos) need browser automation tools that are NOT installed by docsmith itself. This guide walks you through setup before your first `wt` / `record` run.

> 🇻🇳 Bản tiếng Việt: [SETUP.vi.md](SETUP.vi.md)

## TL;DR

| Want to do                          | Need                                                                 |
| ----------------------------------- | -------------------------------------------------------------------- |
| Just write docs, skip product verify | Nothing extra. `init`, `module`, `draft`, `edit`, `translate`, `deploy` work without browser tools. |
| Capture screenshots (`walkthrough`) | Claude in Chrome extension (recommended) OR Playwright MCP server   |
| Record tutorial videos (`record`)   | Same as walkthrough + ffmpeg                                         |
| Fetch from Notion / GDrive / GitHub | Auth tokens set as env vars (no install)                             |

If you're new to this, start with **Path 1: Claude in Chrome** below. It's the easiest.

---

## Path 1: Claude in Chrome extension (recommended for beginners)

This is the official Anthropic browser extension that gives Claude a Chrome browser to control. Best fit for docsmith's `walkthrough` command.

### 1. Install Claude in Chrome

1. Open Chrome (or Chromium / Brave / Edge — anything Chromium-based)
2. Visit https://claude.ai/chrome
3. Click "Add to Chrome" → "Add extension"
4. Sign in with your Anthropic account if prompted
5. Pin the extension to the toolbar (the puzzle icon → pin Claude)

### 2. Verify the extension works

1. Click the Claude extension icon in the toolbar
2. You should see a Claude side panel
3. Type "What page am I on?" — Claude should answer with the current URL
4. If this works, the extension is ready

### 3. Connect Claude Code to the extension

The extension runs as an MCP server that Claude Code can talk to. By default, Claude Code auto-discovers it. Verify:

```bash
# In Claude Code
/mcp list
```

You should see `claude-in-chrome` (or similar) listed as available. If not, the extension may not be installed correctly — restart Chrome and Claude Code.

### 4. Test browser automation

In Claude Code:

```
Open https://example.com in a tab and tell me what's on the page
```

Claude should open a tab via the extension and report content. This proves the integration works.

### 5. You're ready

Now `/docsmith wt` will work. The skill knows to call browser tools via the extension.

**Limitations**:
- Only works on machines where Chrome is installed
- Requires Chrome to be running during walkthrough
- Cannot run headless (Chrome must be visible)

---

## Path 2: Playwright MCP server (for headless / CI use)

If you need walkthrough to run on a server without GUI (CI/CD, remote dev environment), use Playwright instead.

### 1. Install Playwright

```bash
# Node.js required (>=18)
npm install -g @playwright/mcp
npx playwright install chromium
```

This downloads ~200MB for the Chromium binary.

### 2. Configure Claude Code to use Playwright MCP

Add to your Claude Code MCP config (location depends on platform; usually `~/.claude/mcp.json` or via `/mcp add`):

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

Restart Claude Code. Verify with `/mcp list` — should see `playwright`.

### 3. Test

```
Use Playwright to open https://example.com and screenshot it
```

If you get a screenshot back, it's working.

### 4. Headless mode

Playwright runs headless by default. For local development you can switch to headed mode:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headed"]
    }
  }
}
```

### 5. You're ready

`/docsmith wt` will use Playwright tools instead of Claude in Chrome.

**Limitations**:
- More setup overhead
- Browser fingerprint differs slightly from real Chrome (some products detect and block automation)
- Requires Node.js installed

---

## Path 3: For `record` (tutorial videos)

Recording videos needs everything in Path 1 or Path 2 PLUS a video encoder.

### 1. Install ffmpeg

ffmpeg captures the browser viewport and encodes to mp4.

**macOS**:
```bash
brew install ffmpeg
```

**Ubuntu / Debian**:
```bash
sudo apt update && sudo apt install ffmpeg
```

**Windows**:
- Download from https://www.gyan.dev/ffmpeg/builds/
- Extract and add `bin/` to PATH

Verify:
```bash
ffmpeg -version
```

Should print version 5.0+ for best compatibility with browser screen recording.

### 2. Browser must support screen capture API

Modern Chrome/Firefox/Edge support `getDisplayMedia()` natively. If you're using:
- **Claude in Chrome**: works out of the box
- **Playwright**: pass `--enable-features=ScreenCaptureKit` to Chromium args

### 3. Permissions

First time `record` runs, the browser will ask for:
- Screen / window / tab capture permission → grant
- Microphone (if voiceover) → grant or skip

Permissions are remembered for the test product URL after first grant.

### 4. Test

```bash
/docsmith record --test
```

(if implemented; otherwise add a test marker to a draft and run `/docsmith record`)

Expected output: `documentation/videos/<asset-id>.mp4` ~10-90 seconds long.

### 5. Storage considerations

Raw recordings can be large (10-50 MB per minute at 1080p). docsmith automatically:
- Saves raw to `documentation/videos/raw/` (gitignored)
- Encodes final `.mp4` to `documentation/videos/<asset-id>.mp4` at lower bitrate
- Final file size typically 1-5 MB per video

Add `documentation/videos/raw/` to `.gitignore` if not already (docsmith init does this).

### 6. TTS provider install (only if voiceover strategy = "AI synthetic voice")

If your project intake § 11 selects "AI synthetic voice", install one of these TTS providers:

#### Local Piper (default, recommended)

```bash
pip install piper-tts

# Download voice models per locale you'll use
# Browse models: https://github.com/rhasspy/piper/blob/master/VOICES.md
piper --download-voice en_US-amy-medium
piper --download-voice vi_VN-25hours_single-low
piper --download-voice ja_JP-nemo-low
```

Verify:
```bash
echo "Hello world" | piper --model en_US-amy-medium --output_file test.wav
```

#### Local Coqui TTS

```bash
pip install TTS

# List available models
python -m TTS.bin.list_models

# Test synthesis
tts --text "Hello world" --model_name tts_models/en/vctk/vits --out_path test.wav
```

#### OpenAI TTS

```bash
export OPENAI_API_KEY="sk-..."
# No install — uses HTTP API
```

#### ElevenLabs

```bash
export ELEVENLABS_API_KEY="..."
# No install — uses HTTP API
# Browse voices: https://api.elevenlabs.io/v1/voices
```

#### Google Cloud TTS

1. Create GCP project, enable "Cloud Text-to-Speech API"
2. Create service account, download JSON key
3. Set:
```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json
```

#### Azure Cognitive Services

1. Create Azure Speech resource
2. Set:
```bash
export AZURE_SPEECH_KEY="..."
export AZURE_SPEECH_REGION="eastus"   # or your region
```

### 7. Subtitle generation (Whisper for STT, only if voiceover = human)

If voiceover strategy is "Human recorded" AND you want auto subtitle generation, install Whisper:

```bash
pip install openai-whisper
# Or for faster local inference:
pip install faster-whisper
```

Verify:
```bash
whisper test.mp3 --model base --output_format vtt
```

Skip this if you'll provide `.vtt` files manually OR if voiceover is silent/AI (subtitles auto-generate from script in those cases).

---

## Environment variables for sources and credentials

`fetch` and `walkthrough` need credentials. NEVER put these in intake files — only env var NAMES belong there.

### Setting env vars

**macOS / Linux** (temporary, current shell only):
```bash
export NOTION_TOKEN="secret_..."
export GITHUB_TOKEN="ghp_..."
export GOOGLE_DRIVE_TOKEN="ya29..."
export MYCLOUD_TEST_USER="qa@mycloud.test"
export MYCLOUD_TEST_PASS="your-test-password"
```

**macOS / Linux** (persistent, add to shell rc):
```bash
# Add to ~/.zshrc or ~/.bashrc
echo 'export NOTION_TOKEN="secret_..."' >> ~/.zshrc
source ~/.zshrc
```

**Windows PowerShell**:
```powershell
$env:NOTION_TOKEN = "secret_..."
# Persistent:
[System.Environment]::SetEnvironmentVariable('NOTION_TOKEN', 'secret_...', 'User')
```

### Per-project `.env` (recommended for teams)

Use a `.env` file at project root (gitignored):

```bash
# .env (DO NOT COMMIT)
NOTION_TOKEN=secret_...
GITHUB_TOKEN=ghp_...
MYCLOUD_TEST_USER=qa@mycloud.test
MYCLOUD_TEST_PASS=...
```

Then before running docsmith:
```bash
# Source it
source .env
# Or via direnv (auto-loads on cd):
direnv allow
```

Add `.env` to `.gitignore`:
```
echo ".env" >> .gitignore
```

### How to get tokens for each source

#### Notion (NOTION_TOKEN)

1. Go to https://www.notion.so/my-integrations
2. Click "+ New integration"
3. Name it (e.g. "docsmith-fetch"), select workspace, internal type
4. Copy "Internal Integration Token" (starts with `secret_`)
5. Go to the Notion page you want docsmith to read → click "..." top-right → "Add connections" → select your integration
6. Page is now accessible to docsmith via the token

Notion permissions are page-by-page. You must explicitly share each page.

#### GitHub (GITHUB_TOKEN)

For **public repos**: no token needed. Leave the auth env var empty in intake.

For **private repos**:

1. Go to https://github.com/settings/tokens?type=beta
2. "Generate new token" → fine-grained PAT
3. Repository access: select the specific repos docsmith will fetch
4. Permissions:
   - Contents: Read-only
   - Metadata: Read-only (auto-included)
5. Copy the token (starts with `github_pat_`)

For org repos: ask your org admin to install GitHub App or generate org-scoped PAT.

#### Google Drive (GOOGLE_DRIVE_TOKEN)

Two options:

**Option A: Service account** (better for production):
1. Go to https://console.cloud.google.com → create project
2. Enable "Google Drive API"
3. Create service account → download JSON key
4. Set `GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json`
5. Share the GDrive doc with the service account's email

**Option B: OAuth2 user token** (easier for personal):
1. Use https://developers.google.com/oauthplayground
2. Select Drive API v3 → readonly scope
3. Authorize → exchange for access token (note: expires in 1 hour, refresh needed)

Option A recommended for production. Option B for testing.

#### Product walkthrough credentials

Whatever your product uses. Common patterns:

```bash
export MYCLOUD_TEST_USER="qa-bot@example.com"
export MYCLOUD_TEST_PASS="..."

# If 2FA, often skip via test account configuration. Some setups:
export MYCLOUD_TEST_TOTP_SECRET="..."  # if TOTP-based 2FA
export MYCLOUD_TEST_BACKUP_CODE="..."  # if backup code

# If SSO, walkthrough may need session cookie:
export MYCLOUD_TEST_SESSION_COOKIE="session=abc123..."
```

Use a dedicated **test account** with limited permissions. Don't use real production accounts.

---

## Pre-walkthrough checklist

Before first `/docsmith wt`, verify:

- [ ] Browser extension or Playwright installed (Path 1 or 2)
- [ ] `/mcp list` in Claude Code shows the browser tool
- [ ] Test browser automation works (open example.com test)
- [ ] Test account credentials set as env vars
- [ ] Test account can actually log into the product (try manually first)
- [ ] If 2FA: test account has 2FA disabled OR you have backup codes
- [ ] Product URL accessible from your machine (not VPN-blocked)
- [ ] Drafts exist with screenshot placeholders (`{{screenshot:...}}`) or video markers (`<!-- VIDEO ... -->`)

If `walkthrough` still fails, run `/docsmith wt --check --verbose` to see what step breaks.

## Pre-record checklist

Same as walkthrough plus:

- [ ] ffmpeg installed (`ffmpeg -version` works)
- [ ] At least one `<!-- VIDEO ... -->` marker in drafts
- [ ] Browser allows screen capture (test with: in Claude Code, "use the browser to record this tab for 5 seconds")
- [ ] Disk space for raw recordings (~50 MB per minute)

---

## Common errors and fixes

### "No browser tool available"

Cause: neither Claude in Chrome nor Playwright MCP is connected.

Fix: install Path 1 or Path 2 above. Run `/mcp list` to confirm.

### "Cannot click element: not found"

Cause: product UI changed or selector outdated.

Fix: run `/docsmith wt --check` to detect drift. Update test cases or accept new UI.

### "Login failed"

Cause: credentials env var not set OR test account locked.

Fix: 
```bash
echo $MYCLOUD_TEST_USER  # should print, not be empty
```
If empty, re-export. If account locked, unlock via product UI.

### "Notion page not found" / 404

Cause: page not shared with integration.

Fix: in Notion, "..." → "Add connections" → select your integration.

### "ffmpeg: command not found"

Cause: ffmpeg not in PATH.

Fix: install ffmpeg per OS instructions above. Verify with `which ffmpeg`.

### "Headless browser can't access webcam/screen"

Cause: trying to record video in headless mode.

Fix: switch Playwright to `--headed` mode OR use Claude in Chrome (always headed).

### "Walkthrough captures wrong locale"

Cause: product UI not switched to expected language for `wt --locale vi`.

Fix: pre-walkthrough setup script should switch language:
```bash
# In module intake's "Pre-walkthrough setup script":
# (run before browser opens)
mycloud user prefs set --locale vi
```

Or do it manually in browser before running `wt --locale vi`.

---

## Network and firewall considerations

`fetch` and `walkthrough` need outbound HTTPS to:

| Service                 | Domain                                      |
| ----------------------- | ------------------------------------------- |
| Notion API              | `api.notion.com`                            |
| GitHub API              | `api.github.com`, `raw.githubusercontent.com` |
| Google Drive API        | `www.googleapis.com`                        |
| Your product            | `console.your-product.com` (whatever it is) |
| Anthropic (Claude API)  | `api.anthropic.com`                         |
| Browser extension auth  | `claude.ai`, `anthropic.com`                |

If behind corporate firewall, request these be allowlisted. docsmith does NOT need any inbound ports.

---

## Performance tuning

**Walkthrough is slow** (>5 min per module):

- Reduce screenshot count: tighten module intake `Features to document` (fewer features per module)
- Use `wt --check` (no capture, fast drift detect) for daily monitoring; only full `wt` weekly
- Run on faster machine (browser automation is CPU-bound)

**Source fetch is slow** (>1 min per Notion page):

- Notion API rate-limits to 3 req/s. Multiple sources fetch sequentially.
- Cache: `documentation/.cache/sources/` reuses content if hash matches. Don't delete cache between runs.
- Use `update` command to fetch only changed sources, not all.

**Translate is slow** (>2 min per file):

- Per-block mode is slowest (each block = LLM call). Use `batch` mode (default) for normal flow.
- Use `--auto-approve` only after building good glossary

**Browser automation flaky**:

- Most issues are timing. Increase wait time in test cases (intake → walkthrough setup → wait_for selector)
- Use deterministic test data (don't depend on dynamic IDs that change per session)

---

## Optional: docsmith without any browser

If you only want to draft and translate (no product verification, no screenshots), skip Paths 1-3 entirely. The pipeline runs without browser:

```bash
/docsmith init                 # works
/docsmith module myfeature     # works
/docsmith draft myfeature      # works
/docsmith edit myfeature       # works
/docsmith translate myfeature  # works
/docsmith deploy --dry-run     # works
```

What WON'T work:
- `walkthrough` (will warn "no browser available, skipping")
- `record` (same)
- Screenshot placeholders in drafts will remain as placeholders

You can fill screenshots manually later: capture them yourself, save to `documentation/images/<module>/<asset>.png`, replace `{{screenshot:...}}` placeholders in drafts. Then `/docsmith verify` to confirm no orphan placeholders.

---

## Recap: the simplest path forward

For most users on a personal machine:

1. Install Claude in Chrome from https://claude.ai/chrome (5 min)
2. `/mcp list` in Claude Code to verify (10 sec)
3. Set env vars for whatever sources/credentials you'll use:
   ```bash
   export NOTION_TOKEN=secret_...
   export MYCLOUD_TEST_USER=qa@example.com
   export MYCLOUD_TEST_PASS=...
   ```
4. Run docsmith normally:
   ```bash
   /docsmith init
   /docsmith module myfeature
   # edit intake files
   /docsmith run myfeature
   ```

That's it. Only add ffmpeg if you actually need video recording.
