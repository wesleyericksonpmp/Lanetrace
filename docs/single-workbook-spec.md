# Single-Workbook Plans: Spec

**Status:** Draft for review
**Target:** Lanetrace 4.18
**Date:** 2026-11-05

## 1. Problem

A plan in Lanetrace lives in up to three places today:

| File | Holds | Written when |
|---|---|---|
| The user's workbook (for example `Intake-Approach_Lanetrace.xlsx`) | The plan as imported | The user edits it in Excel |
| `program-timeline.data.xlsx` | Lanetrace's copy of the last import | A file is imported with a folder linked |
| `program-timeline.state.json` | Settings, plus edits made in Lanetrace (overlay) | Anything changes in Lanetrace |

Without a linked folder, the settings and edits live only in the browser's storage.

What goes wrong:

- **Edits made in Excel don't show.** After the first import, Lanetrace reads its own copy. Changes saved in Excel appear only if someone imports the file again.
- **Edits made in Lanetrace never reach Excel.** They sit in the state file's overlay until someone exports a new workbook by hand.
- **A workbook opened on its own looks plain.** The title, colors, blackouts and layout stay behind in the browser or the state file. This is why the intake timeline opened with default colors.
- **Renames lose edits.** Overlay edits are keyed by a hash of Title and Swimlane (`rowKeyFor`). Renaming an item in Excel orphans its Lanetrace edits. `docs/data-format.md` says Row ID is the overlay key, but the code doesn't use it.
- **Hand-offs need a README** to explain which file to edit and how to link the folder.

## 2. Goal

The workbook is the document. One `.xlsx` holds the plan and everything Lanetrace needs to draw it. Opening that file anywhere gives the finished timeline. Edits in either Excel or Lanetrace show up in the other without anyone re-importing.

### Success criteria

1. Open a workbook that Lanetrace saved, in a fresh browser profile: the timeline matches the saved look (title, subtitle, colors, blackouts, Today, layout).
2. Edit a date in Excel and save: Lanetrace updates within 2 seconds, with no re-import.
3. Edit a date in Lanetrace: the workbook on disk has the new date within 2 seconds, or as soon as Excel releases its lock.
4. The same row changed in both places before either saved: Lanetrace shows one prompt naming the row and the field, and loses neither value until the user chooses.
5. Rename an item in Excel: its Lanetrace-only fields (if any remain) and its place in the row order survive.
6. The workbook still opens cleanly in Excel desktop, Excel for the web and Excel on Mac, with the user's own formatting, extra sheets, formulas and column widths unchanged.
7. Existing linked folders (`state.json` plus `data.xlsx`) keep working, with an offer to move to one workbook.

### Non-goals

- Real-time co-editing between two people. That is the CRDT problem, and a plan with field rules is a poor fit for it.
- Any server, account or cloud sync. OneDrive or SharePoint can carry the file; Lanetrace doesn't talk to them.
- Editing CSV, Planner or Smartsheet exports in place. Those keep today's import behavior and can be saved as a Lanetrace workbook.

## 3. How it works for the user

### Open

- **Edge or Chrome (version 133 or later):** File > Open plan (or drop the workbook on the page). Lanetrace asks once for edit access to that file. The header shows the file name and "Live".
- **Next visit:** Lanetrace reopens the same file. If the browser needs permission again, a one-line banner shows "Reconnect", like today's sync folder.
- **Firefox, Safari, or a browser that blocks file access:** opening works read-only as an import, as today. Saving downloads a new workbook with everything inside it (see 7.3).

### Edit

- Edits in Lanetrace save to the workbook automatically, after a short pause (about 1 second after the last change).
- Saves in Excel reload in Lanetrace automatically. A toast reads "Updated from Excel: 3 rows changed".
- While Excel has the workbook open, it usually locks the file. Lanetrace keeps the edits, shows "Waiting for Excel to close Intake-Approach.xlsx", and saves once the lock clears. This uses the existing retry-on-lock behavior.

### Hand off

Send the one workbook, plus `lanetrace.html` if the reader doesn't have it. No README steps.

## 4. What goes inside the workbook

### 4.1 Plan rows

The plan stays on the `TimelineExcel` sheet with the documented columns (`docs/data-format.md`). Lanetrace writes edits into those cells: the same rows and columns the user already edits in Excel. Edits no longer go into a separate overlay.

Lanetrace-only fields that have no column today (row order and hidden state) map to columns: `Shown` already exists, and row order is the sheet's row order.

### 4.2 The `Lanetrace` sheet

A new sheet named `Lanetrace`, set to `state="veryHidden"` in `xl/workbook.xml`. Excel's Unhide menu doesn't list veryHidden sheets, so people can't break it by accident, and Excel desktop, Excel for the web and Excel on Mac all keep it on save. (Excel for the web and Mac need a test pass: see 11.)

| Cell | Holds |
|---|---|
| A1 | `lanetrace-workbook` (marker) |
| B1 | Format version, starting at `1` |
| A2 | Lanetrace version that last saved, for example `4.18.0` |
| A3 | Saved at (ISO time) |
| A5 down | Settings JSON, split into chunks of 30,000 characters (Excel's cell limit is 32,767) |
| D5 down | Row fingerprints: Row ID in column D, hash in column E (see 6) |

Settings saved here are the display settings: title, subtitle, theme pack, density, width, packing, milestone placement, label settings, Today, blackouts, legend, lane colors, lane text colors, lane order, hidden lanes, collapsed lanes, and the Data view column layout. Not saved here: `dataOverlay`, `addedItems` and `deletedKeys` (those become rows), and anything that only makes sense in one browser (open panels, the last-used view).

Also mirror the title into the workbook's core properties (`docProps/core.xml` title) so File Explorer and SharePoint show it.

### 4.3 Reading rules

- When a workbook has the `Lanetrace` sheet, its settings replace the browser's settings for that plan.
- When it doesn't, today's behavior applies: browser settings with the import title rules.
- An unknown higher format version opens read-only with a message to update Lanetrace.

## 5. Row identity

Row ID becomes the row key, as `data-format.md` already promises.

- A row with a Row ID is keyed by it.
- A row without one gets the next free integer the first time Lanetrace saves, and Lanetrace writes it into the Row ID column. A toast says how many were filled.
- Duplicate Row IDs: the first keeps it, the rest get new IDs on save, with a toast naming them. Depends On values pointing at a duplicated ID are flagged (they already are when an ID matches no row).
- Workbooks with no Row ID column get one inserted as the first column on first save. Lanetrace asks first, since it changes the sheet's layout.
- `rowKeyFor` (the Title and Swimlane hash) stays only for reading old state files during migration (see 9).

## 6. Keeping Excel and Lanetrace in step

### 6.1 Watching the file

- Keep the `FileSystemFileHandle` in IndexedDB, the same way the sync folder handle is kept today.
- Watch it with `FileSystemObserver` (Chrome and Edge 133 or later). On a `modified` record, re-read the file.
- Where `FileSystemObserver` is missing, check `getFile().lastModified` when the tab gains focus and every 5 seconds while it's visible.
- Ignore change events caused by Lanetrace's own save: compare the file's `lastModified` and size to the values recorded right after writing.

### 6.2 Merging

Lanetrace keeps a **base**: the row values as of its last read or write, stored as a per-row hash in the `Lanetrace` sheet (column E) and as full values in memory.

When the file changes on disk:

1. Read the new rows (**theirs**). Lanetrace's current rows, including edits not yet saved, are **mine**.
2. Match rows by Row ID.
3. For each row and field:
   - Same in theirs and mine: nothing to do.
   - Changed only in theirs: take theirs.
   - Changed only in mine: keep mine.
   - Changed in both, to the same value: nothing to do.
   - Changed in both, to different values: a **conflict**.
4. Rows only in theirs are new rows from Excel. Rows in base but missing from theirs were deleted in Excel, unless mine changed them; that's a conflict too.
5. With no conflicts, apply the merge, redraw, and save if mine had unsaved changes.
6. With conflicts, show one dialog listing each row and field, Excel's value and Lanetrace's value, with "Keep Excel's", "Keep mine" and "Keep all mine" buttons. Nothing saves until it's resolved.

This is a three-way merge keyed by Row ID, not a CRDT. A plan has rules (an end date is required, Depends On must point at real rows), and a per-field merge keeps those rules checkable after every merge.

### 6.3 Writing

- Write with `createWritable()` on the workbook's handle. It writes a temporary file and swaps it in, so a crash mid-save can't leave a half-written workbook.
- If the write fails because Excel or OneDrive holds the file, keep the bytes (as `PENDING_CACHE` does today), show the waiting state, and retry on the next change, on focus, and every 10 seconds.
- Before overwriting, compare `lastModified` to the base. If the file changed since the last read, merge first (6.2), then write.

## 7. Writing the workbook without breaking it

### 7.1 Patch, don't regenerate

`exportExcel` builds a fresh template workbook with `generateXlsxTemplate`. Saving that over a user's workbook would throw away their formatting, extra sheets, formulas, comments and column widths. The save path must patch instead:

- Unzip the current file (the existing `unzip`).
- Replace only `xl/worksheets/sheetN.xml` for `TimelineExcel`, writing cell values into the existing rows and columns and keeping each cell's style index (`s=`).
- Add or replace the `Lanetrace` sheet, its relationship in `xl/_rels/workbook.xml.rels`, its entry in `xl/workbook.xml` and `[Content_Types].xml`.
- Write strings inline (`t="inlineStr"`) so `sharedStrings.xml` stays untouched.
- Leave every other part byte for byte.
- Zip and write.

### 7.2 Cells Lanetrace never touches

Formulas in the `TimelineExcel` sheet stay as formulas: if a cell has `<f>`, Lanetrace reads its cached value and doesn't overwrite it. Editing such a field in Lanetrace shows "This cell is a formula in Excel" and blocks the edit.

### 7.3 Fallback save

Without file write access, Save downloads the patched workbook under the same name. The user replaces their file. The result is still one file with everything inside.

## 8. Permissions and the browser

- File access needs a user gesture, so Open or Drop starts it. Drag and drop uses `DataTransferItem.getAsFileSystemHandle()` to get a writable handle.
- A file opened from a `file://` page works in Edge and Chrome. Managed work browsers can turn file access off by policy. Lanetrace detects that (`showOpenFilePicker` missing or throwing) and falls back to 7.3.
- Inside a sandboxed frame (the case `sandboxBlocked()` already handles, such as a SharePoint or Teams viewer), file access isn't available. Opening works read-only and Save is hidden, as downloads are today.
- Nothing leaves the computer. There are no new network calls.

## 9. Moving from linked folders

- When a linked folder has `program-timeline.state.json` and `program-timeline.data.xlsx`, Lanetrace offers "Save as one workbook". It merges the overlay into rows (matching overlay keys with `rowKeyFor` one last time), writes the settings into the `Lanetrace` sheet, saves to a file the user names, and opens that file live.
- The folder files stay where they are until the user deletes them. Lanetrace stops writing to them once the plan is live in a workbook.
- Sync folders remain supported for plans that aren't moved.

## 10. Interface changes

- Settings > Data & sync gains Open plan, the live file's name and state (Live, Waiting for Excel, Read-only, Reconnect), and Save as one workbook.
- The header shows the file name next to the date range, with a small status dot.
- Export Excel stays as a separate action that downloads a clean copy.
- The user guide (press ?) gets an "Open and save a plan" section, replacing most of the sync-folder text.

## 11. Tests

Automated (Playwright, like the existing scripts):

1. Round trip: open the sample workbook, change 3 fields, save; reopen in a new profile; the rows and every saved setting match.
2. Preservation: a workbook with custom fonts, fills, column widths, a formula column and an extra sheet keeps all of them, byte for byte outside `TimelineExcel` and the new sheet.
3. Merge: base, then Excel changes row 3's end date while Lanetrace changes row 5's title: both land, no prompt.
4. Conflict: both change row 3's end date: one prompt, and nothing written until it's answered.
5. Rename: change a title in the file; the row keeps its Row ID, order and Shown state.
6. Lock: hold the file open with a write lock; the edit waits, then saves after release.
7. Missing Row IDs and duplicates: filled and fixed on save, with the toasts.
8. Migration: a folder with state and data files becomes one workbook with the same look.

Manual, on the dev tenant or a personal machine only:

9. Excel desktop on Windows: open, edit, save; Lanetrace reloads. Excel keeps the veryHidden sheet.
10. Excel for the web and Excel on Mac: the veryHidden sheet survives a save.
11. A workbook in a OneDrive-synced folder: Lanetrace sees OneDrive's updates and waits through OneDrive's lock.

## 12. Risks

| Risk | Mitigation |
|---|---|
| Patching XML corrupts a workbook Excel then repairs | Test 2 on a corpus of real-world workbooks; write only well-formed parts; keep a copy of the last good bytes in memory and offer "Undo last save" |
| Excel for the web drops veryHidden sheets | Test 10; if it does, fall back to a custom XML part (`customXml/item1.xml`), which Excel also keeps |
| OneDrive conflicts create "Intake-Approach-PC-NAME.xlsx" copies | Detect a sibling with that pattern when the folder is visible and warn |
| Work browsers block file access | Fallback save (7.3) still produces one file |
| A user edits the hidden sheet with a macro | Validate the JSON; on failure, use browser settings and warn |

## 13. Phases

1. **4.18.0: one file.** The `Lanetrace` sheet, Row ID keys, patch-on-save, Open and Save, migration. No watching yet: Lanetrace re-reads the file on focus.
2. **4.18.1: live.** `FileSystemObserver`, the three-way merge and the conflict dialog.
3. **Later, if wanted:** a read-only "presenter" mode that opens the workbook and hides editing, for leaders viewing a plan.

## 14. Open questions

1. Should saving to the user's own workbook be the default, or should Lanetrace ask "Save edits back to this workbook?" the first time? Proposed: ask once per file.
2. Should the Data view column layout live in the workbook (shared) or the browser (personal)? Proposed: browser, since it's a viewing preference.
3. Should the `Lanetrace` sheet keep a short change log (who saved, when) for audit? Proposed: no; Excel's version history covers it.
