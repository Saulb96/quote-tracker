# M1S Quote Tracker

Single-file HTML tracker for the Quoting sheet — wide layout, Excel-like grid, large images.

## Files
- `index.html` — the whole app
- `data/quotes.json` — 525 rows exported from `01 QuotingTracker.xlsx` (seed + demo-mode source)
- `images/thumb/<STYLE>.jpg` — 360px grid thumbnails (523 files, 6 MB)
- `images/full/<STYLE>.jpg` — 1400px lightbox images (523 files, 24 MB)

Images are keyed by **Style #**. Add a new product photo by dropping
`STYLE.jpg` into both folders — no code change, no spreadsheet embedding.

## Deploy
1. New repo `quote-tracker` → push these files → enable GitHub Pages.
2. Open the page. It runs immediately in **demo mode** (reads `data/quotes.json`, read-only).

## Go live on SharePoint
1. Create list **Quote_Tracker** on `/sites/Importing` with these columns:

| Internal name | Type |
|---|---|
| Title | Text *(holds Style #)* |
| QuoteDate | Date only |
| ImgKey | Text |
| Descr | Multi-line, plain |
| Sensitive | Text |
| Brand | Text |
| Category | Text |
| Customer | Text |
| StockProgram | Text |
| Season | Text |
| ActionItem | Text |
| Stage | Text |
| QuoteSheet | Text |
| Notes | Multi-line, plain |

   Index **Title**, **Brand**, **ActionItem**, **Stage** (5,000-item threshold).
   Avoid renaming to `DisplayName` — SharePoint silently drops writes to it.
2. Import `data/quotes.json` into the list (or paste the exported xlsx into
   grid view). `img` → `ImgKey`, `style` → `Title`, `desc` → `Descr`.
3. Register an Entra app: platform **Single-page application (SPA)**,
   redirect URI = the Pages URL. Delegated scope `Sites.ReadWrite.All`.
4. In `index.html`, set `clientId`. Demo mode switches off automatically.

## Keyboard
| | |
|---|---|
| Arrows / Tab | move |
| Shift + arrows / Shift+click | select range |
| Enter or F2 | edit · type to overwrite |
| Ctrl+D | fill down through selection |
| Ctrl+C / Ctrl+V | copy · paste (multi-cell, Excel-compatible) |
| Ctrl+Z | undo |
| Delete | clear selection |
| Click thumbnail | lightbox · ← → to browse, Esc to close |

Frozen columns: `#`, Date, Image, Style. Row-height buttons S/M/L/XL/XXL
scale the thumbnails up to 282px in-grid.

---

## Adding quotes

**+ New Quote** — full form with a drag-and-drop photo pane. The image is
resized in the browser (360px thumb + 1400px full), uploaded to SharePoint,
and filed under the style number. Saves immediately; no separate Save click.

**+ Row** — blank inline row for quick data-only entry. Stays local until Save.

**Photo on an existing row** — click the `+ PHOTO` placeholder in the Image
column. To replace a photo that's already there, Alt+click the thumbnail.
Both upload right away, then hit Save to record the key on the row.

### One-time setup for uploads
In `/sites/Importing/Shared Documents`, create a folder **Quote Images**
containing two subfolders: **thumb** and **full**. That's it — the tool
writes into them. Grant the team Contribute on that folder.

Image lookup order: SharePoint library first, repo `images/` as fallback.
So the 523 seeded photos keep loading with no token, and anything added
later comes from SharePoint. Library URLs are pre-authenticated and expire
after roughly an hour; the tool silently refreshes them every 35 minutes.

### Migrating the seeded images to SharePoint (optional)
Only worth it if the repo needs to be private. Upload `images/thumb/*` and
`images/full/*` into the matching SharePoint folders and delete the repo
`images/` directory — the fallback simply stops being used.
