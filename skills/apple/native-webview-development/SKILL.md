---
name: native-webview-development
description: Write web code that renders inside macOS WKWebView with native look and feel. Use when building HTML/CSS/JS content for WKWebView, creating onboarding overlays, in-app web panels, or any web-rendered UI in a macOS native app. Covers layout, transparency, animation performance, system colors, interaction hygiene, scroll handling, Swift bridge patterns, and development workflow.
---

# Native WebView Development for macOS

## Mindset: You Are Not Building a Website

WKWebView is a `NSView` subclass. Your HTML/CSS/JS is the paint implementation of a native view, not a web page.

| Traditional Web                     | WKWebView                                          |
| ----------------------------------- | -------------------------------------------------- |
| Viewport is variable, must adapt    | Viewport is a known constant from Swift            |
| `overflow: hidden` is a restriction | `overflow: hidden` is body's default state         |
| Absolute positioning is a hack      | Absolute positioning is a legitimate first choice  |
| `rem` is a portable relative unit   | `rem` is meaningless — root font-size is undefined |
| Background defaults to white        | Background defaults to transparent                 |
| Scrolling is default behavior       | Scrolling must be explicitly opted-in              |
| Router manages navigation           | State comes from Swift injection, no URL           |
| `localStorage` persists             | `localStorage` may be wiped on process termination |

For the full mental model, see [mental-model.md](mental-model.md).

## Layout

**The viewport is a known constant.** Window size is controlled by Swift. Do not use media queries or breakpoints.

```css
html,
body {
  width: 100%;
  height: 100%;
  overflow: hidden;
  margin: 0;
}
```

`w-full h-full` is the normal default, not a special case.

**Absolute positioning is legitimate.** Overlays, badges, window-anchored elements — use `position: absolute` with precise values from the bridge without hesitation.

**No responsive design.** One viewport size at runtime.

**No global scroll.** `body` is always `overflow: hidden`. Scrollable regions are explicit sub-containers with `overflow-y: auto`.

**Coordinate system:** macOS native uses bottom-left origin (y up). CSS uses top-left (y down). When bridging coordinates: `cssTop = screenHeight - macY - elementHeight`.

## Transparent Background

WKWebView can render over native content (desktop, vibrancy materials). Three layers must align:

**Swift side:**

The full transparency chain must be intact — if any layer is opaque, everything below is invisible. See [mental-model.md](mental-model.md) for the complete layer diagram.

```swift
webView.isOpaque = false
webView.setValue(false, forKey: "drawsBackground") // fallback for older SDKs
```

**CSS side:**

```css
html,
body {
  background: transparent;
}
```

**Browser dev mock:** Real transparency is invisible in the browser. Fake it:

```js
if (!isNative) document.documentElement.dataset.env = 'browser';
```

```css
#root[data-env='browser'] {
  background: url('/wallpaper-light.png') center / cover;
}
@media (prefers-color-scheme: dark) {
  #root[data-env='browser'] {
    background: url('/wallpaper-dark.png') center / cover;
  }
}
```

## Sizing Strategy

| Unit      | When                                                            |
| --------- | --------------------------------------------------------------- |
| `px`      | Fixed: icons, buttons, borders, spacing tokens, control heights |
| `%`       | Relative to parent: panels, columns, fill regions               |
| `vw`/`vh` | Root level only; unreliable inside flex/grid subtrees           |
| `rem`     | Avoid — no meaningful root font size in WebView context         |

Use `px` for anything small/fixed, `%` for filling containers, `calc()` to combine.

## Eliminating "Web Feel"

These are the tells. Eliminate all of them:

| Symptom                    | Fix                                                               |
| -------------------------- | ----------------------------------------------------------------- |
| Hand cursor on buttons     | `* { cursor: default; }`                                          |
| Text selectable everywhere | `* { user-select: none; }` + whitelist inputs                     |
| Images/links draggable     | `* { -webkit-user-drag: none; }`                                  |
| Browser right-click menu   | `contextmenu` → `preventDefault()` (native only)                  |
| Blue focus outline         | Custom `:focus-visible` with system accent color glow             |
| Elastic overscroll bounce  | Swift: disable `NSScrollView` elasticity (CSS alone insufficient) |
| Tap flash on iOS           | `-webkit-tap-highlight-color: transparent`                        |
| Non-native form controls   | `color-scheme: light dark` + `accent-color`                       |

**Global reset:**

```css
*,
*::before,
*::after {
  box-sizing: border-box;
  cursor: default;
  user-select: none;
  -webkit-user-select: none;
  -webkit-user-drag: none;
  -webkit-tap-highlight-color: transparent;
}

input,
textarea,
[contenteditable] {
  cursor: text;
  user-select: text;
  -webkit-user-select: text;
}

*:focus {
  outline: none;
}
*:focus-visible {
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.5);
  border-radius: 4px;
}

body {
  -webkit-font-smoothing: antialiased;
  font-family:
    system-ui,
    -apple-system,
    sans-serif;
  color-scheme: light dark;
}
```

## Animation Performance

**GPU-composited (always prefer):** `transform` and `opacity`.

**Triggers layout (avoid animating):** `width`, `height`, `top`, `left`, `margin`, `padding` — any property that changes box dimensions.

In WKWebView, layout recalculation crosses a process boundary (UI process ↔ WebContent process). Layout thrashing is more expensive than in a standalone browser.

**`will-change` caution:** Each `will-change: transform` creates a compositing layer consuming GPU memory. Use on-demand (toggle via JS before animation), not as a blanket declaration.

**For size changes:** Use `clip-path: inset()` instead of animating `width`/`height` — it's paint-only, no layout.

**Entrance animation pattern:**

```css
.panel {
  opacity: 0;
  transform: translateY(8px);
  transition:
    opacity 200ms ease,
    transform 200ms ease;
}
.panel.visible {
  opacity: 1;
  transform: translateY(0);
}
```

Apply `.visible` in the next frame after mount:

```js
requestAnimationFrame(() => el.classList.add('visible'));
```

**macOS timing conventions:** enter slow (ease-out, ~250ms), exit fast (ease-in, ~200ms), micro-interactions ~100ms.

## Typography

```css
body {
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    sans-serif;
  -webkit-font-smoothing: antialiased;
}
```

`-webkit-font-smoothing: antialiased` is the single highest-impact line for matching native text rendering. Without it, text appears heavier than SwiftUI/AppKit on dark backgrounds.

**System font shorthands** (exact semantic sizing, auto light/dark):

```css
font: -apple-system-body; /* 13px regular */
font: -apple-system-headline; /* 13px bold */
font: -apple-system-title1; /* 22px regular */
font: -apple-system-subheadline; /* 11px regular */
font: -apple-system-footnote; /* 10px regular */
font: -apple-system-caption1; /* 10px regular */
```

**Letter spacing:** Native macOS text is tighter than browser defaults at large sizes. Apply:

```css
.large-title {
  font-size: 28px;
  letter-spacing: -0.025em;
}
.body {
  font-size: 13px;
  letter-spacing: -0.01em;
}
```

## Colors and Appearance

Always declare:

```css
:root {
  color-scheme: light dark;
}
```

Use `-apple-system-*` semantic colors — they auto-respond to Light/Dark mode:

```css
color: -apple-system-label;
color: -apple-system-secondary-label;
color: -apple-system-tertiary-label;
background: -apple-system-background;
background: -apple-system-control-background;
background: -apple-system-fill;
border-color: -apple-system-separator;
```

These are **not** available in Chrome/Firefox. For WKWebView-only code, prefer them over custom hex values — they guarantee consistency with native UI.

**Accent color:** Not directly available in CSS. Bridge from Swift:

```swift
let hex = NSColor.controlAccentColor.toHex()
webView.evaluateJavaScript(
  "document.documentElement.style.setProperty('--accent', '\(hex)')"
)
```

**Vibrancy:** CSS `backdrop-filter: blur()` only blurs within-WebView content, not the desktop behind. Real vibrancy requires `NSVisualEffectView` placed beneath the transparent WKWebView in the native view hierarchy.

## Scroll Handling

Never scroll `body`. Explicit containers only:

```css
.scroll-area {
  overflow-y: auto;
  overflow-x: hidden;
  overscroll-behavior: contain;
}
```

**Elastic bounce must be disabled in Swift** — `overscroll-behavior: none` is insufficient on macOS WKWebView for the enclosing `NSScrollView`:

```swift
if let sv = webView.enclosingScrollView {
  sv.verticalScrollElasticity = .none
  sv.horizontalScrollElasticity = .none
}
```

## Swift Bridge Basics

**Config injection via WKUserScript** (survives HMR reloads):

```swift
let script = WKUserScript(
  source: "window.__native__ = { config: \(jsonString) };",
  injectionTime: .atDocumentStart,
  forMainFrameOnly: true
)
config.userContentController.addUserScript(script)
```

**JS → Swift:**

```js
window.webkit.messageHandlers.bridge.postMessage({ type: 'dismiss' });
```

**Swift → JS:**

```swift
webView.evaluateJavaScript("handleNativeEvent(\(json))")
```

**Retain cycle warning:** `WKUserContentController.add(_:name:)` strongly retains the handler. Use a weak proxy or explicitly `removeScriptMessageHandler` on teardown.

## Development Workflow

**90% browser, 10% WKWebView.** Iterate in Vite (`localhost:5174`). Only verify in WKWebView for:

- Transparent background rendering
- Swift bridge message round-trips
- Font rendering edge cases
- Elastic scroll behavior

**Vite HMR works in WKWebView.** Point WKWebView at dev server URL in `#if DEBUG`. CSS/JS changes reflect instantly without rebuilding the native app.

**Safari Web Inspector:** Required for WKWebView debugging.

```swift
#if DEBUG
webView.isInspectable = true  // macOS 13.3+
#endif
```

Safari → Develop → [Your Mac] → select WKWebView instance.

**Debug window level:** Use `.normal` (not `.floating`) in DEBUG so other windows (including Safari Inspector) can overlay:

```swift
#if DEBUG
window.level = .normal
#else
window.level = .floating
#endif
```

**Cmd+R reload:** Add keyboard shortcut in DEBUG for quick page reload without restarting the app.

## Architecture Guidance

**Good for WKWebView:**

- Onboarding overlays (temporary, complex animation, content-driven)
- Rich text / markdown editors
- Data visualization dashboards
- Release notes / changelog panels

**Keep native:**

- Window chrome (title bar, traffic lights, toolbar)
- System navigation (sidebar, tab bar)
- Controls that must feel like system controls
- Drag-and-drop with other native apps

Arc Browser: native shell + web content = users accept it. 1Password 8 Electron: native controls replaced with web = widespread rejection. The lesson: **chrome native, content web.**

## Reference Files

**Conceptual:**

- [mental-model.md](mental-model.md) — The core mindset shift, transparency layer stack, "your page is a UIView"
- [common-patterns.md](common-patterns.md) — Ready-to-use code: overlay, sidebar, modal, scroll container, bridge
- [anti-patterns.md](anti-patterns.md) — Things you must NOT do (router, rem, localStorage, layout animation...)

**Specification tables:**

- [css-system-colors.md](css-system-colors.md) — Complete `-apple-system-*` and CSS L4 system color mapping
- [macos-control-specs.md](macos-control-specs.md) — Pixel-level macOS control CSS specs
- [wkwebview-css-quirks.md](wkwebview-css-quirks.md) — CSS feature support matrix, known bugs
- [animation-curves.md](animation-curves.md) — macOS timing values, spring parameters, CSS equivalents
- [performance.md](performance.md) — Process architecture, IPC cost, memory, jank causes
- [bridge-patterns.md](bridge-patterns.md) — Swift↔JS communication, lifecycle, security
- [layout-tokens.md](layout-tokens.md) — Spacing scale, window dimensions, typography tokens
