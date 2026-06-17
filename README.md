[README.md](https://github.com/user-attachments/files/29044402/README.md)
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
- **Baptism flow** — dedicated confession screen with the great confession verse, a notes field for the candidate, then straight to review (skips prayer requests — elders always announce baptism decisions publicly)
- **15 prayer need categories** — Spiritual, Life Circumstances, and three confidential crisis flags (thoughts of self-harm, abuse situation, other emergency)
- **Free-write field** — for anything not covered by the checkboxes
- In-person / Streaming attendance toggle

### Elder-Facing (PIN-protected)
- **Responses tab** — live feed pulled from Google Sheets, auto-refreshes every 30 seconds
- **Stats row** — total responses, baptism count, in-person count
- **Filter** — All / In Person / Streaming
- **Baptism candidates** pinned at the top when the alert toggle is on
- **Crisis banner** — red bar across the top of the page whenever a crisis flag is submitted, with the person's name
- **Tally chart** — horizontal bar chart of all response categories sorted by frequency; crisis bars in red, baptism in gold
- **Collapsible feed items** — double-tap any response to expand full detail; collapsed by default for privacy in public settings
- **Copy as text** — one tap copies a formatted pastoral summary of any response to the clipboard

### Settings (PIN-protected)
- Congregation name, full name, city, baptism verse and reference — all editable
- Elder PIN management — change the PIN across all devices by editing `HARDCODED_PIN` in the file
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

The Responses and Settings tabs are hidden from the nav entirely. There is no visible lock icon, no hint that anything else exists. Triple-tapping the church name in the gold header opens a numeric PIN pad. After the correct PIN is entered, both elder tabs appear for the duration of the session. A page refresh resets to the public view.

---

## Configuration

All device-wide configuration lives in three constants at the top of the `<script>` block in `index.html`. Edit these once and re-upload to GitHub — every device picks up the new values on next page load.

```javascript
// Apps Script deployment URL — required for Google Sheets sync
var HARDCODED_WEBHOOK_URL = 'https://script.google.com/macros/s/YOUR_ID/exec';

// Elder PIN — change here to update all devices at once
var HARDCODED_PIN = '1234';

// Congregation settings
var HARDCODED_SETTINGS = {
  churchShort:  'Schrader Lane',
  churchFull:   'Schrader Lane Church of Christ',
  city:         'Nashville, TN',
  baptismVerse: '"Thou art the Christ, the Son of the living God."',
  baptismRef:   'Matthew 16:16',
  alertBaptism: true,
  alertCrisis:  true
};
```

**Settings changed via the Settings UI apply to that device only.** They are stored as local overrides in the browser's `localStorage`. The hardcoded values in the file are the authoritative defaults — they apply to every device that has not explicitly overridden a given setting.

---

## Setup

### 1 — Deploy the HTML file

The entire app is a single `index.html` file. Host it anywhere that serves static HTML.

**GitHub Pages (recommended — free, permanent URL)**
1. Create a free account at [github.com](https://github.com)
2. Create a new repository (public)
3. Upload `index.html` — it must be named exactly `index.html`
4. Go to **Settings → Pages → Source → Deploy from branch → main → / (root) → Save**
5. Your URL will be `https://yourusername.github.io/your-repo-name`

Allow 1–3 minutes for the first deploy. The URL is permanent.

### 2 — Set up Google Sheets

1. Create a new Google Sheet. Copy the Sheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

2. Go to [script.google.com](https://script.google.com) → **New Project**

3. Paste the Apps Script code (available in **Settings → Google Sheets Integration → Webhook Code** inside the app, or from the Apps Script section below)

4. Replace `YOUR_GOOGLE_SHEET_ID_HERE` with your actual Sheet ID

5. Click **Deploy → New Deployment → Web App**
   - Execute as: `Me`
   - Who has access: **Anyone**

6. Click **Deploy**, authorize permissions, and copy the deployment URL — it ends in `/exec`

### 3 — Configure the file

Open `index.html` in any text editor. At the top of the `<script>` block, fill in the three constants:

```javascript
var HARDCODED_WEBHOOK_URL = 'https://script.google.com/macros/s/YOUR_ID/exec';
var HARDCODED_PIN = '1234';   // change to your chosen PIN
var HARDCODED_SETTINGS = {
  churchShort: 'Your Church Short Name',
  churchFull:  'Your Church Full Name',
  city:        'Your City, State',
  // ... etc
};
```

Save the file and re-upload it to GitHub. Every device that opens the app URL is now fully configured — no per-device setup required.

### 4 — Test the connection

Open the app → triple-tap the church name → enter PIN → Settings → scroll to **Google Sheets Integration** → click **Test Connection**. The status dot turns green and a test row appears in the sheet.

### 5 — Generate a bulletin QR code

Settings → QR Code Generator → paste your app URL → click **Response Card** → download the PNG. Add it to your bulletin insert with the caption:

> *"Scan to submit your worship response — prayer requests, decisions, and confidential needs go directly to your shepherds."*

---

## Making Changes After Going Live

All ongoing maintenance is done by editing the three constants in the file and re-uploading to GitHub:

| What changed | What to update |
|---|---|
| Apps Script URL | `HARDCODED_WEBHOOK_URL` |
| Elder PIN | `HARDCODED_PIN` |
| Church name, verse, alerts | `HARDCODED_SETTINGS` |

Re-uploading the file to GitHub takes about 30 seconds. Every device picks up the change on its next page load or refresh — no app store, no push notification, no per-device action.

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
The file must be named exactly `index.html`. GitHub Pages serves `index.html` as the root page. Any other filename will render but JS may not initialize correctly.

**Responses tab shows "No responses yet" on elder devices**
`HARDCODED_WEBHOOK_URL` must be filled in the file. If it is and the issue persists, go to Settings → Google Sheets Integration → Test Connection:
- Green dot: connection works — hit ↻ Refresh on the Responses tab
- Red dot + "Script error": Sheet ID is wrong in the Apps Script
- Red dot + "Connection failed": URL is wrong, or deployment access is not set to "Anyone"
- Nothing happening: the Apps Script was not redeployed after the last code change

**Apps Script changes not taking effect**
Every code change requires a new deployment. Go to **Deploy → Manage Deployments → pencil icon → New version → Deploy**. The URL stays the same — no update needed in the file.

**JSON parse error on the Responses tab**
The Apps Script is returning an HTML page instead of JSON. The `action=fetch` branch is not present in the currently deployed version. Copy the latest code from Settings → Webhook Code, paste into Apps Script, and redeploy as a new version.

**Crisis banner not appearing**
Confirm "Flag crisis submissions immediately" is toggled on in Settings → Elder Alerts. If the toggle was changed in the UI on one device, it only affects that device. Set `alertCrisis: true` in `HARDCODED_SETTINGS` in the file to enforce it on all devices.

**PIN not working on a device**
If `HARDCODED_PIN` was recently changed in the file and re-uploaded, any device that previously changed the PIN via the Settings UI has a local override that takes precedence. Go to Settings → Data Management → Reset to File Defaults to clear it.

**Settings look different on different devices**
Expected behavior if the Settings UI was used on some devices. Reset to file defaults on any device that is out of sync, or clear `localStorage` in the browser.

---

## Adopting for Another Congregation

1. Set `HARDCODED_WEBHOOK_URL` to your own Apps Script endpoint
2. Set `HARDCODED_PIN` to your chosen elder PIN
3. Update `HARDCODED_SETTINGS` with your church name, city, and baptism verse
4. Save and upload to your own GitHub Pages repository

No other code changes required. The Settings UI inside the app can handle everything else.

---

## Architecture

| Layer | Technology |
|-------|-----------|
| Hosting | GitHub Pages (static HTML) |
| Configuration | Constants hardcoded in `index.html` — file is the source of truth for all devices |
| Storage — submissions | Google Sheets via Apps Script GET webhook |
| Storage — local overrides | Browser `localStorage` (device-specific, overrides file defaults) |
| Auth | 4-digit PIN, session-scoped; `HARDCODED_PIN` in file is authoritative |
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
