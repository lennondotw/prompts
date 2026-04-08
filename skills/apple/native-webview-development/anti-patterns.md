# Anti-Patterns for WKWebView Development

Things you must NOT do when writing web code for WKWebView. Each entry explains why it's wrong and what to do instead.

## Routing

**Don't:** Use `react-router`, `wouter`, `@tanstack/router`, or any URL-based router.

**Why:** There is no address bar, no URL history, no back button. URL routing is meaningless. The mode/screen is determined by Swift-injected config at load time.

**Instead:** Use React state or a simple state machine to switch between views.

## Responsive Design

**Don't:** Use `@media (min-width: ...)`, breakpoints, or responsive utility classes.

**Why:** There is exactly one viewport size, controlled by Swift. You know it at build time.

**Instead:** Build for the known size. If the window can resize, use `flex` to fill the container — not breakpoints to rearrange layout.

## rem / em Units

**Don't:** Use `rem` or `em` for sizing.

**Why:** `rem` is relative to the root `<html>` font-size, which has no meaningful user-controlled value in WKWebView (there is no browser font-size setting). It adds indirection with zero benefit.

**Instead:** Use `px` for fixed sizes, `%` for relative sizes.

## localStorage for Important State

**Don't:** Rely on `localStorage` or `sessionStorage` for state that must survive.

**Why:** The WebContent process can be terminated by the OS at any time (memory pressure). When terminated, localStorage may be lost. Additionally, WKWebView storage is subject to Intelligent Tracking Prevention and may be purged.

**Instead:** Persist important state through the Swift bridge to `UserDefaults` or disk. Read it back via `WKUserScript` injection at load time.

## Service Workers

**Don't:** Use Service Workers for caching or offline support.

**Why:** Service Workers are not available in WKWebView unless `WKAppBoundDomains` is configured in Info.plist. Even then, the behavior is restricted.

**Instead:** Bundle all resources locally. Use `WKURLSchemeHandler` for custom resource loading if needed.

## Animating Layout Properties

**Don't:** Animate `width`, `height`, `top`, `left`, `margin`, `padding`.

**Why:** These trigger layout recalculation, which crosses the process boundary in WKWebView (UI process ↔ WebContent process). This is more expensive than in a standalone browser and causes visible jank.

**Instead:** Use `transform` (translate, scale) and `opacity`. For size changes, use `clip-path: inset()` or `scale()` on a fixed-size element.

## `<meta name="viewport">`

**Don't:** Include `<meta name="viewport" content="width=device-width, initial-scale=1">`.

**Why:** WKWebView on macOS ignores this tag. The viewport is set by the WKWebView's frame, controlled by Swift.

**Instead:** Nothing. Remove the tag entirely.

## `position: fixed` for Window Anchoring

**Don't:** Use `position: fixed` to anchor elements to the window edges.

**Why:** In WKWebView, `position: fixed` behaves identically to `position: absolute` relative to the viewport (since there is no scrollable document). It works visually but the semantic is misleading.

**Instead:** Use `position: absolute` with `inset: 0` on the root container. Inside that, position children normally.

## Frequent `evaluateJavaScript` Calls

**Don't:** Call `evaluateJavaScript` from Swift on every frame or mouse move.

**Why:** Each call crosses an XPC IPC boundary (~0.5ms per call). At 60fps, this consumes 30ms of your 16ms frame budget.

**Instead:** Batch updates. Send one message with all data, or let JS poll at its own pace.

## `innerHTML` with Swift-Provided Strings

**Don't:** Set `innerHTML` using strings received from Swift without sanitization.

**Why:** If the string contains `<script>` tags or event handlers, it executes in the web context — this is XSS even within your own app.

**Instead:** Use `textContent` for text, or construct DOM elements programmatically.

## Blanket `will-change`

**Don't:** Apply `will-change: transform` to every element "for performance."

**Why:** Each `will-change` creates a compositing layer that consumes GPU memory (proportional to element size × device pixel ratio). The GPU process has finite memory.

**Instead:** Apply `will-change` only to elements about to animate, and remove it after animation completes.

## Global Scroll

**Don't:** Let `<body>` or `<html>` scroll.

**Why:** WKWebView wraps the page in an internal `NSScrollView`. If the body has scrollable content, the NSScrollView activates and causes elastic bounce, which cannot be fully disabled from CSS.

**Instead:** `body { overflow: hidden; }` always. Create explicit scroll containers for regions that need it.

## `cursor: pointer` on Buttons

**Don't:** Use `cursor: pointer` on buttons or interactive elements.

**Why:** macOS HIG: buttons use the default arrow cursor. Only external links use the hand cursor. Using pointer on buttons is the most visible "this is a web page" tell.

**Instead:** `* { cursor: default; }` globally. Only use `cursor: pointer` on elements that navigate to an external URL.
