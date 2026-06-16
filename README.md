[README.md](https://github.com/user-attachments/files/29023299/README.md)
# Worship Response System

A lightweight, self-hosted worship response card for Churches of Christ congregations. Members submit prayer requests, public decisions, and crisis situations from any device. Elders receive responses in real time through a PIN-protected dashboard backed by Google Sheets.

Built for **Schrader Lane Church of Christ**, Nashville, TN. Configurable for any congregation.

---

## What It Does

**Members see one thing:** a digital response card — the same information they would write on a paper card and drop in a basket. Name, attendance mode, response type, prayer needs, and a free-write field. Submitted responses go directly to the elders.

**Elders access a live dashboard** (PIN-protected, hidden from the public) showing all responses pulled from Google Sheets in real time — tally chart, baptism candidates pinned at top, crisis flags in red, and a collapsible feed of every submission.

---

## Features

### Member-Facing
- **3-step response card** — name and attendance mode → response type → prayer requests & needs → review and submit
- **Six response types** — Prayer Request, Public Confession, Baptism, Rededication, Membership, Other
- **Baptism flow** — dedicated confession screen with Matthew 16:16, a notes field for the candidate, then straight to review (skips prayer requests)
- **15 prayer need categories** — Spiritual, Life Circumstances, and three confidential Crisis flags (self-harm, abuse, other emergency)
- **Free-write field** — for anything not covered by the checkboxes
- In-person / Streaming attendance toggle

### Elder-Facing (PIN-protected)
- **Responses tab** — live feed pulled from Google Sheets, auto-refreshes every 30 seconds
- **Stats row** — total responses, baptism count, in-person count
- **Filter** — All / In Person / Streaming
- **Baptism candidates** pinned at the top when the alert toggle is on
- **Crisis banner** — red bar across the top of the page whenever a crisis flag is submitted, with the person's name
- **Tally chart** — horizontal bar chart of all response categories sorted by frequency; crisis bars in red, baptism in gold
- **Collapsible feed items** — double-tap any response to expand full detail; collapsed by default for privacy
- **Copy as text** — one tap copies a formatted pastoral summary of any response to the clipboard

### Settings (PIN-protected)
- Congregation name, full name, city, baptism verse and reference — all editable for adoption by another church
- Elder PIN management — change from default (1234) to something only elders know
- Elder alert toggles — baptism flag, crisis banner
- QR code generator — scannable code for the bulletin pointing to the Response Card
- Google Sheets integration — Apps Script code with one-click copy, webhook test with live status indicator
- Data management — clear responses, reset settings, export JSON

---

## Access Model

| Who | Sees | How |
|-----|------|-----|
| Everyone | 🙏 Response Card tab only | Default on page load |
| Elders | 📋 Responses + ⚙️ Settings | Triple-tap the church name → PIN |

The Settings and Responses tabs are hidden from the nav entirely. There is no visible lock icon, no hint that anything else exists. Triple-tapping the church name in the gold header opens a numeric PIN pad. After the correct PIN is entered, both elder tabs appear for the duration of the session. A page refresh resets to the public view.

**Default PIN: `1234`** — change this immediately in Settings → Elder PIN before going live.

---

## Setup

### 1 — Deploy the HTML file

The entire app is a single `index.html` file. Host it anywhere that serves static HTML:

**GitHub Pages (recommended — free, permanent URL)**
1. Create a free account at [github.com](https://github.com)
2. Create a new repository (public, add a README)
3. Upload `index.html` — rename it to `index.html` before uploading
4. Go to **Settings → Pages → Source → Deploy from branch → main → / (root) → Save**
5. Your URL will be `https://yourusername.github.io/your-repo-name`

Allow 1–3 minutes for the first deploy. The URL is permanent.

### 2 — Set up Google Sheets

1. Create a new Google Sheet. Copy the Sheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

2. Go to [script.google.com](https://script.google.com) → **New Project**

3. Paste the Apps Script code from **Settings → Google Sheets Integration → Webhook Code** in the app (or from the `apps-script.gs` section below)

4. Replace `YOUR_GOOGLE_SHEET_ID_HERE` with your actual Sheet ID

5. Click **Deploy → New Deployment → Web App**
   - Description: `Worship Response Webhook`
   - Execute as: `Me`
   - Who has access: **Anyone**

6. Click **Deploy**, authorize the permissions, and copy the deployment URL (ends in `/exec`)

### 3 — Wire up the URL

Open `index.html` in a text editor. Find this line near the top of the `<script>` block:

```javascript
var HARDCODED_WEBHOOK_URL = '';
```

Paste your Apps Script URL between the quotes:

```javascript
var HARDCODED_WEBHOOK_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```

Save the file. Re-upload it to GitHub. Every device that opens the app URL is now automatically connected to the sheet — no per-device setup required.

### 4 — Test the connection

Open the app → triple-tap the church name → enter PIN → Settings → scroll to **Google Sheets Integration** → click **Test Connection**. The status dot should turn green and a test row should appear in your Google Sheet.

### 5 — Update congregation details

In Settings → Congregation:
- Set your church's short name (appears in the gold header)
- Set the full name (appears on the response card)
- Update the baptism confession verse and reference if desired

### 6 — Change the elder PIN

Settings → Elder PIN → enter `1234` as the current PIN → set a new 4-digit PIN → Update PIN. Share the new PIN only with your elders.

### 7 — Generate bulletin QR codes

Settings → QR Code Generator → paste your app URL → click **Response Card** → download the PNG. Drop it into your bulletin insert with the caption:

> *"Scan to submit your worship response — prayer requests, decisions, and confidential needs go directly to your shepherds."*

---

## Apps Script Code

```javascript
// Worship Response Webhook — Google Apps Script
var SHEET_ID   = 'YOUR_GOOGLE_SHEET_ID_HERE';
var SHEET_NAME = 'Responses';

function doGet(e) {
  try {
    var p = e.parameter;
    if (p && p.action === 'fetch') {
      var ss    = SpreadsheetApp.openById(SHEET_ID);
      var sheet = ss.getSheetByName(SHEET_NAME);
      if (!sheet) { return jsonOut([]); }
      var rows = sheet.getDataRange().getValues();
      if (rows.length <= 1) { return jsonOut([]); }
      var out = [];
      for (var i = 1; i < rows.length; i++) {
        var r    = rows[i];
        var cats = r[5] ? r[5].toString().split('; ') : [];
        out.push({
          id:           r[0] ? new Date(r[0]).getTime() : i,
          timestamp:    r[0] ? r[0].toString() : '',
          type:         r[1] ? r[1].toString() : '',
          name:         r[2] ? r[2].toString() : 'Anonymous',
          contact:      r[3] ? r[3].toString() : '',
          mode:         r[4] ? r[4].toString() : '',
          categories:   cats,
          notes:        r[6] ? r[6].toString() : '',
          crisis:       r[7] === 'YES',
          baptism:      r[1] === 'Baptism',
          baptismPublic: true
        });
      }
      out.reverse();
      return jsonOut(out);
    }
    if (!p || !p.type) { return okOut(); }
    var ss    = SpreadsheetApp.openById(SHEET_ID);
    var sheet = ss.getSheetByName(SHEET_NAME);
    if (!sheet) {
      sheet = ss.insertSheet(SHEET_NAME);
      sheet.appendRow(['Timestamp','Type','Name','Contact',
        'Mode','Categories','Notes','Crisis']);
    }
    var cats = p.categories ? p.categories.split('|') : [];
    sheet.appendRow([
      new Date().toISOString(),
      p.type || '',
      p.name || 'Anonymous',
      p.contact || '',
      p.mode || '',
      cats.join('; '),
      p.notes || '',
      p.crisis === 'true' ? 'YES' : ''
    ]);
    return okOut();
  } catch(err) {
    return ContentService.createTextOutput('ERROR: ' + err.toString())
      .setMimeType(ContentService.MimeType.TEXT);
  }
}
function jsonOut(data) {
  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
function okOut() {
  return ContentService.createTextOutput('OK')
    .setMimeType(ContentService.MimeType.TEXT);
}
```

---

## Troubleshooting

**Buttons not clickable after uploading to GitHub**
Make sure the file is named exactly `index.html` (not `hold-the-line-response-system.html` or anything else). GitHub Pages serves `index.html` as the root.

**Responses tab shows "No responses yet" on elder devices**
The webhook URL must be baked into `HARDCODED_WEBHOOK_URL` in the file. If it is and the issue persists, go to Settings → Google Sheets Integration → Test Connection. Check what the status shows:
- Green dot: connection works — hit Refresh on the Responses tab
- Red dot + "Script error": Sheet ID is wrong in the Apps Script
- Red dot + "Connection failed": URL is wrong or deployment access is not set to "Anyone"
- No test at all: the Apps Script was not redeployed after the last code change

**Apps Script changes not taking effect**
Every code change requires a new deployment. Go to **Deploy → Manage Deployments → pencil icon → New version → Deploy**. The URL stays the same.

**JSON parse error in the fetch**
The Apps Script is returning an HTML page instead of JSON. This means the `action=fetch` branch is not present in the currently deployed version. Copy the latest code from Settings, paste it into Apps Script, and redeploy as a new version.

**Crisis banner not appearing**
Confirm the "Flag crisis submissions immediately" toggle is on in Settings → Elder Alerts.

---

## Adopting for Another Congregation

This system is designed to be congregation-neutral. To adopt it:

1. Update `HARDCODED_WEBHOOK_URL` with your own Apps Script endpoint
2. Open the app → triple-tap church name → PIN (default: `1234`) → Settings → Congregation
3. Set your church's short name, full name, city, and baptism verse
4. Change the elder PIN
5. Generate a new QR code for your bulletin

No code changes required beyond the webhook URL in Step 1.

---

## Architecture

| Layer | Technology |
|-------|-----------|
| Hosting | GitHub Pages (static HTML) |
| Storage — submissions | Google Sheets via Apps Script GET webhook |
| Storage — settings/cache | Browser `localStorage` |
| Auth | 4-digit PIN, session-scoped, stored in `localStorage` |
| Real-time sync | Polling Google Sheets every 30 seconds on the Responses tab |
| Offline fallback | `localStorage` cache used when sheet is unreachable |

The app is intentionally dependency-light. The only external library is [qrcodejs](https://github.com/davidshimjs/qrcodejs) loaded from cdnjs for the bulletin QR generator.

---

## File Structure

```
index.html     — the entire application (HTML + CSS + JS, single file)
README.md      — this document
```

---

*Built for the shepherds of Schrader Lane Church of Christ, Nashville, TN.*
