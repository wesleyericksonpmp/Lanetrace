# Lanetrace Product Standards

**Version:** 1.1
**Date:** 2026-09-24
**Change log:** v1.1 adds positioning and screenshot/demo standards from the design review.
**Status:** Draft

---

## 1. Product Principles

### Local-First
- No server required for core functionality
- User data stays on their machine unless they choose otherwise
- Works offline after initial load
- No telemetry, analytics, or tracking

### Single-File Simplicity
- The application is one HTML file
- No build step for end users
- Drag-and-drop deployment anywhere
- Zero installation, zero IT approval

### Excel-Native
- Excel is the primary data input format
- Column schema is documented and stable
- Template generation helps users get started
- Don't fight Excel; embrace it

### Honest About Capabilities
- Don't call it "Gantt" without dependency arrows
- Don't claim "integration" for simple compatibility
- Be clear about what exports actually produce
- Document limitations alongside features

### Boardroom Ready
- Default styling should look professional
- Print/PDF output should need no adjustment
- SVG exports should be high resolution
- PowerPoint exports should be usable as-is

### Positioning
- **What it is:** a single HTML file that turns an Excel sheet into a swimlane timeline for a project, program or portfolio, with a data grid and boardroom-ready exports
- **What it is not:** a dependency-driven schedule with critical path or resource leveling. That boundary is a feature
- **Who it is for:** a PM or PMO who needs a steering committee visual, not a new system of record
- **Positioning line:** "Timelines, without the platform." Alternates: "Your data. Your timeline. One file." / "The timeline tool with no login screen." / "From spreadsheet to steering committee."

---

## 2. Naming Rules

### Product Name
- **Lanetrace**: one word, capital L
- Never "Lane Trace" or "LaneTrace" or "lanetrace"
- Tagline: "Timelines, without the platform." Secondary: "From spreadsheet to steering committee."
- Wordmark (brand materials): lowercase monospace "lanetrace" is a logo treatment only; in text, always "Lanetrace"

### Technical Names
- Use PascalCase for column names: `RowID`, `StartDate`, `ItemType`
- Use camelCase for JavaScript variables: `swimlaneData`, `itemType`
- Use kebab-case for CSS classes: `data-grid`, `item-bar`
- Use kebab-case for file names: `data-format.md`, `design-notes.md`

### UI Labels
- Use sentence case for buttons: "Download SVG", "Add item"
- Use title case for section headers: "Timeline View", "Data Grid"
- Use lowercase for shortcuts: "Ctrl+K", "Esc"

### Swimlane Terminology
- "Swimlane": not "lane" or "track" or "row"
- "Sub-swimlane": for nested groupings
- Plurals: "swimlanes", "sub-swimlanes"

### Item Terminology
- "Item": generic term for anything on the timeline
- "Phase": bar with duration, represents a span of work
- "Task": bar with duration, smaller than phase
- "Milestone": single point in time, no duration
- "Event": milestone variant, external occurrence
- "Decision": milestone variant, choice point

---

## 3. UX Rules

### Interactions
- Single click selects an item
- Double-click or Enter edits an item
- Right-click shows context menu (where supported)
- Escape closes any modal/panel
- Undo is always Ctrl+Z, Redo is Ctrl+Y

### Feedback
- Every action shows a toast confirmation
- Error states are red, success states are green
- Loading states show visual indicator
- Empty states explain what to do next

### Modals
- Focus traps inside modal when open
- Escape key closes modal
- Click outside closes modal (with backdrop)
- Primary action on the right

### Settings
- Changes apply immediately (no "Apply" button)
- Settings persist across sessions
- "Reset" options available for customizations
- Dangerous actions require confirmation

### Accessibility
- All interactive elements keyboard accessible
- Buttons have visible labels or aria-labels
- Color is not the only indicator of state
- Focus states are visible

---

## 4. Data Privacy Rules

### No Data Collection
- No analytics scripts
- No error reporting services
- No usage tracking
- No cookies for tracking

### Local Storage Only
- localStorage for preferences
- File System Access API for sync (opt-in)
- No cloud storage unless user configures

### User Control
- User chooses if/where to save
- "Clear my edits" removes all modifications
- No hidden data retention
- Export is always available

### Sample Data
- Sample data is clearly fictional
- No real company names, dates, or projects
- Generic industry-neutral terminology
- Can be replaced/cleared easily
- Realistic, not placeholder: named phases, dated milestones, and role-based owners ("Program Sponsor") read as credible. "Task 1, Task 2" does not

---

## 5. Feature Quality Rules

### Before Shipping a Feature

1. **Works in all themes**: light, dark, and auto
2. **Works at all densities**: ultra, compact, normal, spacious
3. **Works without network**: no CDN dependency for core function
4. **Undoable if destructive**: delete, edit, clear all have undo
5. **Has keyboard support**: accessible without mouse
6. **Has feedback**: toast or visual state change
7. **Documented**: in help modal or docs

### Feature Flags
- Use `data-*` attributes for feature detection
- Don't ship half-implemented features
- Use version number in output files

### Performance
- Initial render < 2 seconds
- Interactions respond < 100ms
- Export operations show progress for > 1s
- No jank during drag operations

---

## 6. Release Readiness Checklist

### Pre-Release
- [ ] All "Keep v1" features verified working
- [ ] Sample data is generic (no client references)
- [ ] README is complete and accurate
- [ ] LICENSE file present
- [ ] data-format.md documents all columns
- [ ] Screenshots show generic data
- [ ] Browser compatibility tested (Chrome, Edge, Firefox, Safari)
- [ ] Print/PDF tested
- [ ] All exports tested

### Sanitization
- [ ] No employer, client or vendor names
- [ ] No industry-specific terminology in samples
- [ ] No real dates tied to real projects
- [ ] No internal team/person names
- [ ] Only the Corporate pack carries the author's work palette; every other pack is neutral

### Documentation
- [ ] Quick start in README
- [ ] Data format documented
- [ ] Known limitations documented
- [ ] SharePoint notes are honest (compatible, not integrated)
- [ ] Roadmap is realistic

### Git
- [ ] Clean commit history
- [ ] No sensitive files in repo
- [ ] .gitignore covers node_modules, *.log, etc.
- [ ] CHANGELOG started
- [ ] Version tag applied

### Screenshots and Demo
- [ ] Hero shot shows the export, not only the editor
- [ ] Browser chrome cropped out; frame the app only
- [ ] One theme in the hero shot, one alternate further down
- [ ] README demo loop is silent and under 10 seconds: drop .xlsx, render, drag one bar, export
- [ ] Data in every image passes the Sanitization list above
- [ ] No stock photography anywhere
- [ ] README badges are verifiable claims only (single file, zero dependencies, MIT license, works in SharePoint); no stars/forks badges at launch

---

## 7. Version Numbering

### Format: MAJOR.MINOR.PATCH

- **MAJOR**: Breaking changes to data format or major feature removal
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, minor improvements

### Version Display
- Footer of app shows version
- Exported files include version in filename
- State JSON includes version for migration

### Pre-Release
- Use "pre-release" or "early prototype" in marketing
- Don't claim stability until v1.0

---

## 8. Attribution

### Creator Credit
- "Created by Wesley Erickson"
- Link to pmpmaker.com where appropriate

### License
- MIT license (permissive, simple)
- Include full license text
- Note in README

### Dependencies
- Document all embedded libraries
- Respect library licenses (PptxGenJS, Lucide icons)
- CDN sources documented
