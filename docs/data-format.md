# Lanetrace Data Format

This document describes the column schema for Lanetrace timeline data, loaded from Excel or CSV.

## Excel Requirements

- Lanetrace reads the sheet named **TimelineExcel**, as in the template. Without one, it reads Planner's Consolidated Data sheet, or else the first sheet
- Data should be in a table format with headers in the first row. Template header names load directly; capitals do not matter. Lanetrace warns on load when a required column is missing or rows are skipped for having no title
- Download the template from Settings > Download template for a pre-formatted starting point

## CSV

A CSV file with the same headers loads the same way: Lanetrace's own Export to CSV, Excel's CSV UTF-8, or text from another tool. Commas, semicolons or tabs all work; Lanetrace uses whichever the header row has. Quote a value that contains the separator or a line break. Write dates as 2026-03-31 or 3/31/2026.

## Planner, Smartsheet and other exports

A file whose headers don't match the template is mapped onto it on load.

- **Microsoft Planner:** use the plan's Export as Excel. Buckets become swimlanes and the plan name becomes the timeline title. Planner has no % complete, so Completed reads 100%, Not started 0%, and In progress the share of checklist items done, or 50% without a checklist. Checklist items are added to the notes.
- **Smartsheet:** use Export to Excel. Indented rows become swimlanes (the first level under the project row) and sub-swimlanes (the next level). Parent rows act as grouping, unless their children have no dates. A Duration of 0 makes a milestone. Predecessors become Depends on, and links to parent rows are dropped. A Health column with Smartsheet's colored balls maps across, with Blue as Auto.
- **Status, in either:** Complete sets 100%, Blocked makes Health Off Track, On Hold makes it At Risk, and Canceled rows are hidden.
- **Rows with no dates** load hidden, so they are in the Data view but not on the timeline.
- **Status, Priority, Labels and Goal** come in as Data view columns. They are not saved when the page reloads, and Export to Excel writes them as extra columns.
- **Anything else** opens Match the columns: pick which column holds the title, the dates, the swimlane and the rest, with a preview. Lanetrace remembers the choice for files with the same columns, and Change mapping in the load message reopens it.

Microsoft Project exports are not tested.

## Column Reference

### Required Columns

| Column | Internal Name | Type | Description |
|--------|---------------|------|-------------|
| Row ID | RowID | Integer/Text | Unique identifier for each row. Used for overlay edits. |
| Swimlane | Swimlane | Text | Top-level grouping (e.g., "Engineering", "Product") |
| Milestone/Task Title | Title | Text | Item label displayed on the timeline. The header must read exactly "Milestone/Task Title". |
| End date | EndDate | Date | Required for all items |

### Optional Columns

| Column | Internal Name | Type | Default | Description |
|--------|---------------|------|---------|-------------|
| Sub-swimlane | SubSwimlane | Text | (blank) | Nested grouping within a swimlane |
| Description | Description | Text | (blank) | What the item covers: scope, deliverables, definition of done. Shows under the title in the tooltip and item form |
| Start date | StartDate | Date | (blank) | Blank = milestone (single date) |
| % Complete | PercentComplete | Integer 0-100 | 0 | Progress indicator. Cells formatted as % in Excel (0.5 = 50%) are converted when every value in the column is 0-1. Values outside 0-100 are capped. |
| Owner | TaskOwner | Text | (blank) | Person or team responsible |
| Depends on | DependsOn | Text | (blank) | Row IDs of items that must finish before this one starts, separated by commas, semicolons or spaces (e.g., `3, 5`). Finish-to-start only. Lanetrace flags an item that starts before a predecessor finishes (the same day is fine), IDs that match no row, self-references and loops. It never moves dates. |
| Notes | Notes | Text | (blank) | Free text, supports Markdown |
| Updates | Updates | Text | (blank) | Running status log, newest first. Each entry starts with its date and time in brackets, one per line: `[10/29/2026 2:05 PM] Vendor confirmed the SFTP account`. Lanetrace adds new entries with the stamp; text without stamps reads as one entry |
| Item Type | ItemType | Text | Auto | Task/Phase/Milestone/Event/Decision |
| Label Position | LabelPosition | Text | Auto | Auto/Above/Below/Left/Right/Inside (any case) |
| Milestone Shape | MilestoneShape | Text | (type default) | diamond, diamond-tall, circle, triangle-right, triangle-down, triangle-down-large, star, flag (any case) |
| Bar Style | BarStyle | Text | Auto | Flat/Chevron/Pill. Auto gives tasks a chevron and phases rounded ends |
| Milestone Row | MilestoneTier | Integer 1-6 | (blank) | Moves a milestone, event or decision into that row of the strip above the timescale. If another pinned label already uses that row, the item moves up to the next row with room. Settings > Timeline > Milestones decides how it applies: In swimlanes ignores it, As set per item (the default) uses it, Above timeline moves every milestone up and uses it to pick the row |
| Shown | IsShown | Boolean | TRUE | FALSE, No, N, 0, Off or Hidden (any case) hides the item. Anything else, or blank, shows it. |
| Color Category | ColorCategory | Text | (blank) | Hex color override (e.g., #78BE20) |
| Title Color | TitleColor | Text | Auto | Auto/White/Dark or a hex color for the item label |
| Health | Health | Text | Auto | On Track/At Risk/Off Track (any case; Green, Amber, Yellow and Red also map). Blank is Auto: Lanetrace works it out from the dates and % complete (overdue = Off Track, behind pace = At Risk, otherwise On Track; complete items have none). Other text is kept as typed. An item marked On Track that is overdue shows in Needs attention, and At Risk or Off Track feeds the Status update rating |

## Item Type Inference

If ItemType is blank, Lanetrace infers the type:

| Condition | Inferred Type |
|-----------|---------------|
| StartDate blank, EndDate present | Milestone |
| StartDate and EndDate on the same day | Milestone |
| Duration of 92 days or less | Task |
| Duration over 92 days | Phase |

Explicit ItemType values override inference:
- `Task`
- `Phase`
- `Milestone`
- `Event`
- `Decision`

Matching ignores case and surrounding spaces (`task ` works). Unrecognized values fall back to inference.

For a one-day task drawn as a bar (a workshop or cutover day), set Item Type to `Task`. Otherwise a row with the same start and end date becomes a milestone.

Bars include their end date: a task from 2026-03-01 to 2026-03-31 runs through the end of March 31.

## Date Formats

Lanetrace accepts dates in these formats:
- ISO 8601: `2026-01-15`
- US format: `1/15/2026` or `01/15/2026`
- Excel serial numbers (handled automatically)

Unreadable or impossible dates (`15/01/2026`, `2026-02-30`, `TBD`) are not drawn. They show in the "needs attention" chip with the original value.

## Markdown in Notes

The Notes field supports basic Markdown:

```markdown
**bold** and *italic*

- Bullet list
- Another item

1. Numbered list
2. Second item

`inline code`
```

## Character Limits

| Field | Max Characters |
|-------|----------------|
| Timeline title | 50 |
| Subtitle | 80 |
| Swimlane name | 30 |
| Sub-swimlane | 30 |
| Item title | 100 |
| Description | Unlimited |
| Updates | Unlimited |
| Notes | Unlimited |

## Row ID Best Practices

The RowID column is critical for the overlay edit system:

- Use stable, unique identifiers
- Don't change RowIDs after initial load
- Recommended: sequential integers (1, 2, 3, ...)
- Alternative: meaningful codes (PROD-001, ENG-002)

If you re-import a file with changed RowIDs, existing overlay edits may be orphaned.

## Example Data

```
Row ID | Swimlane    | Milestone/Task Title | Start date | End date   | Item Type
-------|-------------|----------------------|------------|------------|----------
1      | Product     | Discovery Phase      | 2026-07-01 | 2026-09-30 | Phase
2      | Engineering | API Development      | 2026-09-15 | 2026-12-31 | Phase
3      | Product     | Requirements Sign-off|            | 2026-09-30 | Milestone
4      | Engineering | Architecture Review  |            | 2026-09-15 | Decision
5      | Marketing   | Launch Announcement  |            | 2027-03-01 | Event
```
