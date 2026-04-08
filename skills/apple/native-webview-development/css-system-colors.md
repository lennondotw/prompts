# CSS System Colors for WKWebView on macOS

## `-apple-system-*` Private Keywords (WebKit only)

### Label / Text

| CSS Keyword                      | NSColor                | Notes        |
| -------------------------------- | ---------------------- | ------------ |
| `-apple-system-label`            | `labelColor`           | Primary text |
| `-apple-system-secondary-label`  | `secondaryLabelColor`  |              |
| `-apple-system-tertiary-label`   | `tertiaryLabelColor`   |              |
| `-apple-system-quaternary-label` | `quaternaryLabelColor` |              |
| `-apple-system-placeholder-text` | `placeholderTextColor` |              |
| `-apple-system-header-text`      | `headerTextColor`      |              |
| `-apple-system-selected-text`    | `selectedTextColor`    | macOS only   |

### Background

On macOS, most background variants map to `NSColor.textBackgroundColor` (iOS has distinct values).

| CSS Keyword                          | NSColor                  |
| ------------------------------------ | ------------------------ |
| `-apple-system-background`           | `textBackgroundColor`    |
| `-apple-system-secondary-background` | `textBackgroundColor`    |
| `-apple-system-tertiary-background`  | `textBackgroundColor`    |
| `-apple-system-control-background`   | `controlBackgroundColor` |

### Selection

Some are fingerprint-protected (hardcoded in normal WKWebView, real values only with `UseSystemAppearance`):

| CSS Keyword                                              | NSColor                                      | Protected       |
| -------------------------------------------------------- | -------------------------------------------- | --------------- |
| `-apple-system-selected-content-background`              | `selectedContentBackgroundColor`             | Yes (`#0063E1`) |
| `-apple-system-unemphasized-selected-content-background` | `unemphasizedSelectedContentBackgroundColor` | No              |
| `-apple-system-even-alternating-content-background`      | `alternatingContentBackgroundColors[0]`      | No              |
| `-apple-system-odd-alternating-content-background`       | `alternatingContentBackgroundColors[1]`      | No              |

### Fill / Separator

| CSS Keyword                      | NSColor                   | Notes     |
| -------------------------------- | ------------------------- | --------- |
| `-apple-system-tertiary-fill`    | `tertiarySystemFillColor` | macOS 13+ |
| `-apple-system-opaque-fill`      | `systemFillColor`         | macOS 13+ |
| `-apple-system-separator`        | `separatorColor`          |           |
| `-apple-system-grid`             | `gridColor`               |           |
| `-apple-system-container-border` | `containerBorderColor`    |           |

### System Tints

| CSS Keyword            | NSColor             |
| ---------------------- | ------------------- |
| `-apple-system-blue`   | `systemBlueColor`   |
| `-apple-system-brown`  | `systemBrownColor`  |
| `-apple-system-gray`   | `systemGrayColor`   |
| `-apple-system-green`  | `systemGreenColor`  |
| `-apple-system-orange` | `systemOrangeColor` |
| `-apple-system-pink`   | `systemPinkColor`   |
| `-apple-system-purple` | `systemPurpleColor` |
| `-apple-system-red`    | `systemRedColor`    |
| `-apple-system-yellow` | `systemYellowColor` |

### Accent

| CSS Keyword                    | NSColor              | Protected       |
| ------------------------------ | -------------------- | --------------- |
| `-apple-system-control-accent` | `controlAccentColor` | Yes (`#007AFF`) |

## CSS Color Level 4 Standard Keywords (cross-browser)

| CSS Keyword     | NSColor Equivalent            | Light              | Dark                     |
| --------------- | ----------------------------- | ------------------ | ------------------------ |
| `AccentColor`   | `controlAccentColor`          | `#007AFF`          | `#007AFF`                |
| `Canvas`        | `textBackgroundColor`         | `#FFFFFF`          | `#1E1E1E`                |
| `CanvasText`    | `textColor`                   | `#000000`          | `#FFFFFF`                |
| `Field`         | `textBackgroundColor`         | `#FFFFFF`          | `#1E1E1E`                |
| `FieldText`     | `textColor`                   | `#000000`          | `#FFFFFF`                |
| `GrayText`      | `disabledControlTextColor`    | `rgba(0,0,0,0.25)` | `rgba(255,255,255,0.25)` |
| `Highlight`     | `selectedTextBackgroundColor` | `#B3D7FF`          | `#3F638B`                |
| `HighlightText` | `selectedTextColor`           | `#000000`          | `#FFFFFF`                |
| `LinkText`      | `linkColor`                   | `#0068DA`          | `#419CFF`                |

## Light/Dark Approximate Values

| Keyword                         | Light              | Dark                     |
| ------------------------------- | ------------------ | ------------------------ |
| `-apple-system-label`           | `rgba(0,0,0,0.85)` | `rgba(255,255,255,0.85)` |
| `-apple-system-secondary-label` | `rgba(0,0,0,0.50)` | `rgba(255,255,255,0.50)` |
| `-apple-system-tertiary-label`  | `rgba(0,0,0,0.26)` | `rgba(255,255,255,0.26)` |
| `-apple-system-separator`       | `rgba(0,0,0,0.10)` | `rgba(255,255,255,0.15)` |
| `-apple-system-blue`            | `#007AFF`          | `#0A84FF`                |
| `-apple-system-green`           | `#34C759`          | `#30D158`                |
| `-apple-system-red`             | `#FF3B30`          | `#FF453A`                |
