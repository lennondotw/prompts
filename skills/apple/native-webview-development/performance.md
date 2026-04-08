# WKWebView Performance

## Process Architecture

```
Host App (UI process)
  ├── WebContent process (1 per WKWebView, DOM/JS/layout/paint)
  ├── Networking process (session-level shared)
  └── GPU process (compositing, WebGL, media)
```

Shared `WKProcessPool` allows multiple WKWebViews to share a WebContent process (after hitting an implementation-defined limit).

## IPC Overhead

| Channel                  | Round-trip | Direction      |
| ------------------------ | ---------- | -------------- |
| `evaluateJavaScript`     | ~0.4ms     | Swift→JS→Swift |
| `WKScriptMessageHandler` | ~0.6ms     | JS→Swift       |

These are 2015 iOS measurements. Modern macOS is faster but exact figures require benchmarking. Key principle: **minimize cross-process calls, batch updates**.

## Memory

- Empty WKWebView: ~30–50MB (WebContent process startup)
- Light React UI: 100–150MB
- Medium complexity: 200–350MB
- Each compositing layer: proportional to width × height × devicePixelRatio × 4 bytes

`window.addEventListener('memorypressure')` does NOT work on macOS. Handle memory pressure natively via `webViewWebContentProcessDidTerminate`.

## Cold Start Optimization

1. **Pre-warm**: Create WKWebView at app launch, load `about:blank`
2. **Share WKProcessPool**: Reuse process pool across views
3. **Inline critical CSS**: Avoid FOUC
4. **Measure FCP**: Use Paint Timing API + bridge to report

## GPU Compositing Rules

Only these properties are pure compositor updates (no main thread involvement):

- `transform` (translate, scale, rotate)
- `opacity`

Everything else triggers layout or paint:

- `width`, `height`, `top`, `left`, `margin` → layout + paint
- `background-color`, `border-radius`, `box-shadow` → paint only
- `clip-path` → paint only (better than width/height for size animation)

Use `will-change: transform, opacity` to pre-promote elements to compositing layers. But UA may ignore the hint under memory pressure.

## What Causes Jank

| Cause                         | Why worse in WKWebView                                    |
| ----------------------------- | --------------------------------------------------------- |
| Non-composited animations     | Extra IPC to GPU process per frame                        |
| Frequent `evaluateJavaScript` | XPC serialization blocks main thread                      |
| Large DOM                     | WebContent process memory pressure, potential termination |
| Active scroll listeners       | Bypasses passive scroll fast-path                         |
| Sync image decoding           | WebContent main thread blocked                            |
| JIT cold start                | First JS execution walks through interpreter tiers        |

## Useful APIs

- `content-visibility: auto` (Safari 18+): skip layout/paint for offscreen content
- `decoding="async"` on images: non-blocking decode
- Passive event listeners: `{ passive: true }` for scroll/wheel
