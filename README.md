# Fred & Jo – wedding invitation site

A single static page (`index.html` + `assets/`). No build step, no server, no framework. Works on any free static host.

## Files

- `index.html` – the whole site (HTML, CSS and JS in one file)
- `assets/img/` – couple photo, 4 gallery photos, envelope / flower / paper artwork
- `assets/fonts/` – Viaoda Libre and Lora (the other fonts load from Google Fonts)

## Editing the content

Open `index.html` and find the `CFG = { ... }` block near the bottom. Every name, date, time, venue and photo list is in there. Change the values, save, refresh. Nothing else in the file needs touching.

To swap photos, drop new files into `assets/img/` and update the `gallery` list (any number of photos works; the grid is 2 wide).

## Hosting (free)

**Option A – Netlify Drop (fastest, ~2 minutes)**
1. Go to https://app.netlify.com/drop and sign in (free).
2. Drag the whole `fredr` folder onto the page.
3. You get a `something.netlify.app` link immediately. Rename it under *Site settings → Change site name*.

**Option B – GitHub Pages**
1. Create a repo, push this folder to it.
2. *Settings → Pages → Branch: main, folder: / (root)*.
3. The site appears at `https://<user>.github.io/<repo>/`.

Both support a custom domain if they already own one.

## Guestbook (shared wishes)

Already wired up. Wishes are stored in a Google Sheet owned by the Google account that created it:

- Sheet: https://docs.google.com/spreadsheets/d/1ugDlX7FYQlm49LYN1CNAN1kvbsG525OxuAhPCUfrHPg/edit
- Apps Script project: https://script.google.com/home/projects/1xS_DKPC7sYRN_0vv5XQyrxeaaLKa5iNA6O0kObys_v9M2TV3LRiaxwbv/edit
- Web app URL is in `guestbookEndpoint` inside `index.html`

Share the sheet with the couple so they can read wishes; deleting a row removes that wish from the site. If the sheet is ever deleted or the script undeployed, the site silently falls back to per-device storage.

To recreate it on another account: new Sheet with header row `name | message | at`, new Apps Script with the code below (set `SHEET_ID`), Deploy → Web app → execute as *Me*, access *Anyone*, paste the `/exec` URL into `guestbookEndpoint`.

```js
const SHEET_ID = 'PASTE_SHEET_ID';
function sheet() { return SpreadsheetApp.openById(SHEET_ID).getSheets()[0]; }
function doGet() {
  const rows = sheet().getDataRange().getValues().slice(1).filter(r => r[0])
    .map(r => ({ name: r[0], message: r[1], at: r[2] }));
  return ContentService.createTextOutput(JSON.stringify(rows)).setMimeType(ContentService.MimeType.JSON);
}
function doPost(e) {
  const d = JSON.parse(e.postData.contents);
  sheet().appendRow([String(d.name || '').slice(0, 80), String(d.message || '').slice(0, 1000), d.at || new Date().toISOString()]);
  return ContentService.createTextOutput('ok');
}
```

## Local preview

```bash
python -m http.server 8765
```

then open http://localhost:8765 (opening `index.html` directly from the file system also works, except the map embed).
