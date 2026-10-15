# Lanetrace Design Notes

Design philosophy and visual system documentation.

## Design Philosophy

### Quiet Chrome, Loud Data
The UI should recede. Timeline bars, milestones, and swimlanes are the focus. Controls, settings, and navigation should be present but not prominent.

### Professional by Default
A new user should be able to export a timeline and present it immediately. No design skills required. Default colors, spacing, and typography should be boardroom-ready.

### Local-First
No accounts, no cloud, no dependencies. The tool works offline and stores nothing externally.

### Looks Like a Tool, Not a Product Demo
Sharp edges, flat surfaces, one confident accent, monospace where precision matters. Avoid the generic SaaS look (blue gradients, heavy rounded corners, soft floating shadows); that look belongs to the tools Lanetrace is the alternative to.

## Visual Identity

### Brand Palette vs. Theme Packs
The brand palette is what Lanetrace itself looks like: logo, README, landing page. Theme packs (below) are user-selectable so a deployed timeline can match whoever is presenting it. Keep the two separate.

| Token | Hex | Role |
|-------|-----|------|
| Ink | #14181F | Primary text, wordmark |
| Paper | #FAF9F5 | Background; warm off-white reads as paper and makes exports look printed |
| Trace Blue | #1E4FD8 | Brand accent; sharper than the in-app Corporate Blue (#0072CE) |
| Milestone | #E2A32D | Milestone diamond in the mark |
| Danger | #B23B32 | Destructive actions in brand materials |

Open decision: whether Paper and Trace Blue become the app's default theme or stay brand-only.

### Logo Direction
| Concept | Use |
|---------|-----|
| A. Lane mark: three lanes at different lengths, one milestone diamond breaking the top bar | Icon, favicon, GitHub avatar |
| C. Wordmark: monospace "lanetrace", milestone diamond as the period | README header and app chrome, paired with A |
| B. Trace mark: stepped line crossing three lane rules | Optional loading animation only; weak at small sizes |

## Visual System

### Color Palette

**Swimlane Colors (6-color cycle)**
| Name | Hex | Usage |
|------|-----|-------|
| Blue | #0072CE | Primary swimlane |
| Teal | #0092BC | Secondary |
| Purple | #743A83 | Tertiary |
| Green | #78BE20 | Success, completion |
| Navy | #00417D | Headers, emphasis |
| Amber | #F2A900 | Warnings, decisions |

**Semantic Colors**
| Name | Hex | Usage |
|------|-----|-------|
| Red | #D50032 | Errors, destructive |
| Green | #78BE20 | Success, complete |
| Amber | #F2A900 | Warning, at-risk |

### Typography

System font stack:
```css
-apple-system, "Segoe UI", Inter, system-ui, sans-serif
```

No custom fonts required. This ensures fast loading and consistent rendering across platforms.

**Monospace for precision (proposed).** Use a local monospace stack (`ui-monospace, Consolas, monospace`) for the wordmark, section labels, and anything tabular: dates, percentages, IDs. Brand materials may use IBM Plex Mono or JetBrains Mono (open license); the shipping app must never request a webfont. Do not use monospace for paragraph copy.

**Tabular figures.** Numbers in the data grid use `font-variant-numeric: tabular-nums` so dates and percentages line up. Applied today to Row ID, % Complete, Start, and End.

**Type Scale**
| Element | Size | Weight |
|---------|------|--------|
| Timeline title | 20px | 700 |
| Swimlane label | 12px | 600 |
| Item label | 11-13px | 400-600 |
| Settings labels | 11px | 700 |
| Body text | 13-14px | 400 |

### Spacing

**Density Levels**
| Level | Row Height | Use Case |
|-------|------------|----------|
| Ultra | Smallest | Many items, overview |
| Compact | Small | Default for most uses |
| Normal | Default | Readability focus |
| Spacious | Large | Presentations, accessibility |

### Item Rendering

**Task/Phase Bars**
- Three end-cap styles:
  - Flat: square ends
  - Chevron: pointed right end
  - Pill: rounded both ends
- Labels inside bar if space permits, otherwise to the right
- Percent complete shown as text label past bar end

**Milestone Markers**
| Shape | SVG | Use Case |
|-------|-----|----------|
| Diamond | Standard | Default milestone |
| Hollow Diamond | Stroke only | Decision points |
| Star | 5-point | Launches, releases |
| Triangle | Directional | Events, kickoffs |
| Circle | Round | Alternative |

## Theme System

### Theme Contract (19 variables)
Every pack defines all 19 variables in both light and dark, even when values repeat. A missing variable silently renders part of the chrome as nothing.

| Variable | Role |
|----------|------|
| `--bg`, `--card`, `--text`, `--muted`, `--border`, `--shadow` | Surfaces, ink, hairlines, elevation |
| `--blue` | Primary accent (focus, active tab, primary button, hover). Load-bearing |
| `--red` | Danger (delete, clear, remove). Load-bearing |
| `--teal`, `--green`, `--navy`, `--purple`, `--amber` | Categorical palette; feeds swatch presets via `getColorPresets()` |
| `--grid-minor`, `--grid-major`, `--band` | Chart gridlines, swimlane tint, grid row hover |
| `--scale-year`, `--scale-q`, `--scale-mo` | Date-scale header rows, darkest to lightest |
| `--bar-text-dark`, `--bar-text-light` | Label ink on light vs. dark bar fills |

### Packs
Nine packs ship, each with light and dark variants. Every pack's bar label colors and lane colors are tuned so labels reach 4.5:1 contrast.

| Pack | Mood |
|------|------|
| Corporate (default) | Cool gray, safety blue, broad series |
| Slate | Tonal ladder of one hue; print-safe |
| Warm | Cream, umber, terracotta |
| Nord | Arctic blues and muted accents |
| Solarized | Classic Solarized palette |
| Catppuccin | Soft pastels |
| GitHub | GitHub's light and dark palettes |
| Gruvbox | Retro warm tones |
| Dracula | High-contrast purples |

### Adding a Pack
1. Add a pack with `addPack()` (or an entry in the `THEME_PACKS` object) with light and dark variable sets and a lane color list.
2. Add its dark bar-label color to `BAR_TEXT_DARK`, and check every lane color reaches 4.5:1 with white or that dark color.
3. Verify both modes, every density, and all exports.

### Theme Rules
- A pack is text inside the one file, never a `<link>`.
- `--blue` and `--red` always get real, contrasting values.
- Do not rename variables in a theming change. `--blue` holding a non-blue accent is a documented cost; a rename to `--accent`, `--danger`, `--series-1..5` is separate work that must update every call site.
- Keep series colors distinct from the accent, and a pack's purple distinct from a non-blue accent.

## UI Patterns

### Cards
- White/dark background
- 1px hairline border, no shadow. Soft shadows are the fastest tell of generic SaaS
- 3px border radius

### Modals
- Centered, max-width 560px
- 3px border radius, hairline border, no shadow; the backdrop separates it
- Backdrop with click-to-close
- Escape key closes

### Panels
- Slide-in from right
- Fixed width (340-380px)
- Scrollable content area
- Fixed header and footer

### Buttons
- 34px height (standard)
- 3px border radius
- Border + background
- Hover: border color change

### Inputs
- 3px border radius
- 1px border
- Focus: blue border
- Background: slightly darker than card

## Accessibility

- All interactive elements keyboard accessible
- Focus states visible
- Color is not the only indicator of state
- Semantic HTML (buttons, headings, labels)
- ARIA labels where needed

## Print Styles

- Hide all controls
- White background
- No shadows
- Landscape orientation
- 12mm margins

## UI Polish Backlog

| Item | Status |
|------|--------|
| Flatten elevation to hairline borders | Done: 3px radius everywhere, no soft shadows. Focus rings stay |
| Lane mark (20px) left of the editable title | Waiting on the mark |
| Empty state states the value prop plus two entry points: "Drop your .xlsx, or load the sample timeline." | Partial |
| Swatch presets follow the active theme | Done |
| Keep the Today marker red exactly as is | Keep |
| Tabular figures everywhere numbers appear | Partial (grid only) |
| One icon set; audit toolbar stroke weight and corner radius | Open |

## Future Considerations

- Consider custom icon set for consistency
- Evaluate need for high-contrast mode
- Consider animation preferences (prefers-reduced-motion)
- Evaluate responsive breakpoints for mobile
