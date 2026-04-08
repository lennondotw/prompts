# Common Code Patterns for WKWebView

## Full-Screen Overlay

The most basic WKWebView pattern: a transparent layer covering the entire screen.

```css
.overlay {
  position: fixed;
  inset: 0;
  background: transparent;
}

.dim-layer {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0);
  transition: background 1s cubic-bezier(0, 0, 0.2, 1);
}

.dim-layer.active {
  background: rgba(0, 0, 0, 0.7);
}
```

```javascript
requestAnimationFrame(() => dimLayer.classList.add('active'));
```

**Swift side:** borderless window, `.floating` level, `isOpaque = false`, full screen frame.

## Panel + Sidebar Layout

```css
.shell {
  display: flex;
  width: 100%;
  height: 100%;
}

.sidebar {
  width: 220px;
  flex-shrink: 0;
  border-right: 0.5px solid -apple-system-separator;
  display: flex;
  flex-direction: column;
}

.sidebar-list {
  flex: 1;
  overflow-y: auto;
  overscroll-behavior: contain;
}

.main {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.main-content {
  flex: 1;
  overflow-y: auto;
  overscroll-behavior: contain;
}
```

Key points:

- Sidebar width is fixed px (not responsive)
- Each scrollable area is its own `overflow-y: auto` container
- `overscroll-behavior: contain` prevents scroll leaking between regions
- Body never scrolls

## Modal / Sheet

macOS sheets slide down from the top. Modals scale in from center.

```css
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.15);
  opacity: 0;
  transition: opacity 200ms ease;
}

.modal-backdrop.visible {
  opacity: 1;
}

.modal-panel {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0.95);
  opacity: 0;
  transition:
    transform 250ms cubic-bezier(0.34, 1.2, 0.64, 1),
    opacity 200ms ease;
  background: -apple-system-background;
  border-radius: 12px;
  box-shadow:
    0 30px 60px rgba(0, 0, 0, 0.3),
    0 0 0 0.5px rgba(0, 0, 0, 0.12);
}

.modal-panel.visible {
  transform: translate(-50%, -50%) scale(1);
  opacity: 1;
}
```

**Exit:** reverse the animation, then remove from DOM.

## Sheet (slides from top)

```css
.sheet {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translate(-50%, -100%);
  transition: transform 250ms cubic-bezier(0.32, 0.72, 0, 1);
  width: 480px;
  background: -apple-system-background;
  border-radius: 0 0 10px 10px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.25);
}

.sheet.visible {
  transform: translate(-50%, 0);
}
```

## Scroll Container

```css
.scroll-container {
  overflow-y: auto;
  overflow-x: hidden;
  overscroll-behavior-y: contain;
  scrollbar-width: thin;
}

/* WebKit custom scrollbar */
.scroll-container::-webkit-scrollbar {
  width: 6px;
}
.scroll-container::-webkit-scrollbar-track {
  background: transparent;
}
.scroll-container::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}
```

Height must come from a parent constraint (flex, grid, or explicit px). Never let the scroll container determine its own height from content.

## Bridge Message Pattern

```typescript
type NativeMessage =
  | { type: 'ready'; frame: { x: number; y: number; width: number; height: number } }
  | { type: 'dismiss' }
  | { type: 'persist'; key: string; value: string };

const isNative = !!window.webkit?.messageHandlers?.bridge;

function postMessage(msg: NativeMessage): void {
  if (isNative) {
    window.webkit!.messageHandlers.bridge.postMessage(msg);
  } else {
    console.log('[bridge:mock]', msg.type, msg);
  }
}
```

## Window Shape Animation (for onboarding overlay)

Animate a CSS rectangle from large to target window size using only `transform`:

```css
.window-shape {
  position: absolute;
  /* set width/height/left/top to TARGET window size */
  border-radius: 10px;
  background: rgba(30, 30, 30, 0.95);
  border: 0.5px solid rgba(255, 255, 255, 0.1);

  /* start scaled up */
  transform: scale(1.15);
  opacity: 0;
  transition:
    transform 700ms cubic-bezier(0, 0, 0.2, 1),
    opacity 400ms ease;
}

.window-shape.visible {
  transform: scale(1);
  opacity: 1;
}
```

This avoids animating `width`/`height`/`left`/`top` (layout properties). The element is sized to the target, and `scale()` provides the "shrinking" visual.
