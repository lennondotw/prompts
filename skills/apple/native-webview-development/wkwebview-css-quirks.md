# WKWebView CSS Quirks and Feature Support

## WebKit Version Mapping

| macOS          | Safari | WebKit  |
| -------------- | ------ | ------- |
| 15.x (Sequoia) | 18.x   | 20619.x |
| 26.x (Tahoe)   | 26.x   | 21624.x |

WKWebView shares the exact same WebKit as Safari. No separate rendering path.

## Key Differences from Chrome/Firefox

### `-webkit-app-region: drag` — NOT SUPPORTED

This is Chromium-only. WKWebView silently ignores it. Window dragging must be implemented via JS→Swift bridge using `NSWindow.performWindowDrag(with:)`.

### `backdrop-filter`

- Safari 18.0+: no prefix needed
- Older: requires `-webkit-backdrop-filter`
- Only blurs content within WKWebView, NOT the desktop behind the window
- **Safari 26.0+**: With `webView.isOpaque = false`, `backdrop-filter` CAN penetrate to native layers

### Viewport Units

On macOS WKWebView, all viewport units are stable (no browser chrome interference):

- `vw`/`vh` = frame size, fixed
- `svh`/`lvh`/`dvh` = all equivalent on macOS (no dynamic UI chrome)

### `env()` Variables

- `env(safe-area-inset-*)`: Always `0px` on macOS desktop
- `env(titlebar-area-*)`: NOT supported (Chromium PWA only)
- For titlebar safe area, inject via Swift bridge

### `color-scheme`

WebKit does NOT auto-darken content. Must explicitly opt in:

```css
:root {
  color-scheme: light dark;
}
```

`prefers-color-scheme` follows `webView.appearance` (highest priority), then parent view, then system setting.

### System Fonts

`-apple-system`, `system-ui`, `BlinkMacSystemFont` all resolve to SF Pro. Auto-switches between Text (≤19pt) and Display (≥20pt) variants.

### `@font-face`

Fully supported. Options for local resource loading:

1. `loadFileURL` with `allowingReadAccessTo`
2. `WKURLSchemeHandler` (production recommended)
3. Base64 inline (escape hatch)

`local()` may fail for some system fonts in sandboxed apps. Use `-apple-system` keyword instead.

## CSS Feature Support Timeline

| Feature                       | Safari Version |
| ----------------------------- | -------------- |
| `@layer`                      | 15.4+          |
| `dvh`/`svh`/`lvh`             | 15.4+          |
| Container Size Queries        | 16.0+          |
| `light-dark()`                | 17.5+          |
| `@starting-style`             | 17.5+          |
| View Transitions (same-doc)   | 18.0+          |
| `backdrop-filter` (no prefix) | 18.0+          |
| `content-visibility`          | 18.0+          |
| `scrollbar-width`             | 18.2+          |
| `text-box-trim`               | 18.2+          |
| CSS Anchor Positioning        | 26.0+          |
| Scroll-driven Animations      | 26.0+          |
| CSS Houdini Paint API         | Never          |

## Never Supported in WKWebView

- `-webkit-app-region: drag`
- `env(titlebar-area-*)`
- CSS Houdini Paint API
- Service Workers (unless App-Bound Domains enabled)
