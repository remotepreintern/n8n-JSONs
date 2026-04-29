# n8n-JSONs

Ready-to-import n8n workflow JSON files for automated social media video publishing.

---

## Workflows

### v1 — `Social_Media_Video_Publisher.json`
Baseline: Google Drive trigger → AI content → **parallel** upload → email on error.

### v2 — `Social_Media_Video_Publisher_v2.json`
Adds: WhatsApp approval gate, sequential [1/10]→[10/10] uploads, Drive move, Sheets log, WhatsApp errors.
> ⚠️ Sequential chain: if one platform fails completely, platforms behind it may not run.

### v3 — `Social_Media_Video_Publisher_v3.json` ✅ Latest — Recommended
Full professional setup with smart parallelism, AI fallback, and complete configurations.

---

## v3 Features

| Feature | Details |
|---------|---------|
| **Parallel uploads** | All 10 platforms start simultaneously — one failure never blocks others |
| **AI fallback chain** | OpenRouter (free) → Gemini (free) → OpenAI (paid) — auto-picks first valid response |
| **Full error isolation** | Every HTTP node: `onError: continueRegularOutput`. Every Code node: try/catch wrapper |
| **CAP status nodes** | Each branch ends with a Capture node — always emits exactly 1 status item |
| **Platform Merge** | 10-input Merge collects all results before final report is compiled |
| **Meta API v24.0** | All Facebook/Instagram endpoints use the latest API version |
| **WhatsApp approval** | Preview + content → tap APPROVE/CANCEL → 2-min auto-proceed |
| **WhatsApp final report** | Formatted [1/10]→[10/10] ordered list with ✅/❌ + error details |
| **Drive folder move** | File automatically moved to Completed folder after all uploads |
| **Google Sheets log** | Row per video: per-platform results + timestamp + overall status |
| **Multiple videos** | Each file gets its own independent parallel execution |
| **Free forever** | OpenRouter + Gemini free tiers are the primary AI providers |

---

## AI Models — Free vs Paid

### Recommended: OpenRouter (100% free, no credit card)
Sign up at [openrouter.ai](https://openrouter.ai) → Create API Key.

| Model | Quality | Notes |
|-------|---------|-------|
| `google/gemini-2.5-pro-exp-03-25:free` | ⭐⭐⭐⭐⭐ | **Best overall** — creative, long context |
| `google/gemini-2.0-flash-exp:free` | ⭐⭐⭐⭐ | Fast, reliable |
| `meta-llama/llama-4-scout:free` | ⭐⭐⭐⭐ | Good JSON output |
| `mistralai/mistral-7b-instruct:free` | ⭐⭐⭐ | Lightweight fallback |

### Backup: Google Gemini (free tier — 1,500 req/day)
Get API key at [aistudio.google.com](https://aistudio.google.com).
Model: `gemini-2.0-flash`

### Optional Upgrade: OpenAI (paid but cheap)
`gpt-4o-mini` at ~$0.002 per video. Best quality if budget allows.

### Workflow Fallback Order
```
1. OpenRouter (free) — tried first
2. Gemini (free)     — if OpenRouter fails/times out
3. OpenAI (paid)     — if Gemini also fails
4. Built-in defaults — if ALL AIs fail (still uploads, uses generic captions)
```

---

## v3 Architecture

```
📁 New Video in Upload Folder
    ↓
🔍 Check: Is it a video file?
    ↓
⚙️ SET: Extract file info + config IDs
    ↓
🔓 Make file publicly accessible on Drive
    ↓ (3 parallel AI calls)
🤖 OpenRouter ──┐
🤖 Gemini ──────┤→ 🔀 AI Merge → 🧠 Pick Best Response
🤖 OpenAI ──────┘
    ↓
📥 Download video from Drive
    ↓
🧩 Assemble final data (AI content + file info + binary)
    ↓
📲 Build WhatsApp preview message
    ↓
📲 Send preview + APPROVE/CANCEL links to +977 9845670262
    ↓
⏳ Wait for reply (max 2 minutes, then auto-proceed)
    ↓
🔒 Was upload rejected?
  → YES: 📲 Send "Cancelled" WA → 🛑 HALT
  → NO:  Fan out to all 10 platforms simultaneously
    ├── [1/10] 📘 Facebook Page Post
    ├── [2/10] 📘 Facebook Reels (init → upload → publish)
    ├── [3/10] 📘 Facebook Story (init → upload → publish)
    ├── [4/10] 📸 Instagram Post (container → wait 90s → publish)
    ├── [5/10] 📸 Instagram Reels (container → wait 90s → publish)
    ├── [6/10] 📸 Instagram Story (container → wait 60s → publish)
    ├── [7/10] ▶️ YouTube Video (resumable init → prep → upload)
    ├── [8/10] ▶️ YouTube Shorts (resumable init → prep → upload)
    ├── [9/10] 🎵 TikTok Video (init → prep → upload)
    └── [10/10] 🎵 TikTok Story (init → prep → upload)
         ↓ each branch ends with 📊 CAP status node
    → 🔀 Platform Merge (waits for all 10)
    ↓
📊 Compile Final Report
    ↓
📲 Send WhatsApp Report (ordered ✅/❌ per platform + error details)
    ↓
📁 Move file to Completed Folder
    ↓
📊 Append row to Google Sheets
```

---

## Error Isolation (How Parallel Works Safely)

**The problem with sequential:** if platform 5 has a code error, platforms 6-10 never run.

**v3 solution — 3 layers of isolation:**
1. **`onError: continueRegularOutput`** on every HTTP node — API failures produce an error item instead of stopping the branch
2. **try/catch** in every Code node — code errors return an error status item instead of throwing
3. **Independent branches** — each platform runs in its own n8n execution branch, completely isolated

Even if a branch completely fails, the `CAP` (Capture) node at the end of each branch always outputs exactly 1 item with `{ok: false, error: "..."}`. The Platform Merge node waits for all 10 CAP nodes, then the report is compiled.

---

## Setup (5 Steps)

### Step 1: Google Drive Folder Structure
```
📁 Social Media Publisher (Parent)
  📁 Upload Folder     ← n8n watches this (copy ID)
  📁 Completed Folder  ← n8n moves files here (copy ID)
```

### Step 2: Create 9 Credentials in n8n (Settings → Credentials)
Use these EXACT names so n8n auto-matches on import:

| Credential Name | Type | What to Set |
|----------------|------|-------------|
| `Google Drive account` | Google Drive OAuth2 | Complete OAuth2 flow |
| `Google Sheets account` | Google Sheets OAuth2 | Complete OAuth2 flow |
| `Meta (Facebook/Instagram) API` | HTTP Header Auth | `Authorization: Bearer YOUR_PAGE_TOKEN` |
| `YouTube account` | Google OAuth2 | Scope: youtube.upload |
| `TikTok API` | HTTP Header Auth | `Authorization: Bearer YOUR_TT_TOKEN` |
| `OpenRouter API` | HTTP Header Auth | `Authorization: Bearer sk-or-...` |
| `Google Gemini API` | HTTP Header Auth | `x-goog-api-key: YOUR_GEMINI_KEY` |
| `OpenAI API` | HTTP Header Auth | `Authorization: Bearer sk-...` |
| `WhatsApp Business API` | HTTP Header Auth | `Authorization: Bearer YOUR_WA_TOKEN` |

### Step 3: Set 6 IDs in the ⚙️ SET node
- `YOUR_UPLOAD_FOLDER_ID` — Google Drive Upload subfolder ID
- `YOUR_COMPLETED_FOLDER_ID` — Google Drive Completed subfolder ID
- `YOUR_WA_PHONE_NUMBER_ID` — From Meta Developer Console → WhatsApp → API Setup
- `YOUR_FACEBOOK_PAGE_ID` — Facebook Page numeric ID
- `YOUR_INSTAGRAM_BUSINESS_USER_ID` — Instagram Business account ID
- `YOUR_GOOGLE_SHEET_ID` + `YOUR_GOOGLE_SHEET_NAME` — Sheets report

### Step 4: Google Sheets Headers
Create a sheet with these column headers in row 1:
```
Timestamp | File | FileID | FB Post | FB Reels | FB Story | IG Post | IG Reels | IG Story | YT Video | YT Shorts | TT Video | TT Story | Overall
```

### Step 5: Activate the Workflow
Toggle the workflow to Active. Drop a video in the Upload Folder.

---

## WhatsApp Approval

When a video is detected:
1. You receive a WhatsApp message at +977 9845670262 with:
   - File name and size
   - Preview link (Google Drive)
   - Content preview (captions for each platform)
   - APPROVE link (tap to start upload)
   - CANCEL link (tap to stop)
2. **Tap APPROVE** → all 10 platforms start uploading in parallel
3. **Tap CANCEL** → workflow stops, file remains in Upload folder
4. **No reply in 2 minutes** → auto-proceeds with upload

> **Important**: Your n8n instance must be at a public HTTPS URL for the tap links to work.
> For self-hosted: configure `N8N_WEBHOOK_BASE_URL=https://your-domain.com`

---

## Can Mega.nz Replace Google Drive for Instagram?

**Short answer: No.**

Mega.nz uses end-to-end encryption with the decryption key embedded in the URL fragment (`#key`). Instagram's servers cannot read URL fragments, so they cannot access the file content. There is no plain `https://` direct download URL from Mega.

**Best alternatives if Google Drive URL is rejected by Instagram:**

| Option | Free? | Direct URL? |
|--------|-------|-------------|
| Google Drive (current) | ✅ | ✅ Works when file is public |
| **Cloudinary** | ✅ 25 GB | ✅ Best — instant CDN URL |
| Google Cloud Storage | ✅ trial | ✅ Public bucket URL |
| Amazon S3 | Limited | ✅ Pre-signed URL |

If Instagram still rejects the Drive URL, replace the `video_url` value in the IG container nodes with a Cloudinary URL.

---

## How to Import

1. Open n8n → Workflows → Add Workflow → **Import from File**
2. Select `Social_Media_Video_Publisher_v3.json`
3. Create all 9 credentials with the exact names listed above
4. Replace the 6 ID placeholders in the ⚙️ SET node
5. Activate the workflow
