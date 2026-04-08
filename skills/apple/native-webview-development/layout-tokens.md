# macOS Layout and Typography Tokens

## Spacing Scale (8pt grid, 4pt subdivisions)

| Token     | Value | Typical Use                            |
| --------- | ----- | -------------------------------------- |
| `space.1` | 4px   | Tightest spacing, icon padding         |
| `space.2` | 8px   | Control gap (Regular), inline gap      |
| `space.3` | 12px  | Group extra spacing, separator padding |
| `space.4` | 16px  | Card padding, GroupBox padding         |
| `space.5` | 20px  | Window edge margins (L/R/Bottom)       |
| `space.6` | 24px  | Group-to-group whitespace              |
| `space.7` | 32px  | Large section separation               |
| `space.8` | 48px  | Major layout divisions                 |

## Window Content Margins [official HIG]

| Location                      | Regular | Small/Mini |
| ----------------------------- | ------- | ---------- |
| Left, Right, Bottom           | 20px    | 10px       |
| From toolbar to first control | 14px    | 8px        |
| Tab View inner edge           | 16px    | —          |
| GroupBox inner padding        | 16px    | —          |

## Control-to-Control Spacing [official]

| Context                  | Regular | Small | Mini |
| ------------------------ | ------- | ----- | ---- |
| Vertical stack (general) | 6px     | 6px   | 4px  |
| Push Buttons             | 12px    | 10px  | 8px  |
| Checkboxes               | 8px     | 8px   | 7px  |
| Radio Buttons            | 6px     | 6px   | 5px  |
| Label colon to control   | 6px     | 6px   | 5px  |

## Window Dimensions

| Element                      | Value    | Source     |
| ---------------------------- | -------- | ---------- |
| Standard titlebar            | ~28–30px | [measured] |
| Unified title + toolbar      | ~52–60px | [measured] |
| Compact toolbar              | ~38–44px | [measured] |
| Traffic light diameter       | ~14px    | [measured] |
| Traffic light center spacing | ~20px    | [measured] |
| Traffic light left edge      | ~20px    | [measured] |

Titlebar height MUST be computed at runtime:

```swift
let titlebarHeight = window.frame.height - window.contentLayoutRect.height
```

## Sidebar

| Property              | Value     |
| --------------------- | --------- |
| Width range           | 225–400px |
| Default width         | ~250px    |
| Row inset margins     | 10px      |
| List top/bottom inset | 10px      |

Sidebar rows vary by user preference:

| Setting          | Row Height | Icon Size | Font               |
| ---------------- | ---------- | --------- | ------------------ |
| Small            | 24px       | 16×16     | 11px (subheadline) |
| Medium (default) | 28px       | 20×20     | 13px (body)        |
| Large            | 32px       | 24×24     | 15px (title3)      |

## Table Rows [official]

| RowSizeStyle | Height                   |
| ------------ | ------------------------ |
| `.small`     | 17px                     |
| `.medium`    | 22px                     |
| `.large`     | 24px (default macOS 11+) |

Intercell spacing: 3px horizontal, 2px vertical.

## Typography Scale [official]

| Style          | Size | Default Weight | Emphasized Weight |
| -------------- | ---- | -------------- | ----------------- |
| `.largeTitle`  | 26px | Regular        | Bold              |
| `.title1`      | 22px | Regular        | Bold              |
| `.title2`      | 17px | Regular        | Bold              |
| `.title3`      | 15px | Regular        | Semibold          |
| `.headline`    | 13px | **Bold**       | Heavy             |
| `.body`        | 13px | Regular        | Semibold          |
| `.callout`     | 12px | Regular        | Semibold          |
| `.subheadline` | 11px | Regular        | Semibold          |
| `.footnote`    | 10px | Regular        | Semibold          |
| `.caption1`    | 10px | Regular        | Medium            |
| `.caption2`    | 10px | **Medium**     | Semibold          |

System constants: `NSFont.systemFontSize` = 13px, `NSFont.smallSystemFontSize` = 11px.

## SF Pro Tracking [official]

| Size    | Tracking | Variant |
| ------- | -------- | ------- |
| 10px    | +0.12px  | Text    |
| 11px    | +0.06px  | Text    |
| 12–15px | 0px      | Text    |
| 17px    | −0.43px  | Text    |
| 20px    | −0.60px  | Display |
| 22px    | −0.70px  | Display |
| 28px    | −0.80px  | Display |

System API applies tracking automatically from the font's `trak` table.

## Corner Radii [measured]

### macOS 13–15 (Ventura–Sequoia)

| Component              | Radius   |
| ---------------------- | -------- |
| NSWindow               | ~10–12px |
| NSPopover / NSAlert    | ~10–12px |
| NSMenu                 | ~8px     |
| NSButton / NSTextField | ~4–5px   |

### macOS 26 (Tahoe, Liquid Glass)

| Window Type     | Radius |
| --------------- | ------ |
| Unified toolbar | 26px   |
| Compact toolbar | 20px   |
| Title-only      | 16px   |

## Quick Reference

```
TYPOGRAPHY
  largeTitle:26  title1:22  title2:17  title3:15
  headline:13b  body:13  callout:12  subheadline:11  footnote/caption:10

SPACING
  Window margins: 20  From toolbar: 14  Control stack: 6
  Group gap: 12  Card padding: 16  Button gap: 12

SIDEBAR
  Width: 225–400 (default ~250)  Row: 24/28/32  Icons: 16/20/24

CONTROLS
  Button: 22/19/15  TextField: 22/19/15  Switch: 38×22

CORNER RADII (Sequoia)
  Window: ~10–12  Menu: ~8  Button: ~4–5
```
