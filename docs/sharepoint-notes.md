# Lanetrace SharePoint Compatibility

Notes on using Lanetrace with SharePoint and OneDrive.

## Compatibility Status

**Tested and Working:**
- Opening the HTML file in SharePoint's OneUp viewer, to view and present (see "Verified in OneUp" below for what works there)
- Downloading and using locally
- Storing the HTML file in a document library

**Not Tested / Uncertain:**
- Embedded WebPart experience
- Bi-directional sync with SharePoint lists
- Power Automate integration

**Known Limitations:**
- Nothing persists in the SharePoint preview: edits, settings and the Data view layout reset when the file is opened again
- Downloads, printing and browser prompts are blocked in the preview (iframe sandbox), so no export leaves it
- File System Access API (sync folder) does not work in the preview

## SharePoint OneUp Viewer

When you click an HTML file in a SharePoint document library, it opens in the OneUp viewer. The viewer runs the file in an iframe with `sandbox="allow-scripts"` and nothing else. That means no `allow-same-origin` (no storage), no `allow-downloads`, no `allow-modals` (no print, `prompt` or `confirm`) and no `allow-popups`. Lanetrace detects this and shows a "Limited mode" banner.

### Verified in OneUp, 2026-10-27

Tested in the 3nf7cm tenant (pmpmaker site, Documents › Lanetrace › lanetrace.html), with the build at commit `de62288`, in the Claude desktop app's browser pane (Chromium). The SHA-256 of the uploaded file matched the local build.

**Works:**
- The timeline and the Data view render, including the Limited mode banner
- Data view: the List | By swimlane switch, sub-swimlane groups and the hidden lane columns
- Editing a Data view cell (the change shows on screen for that session)
- Settings panel
- Status update: Copy prompt put the full prompt (5,677 characters) on the clipboard

**Doesn't work:**
- **Saving:** after reopening the file, both test edits were gone, the Data view was back to List and the theme had reset. No message says edits will be lost.
- **Export to Excel:** the toast said "Exported 22 items to Excel", but no file arrived
- **Download PNG:** the toast said "PNG downloaded!", but no file arrived
- **Download SVG:** no file and no message
- **Copy timeline image:** the clipboard is blocked (`NotAllowedError`). The toast said it downloaded the PNG instead, but no file arrived
- **Print / PDF:** no print dialog appeared
- **+ Column:** nothing happens (it uses `prompt`, which the sandbox blocks)

**Not tested here:**
- Export for PPT and Export to CSV. The clicks missed; both use the same download path as Excel and PNG, so expect them to be blocked too
- Loading a file (file picker or drop), Planner and Smartsheet imports, Match the columns
- Dragging rows between swimlanes. The browser pane's simulated drag doesn't start HTML drag and drop
- Microsoft Edge and Chrome themselves. The sandbox flags are the same in any browser, so downloads, storage and print should be blocked there too

**Changed in 4.15.0, after this test:** the banner now says edits made in the preview aren't saved, and the export, download, print and copy-image buttons say they're blocked instead of reporting success. + Column and every confirmation (Delete item and others) use Lanetrace's own dialogs, since the sandbox blocks the browser's `prompt` and `confirm`. Not yet re-tested in OneUp.

**Bug found and fixed (commit `08478ac`):** pressing Enter in a Data view cell threw `NotFoundError: Failed to execute 'remove' on 'Element'`, which SharePoint showed as "Some content didn't load. A script didn't run correctly on this page." The same fault made Escape save an edit instead of cancelling it. It happened in every browser; OneUp was just the only place that showed the error.

### Workaround

For full functionality in SharePoint:

1. Click the **...** menu on the file
2. Select **Download**
3. Open the downloaded file in your browser

This runs the file outside the iframe sandbox, enabling all features.

## CSP Compliance

Lanetrace is designed to work within SharePoint's Content Security Policy:

- No external resources at runtime (CSS/JS inline)
- No external fonts
- No tracking or analytics
- Only optional CDN: PptxGenJS for PowerPoint export. It loads from jsDelivr on the first PowerPoint export (pinned version, integrity-checked), never at startup. If it cannot load, the export downloads a PNG instead

## OneDrive Sync

If you store lanetrace.html in a OneDrive-synced folder:

1. The file syncs normally as any other file
2. Open it locally for full functionality
3. Changes to the HTML file itself sync back to OneDrive
4. Overlay edits (localStorage) are browser-local only

### Sync Folder Feature

The sync folder feature (Settings > Sync folder) works with OneDrive:

1. Pick a folder in your OneDrive
2. Lanetrace writes `program-timeline.state.json` to that folder
3. This file syncs to OneDrive
4. Other machines with the same OneDrive folder can pick up the state

**Note:** This only works in Chromium browsers (Chrome, Edge) when running the file locally. It does not work in the SharePoint preview.

## Future Considerations

### LiveData Pattern

SharePoint's OneUp viewer supports a LiveData manifest pattern that could allow Lanetrace to read data directly from SharePoint lists. This is documented but not implemented.

### Graph API Publishing

A separate skill/tool could publish Lanetrace HTML files to SharePoint via Microsoft Graph API. That tool is not part of this repo.

## Recommendations

For production use with SharePoint:

1. **Simple approach:** Store the HTML file in a document library. Users download and open locally.

2. **Team approach:** Store in a shared OneDrive folder. Use the sync folder feature to share settings.

3. **Don't claim "SharePoint integration"** unless you've implemented actual integration features. Hosting compatibility is not integration.
