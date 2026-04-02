---
name: stable-width-button
description: Prevent button width jitter when content changes dynamically. Use when building buttons or tabs whose label switches between multiple known values, or when layout shift occurs on state-driven content change.
---

# Stable-Width Button

A CSS technique for keeping a button's inline size stable when its visible content changes between a known set of values (e.g. "Save" / "Saving..." / "Saved").

## Problem

When a button's text changes on interaction, its intrinsic width changes too. This causes the button — and everything around it — to shift, producing layout jitter.

Common workarounds and their downsides:

| Approach                                      | Downside                                           |
| --------------------------------------------- | -------------------------------------------------- |
| Fixed `width`                                 | Breaks when translations or content vary in length |
| `min-width` guess                             | Under- or over-sized; fragile                      |
| CSS `text-indent` / invisible pseudo-elements | Hard to maintain; content isn't in the DOM         |

## Pattern

Render **all possible contents** inside the button. Only the active content is visible; the rest occupy zero height but retain their natural width, forcing the button to adopt the widest variant's inline size.

```tsx
export const Button: FC<
  ButtonHTMLAttributes<HTMLButtonElement> & {
    allPossibleContents?: ReactNode[];
  }
> = ({ children, allPossibleContents, ...props }) => {
  return (
    <button {...props}>
      {children}
      {allPossibleContents && allPossibleContents.length > 0 && (
        <div className="invisible flex h-0 flex-col overflow-clip leading-0">
          {allPossibleContents.map((content, index) => (
            <div key={index}>{content}</div>
          ))}
        </div>
      )}
    </button>
  );
};
```

## How It Works

| Class           | Purpose                                                                     |
| --------------- | --------------------------------------------------------------------------- |
| `h-0`           | Container contributes zero height — button height is unaffected             |
| `flex-col`      | Stacks all variants vertically so each gets its own line to determine width |
| `invisible`     | Hides content visually but preserves layout contribution                    |
| `overflow-clip` | Prevents hidden content from being scrollable or focusable                  |
| `leading-0`     | Eliminates line-height so the zero-height container doesn't leak space      |

The hidden container participates in the button's inline-size calculation. The button's width resolves to `max(active content width, widest hidden variant width)`, keeping it stable across state changes.

## Why `invisible` Instead of `hidden` or `display: none`

`display: none` and the `hidden` attribute remove the element from layout entirely — no width contribution. `visibility: hidden` (`invisible` in Tailwind) keeps the element in flow and preserves its box dimensions while making it non-interactive and invisible. This is exactly what we need: width participation without visual presence.

## Usage

Pass all possible label values via `allPossibleContents`:

```tsx
<Button allPossibleContents={[t('saving'), t('saved')]}>{isLoading ? t('saving') : t('saved')}</Button>
```

The current content naturally appears via `children`; the `allPossibleContents` array only needs to include **all variants** (including the current one is harmless — it just adds a redundant width contributor).

## When to Use

- Buttons that toggle between known states ("Edit" / "Editing" / "Done")
- Tab triggers where switching labels causes container reflow
- Any inline element with a finite set of content variants that should not cause width change

## When NOT to Use

- Content is user-generated or unbounded (use `min-width` or fixed width instead)
- The button already has `width: 100%` / fills its container (width is externally determined)
- Only one possible content value (no jitter risk)

## Generalization

This pattern is not limited to buttons. It applies to any element where:

1. The content switches between a known finite set of values.
2. The element's inline size is intrinsic (not externally constrained).
3. Width stability matters for the surrounding layout.

Tabs, badges, status indicators, and inline labels are all candidates.
