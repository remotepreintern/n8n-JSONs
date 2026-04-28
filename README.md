# n8n-JSONs

A collection of ready-to-import n8n workflow JSON files for social media automation.

---

## Workflows

### 1. `Social_Media_Video_Publisher.json` — v1 (Baseline)

Original workflow: Google Drive trigger → AI content → parallel upload to 10 platforms (Facebook, Instagram, YouTube, TikTok). Error notification via email.

### 2. `Social_Media_Video_Publisher_v2.json` — v2 (Recommended)

Full-featured workflow with all improvements:

| Feature | Details |
|---------|---------|
| WhatsApp Approval Gate | Sends video preview + AI content to WhatsApp before uploading. Tap APPROVE or CANCEL. Auto-proceeds after 2 minutes. |
| Sequential Uploads | All 10 platforms run in order [1/10] to [10/10] with per-platform status tracking. |
| WhatsApp Final Report | Formatted report showing success/failure per platform, error details, completion time (NPT). |
| WhatsApp Error Alerts | All errors sent to WhatsApp instead of email. |
| Completed Folder | Files automatically moved to a "Completed" subfolder after all uploads. |
| Google Sheets Log | Appends a row per video with per-platform results, timestamp, file ID. |
| Folder Structure | Parent > Upload Folder (watched) + Completed Folder (destination) |
| Multiple Videos | Each new file triggers a separate independent execution automatically. |

#### Google Drive Folder Structure

```
Parent Folder
  Upload Folder     <- Drop videos here (n8n watches this)
  Completed Folder  <- n8n moves files here after upload
```

#### WhatsApp Approval Flow

```
New video detected in Upload Folder
  -> AI generates captions/descriptions for all 10 platforms
  -> WhatsApp preview sent to +977 9845670262:
       File: video.mp4
       Preview link
       Content summary
       [APPROVE link]  <- tap to start upload
       [CANCEL link]   <- tap to stop
       Auto-proceeds in 2 min if no response
  -> Uploads run sequentially [1/10] through [10/10]
  -> Final report sent via WhatsApp with per-platform status
  -> File moved to Completed Folder
  -> Row added to Google Sheet
```

#### Platforms Covered

| # | Platform | Method |
|---|----------|--------|
| 1 | Facebook Page Post | URL-based |
| 2 | Facebook Reels | 3-phase binary upload |
| 3 | Facebook Story | 3-phase binary upload |
| 4 | Instagram Post | Container > wait > publish |
| 5 | Instagram Reels | Container > wait > publish |
| 6 | Instagram Story | Container > wait > publish |
| 7 | YouTube Video | Resumable upload |
| 8 | YouTube Shorts | Resumable upload (#Shorts auto-tag) |
| 9 | TikTok Video | Init > upload |
| 10 | TikTok Story | Init > upload (requires TikTok approval) |

> YouTube Stories were discontinued by Google in June 2023 and are not included.

#### Required Credentials (7 total)

| Credential | Type |
|-----------|------|
| Google Drive OAuth2 | Google Drive OAuth2 API |
| OpenAI API Key | HTTP Header Auth |
| Facebook Page Access Token | HTTP Header Auth |
| YouTube OAuth2 | OAuth2 (Google, youtube.upload scope) |
| TikTok Access Token | HTTP Header Auth |
| WhatsApp Cloud API Token | HTTP Header Auth |
| Google Sheets OAuth2 | Google Sheets OAuth2 API |

---

## Can Mega.nz Replace Google Drive for Instagram?

**Short answer: No — Mega.nz does not work for Instagram video URLs.**

Instagram requires a publicly accessible plain HTTPS URL to fetch the video during the container creation step. Mega.nz is technically incompatible because:

1. **Encrypted links**: Mega uses end-to-end encryption with the decryption key embedded in the URL fragment (#). HTTP servers cannot read fragments, so Instagram cannot access the file.
2. **No direct download URL**: Mega requires its browser extension or SDK to decrypt and stream files. There is no plain https:// download URL.
3. **No official API**: Mega has no public REST API for generating usable direct links.
4. **No n8n node**: Mega has no native n8n integration.

Mega.nz free tier is 20 GB, but the above limitations make it unsuitable for this use case.

### Alternatives That Work for Instagram

| Option | Free? | Notes |
|--------|-------|-------|
| Google Drive (current) | Yes | Works when file is shared publicly (already handled by workflow) |
| Cloudinary | Yes (25 GB) | Best option — native CDN URL, no auth needed |
| Google Cloud Storage | Free trial | Public bucket gives a direct URL |
| Amazon S3 | Limited free tier | Pre-signed URLs work |
| Bunny CDN | Paid | Very fast CDN delivery |

**Recommendation**: Keep Google Drive (already configured). If Instagram rejects the Drive URL, upload to Cloudinary first and use its CDN URL in the video_url field of the Instagram container nodes.

---

## How to Import

1. Open your n8n instance
2. Go to **Workflows > Add Workflow > Import from File**
3. Select the `.json` file
4. Replace all placeholders (see the SETUP GUIDE sticky note inside the workflow)
5. Add credentials under **Settings > Credentials**
6. Activate the workflow
