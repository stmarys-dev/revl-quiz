# REVL Training — Interactive Quiz

Interactive training questionnaire that recommends a personalised REVL program based on 5 questions. Captures name, email and mobile number before showing the result, and sends leads to Google Sheets automatically.

---

## Deployment (Vercel)

This project deploys under the **stmarys-5100** Vercel account.

```bash
npm i -g vercel
cd revl-quiz
vercel login        # log in as stmarys-5100 if not already
vercel              # follow prompts — select stmarys-5100 scope
```

Follow the prompts — takes under 2 minutes. You'll get a URL like `revl-quiz.vercel.app`.

To redeploy after any changes:

```bash
vercel --prod
```

To connect a custom domain, go to the Vercel dashboard → your project → Domains.

---

## Google Sheets Lead Capture Setup

Every quiz submission (name, email, quiz answers, timestamp) gets saved to a Google Sheet automatically. Free, no limits.

### Step 1 — Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet
2. Name it **REVL Quiz Leads**
3. The Apps Script will create the column headers automatically on first submission

### Step 2 — Add the Apps Script

1. In your Google Sheet, click **Extensions → Apps Script**
2. Delete any existing code and paste the following:

```javascript
function doPost(e) {
  try {
    const data  = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Add headers on first run
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        'Timestamp', 'Name', 'Email', 'Phone', 'Marketing Consent',
        'Timing', 'Goal', 'Days', 'Team', 'Experience'
      ]);
    }

    sheet.appendRow([
      data.timestamp  || new Date().toISOString(),
      data.name       || '',
      data.email      || '',
      data.phone      || '',
      data.marketing  ? 'Yes' : 'No',
      data.timing     || '',
      data.goal       || '',
      data.days       || '',
      data.team       || '',
      data.experience || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click **Save** (name the project anything, e.g. "REVL Quiz")

### Step 3 — Deploy the Apps Script

1. Click **Deploy → New deployment**
2. Click the gear icon next to "Type" → select **Web app**
3. Set **Execute as**: Me
4. Set **Who has access**: Anyone
5. Click **Deploy**
6. Copy the **Web app URL** — it looks like `https://script.google.com/macros/s/XXXX/exec`

### Step 4 — Add the URL to index.html

Open `index.html` and find this line near the top of the `<script>` tag:

```javascript
const LEAD_CAPTURE_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
```

Replace `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` with your web app URL.

### Step 5 — Redeploy to Vercel

```bash
vercel --prod
```

That's it. Leads will now appear in your Google Sheet within seconds of each submission.

---

## Updating Content

All content that REVL may need to update is clearly marked with `// TODO` comments in `index.html`.

| What to update | Where in index.html |
|---|---|
| EOFY offer headline, detail, urgency copy | Search `eofy-headline` |
| Membership prices or inclusions | Search `membership-grid` |
| App download / CTA link | `const APP_URL = ...` at top of script |
| Lead capture endpoint | `const LEAD_CAPTURE_URL = ...` at top of script |
| Instagram handle | Search `@revltraining` |

---

## Assets Required

Place these files in the `assets/` folder:

| File | Description |
|---|---|
| `assets/logo.png` | REVL logo — white version for dark backgrounds |
| `assets/logo-black.png` | REVL logo — black version (for future use) |
| `assets/og-image.png` | 1200×630px image for social link previews |
| `assets/favicon.ico` | Browser tab icon |

The welcome screen will show a text fallback ("REVL") if `logo.png` is missing.

---

## Connecting to Xoda (Future)

When REVL is ready to sync leads directly into Xoda, there are two approaches:

**Option A — Zapier bridge (no-code, recommended):**
1. Connect Google Sheets to Zapier
2. Trigger: "New Row in Google Sheet"
3. Action: Create a lead/contact in Xoda (if Xoda has a Zapier integration)

**Option B — Direct API call from Apps Script:**
Add a second `fetch()` call inside the `doPost` function in your Apps Script. When Xoda provides API documentation, the Apps Script can POST to both Google Sheets and Xoda simultaneously — no changes needed to the quiz itself.

---

## File Structure

```
revl-quiz/
  index.html      ← entire app (HTML + CSS + JS)
  vercel.json     ← Vercel routing config
  README.md       ← this file
  assets/
    logo.png      ← add your logo here
    og-image.png  ← add social preview image here
    favicon.ico   ← add favicon here
```
