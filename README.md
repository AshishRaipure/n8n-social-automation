# 🎬 n8n Social Media Automation
### Daily Instagram & YouTube Video Poster with WhatsApp Notifications

[![n8n](https://img.shields.io/badge/n8n-Workflow-EA4B71?logo=n8n)](https://n8n.io)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Docker-2496ED?logo=docker)](https://docker.com)

> **Fully automated daily video posting from Google Drive → Instagram Reels + YouTube, with WhatsApp success/failure notifications. Set it once, forget it.**

---

## ✨ Features

| Feature | Details |
|---------|---------|
| ⏰ **Daily Schedule** | Posts one video every day at **9:00 AM IST** automatically |
| 📁 **Google Drive Source** | Pulls videos sequentially from your Google Drive folder |
| 📸 **Instagram Reels** | Posts as Instagram Reels via Graph API |
| ▶️ **YouTube Upload** | Uploads to YouTube with SEO-optimized title & description |
| ✍️ **Smart Captions** | Auto-generates engaging captions with strategic hashtags |
| 📱 **WhatsApp Success Alert** | Notifies you on WhatsApp when posting succeeds |
| 🚨 **Deadline Enforcement** | Checks at 10 AM IST — sends WhatsApp if post was missed |
| 📊 **Google Sheets Tracker** | Logs every post, status, and links automatically |
| 🔄 **No-Repeat Logic** | Tracks posted videos to avoid duplicates |
| ♻️ **Error Recovery** | Catches all errors and alerts you via WhatsApp |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      WORKFLOW 1: MAIN POSTER                    │
│                         (9:00 AM IST)                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⏰ Schedule → 📊 Google Sheets → 📁 Google Drive               │
│                      │                   │                      │
│                       ──────────────────                        │
│                              │                                  │
│                    🧠 Pick Next Video                            │
│                              │                                  │
│                    ⬇️ Download Video                             │
│                              │                                  │
│                    ✍️ Generate Caption                           │
│                           ┌──┴──┐                               │
│                    ▶️ YouTube  📸 Instagram                      │
│                           └──┬──┘                               │
│                    📊 Update Tracker                             │
│                              │                                  │
│                    📱 WhatsApp Success                           │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                   WORKFLOW 2: DEADLINE CHECKER                   │
│                         (10:00 AM IST)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⏰ Schedule → 📊 Sheets → 🔍 Check Posted?                     │
│                                    │                            │
│                          ┌─────────┴─────────┐                 │
│                    ✅ Yes: Send OK         ❌ No: Send Report  │
│                    📱 Success Alert        📱 Failure Report    │
│                    📊 Log to Sheets        📊 Log Missed        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Prerequisites

Before you start, you need accounts on these platforms:

- ✅ **[n8n](https://n8n.io)** — The automation engine (self-hosted via Docker)
- ✅ **[Google Cloud](https://console.cloud.google.com)** — For Drive + Sheets + YouTube APIs
- ✅ **[Google Drive](https://drive.google.com)** — Where your videos are stored
- ✅ **[Google Sheets](https://sheets.google.com)** — For tracking post history
- ✅ **[Facebook Developers](https://developers.facebook.com)** — For Instagram API
- ✅ **Instagram Business Account** — Connected to a Facebook Page
- ✅ **YouTube Channel** — With verified account
- ✅ **WhatsApp** — For CallMeBot notifications (free!)
- ✅ **[Docker Desktop](https://www.docker.com/products/docker-desktop/)** — To run n8n

---

## 🚀 Quick Start (Step-by-Step)

### Step 1: Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/n8n-social-automation.git
cd n8n-social-automation
```

### Step 2: Set Up Environment Variables
```bash
# Copy the example file
cp .env.example .env

# Edit with your values (use any text editor)
notepad .env   # Windows
nano .env      # Linux/Mac
```

Fill in all values. See [docs/api_setup.md](docs/api_setup.md) for detailed setup instructions.

### Step 3: Set Up Google Sheets Tracker

Create a new Google Spreadsheet with **3 sheets** (exact names matter!):

**Sheet 1: `Tracker`**
Add these column headers in Row 1:
```
Video ID | Video Name | YouTube Title | YouTube Video ID | Instagram Post ID | Posted Date | Status | Instagram URL | YouTube URL | Posted At
```

**Sheet 2: `MissedDeadlines`**
```
Date | Checked At | Minutes Late | Status | Action Taken | Total Posted All Time
```

**Sheet 3: `DailyReport`**
```
Date | Status | Video Name | YouTube URL | Instagram URL | Posted At | Total Posted | 10AM Check
```

### Step 4: Set Up WhatsApp CallMeBot (2 minutes!)
1. Save **+34 644 59 35 89** as a WhatsApp contact (name: "CallMeBot")
2. Send: `I allow callmebot to send me messages`
3. You'll get your API key in ~2 minutes
4. Add it to your `.env` file

### Step 5: Prepare Your Videos
1. Create a folder in Google Drive named `Daily Post Videos`
2. Upload your MP4 videos
3. **Name them numerically** for ordered posting:
   ```
   001_intro_video.mp4
   002_tutorial_day1.mp4
   003_behind_scenes.mp4
   ```
4. Copy the folder ID from the URL

### Step 6: Start n8n
```bash
docker-compose up -d
```

Wait 30 seconds, then open: **http://localhost:5678**

Login with:
- Username: `admin`
- Password: (what you set in `.env`)

### Step 7: Set Up API Credentials in n8n
See [docs/api_setup.md](docs/api_setup.md) for:
- Google OAuth2 setup
- YouTube credential
- Instagram token setup

### Step 8: Import Workflows
1. In n8n, go to **Workflows → Import from File**
2. Import `workflows/main_social_poster.json`
3. Import `workflows/deadline_checker.json`
4. Open each workflow and attach your credentials to the nodes

### Step 9: Configure Workflow Environment Variables
In n8n, go to **Settings → Variables** and add:
```
GOOGLE_DRIVE_FOLDER_ID = your_folder_id
TRACKER_SHEET_ID = your_sheet_id
INSTAGRAM_ACCOUNT_ID = your_instagram_id
INSTAGRAM_ACCESS_TOKEN = your_token
WHATSAPP_PHONE = 91XXXXXXXXXX
CALLMEBOT_API_KEY = your_key
N8N_URL = http://localhost:5678
```

### Step 10: Activate Both Workflows
1. Open each workflow
2. Click the **toggle** in the top-right to **activate**
3. Both workflows should show green "Active" status

✅ **You're done! Videos will start posting automatically at 9 AM IST.**

---

## 📱 WhatsApp Notification Examples

### ✅ Success Notification (9 AM IST area)
```
✅ Daily Post SUCCESS! 🎉

📅 Date: 03/03/2026
🎬 Video: My Amazing Tutorial

▶️ YouTube: https://youtube.com/watch?v=abc123
📸 Instagram: Posted Successfully

📊 Queue: 47 videos remaining
⏰ Posted at: 9:02:34 AM IST
```

### 📊 10 AM IST Confirmation Report
```
✅ 10AM IST Check — All Good! 📊

📅 Date: 03/03/2026
🎬 Video: My Amazing Tutorial
▶️ YouTube: https://youtube.com/watch?v=abc123
📸 Instagram: https://instagram.com/p/xyz789/

⏰ Posted at: 9:02:34 AM IST
📊 Total videos posted so far: 15
```

### 🚨 Missed Deadline Report (10 AM IST)
```
🚨 DEADLINE MISSED REPORT 🚨

📅 Date: 03/03/2026
🕐 Checked at: 10:00:35 AM IST
⏱️ Minutes past deadline: 0 min

❌ STATUS: NOT POSTED

📊 Statistics:
• Total videos posted (all time): 14
• Last posted date: 02/03/2026
• Last video: Day 14 Workout

🔧 Action Required:
1. Check n8n workflow logs
2. Verify API credentials are valid
3. Check Google Drive folder has videos
4. Manually trigger workflow if needed
```

---

## 💡 Caption & Hashtag Strategy

The automation generates **5 rotating caption templates** designed for maximum engagement:

- 🔥 **Emotional hook** captions that drive comments
- 📌 **Save-worthy** content prompts
- 🏷️ **70+ strategic hashtags** covering:
  - Viral tags (`#viral #trending #fyp #foryou`)
  - Engagement tags (`#follow #like #share #comment`)
  - Niche growth tags (`#contentcreator #dailycontent`)

**YouTube SEO includes:**
- Keyword-rich title with date
- Full description with timestamps-style structure
- 18+ video tags for discoverability
- Channel subscription CTA
- Social links section

---

## 📁 Project Structure

```
n8n-social-automation/
│
├── 📄 README.md                    # This file
├── 📄 docker-compose.yml           # n8n Docker setup
├── 📄 .env.example                 # Environment template
├── 📄 .gitignore                   # Protects secrets
│
├── 📂 workflows/
│   ├── 🔄 main_social_poster.json      # Main posting workflow
│   └── ⏰ deadline_checker.json        # 10 AM deadline monitor
│
└── 📂 docs/
    └── 📖 api_setup.md                 # Detailed API setup guide
```

---

## 🔧 Troubleshooting

### ❌ "Instagram container status: ERROR"
- Check if video URL from Google Drive is publicly accessible
- Ensure your Drive video is set to **"Anyone with the link can view"**
- Instagram Reels must be: MP4, H.264 codec, 500MB max, 15s-90s duration

### ❌ "YouTube upload failed"
- Verify YouTube channel is not restricted
- Check quota limits (10,000 units/day on free tier)
- Ensure video is under 128GB and 12 hours long

### ❌ "WhatsApp message not received"
- Test URL manually in browser (see [api_setup.md](docs/api_setup.md))
- Verify phone format: country code + number, no spaces, no `+`
- Check CallMeBot status: callmebot.com

### ❌ "All videos posted" alert received
- Add more videos to your Google Drive folder
- The system will pick them up on the next run

### ❌ Instagram Token expired
- Instagram long-lived tokens expire in **60 days**
- Visit developers.facebook.com to generate a new one
- Update `INSTAGRAM_ACCESS_TOKEN` in n8n Environment Variables

---

## 🔄 Routine Maintenance

| Task | Frequency | Action |
|------|-----------|--------|
| Add new videos | Weekly | Upload to Google Drive folder |
| Refresh Instagram token | Every 50 days | Generate new token at developers.facebook.com |
| Check n8n logs | Weekly | View execution history in n8n |
| Check tracker sheet | Weekly | Verify posts are being recorded |
| n8n updates | Monthly | `docker pull n8nio/n8n:latest && docker-compose up -d` |

---

## 💰 Monetization & Engagement Tips

### YouTube Monetization
- ✅ **1,000 subscribers + 4,000 watch hours** to join YouTube Partner Program
- The automation posts daily which accelerates this naturally
- Consistent posting signals to YouTube algorithm = more recommendations

### Instagram Monetization
- ✅ **Target 10,000 followers** → Unlock Instagram Shopping + paid partnerships
- ✅ Use **Instagram Creator Marketplace** for brand deal opportunities
- ✅ High Reel engagement → Instagram Reels Play Bonus (available in some regions)

### Strategy Tips Built Into This Automation
1. **Rotating captions** keep your content fresh and avoid shadow-banning
2. **70+ hashtags** per post = maximum discoverability
3. **Daily posting** builds the algorithm trust on both platforms
4. **Consistent timing** (9 AM IST) trains your audience when to expect content

---

## 🛡️ Security Notes

- ✅ `.env` is in `.gitignore` — your secrets are never committed
- ✅ n8n runs behind basic auth
- ✅ All credentials stored encrypted in n8n's credential store
- ⚠️ Never share your `.env` file or access tokens publicly
- ⚠️ Rotate your `N8N_ENCRYPTION_KEY` if you suspect compromise

---

## 📄 License

MIT License — Free to use and modify for personal and commercial purposes.

---

## 🤝 Contributing

PRs welcome! Ideas for improvement:
- [ ] TikTok posting support
- [ ] AI-powered caption generation (OpenAI/Gemini)
- [ ] Thumbnail auto-generation for YouTube
- [ ] Instagram Story posting (in addition to Reels)
- [ ] Analytics tracking (views, likes, comments)

---

*Built with ❤️ using [n8n](https://n8n.io) — the powerful open-source workflow automation tool*
