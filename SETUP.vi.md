# Setup Guide — Chuẩn bị môi trường cho Walkthrough & Record

`walkthrough` (verify doc với product live, chụp screenshot) và `record` (quay video tutorial) cần browser automation tools mà docsmith không tự cài. Hướng dẫn này dẫn bạn setup trước khi chạy `wt` / `record` lần đầu.

> 🇬🇧 English version: [SETUP.md](SETUP.md)

## TL;DR

| Bạn muốn làm gì                        | Cần gì                                                            |
| -------------------------------------- | ----------------------------------------------------------------- |
| Chỉ viết doc, skip product verify      | Không cần gì thêm. `init`, `module`, `draft`, `edit`, `translate`, `deploy` chạy không cần browser. |
| Chụp screenshot (`walkthrough`)        | Claude in Chrome extension (recommend) HOẶC Playwright MCP server |
| Quay video tutorial (`record`)         | Như walkthrough + ffmpeg                                          |
| Fetch từ Notion / GDrive / GitHub      | Auth token set vào env vars (không cần install)                   |

Mới bắt đầu thì chọn **Path 1: Claude in Chrome** bên dưới. Dễ nhất.

---

## Path 1: Claude in Chrome extension (recommend cho người mới)

Đây là extension chính thức của Anthropic cho Claude điều khiển Chrome. Hợp với `walkthrough` của docsmith.

### 1. Cài Claude in Chrome

1. Mở Chrome (hoặc Chromium / Brave / Edge — bất cứ Chromium-based browser)
2. Vào https://claude.ai/chrome
3. Click "Add to Chrome" → "Add extension"
4. Login bằng tài khoản Anthropic nếu được hỏi
5. Pin extension lên toolbar (icon puzzle → pin Claude)

### 2. Verify extension hoạt động

1. Click icon Claude trên toolbar
2. Side panel Claude hiện ra
3. Gõ "What page am I on?" — Claude trả lời với URL hiện tại
4. Work là OK

### 3. Connect Claude Code với extension

Extension chạy như MCP server, Claude Code auto-discover. Verify:

```bash
# Trong Claude Code
/mcp list
```

Nên thấy `claude-in-chrome` (hoặc tương tự) trong list. Không thấy → restart Chrome và Claude Code.

### 4. Test browser automation

Trong Claude Code:

```
Open https://example.com in a tab and tell me what's on the page
```

Claude mở tab qua extension và trả lời content. Đó là proof integration work.

### 5. Sẵn sàng

Giờ `/docsmith wt` sẽ chạy. Skill biết cách gọi browser tools qua extension.

**Limitations**:
- Chỉ chạy được trên máy có Chrome cài
- Chrome phải đang chạy lúc walkthrough
- Không headless được (Chrome phải hiện ra)

---

## Path 2: Playwright MCP server (cho headless / CI)

Nếu cần walkthrough chạy trên server không có GUI (CI/CD, remote dev), dùng Playwright thay vì Claude in Chrome.

### 1. Cài Playwright

```bash
# Cần Node.js (>=18)
npm install -g @playwright/mcp
npx playwright install chromium
```

Download ~200MB cho Chromium binary.

### 2. Cấu hình Claude Code dùng Playwright MCP

Thêm vào MCP config của Claude Code (path tùy platform; thường là `~/.claude/mcp.json` hoặc qua `/mcp add`):

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

Restart Claude Code. Verify với `/mcp list` — nên thấy `playwright`.

### 3. Test

```
Use Playwright to open https://example.com and screenshot it
```

Get screenshot lại = work.

### 4. Headed mode (nếu cần xem browser)

Mặc định Playwright headless. Local dev có thể chuyển headed:

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

### 5. Sẵn sàng

`/docsmith wt` dùng Playwright thay Claude in Chrome.

**Limitations**:
- Setup overhead nhiều hơn
- Browser fingerprint khác Chrome thật chút (vài product detect và block automation)
- Cần Node.js cài sẵn

---

## Path 3: Cho `record` (video tutorial)

Quay video cần mọi thứ ở Path 1 hoặc 2 CỘNG video encoder.

### 1. Cài ffmpeg

ffmpeg capture browser viewport và encode ra mp4.

**macOS**:
```bash
brew install ffmpeg
```

**Ubuntu / Debian**:
```bash
sudo apt update && sudo apt install ffmpeg
```

**Windows**:
- Download từ https://www.gyan.dev/ffmpeg/builds/
- Extract và thêm `bin/` vào PATH

Verify:
```bash
ffmpeg -version
```

Nên print version 5.0+ cho compatibility tốt nhất với browser screen recording.

### 2. Browser support screen capture API

Chrome/Firefox/Edge hiện đại support `getDisplayMedia()` native. Nếu dùng:
- **Claude in Chrome**: work ngay
- **Playwright**: pass `--enable-features=ScreenCaptureKit` cho Chromium args

### 3. Permissions

Lần đầu `record` chạy, browser hỏi:
- Screen / window / tab capture permission → grant
- Microphone (nếu voiceover) → grant hoặc skip

Permission được nhớ cho test product URL sau lần grant đầu.

### 4. Test

```bash
/docsmith record --test
```

(nếu implemented; otherwise thêm test marker vào draft và chạy `/docsmith record`)

Output expected: `documentation/videos/<asset-id>.mp4` ~10-90 giây.

### 5. Storage

Recording raw có thể to (10-50 MB / phút ở 1080p). docsmith tự động:
- Save raw vào `documentation/videos/raw/` (gitignored)
- Encode `.mp4` cuối ra `documentation/videos/<asset-id>.mp4` ở bitrate thấp hơn
- File cuối thường 1-5 MB / video

Add `documentation/videos/raw/` vào `.gitignore` nếu chưa (init làm rồi).

---

## Environment variables cho sources và credentials

`fetch` và `walkthrough` cần credentials. ĐỪNG bao giờ để trong intake file — chỉ TÊN env var thôi.

### Set env vars

**macOS / Linux** (tạm thời, chỉ shell hiện tại):
```bash
export NOTION_TOKEN="secret_..."
export GITHUB_TOKEN="ghp_..."
export GOOGLE_DRIVE_TOKEN="ya29..."
export MYCLOUD_TEST_USER="qa@mycloud.test"
export MYCLOUD_TEST_PASS="your-test-password"
```

**macOS / Linux** (vĩnh viễn, thêm vào shell rc):
```bash
# Thêm vào ~/.zshrc hoặc ~/.bashrc
echo 'export NOTION_TOKEN="secret_..."' >> ~/.zshrc
source ~/.zshrc
```

**Windows PowerShell**:
```powershell
$env:NOTION_TOKEN = "secret_..."
# Vĩnh viễn:
[System.Environment]::SetEnvironmentVariable('NOTION_TOKEN', 'secret_...', 'User')
```

### Per-project `.env` (recommend cho team)

Dùng file `.env` ở project root (gitignored):

```bash
# .env (KHÔNG COMMIT)
NOTION_TOKEN=secret_...
GITHUB_TOKEN=ghp_...
MYCLOUD_TEST_USER=qa@mycloud.test
MYCLOUD_TEST_PASS=...
```

Trước khi chạy docsmith:
```bash
# Source nó
source .env
# Hoặc qua direnv (auto-load khi cd):
direnv allow
```

Add `.env` vào `.gitignore`:
```
echo ".env" >> .gitignore
```

### Lấy token cho từng source thế nào

#### Notion (NOTION_TOKEN)

1. Vào https://www.notion.so/my-integrations
2. Click "+ New integration"
3. Đặt tên (vd "docsmith-fetch"), select workspace, internal type
4. Copy "Internal Integration Token" (bắt đầu với `secret_`)
5. Vào Notion page bạn muốn docsmith đọc → click "..." top-right → "Add connections" → chọn integration của bạn
6. Page giờ accessible cho docsmith qua token

Notion permissions theo từng page. Phải explicit share mỗi page.

#### GitHub (GITHUB_TOKEN)

Cho **public repos**: không cần token. Để trống auth env var trong intake.

Cho **private repos**:

1. Vào https://github.com/settings/tokens?type=beta
2. "Generate new token" → fine-grained PAT
3. Repository access: chọn các repo cụ thể docsmith sẽ fetch
4. Permissions:
   - Contents: Read-only
   - Metadata: Read-only (auto-included)
5. Copy token (bắt đầu với `github_pat_`)

Cho org repos: nhờ org admin install GitHub App hoặc generate org-scoped PAT.

#### Google Drive (GOOGLE_DRIVE_TOKEN)

2 options:

**Option A: Service account** (tốt cho production):
1. Vào https://console.cloud.google.com → tạo project
2. Enable "Google Drive API"
3. Tạo service account → download JSON key
4. Set `GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json`
5. Share GDrive doc với email của service account

**Option B: OAuth2 user token** (dễ hơn cho personal):
1. Dùng https://developers.google.com/oauthplayground
2. Select Drive API v3 → readonly scope
3. Authorize → exchange lấy access token (lưu ý: hết hạn sau 1 giờ, cần refresh)

Option A recommend cho production. Option B cho test.

#### Product walkthrough credentials

Tùy product. Common patterns:

```bash
export MYCLOUD_TEST_USER="qa-bot@example.com"
export MYCLOUD_TEST_PASS="..."

# Nếu 2FA, thường skip qua test account config. Vài setup:
export MYCLOUD_TEST_TOTP_SECRET="..."  # nếu TOTP-based 2FA
export MYCLOUD_TEST_BACKUP_CODE="..."  # nếu backup code

# Nếu SSO, walkthrough có thể cần session cookie:
export MYCLOUD_TEST_SESSION_COOKIE="session=abc123..."
```

Dùng **test account riêng** với permissions giới hạn. Đừng dùng tài khoản production thật.

---

## Pre-walkthrough checklist

Trước khi chạy `/docsmith wt` lần đầu, verify:

- [ ] Browser extension hoặc Playwright cài (Path 1 hoặc 2)
- [ ] `/mcp list` trong Claude Code thấy browser tool
- [ ] Test browser automation work (mở example.com)
- [ ] Test account credentials set vào env vars
- [ ] Test account thực sự login được product (test thủ công trước)
- [ ] Nếu 2FA: test account đã disable 2FA HOẶC có backup code
- [ ] Product URL accessible từ máy bạn (không bị VPN block)
- [ ] Drafts có screenshot placeholders (`{{screenshot:...}}`) hoặc video markers (`<!-- VIDEO ... -->`)

`walkthrough` vẫn fail → chạy `/docsmith wt --check --verbose` xem step nào break.

## Pre-record checklist

Như walkthrough cộng:

- [ ] ffmpeg cài (`ffmpeg -version` work)
- [ ] Có ít nhất 1 `<!-- VIDEO ... -->` marker trong drafts
- [ ] Browser allow screen capture (test: trong Claude Code, "use the browser to record this tab for 5 seconds")
- [ ] Disk space cho raw recordings (~50 MB / phút)

---

## Lỗi phổ biến và cách fix

### "No browser tool available"

Nguyên nhân: cả Claude in Chrome lẫn Playwright MCP đều chưa connect.

Fix: cài Path 1 hoặc Path 2. Chạy `/mcp list` confirm.

### "Cannot click element: not found"

Nguyên nhân: product UI đổi hoặc selector cũ.

Fix: chạy `/docsmith wt --check` để detect drift. Update test cases hoặc accept new UI.

### "Login failed"

Nguyên nhân: credentials env var chưa set HOẶC test account bị lock.

Fix: 
```bash
echo $MYCLOUD_TEST_USER  # nên print, không empty
```
Empty thì re-export. Account lock thì unlock qua product UI.

### "Notion page not found" / 404

Nguyên nhân: page chưa share với integration.

Fix: trong Notion, "..." → "Add connections" → chọn integration của bạn.

### "ffmpeg: command not found"

Nguyên nhân: ffmpeg không trong PATH.

Fix: cài ffmpeg theo OS instructions trên. Verify với `which ffmpeg`.

### "Headless browser can't access webcam/screen"

Nguyên nhân: cố quay video ở headless mode.

Fix: switch Playwright sang `--headed` HOẶC dùng Claude in Chrome (luôn headed).

### "Walkthrough captures wrong locale"

Nguyên nhân: product UI chưa switch sang ngôn ngữ expected cho `wt --locale vi`.

Fix: pre-walkthrough setup script switch language:
```bash
# Trong "Pre-walkthrough setup script" của module intake:
# (chạy trước khi browser mở)
mycloud user prefs set --locale vi
```

Hoặc làm thủ công trong browser trước khi chạy `wt --locale vi`.

---

## Network / firewall

`fetch` và `walkthrough` cần outbound HTTPS đến:

| Service                 | Domain                                      |
| ----------------------- | ------------------------------------------- |
| Notion API              | `api.notion.com`                            |
| GitHub API              | `api.github.com`, `raw.githubusercontent.com` |
| Google Drive API        | `www.googleapis.com`                        |
| Product của bạn         | `console.your-product.com` (whatever)       |
| Anthropic (Claude API)  | `api.anthropic.com`                         |
| Browser extension auth  | `claude.ai`, `anthropic.com`                |

Nếu sau corporate firewall, request allowlist các domain này. docsmith KHÔNG cần inbound port nào.

---

## Performance tuning

**Walkthrough chậm** (>5 phút / module):

- Giảm screenshot count: tighten module intake `Features to document` (ít feature / module hơn)
- Dùng `wt --check` (không capture, fast drift detect) cho monitor hằng ngày; chỉ chạy full `wt` weekly
- Chạy trên máy mạnh hơn (browser automation CPU-bound)

**Source fetch chậm** (>1 phút / Notion page):

- Notion API rate-limit 3 req/s. Multiple sources fetch tuần tự.
- Cache: `documentation/.cache/sources/` reuse content nếu hash match. Đừng xóa cache giữa các run.
- Dùng `update` command để fetch chỉ source đã đổi, không phải tất cả.

**Translate chậm** (>2 phút / file):

- Per-block mode chậm nhất (mỗi block = 1 LLM call). Dùng `batch` mode (default) cho normal flow.
- Dùng `--auto-approve` chỉ sau khi build glossary tốt

**Browser automation flaky**:

- Đa số issue là timing. Tăng wait time trong test cases (intake → walkthrough setup → wait_for selector)
- Dùng deterministic test data (đừng phụ thuộc dynamic ID đổi mỗi session)

---

## Optional: docsmith không cần browser

Nếu chỉ muốn draft và translate (không cần product verification, không cần screenshot), skip Path 1-3. Pipeline chạy không cần browser:

```bash
/docsmith init                 # work
/docsmith module myfeature     # work
/docsmith draft myfeature      # work
/docsmith edit myfeature       # work
/docsmith translate myfeature  # work
/docsmith deploy --dry-run     # work
```

Cái KHÔNG work:
- `walkthrough` (warn "no browser available, skipping")
- `record` (same)
- Screenshot placeholders trong drafts vẫn là placeholders

Có thể fill screenshot thủ công sau: tự capture, save vào `documentation/images/<module>/<asset>.png`, replace `{{screenshot:...}}` placeholders trong drafts. Sau đó `/docsmith verify` confirm không còn orphan placeholder.

---

## Tổng kết: con đường đơn giản nhất

Cho đa số user trên máy cá nhân:

1. Cài Claude in Chrome từ https://claude.ai/chrome (5 phút)
2. `/mcp list` trong Claude Code verify (10 giây)
3. Set env vars cho source/credentials sẽ dùng:
   ```bash
   export NOTION_TOKEN=secret_...
   export MYCLOUD_TEST_USER=qa@example.com
   export MYCLOUD_TEST_PASS=...
   ```
4. Chạy docsmith bình thường:
   ```bash
   /docsmith init
   /docsmith module myfeature
   # edit intake files
   /docsmith run myfeature
   ```

Hết. Chỉ thêm ffmpeg nếu thực sự cần quay video.
