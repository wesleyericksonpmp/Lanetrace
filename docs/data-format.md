# Lanetrace Data Format

This document describes the Excel column schema for Lanetrace timeline data.

## Excel Requirements

- The workbook must contain a sheet named **TimelineExcel**
- Data should be in a table format with headers in the first row. Header names must match the template exactly; Lanetrace warns on load when a required column is missing or rows are skipped for having no title
- Download the template from Settings > Download template for a pre-formatted starting point

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
| Start date | StartDate | Date | (blank) | Blank = milestone (single date) |
| % Complete | PercentComplete | Integer 0-100 | 0 | Progress indicator. Cells formatted as % in Excel (0.5 = 50%) are converted when every value in the column is 0-1. Values outside 0-100 are capped. |
| Owner | TaskOwner | Text | (blank) | Person or team responsible |
| Depends on | DependsOn | Text | (blank) | Row IDs of items that must finish before this one starts, separated by commas, semicolons or spaces (e.g., `3, 5`). Finish-to-start only. Lanetrace flags an item that starts before a predecessor finishes (the same day is fine), IDs that match no row, self-references and loops. It never moves dates. |
| Notes | Notes | Text | (blank) | Free text, supports Markdown |
| Item Type | ItemType | Text | Auto | Task/Phase/Milestone/Event/Decision |
| Label Position | LabelPosition | Text | Auto | Auto/Above/Below/Left/Right/Inside (any case) |
| Milestone Shape | MilestoneShape | Text | (type default) | diamond, diamond-tall, circle, triangle-right, triangle-down, triangle-down-large, star, flag (any case) |
| Bar Style | BarStyle | Text | Auto | Flat/Chevron/Pill. Auto gives tasks a chevron and phases rounded ends |
| Milestone Row | MilestoneTier | Integer 1-6 | (blank) | Moves a milestone, event or decision into that row of the strip above the timescale. Settings > Timeline > Milestones decides how it applies: In swimlanes ignores it, As set per item (the default) uses it, Above timeline moves every milestone up and uses it to pick the row |
| Shown | IsShown | Boolean | TRUE | FALSE, No, N, 0, Off or Hidden (any case) hides the item. Anything else, or blank, shows it. |
| Color Category | ColorCategory | Text | (blank) | Hex color override (e.g., #78BE20) |
| Title Color | TitleColor | Text | Auto | Auto/White/Dark or a hex color for the item label |

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
