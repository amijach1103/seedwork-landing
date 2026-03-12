# Seedwork Landing Page — Setup Guide

## Files

| File | URL Path | Persona |
|------|----------|---------|
| `start.html` | joinseedwork.com/start | Variant A: Stuck Dreamer |
| `grow.html` | joinseedwork.com/grow | Variant B: Informal Operator |

## Step 1: Create GitHub Repo

```bash
# Create new repo (separate from VE-prototype)
gh repo create seedwork-landing --public --clone
# Copy landing page files into it
cp start.html grow.html SETUP.md seedwork-landing/
cd seedwork-landing
git add . && git commit -m "Initial landing pages — Variant A + B"
git push
```

Then enable GitHub Pages: Repo → Settings → Pages → Source: main branch → root.

## Step 2: Connect Custom Domain

1. In GitHub repo Settings → Pages → Custom domain: `joinseedwork.com`
2. In your domain registrar (wherever you bought joinseedwork.com), add DNS records:
   - `A` record → `185.199.108.153`
   - `A` record → `185.199.109.153`
   - `A` record → `185.199.110.153`
   - `A` record → `185.199.111.153`
   - `CNAME` for `www` → `your-github-username.github.io`
3. Check "Enforce HTTPS" in GitHub Pages settings.

## Step 3: Set Up PostHog (Analytics)

1. Go to [posthog.com](https://posthog.com) → Sign up (free tier: 1M events/month)
2. Create a project called "Seedwork"
3. Copy your Project API Key from Settings → Project → API Key
4. In BOTH `start.html` and `grow.html`:
   - Uncomment the PostHog `<script>` block in the `<head>`
   - Replace `YOUR_PROJECT_API_KEY` with your key

**What PostHog tracks automatically:**
- Pageviews (with variant tag)
- Scroll depth
- Time on page
- Device / browser / OS
- Referrer
- UTM parameters

**Custom events we fire:**
- `waitlist_signup` — with variant, contact_me, and UTM data

## Step 4: Set Up Google Sheets Backend (Email Capture)

### Create the Sheet
1. Create a new Google Sheet called "Seedwork Signups"
2. Add headers in Row 1: `email | contact_me | variant | utm_source | utm_medium | utm_campaign | utm_content | referrer | page_url | timestamp | user_agent`

### Create the Apps Script
1. In the Sheet: Extensions → Apps Script
2. Replace the code with:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    data.email,
    data.contact_me,
    data.variant,
    data.utm_source,
    data.utm_medium,
    data.utm_campaign,
    data.utm_content,
    data.referrer,
    data.page_url,
    data.timestamp,
    data.user_agent
  ]);

  return ContentService.createTextOutput(JSON.stringify({status: 'success'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Deploy: Deploy → New deployment → Web app
   - Execute as: Me
   - Who has access: Anyone
4. Copy the Web App URL
5. In BOTH `start.html` and `grow.html`:
   - Replace the empty `GOOGLE_SHEETS_URL` with your Web App URL

### Share the Sheet
Share the Google Sheet with Jessica so she can see signups in real time.

## Step 5: Test

1. Open each page locally, submit a test email
2. Check the browser console — you should see `SIGNUP DATA` logged
3. After connecting Google Sheets, verify the row appears in the sheet
4. After connecting PostHog, verify events appear in the PostHog dashboard

## UTM Link Examples

For organic posting:
```
# Reddit → Variant A (Stuck Dreamer)
https://joinseedwork.com/start?utm_source=reddit&utm_medium=organic&utm_campaign=launch&utm_content=r_entrepreneur

# Reddit → Variant A
https://joinseedwork.com/start?utm_source=reddit&utm_medium=organic&utm_campaign=launch&utm_content=r_sidehustle

# Facebook → Variant B (Informal Operator)
https://joinseedwork.com/grow?utm_source=facebook&utm_medium=organic&utm_campaign=launch&utm_content=cleaning_group

# LinkedIn → Variant A
https://joinseedwork.com/start?utm_source=linkedin&utm_medium=organic&utm_campaign=launch
```

For paid ads (when ready):
```
# Reddit Ads → Variant A
https://joinseedwork.com/start?utm_source=reddit&utm_medium=paid&utm_campaign=stuck_dreamer&utm_content=ad1

# Facebook Ads → Variant B
https://joinseedwork.com/grow?utm_source=facebook&utm_medium=paid&utm_campaign=informal_operator&utm_content=ad1
```

## Data You'll Have for March 27 Decision

| Source | Data |
|--------|------|
| PostHog | Visitors, conversion rate, scroll depth, time on page, device, UTM breakdown, variant comparison |
| Google Sheet | Every signup with email, contact-me flag, variant, source, timestamp |
| Calculated | Signup rate = sheet rows / PostHog unique visitors. Contact-me rate = "Yes" rows / total rows. |
