# Changelog

All notable changes to Lanetrace will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [4.13.0] - 2026-10-17

### Added
- CSV import: load a CSV with the template's headers, such as Lanetrace's own CSV export or Excel's CSV UTF-8, from Settings > Data & sync > Load file, the command palette, or by dropping it on the page. Commas, semicolons or tabs. With a sync folder linked, Lanetrace keeps an Excel copy of the loaded CSV
- Column headers match without regard to capitals, in Excel and CSV files
- "Needs attention" list: clicking the chip lists each flagged item with the reason, and clicking an item opens it. The Data view marks flagged rows with ⚠ in the Quick look column, with the reason on hover, and the marks update as you edit
- Duration on bars (Settings > Timeline > Timescale > Duration): adds each task's or phase's length after its title, such as "Integration Testing · 6 wks". Off by default. Auto uses days under 2 weeks, weeks up to 16 weeks, then months; Days, Weeks and Months fix the unit. Calendar days, counting the start and end day
- Timescale rows (Settings > Timeline > Timescale > Rows). Auto, the default, fits the date range: Month · Week up to about 3 months, Year · Quarter · Month up to about 2 years, Year · Quarter beyond that. You can also pick Year · Quarter · Month, Year · Quarter, Year · Month, Quarter · Month, Quarter · Month · Week or Month · Week. Labels shorten (January 2026, Jan, J) or drop out when space runs out. Grid lines follow the lowest row. Pill shows the lowest row
- Timelines longer than about 2 years now fit the default width. The month row kept the chart from shrinking below 2.4 px per day, so a 3-year portfolio scrolled sideways with about 18 months in view
- Settings panel: Expand all (or Collapse all once every section is open) in the header, and Collapse all and Expand all swimlane buttons in the Swimlanes section, next to Reset colors
- Elapsed time: an optional red bar along the bottom of the timescale from the start of the chart to today (Settings > Timeline > Timeline chrome)

### Changed
- A more compact, modern layout:
  - One-line header: title and dates, Timeline/Data, item count, then Filter, Key, Status update, Settings and a ⋯ menu with Help and the theme switch. The "Add a subtitle" hint shows only while hovering the title or editing
  - The legend row is now a Key button that opens the swimlane colors and item shapes. Settings > Timeline chrome > Key button shows or hides it
  - Larger bars and labels: Compact draws 20 px bars with 12 px labels (was 13 px and 10 px); Normal and Spacious grow to match
  - Rows are packed by default, so items share a row when they do not overlap. Saved timelines keep their row layout
  - On phones the header actions wrap, and Status update and Settings show only their icons. The ⋯ menu moves keyboard focus to its first item
  - Auto width fits the window, so the whole date range shows without sideways scrolling, and refits when the window is resized
- Today marker: the badge hangs from the bottom edge of the timescale with a notch at the exact date, in every density and timescale shape. Before, it drifted with density: over the month names in Ultra, inside the month row in Compact, half on and half off in Normal. The line is thin and solid and draws under the items, so it no longer strikes through labels
- Item form: Bar style and Marker shape are button pickers with drawn previews, matching the Data view, instead of text lists. Auto shows the shape the item will get
- Data view Quick look: the open and menu buttons appear when you hover over a row, select it or tab into it; the notes icon always shows. Touch screens always show them. The row menu adds Duplicate and Hide from timeline (Show on timeline for hidden items)
- Notes toolbar: a text style list (Normal, Heading 1, 2, 3) that applies to the current line, then bold, italic, numbered list and bullet list. Headings and numbered lists were missing from the 4.12.0 toolbar. The link button is removed for now, because links in notes could not be clicked anywhere in the app

### Fixed
- Two swimlanes could get the same automatic color, because a swimlane's color came from its name and the default palette listed one purple twice. Automatic colors now skip colors already in use, including ones you picked, and repeat only after all 7 are used. Swimlanes that did not collide keep their color
- Print / PDF cut the timeline off at the right edge, since it printed at full width; it now scales to the page. Printing in dark mode gave a dark page with a white band; printing now always uses light colors, like the image exports, and switches back afterwards. The Timeline/Data switch and the grey page background no longer print
- Milestones pinned to the same Strip Row overlapped until the labels could not be read. The Strip Row is now the lowest row an item uses: when another pinned label already sits there, the item moves up to the next row with room, and with all 6 rows full it keeps its marker and drops the label
- Sync folder: Lanetrace showed a folder link left by an older Timeline Studio build ("timelinev2" in one report). Browsers give every HTML file opened from disk the same storage, and both used the same names. Lanetrace now uses its own names: settings and edits saved under the old name move over once, and the folder link does not carry over, so you choose the folder once
- Sync folder: a linked folder that was moved or deleted stayed listed as linked. It is now unlinked on startup with a message. When the browser needs permission again, the status reads "Last used folder: … · the browser needs your permission again", with Reconnect, Choose a different folder and Disconnect
- A one-time tip after the first file load suggests linking a sync folder
- Text Row IDs such as ENG-004 loaded as "not a number" from Excel. They now stay as typed, as the template allows
- Loading a spreadsheet kept edits made in Lanetrace earlier and reapplied them to any row with the same title and swimlane, silently overriding the new file. Lanetrace now says how many edits would apply and asks whether to keep them or use the file as it is. Ctrl+Z brings them back. Restoring from the sync folder keeps them without asking
- Settings > Timeline > Milestones has three choices: In swimlanes (every milestone, event and decision stays in its swimlane, even with a Strip Row), As set per item (the default: only items with a Strip Row go above) and Above timeline. Before, "In swimlanes" could not bring down items with a Strip Row, so the toggle looked broken. A note under the choices counts items with a Strip Row and offers Clear Strip Rows. Saved timelines keep their look
- In Ultra density the first row started 4px inside the timescale. The space under the timescale is now the same in every density, with room for the Today badge
- Labels set to Above or Below ran past the right edge of the chart when their item sat near it, such as "General Availability · Mar 15" at narrower widths. Centered labels now slide inward at both edges
- Labels set to Left on items at the start of the timeline ran into the swimlane names. They now move to the right of the item when the left has no room
- Data view in dark mode: the selected row was light blue with white text, which was unreadable. It now mixes the theme's accent into the dark background, with at least 4.5:1 text contrast in every theme pack
- Data view: editing a cell showed the old text and the edit box stacked, doubling the row height. The edit box now replaces the text
- Data view: the row menu (⋮ or right-click) opened below the bottom of the window for rows near the bottom, hiding Delete and the other options. It now opens upward when there is no room below and always stays inside the window
- Duplicate gave the copy no Row ID, which broke Depends on references and Excel round trips. Copies now get the next Row ID
- Numbered lists in notes showed as unnumbered items in tooltips and the item card

## [4.12.0] - 2026-10-15

First public release.

### Added
- Dependencies: a Depends on column takes the Row IDs of items that must finish before an item starts. It is in the Excel template, import, export, the Data view and the item form, where a picker adds items by title and each predecessor shows its finish date
- The "needs attention" chip flags an item that starts before a predecessor finishes, a Row ID that matches no item, an item that depends on itself, and loops. Lanetrace never moves dates
- Status update prompt: dependency conflicts go to DATA TO CONFIRM, a new DEPENDENCY RISKS block lists open items waiting on overdue or behind-pace work, and each item lists what it depends on
- Sample data and the template's sample rows include dependencies
- Dependency arrows, off by default. Turn them on with "Dependency arrows" in Settings > Timeline or from the command palette. Arrows run from the end of each predecessor to the start of its successor, go under the items, and leave milestones from the top or bottom so they miss the label. Hovering over or tabbing to an item highlights its whole chain and fades the rest. A conflict draws red and dashed. Items in a collapsed swimlane connect through its summary bar. Exports include the arrows when they are on
- Swimlane labels wrap by default (Wrap, Shrink or Truncate), with adjustable label and sub-swimlane widths
- Swimlanes without sub-swimlanes show a wide horizontal label; with sub-swimlanes the name runs vertically in the tab
- Timescale shapes: Classic, Rounded, and a single-row Pill
- Option to place milestones, events, and decisions in a strip above the timescale
- Today marker follows the system date and moves at midnight; optional custom "as of" date in Settings
- Font picker using system fonts only (applies to the page and all exports)
- Fine-tune sliders for row height, swimlane spacing, and timeline width on top of the density presets, plus a reset-display icon
- Status update dialog: builds a copyable prompt from current dates and progress, and copies the timeline image for email
- Load warnings for missing required columns and for rows skipped because they have no title
- Labels that cross the edge of the progress fill draw in two colors, one for each background
- Unreadable or impossible dates show in the "needs attention" chip with the original value
- Timeline items are keyboard accessible: Tab reaches each item, its tooltip shows on focus, Enter or Space opens the editor, and screen readers announce title, type, dates and % complete
- Closing the item editor returns focus to the item that opened it, including after a save
- Collapsible swimlanes: click a swimlane name (or Tab to it and press Enter) to show it as one summary bar with its date span, item count and duration-weighted % complete. Click the summary bar to expand. "Collapse all" and "Expand all" are in the command palette (Ctrl+K). The chevrons stay out of exports and print
- Row layout "Auto", the new default: one item per row up to 100 items, then Compact. At 2,000 items this cuts the chart from about 56,000 px tall to 11,500 px. Existing saved choices are kept
- Favicon: the lane mark from the design notes
- Swimlane label color: an "Aa" button in Settings > Swimlanes cycles Auto, White and Dark for each swimlane, previews the result on the lane color, and warns below 3:1 contrast. Undo, rename and Reset colors cover it
- Redesigned Excel template:
  - Styled header (blue = required, ink = optional), sized columns, no gridlines, frozen header and title columns, filter buttons, landscape print fit to one page wide
  - Dropdowns for Item Type, Label Position, Milestone Shape, Bar Style, Shown and Title Color; range checks for dates, % Complete and Milestone Row; a hint on every column when a cell is selected
  - Real date and number cells instead of text
  - Sample rows dated from the current month, covering every item type
  - Instructions sheet: quick start, column reference with accepted values and examples, item types, how Lanetrace reads your data, glossary, and load messages with fixes
- Export to Excel and Export to CSV (Settings > Export & share, the Data view toolbar, and the command palette). Exports every item with your edits, hidden items as Shown = No, and custom columns. The Excel file uses the template layout and loads back into Lanetrace unchanged; the CSV is UTF-8 for other tools
- Initial open-source release
- Generic sample program for the built-in demo data
- Documentation for data format, design, and SharePoint compatibility

### Changed
- Flatter interface: 3px corners on every card, button, input, menu and dialog (they were 4 to 14px), and hairline borders instead of soft shadows. Focus rings are unchanged
- Dark mode uses dark scrollbars and dark date pickers
- A completed milestone, event or decision shows a check before its date ("Sign-off · ✓ Sep 30") instead of a small green dot on the marker, which read as a stray mark and relied on color alone
- The Notes toolbar has four buttons: bold, italic, bullet list and link, the formatting the tooltip and template describe. Headings, numbered lists and code still render when typed
- Bar label color follows contrast with the item's color instead of % complete. Every label in every theme pack, light and dark, now reaches 4.5:1
- Each theme pack has fixed bar label colors: white plus the pack's darkest color. Dark mode previously used one light color for both
- Fifteen lane colors in the Nord, Dracula, Solarized, Warm, Catppuccin, Gruvbox and Slate packs shifted 3-12% lighter or darker to reach 4.5:1. Corporate is unchanged
- Bars run through the end of their end date, and a chevron's point lands on that date
- Items with the same start and end date are milestones when Item Type is blank. Set Item Type to Task for a one-day bar
- Item Type matching ignores case and surrounding spaces. Unknown values fall back to inference instead of drawing as milestones
- Excel cells formatted as % (0.5 = 50%) import correctly, and % Complete is capped at 0-100
- Shown accepts No, N, Off and Hidden in any case to hide an item. Anything else, or blank, shows it
- The PowerPoint export library loads on the first PowerPoint export instead of at startup, pinned to 3.12.0 with an integrity check. The app makes no network requests until then. If the library cannot load, the export downloads a PNG and says why
- Settings footer shows the license instead of a contact email
- Editing a cell in the Data view rebuilds only that row, and the hidden timeline redraws once edits pause. At 2,000 rows an edit takes about 50 ms instead of about 1.1 s
- Data view Notes show as one line of plain text, with the full note on hover, so rows keep an even height
- The legend shows on the Timeline tab only
- Swimlane labels pick white or dark text from each theme's own colors, matching bar labels, instead of a fixed dark color
- Label Position and Milestone Shape import in any case ("above", "Flag")
- Status update prompts:
  - The app computes the suggested Red/Yellow/Green rating from a fixed rule and states the reason, instead of asking the model to judge "several items behind pace"
  - Each audience gets its own format: the executive brief asks for exactly five labeled lines, steering committee leads with decisions needed, project team lists owners and next steps
  - New KEY DATES block with the next milestone and next decision, even when they fall outside the look-ahead window
  - "Coming up" includes in-progress work due inside the window, not only work that starts in it
  - "Decisions not yet made" listed only the look-ahead window; it is now "Decisions due through <date>" plus the count of all open decisions
  - New DATA TO CONFIRM block, checked by rule: progress recorded before an item starts, progress far ahead of the schedule, partial progress on single-date items, and open items with no owner. The model turns these into questions for the program manager
  - Asks for a subject line, and writes [Program name] when the timeline title is still the default instead of calling the program "Lanetrace"
- Milestone Row now moves a single milestone, event or decision into the strip above the timescale even when Settings keeps milestones in their swimlanes. "Auto" follows Settings
- Sub-swimlane suggests the sub-swimlanes already used in the chosen swimlane, in the item editor and in the Data view; typing a new one still works
- One item form everywhere. The Data view's slide-out detail panel is gone; its open button, the timeline, the command palette and New item all use the same form, with the same validation (a Phase needs a start date, every item needs an end date)
- The item form is grouped into sections: item (Title first, type, owner, swimlane), Schedule (dates and % complete on one row), Notes, Appearance (folded, with a one-line summary; opens on its own when the item is hidden or its title has low contrast) and More fields (custom Data view columns). Bar Style is now in the form
- Data view: "Add row" is now "New item" and opens the form titled "New item", pre-filled with the swimlane you filtered to. The new row gets the next Row ID and is selected and scrolled into view. Insert above and below still add blank rows for inline editing
- Data view: a Quick look column after Title holds Open item, a notes indicator with a preview on hover, and the row menu. Delete moved into the row menu (and stays on the toolbar and the Delete key) so it is no longer one click away at the far right
- Data view: Bar Style cells stay on one line, so every row has the same height
- Product name from "Timeline Studio" to "Lanetrace"
- Sample swimlanes to generic: Product, Engineering, Operations, Marketing, Customer Success

### Fixed
- Unreadable dates (15/01/2026, TBD) and impossible dates (2026-02-30) no longer blank the whole timeline
- An inferred item type no longer sticks after the dates change, for example a milestone that gains a start date
- Lowercase "true" or "Yes" in the Shown column no longer hides the item on import
- Grid Shown checkbox matches the timeline: blank shows as checked, lowercase "no" as unchecked
- Data format docs named the title column "Title"; the template and parser use "Milestone/Task Title"
- Tab no longer lands on invisible controls inside closed panels and dialogs (38 controls before)
- Help and the Excel template no longer claim drag-to-reposition or right-click on timeline items, which do not exist
- The template downloads as Lanetrace-Template.xlsx instead of TimelineStudio-Template.xlsx
- The template's sample milestones put their date in Start date, so they did not draw. Its instructions also had the date rules reversed, listed bar styles and shapes the app does not have, and called Color Category a name instead of a hex color
- Milestone Shape and Bar Style columns were ignored on import
- Copy to Clipboard crashed with "Cannot read properties of undefined (reading 'write')" where the browser offers no clipboard (previews, non-secure pages, some locked-down browsers). It now downloads the PNG and explains why
- Items imported with dates like 3/31/2026 opened in the item form with empty date boxes, and saving would erase the dates. The form now converts them
- Items added through the item form had no Row ID
- The item editor's "New swimlane" field was never read on Save. Choosing "＋ New swimlane…" in the Swimlane list now shows a name field under it, and Save creates the swimlane and moves the item into it
- Sub-swimlane separator lines no longer cut across the colored swimlane tab
- PNG, clipboard, and PowerPoint exports now write all styles inline, removing black bands and serif text
- Date-only values no longer display one day early in US time zones
- PNG, clipboard, and PowerPoint exports now keep bands, grid lines, and fonts

### Removed
- Prototype sample data, replaced by the generic sample program

## [0.1.0] - 2026-09-24

### Added
- Core timeline rendering engine
- Excel import via drag-drop
- Data grid view with inline editing
- Undo/redo system
- Command palette (Ctrl+K)
- SVG, PDF, PowerPoint exports
- Theme support (light/dark/auto)
- Density options (ultra/compact/normal/spacious)
- Theme packs
- File System Access API sync (Chromium)
- Accessibility improvements
