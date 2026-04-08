# Mental Model: Web as Native View Renderer

## The Core Shift

A WKWebView is not a browser. It is a `NSView` subclass that happens to use WebKit for rendering. Your HTML/CSS/JS is not a "web page" — it is the paint implementation of a native view.

## Thinking Differently

| Traditional Web Thinking                | WKWebView Thinking                                             |
| --------------------------------------- | -------------------------------------------------------------- |
| Viewport is variable, must adapt        | Viewport is a known constant from Swift                        |
| `overflow: hidden` is a restriction     | `overflow: hidden` is the body's default state                 |
| Absolute positioning is a hack          | Absolute positioning is a legitimate first choice              |
| `rem` is a portable relative unit       | `rem` is meaningless — root font-size is undefined             |
| `background: white` is the default      | `background: transparent` is the default                       |
| Scrolling is the default behavior       | Scrolling must be explicitly opted-in                          |
| Router manages navigation state         | State comes from Swift injection, there is no URL              |
| `localStorage` persists across sessions | `localStorage` may be wiped when WebContent process terminates |
| You build for many browsers             | You build for exactly one WebKit version                       |
| CSS is for styling documents            | CSS is closer to SwiftUI view modifiers                        |

## Your Page is a UIView

Think of your `<body>` as a single `NSView`. It has:

- A fixed frame (set by Swift)
- No intrinsic scroll behavior
- No default background (transparent)
- No user-facing chrome (no address bar, no scroll bars, no status bar)

Everything inside it is positioned relative to this fixed frame — just like subviews in AppKit.

## The Transparency Stack

Understanding the layer hierarchy is critical. When any layer breaks the transparency chain, you get a white flash or opaque background where you expected transparency.

```
NSWindow (backgroundColor: .clear, isOpaque: false)
  └── NSVisualEffectView (optional — provides real system vibrancy/blur)
        └── WKWebView (isOpaque: false OR setValue(false, forKey: "drawsBackground"))
              └── <html> (background: transparent)
                    └── <body> (background: transparent)
                          └── Your content
```

**If any layer is opaque, everything below it is invisible:**

| Broken Layer                                | Symptom                                     |
| ------------------------------------------- | ------------------------------------------- |
| NSWindow `isOpaque = true`                  | Entire window has system default background |
| WKWebView `drawsBackground` not disabled    | White rectangle where WebView sits          |
| `<html>` or `<body>` has a background color | Covers the native layers beneath            |
| NSVisualEffectView missing                  | No vibrancy, but transparency still works   |

**Debugging transparency:** If you see unexpected white/opaque areas, check each layer from top to bottom. The most common mistake is forgetting to set `html, body { background: transparent; }` in CSS.

## Sizing is Known

In traditional web, you write defensive CSS because the viewport could be any size. In WKWebView:

- The viewport is `NSWindow.contentView.bounds` — you know it at load time
- It only changes if Swift explicitly resizes the window (and you get notified)
- `100vw × 100vh` IS your entire rendering area, period
- You can use absolute pixel values without guilt — the target is one specific screen

## Every Scroll is Opt-In

Traditional web: content overflows, page scrolls. WKWebView: content overflows, it clips.

```css
/* This is NOT a restriction. This is the correct default. */
body {
  overflow: hidden;
}

/* Scrollable areas are explicit, contained regions: */
.sidebar-list {
  flex: 1;
  overflow-y: auto;
  overscroll-behavior: contain;
}
```

The `overscroll-behavior: contain` is important — without it, scroll events propagate to the parent, eventually reaching the WKWebView's internal NSScrollView and causing elastic bounce (which must also be disabled in Swift).

## State Lives in Swift

Your JS state is ephemeral. The WebContent process can be terminated by the OS at any time (memory pressure). When the user closes the window, everything is gone.

**Persist through bridge:**

```javascript
// Wrong: relies on JS memory / localStorage
let userChoice = 'dark';

// Right: persist to Swift, read back on init
bridge.postMessage({ type: 'persist', key: 'theme', value: 'dark' });
const theme = nativeConfig.theme; // injected by Swift at load time
```

## No Navigation

There are no pages. There is no history stack. There is no URL bar. Your app is a single view that changes its content based on state — like a SwiftUI view, not like a website.

Do not use `react-router`, `wouter`, or any URL-based router. Use React state to switch between "screens":

```tsx
function App() {
  const mode = getConfig().mode;
  if (mode === 'onboarding') return <OnboardingFlow />;
  return <OverlayScene />;
}
```
