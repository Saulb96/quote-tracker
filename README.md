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
1. Create list **Quote_Tracker** on `/sites/Production` with these columns:

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
| Shift + arrows / Shift+click | extend selection |
| Click `#` cell | select whole row |
| Click header label | select whole column |
| Ctrl+A | select all |
| Enter · F2 · Space | edit · type to overwrite |
| Ctrl+D | fill down through selection |
| Ctrl+C / Ctrl+V | copy · paste (multi-cell, Excel-compatible) |
| Ctrl+Z | undo |
| Delete | clear selection |
| Click thumbnail | lightbox · arrows to browse, Esc to close |
| Alt+click thumbnail | replace that photo |

Dropdown columns (Design/Color Sensitive, Brand, Category, Customer, Stock or
Program, Season, Action, Stage) open on a single click and show a small caret.
Sorting is the small control to the left of each header label, so clicking the
label itself selects the column. Frozen columns: `#`, Date, Image, Style.

## Left filter panel

Checkboxes for every Action and Stage value with live counts. Multiple ticks
inside one group are OR; the two groups combine as AND. Counts for a group are
computed ignoring that group's own ticks, so they show what each option would
add. `‹` collapses the panel to a rail.

## Performance notes

Only the visible rows are in the DOM, so row count barely affects speed. Cell
edits patch a single cell rather than re-rendering. Tokens are acquired per
Graph call (MSAL serves from cache) with one forced-refresh retry on a 401.
Expired image links are detected on image load failure and refreshed once,
shared across all images in flight.

## Autosave

There is no Save button. Edits persist about a second after you stop typing;
the status bar reads `Pending…` → `Saving…` → `✓ All changes saved`. A failure
shows `Save failed — click to retry` and keeps the change queued rather than
dropping it. Closing the tab mid-save prompts a warning, and switching away
from the tab forces an immediate flush.

Deletions and undo persist the same way. The panel starts collapsed and row
height starts at M; both remember whatever you last chose.

## Selecting rows

Every row has a checkbox in the leftmost column. The header checkbox tags all
visible rows and shows a dash when only some are tagged. Tagged rows highlight
blue and both export buttons show the count. Ctrl+click on a row number still
toggles a tag. With nothing tagged, the exports fall back to whatever rows the
cell selection covers.

Changing a filter drops tags for rows that are no longer visible, so a hidden
selection can't be exported by accident.

## Request files

**Sample Request** and **Dev Tracker** both open the same dialog, which
collects the values that apply to the whole request and lets you tick which
files to produce. The button you clicked just pre-ticks its own output; tick
both to get both from one pass.

Dialog fields: Supplier, Sample Qty Needed, Sample Type. Supplier is a strict
dropdown fed from the **Factories** list on the same site — no free typing, so
names always match the factory records. Sample Type defaults to Artwork Sample
and only affects the Dev Tracker file.

Artwork Receive Date (F), SRD (G) and Notes (I) export blank for filling in
Excel; F and G come pre-formatted `m/d/yyyy`. Sample Time (H) carries the formula
`=IF(OR(F2="",G2=""),"",G2-F2)`, so it reports the day count as soon as both
dates are entered and stays blank until then.

### Sample Request file
Matches Sample_Request_Template: Supplier, Picture, Style#, Description,
Sample Qty Needed, Artwork Receive Date, SRD, Sample Time, Notes. Product
photos are embedded at true aspect ratio and centered, so the file goes to the
factory as-is. Style# and Description come from the row; Supplier and Sample
Qty from the dialog. Artwork Receive Date, SRD and Notes export blank, and
Sample Time calculates from the two dates.

The tracker's own Note column is deliberately **not** carried over — those are
internal remarks and this file goes to the factory. Rows with no photo export with a blank Picture cell, and the
dialog says how many that will be up front.

Image bytes are pulled from SharePoint (pre-authenticated link first, then the
Graph content endpoint, then the repo folder).

### Development Tracker file
Column order: Status, Date Created, Supplier, PO#, Quote#,
SR / BF / BS / FF #, PPS Comment, Sample Type, Customer, Style #, Item
Description.

| Column | Source |
|---|---|
| Status | `Open` |
| Date Created | today's export date |
| Supplier | dialog |
| Quote# | Quote Sheet Name |
| Sample Type | dialog, defaults to Artwork Sample |
| Customer · Style # · Item Description | the row |
| PO# · SR/BF/BS/FF # · PPS Comment | blank |

Both files use real Excel date values formatted `m/d/yyyy`, styled headers,
thin borders, a frozen header row, and landscape fit-to-width page setup.
Filenames include the supplier name and date.
