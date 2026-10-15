# Lanetrace SharePoint Compatibility

Notes on using Lanetrace with SharePoint and OneDrive.

## Compatibility Status

**Tested and Working:**
- Opening the HTML file in SharePoint's OneUp viewer
- Downloading and using locally
- Storing the HTML file in a document library

**Not Tested / Uncertain:**
- Embedded WebPart experience
- Bi-directional sync with SharePoint lists
- Power Automate integration

**Known Limitations:**
- File System Access API does not work in SharePoint preview (iframe sandbox)
- Export buttons may be blocked in preview mode

## SharePoint OneUp Viewer

When you click an HTML file in a SharePoint document library, it opens in the OneUp viewer. This is an iframe-based preview.

### What Works
- Timeline rendering
- Theme switching
- Settings panel
- Data grid view
- Print (opens browser print dialog)

### What Doesn't Work
- File System Access API (sync folder feature)
- Some exports may fail due to CSP restrictions
- localStorage may not persist across sessions

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
