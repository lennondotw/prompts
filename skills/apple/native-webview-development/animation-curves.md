# macOS Animation Curves and Timing

## CAMediaTimingFunction → CSS cubic-bezier

| Constant                              | CSS cubic-bezier           | CSS keyword   |
| ------------------------------------- | -------------------------- | ------------- |
| `kCAMediaTimingFunctionLinear`        | `(0, 0, 1, 1)`             | `linear`      |
| `kCAMediaTimingFunctionEaseIn`        | `(0.42, 0, 1, 1)`          | `ease-in`     |
| `kCAMediaTimingFunctionEaseOut`       | `(0, 0, 0.58, 1)`          | `ease-out`    |
| `kCAMediaTimingFunctionEaseInEaseOut` | `(0.42, 0, 0.58, 1)`       | `ease-in-out` |
| **`kCAMediaTimingFunctionDefault`**   | **`(0.25, 0.1, 0.25, 1)`** | **`ease`**    |

## SwiftUI Presets (iOS 17+ / macOS 14+)

`Animation.default` changed from `.easeInOut` to a critically-damped spring in iOS 17.

| Preset    | Base Bounce | Default Duration | dampingFraction |
| --------- | ----------- | ---------------- | --------------- |
| `.smooth` | 0.0         | 0.5s             | 1.0 (critical)  |
| `.snappy` | 0.15        | 0.5s             | 0.85            |
| `.bouncy` | 0.30        | 0.5s             | 0.70            |

## System Animation Durations (measured)

| Animation          | Duration   | Curve         |
| ------------------ | ---------- | ------------- |
| Window open        | ~250ms     | easeOut       |
| Window close       | ~200ms     | easeIn        |
| Sheet slide down   | ~250ms     | easeInEaseOut |
| NSPopover appear   | ~200ms     | easeOut       |
| NSPopover dismiss  | ~150ms     | easeIn        |
| Menu appear        | ~100–150ms | easeOut       |
| List row selection | ~100ms     | easeOut       |
| Button press       | ~50–80ms   | easeInEaseOut |

Apple design principle: **enter slow (easeOut), exit fast (easeIn)**.

## CSS Token System

```css
:root {
  --duration-micro: 100ms;
  --duration-standard: 200ms;
  --duration-panel: 280ms;
  --ease-out: cubic-bezier(0.25, 0.46, 0.45, 0.94);
  --ease-panel-enter: cubic-bezier(0.32, 0.72, 0, 1);
  --ease-panel-exit: cubic-bezier(0.4, 0, 1, 1);
  --ease-spring-snappy: cubic-bezier(0.34, 1.2, 0.64, 1);
  --ease-spring-bouncy: cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

## Spring → CSS Conversion

`cubic-bezier` cannot express oscillation. For real spring physics:

1. **CSS `linear()` function** (Safari 17.2+): sample the spring curve at many points
2. **GSAP / Motion library**: real spring solver in JS
3. **cubic-bezier approximation**: only for light overshoot, no oscillation

Spring parameter conversion formula (WWDC 2023):

```
stiffness = (2pi / duration)^2
damping = bounce >= 0 ? 4pi * (1 - bounce) / duration
                      : 4pi / (duration + 4pi * bounce)
```

Tools:

- [linear-easing-generator.netlify.app](https://linear-easing-generator.netlify.app/) — mass/stiffness/damping → CSS `linear()`
- [open-props.style](https://open-props.style/) — pre-built `--ease-spring-1` to `--ease-spring-5`
