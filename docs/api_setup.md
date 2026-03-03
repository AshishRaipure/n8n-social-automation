# Step-by-Step API Setup Guide

This guide walks you through setting up each API needed for the automation.

---

## 1️⃣ Google Drive & Google Sheets API

### Goal: Access videos from Drive + track posting status in Sheets

**Step 1: Create a Google Cloud Project**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click **New Project** → Name it `n8n-social-automation`
3. Click **Create**

**Step 2: Enable APIs**
1. Go to **APIs & Services → Library**
2. Search and enable:
   - **Google Drive API**
   - **Google Sheets API**

**Step 3: Create OAuth2 Credentials**
1. Go to **APIs & Services → Credentials**
2. Click **Create Credentials → OAuth 2.0 Client IDs**
3. Application type: **Web Application**
4. Name: `n8n-social-automation`
5. Authorized redirect URIs: `http://localhost:5678/rest/oauth2-credential/callback`
6. Save your **Client ID** and **Client Secret**

**Step 4: Configure in n8n**
1. Open n8n → **Credentials → New → Google Sheets OAuth2 API**
2. Enter Client ID and Client Secret
3. Click **Connect** and authorize with your Google account
4. Repeat for **Google Drive OAuth2 API**

**Step 5: Set Up the Google Sheets Tracker**
Create a new Google Spreadsheet with these exact sheet tabs:

**Sheet: `Tracker`** (columns A–J)
| Video ID | Video Name | YouTube Title | YouTube Video ID | Instagram Post ID | Posted Date | Status | Instagram URL | YouTube URL | Posted At |

**Sheet: `MissedDeadlines`** (columns A–F)
| Date | Checked At | Minutes Late | Status | Action Taken | Total Posted All Time |

**Sheet: `DailyReport`** (columns A–H)
| Date | Status | Video Name | YouTube URL | Instagram URL | Posted At | Total Posted | 10AM Check |

Copy the Spreadsheet ID from the URL and set as `TRACKER_SHEET_ID`

---

## 2️⃣ Google Drive Folder Setup

1. Create a folder in Google Drive for your videos
2. Name it: `Daily Post Videos` (or anything you like)
3. Upload your videos to this folder (MP4 format recommended)
4. Get the folder ID from the URL: `drive.google.com/drive/folders/[FOLDER_ID_HERE]`
5. Set as `GOOGLE_DRIVE_FOLDER_ID`

> ⚠️ Videos will be posted in alphabetical order. Name them like:
> - `001_my_first_video.mp4`
> - `002_second_video.mp4`
> - etc.

---

## 3️⃣ YouTube API

### Goal: Upload videos to YouTube automatically

**Step 1: Enable YouTube Data API v3**
1. In your Google Cloud Project → **APIs & Services → Library**
2. Search **YouTube Data API v3** → Enable
3. Go to the same OAuth2 credentials from Step 1
4. This credential already works for YouTube if it's the same Google account

**Step 2: Configure n8n YouTube Credential**
1. n8n → **Credentials → New → YouTube OAuth2 API**
2. Enter the same Google OAuth2 Client ID and Secret
3. **Scope required**: `https://www.googleapis.com/auth/youtube.upload`
4. Click **Connect** and authorize with your YouTube channel's Google account

**Step 3: Verify Channel Settings**
- Your YouTube channel must be verified to upload videos > 15 minutes
- Go to [YouTube Studio](https://studio.youtube.com/) → Settings → Channel → Verify

---

## 4️⃣ Instagram Graph API

### Requirements
- ✅ A **Facebook Business Account**
- ✅ An **Instagram Business or Creator Account** (NOT personal)
- ✅ Your Instagram account must be **connected to a Facebook Page**

### Step 1: Create Facebook App
1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Click **My Apps → Create App**
3. Type: **Business**
4. Add product: **Instagram Graph API**

### Step 2: Get Instagram Account ID
1. Go to **Instagram Graph API → Tools → Graph API Explorer**
2. Select your app
3. Generate a User Token with permissions:
   - `instagram_basic`
   - `instagram_content_publish`
   - `pages_show_list`
   - `pages_read_engagement`
4. Make GET request to: `https://graph.facebook.com/me/accounts`
5. Find your Page ID, then: `https://graph.facebook.com/{page-id}?fields=instagram_business_account`
6. Copy the Instagram Account ID → Set as `INSTAGRAM_ACCOUNT_ID`

### Step 3: Get Long-Lived Access Token
1. In Graph API Explorer, get a User Access Token
2. Exchange for long-lived token (60 days):
```
GET https://graph.facebook.com/oauth/access_token
  ?grant_type=fb_exchange_token
  &client_id={app-id}
  &client_secret={app-secret}
  &fb_exchange_token={short-lived-token}
```
3. Set the result as `INSTAGRAM_ACCESS_TOKEN`

> ⚠️ **Token Expires in 60 days!** Set a calendar reminder to refresh.
> Run the token refresh workflow or do it manually every 50 days.

### Step 4: Enable Video Posting (Reels)
- Instagram Reels via API requires **Business account** with **Reel posting permission**
- Apply for `instagram_content_publish` permission in your app's App Review

---

## 5️⃣ WhatsApp Setup (CallMeBot — FREE)

### This is the easiest and completely free!

**Step 1:**
1. Save this number to your WhatsApp contacts: **+34 644 59 35 89** (Name: "CallMeBot")

**Step 2:**
1. Send this EXACT message to CallMeBot on WhatsApp:
   ```
   I allow callmebot to send me messages
   ```

**Step 3:**
1. Within 2 minutes, you'll receive your API key like: `12345`
2. Set `CALLMEBOT_API_KEY=12345`
3. Set `WHATSAPP_PHONE=91XXXXXXXXXX` (your number with country code, no + or spaces)

**Test it:**
Open in browser (replace values):
```
https://api.callmebot.com/whatsapp.php?phone=919876543210&text=Test+Message&apikey=12345
```

> 💡 **CallMeBot Free Limits**: Works great for personal use notifications.
> For high-volume or business use, upgrade to **Twilio** (see README for Twilio setup).

---

## 6️⃣ Final Configuration Checklist

Before activating workflows, verify:

- [ ] Google Drive folder ID is set and has videos
- [ ] Google Sheets tracker is created with correct sheet names
- [ ] YouTube OAuth2 credential is authorized in n8n
- [ ] Instagram long-lived token is valid
- [ ] WhatsApp CallMeBot API key received and tested
- [ ] `.env` file created from `.env.example` with all values
- [ ] Docker container is running (`docker-compose up -d`)
- [ ] Both workflows are imported in n8n
- [ ] Both workflows are **ACTIVATED** (toggle on)

---

## 🔄 Token Renewal Reminders

| Token | Validity | Action |
|-------|----------|--------|
| Instagram Access Token | 60 days | Refresh manually every 50 days |
| Google OAuth2 | Auto-refreshes | No action needed |
| YouTube OAuth2 | Auto-refreshes | No action needed |
| CallMeBot API Key | Permanent | No renewal needed |
