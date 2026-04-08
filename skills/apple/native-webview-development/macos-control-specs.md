# macOS Native Control CSS Specifications

All sizes in pt (= CSS px on macOS). Data from Xel Toolkit CSS, Xcode View Debugger measurements, and `NSControl.intrinsicContentSize`.

## Control Size Matrix

| Control                    | Regular        | Small | Mini |
| -------------------------- | -------------- | ----- | ---- |
| Push Button height         | 22px           | 19px  | 15px |
| Push Button (Large)        | 32px           | —     | —    |
| Push Button border-radius  | 5px            | 4px   | 3px  |
| Push Button (Large) radius | 7px            | —     | —    |
| TextField height           | 22px           | 19px  | 15px |
| SearchField height         | 22px           | 19px  | —    |
| SearchField radius         | 11px (capsule) | 9.5px | —    |
| PopUpButton height         | 22px           | 19px  | 15px |
| SegmentedControl height    | 22px           | 19px  | 15px |
| NSSwitch                   | 38×22px        | —     | —    |
| Slider thumb               | 17px circle    | 11px  | 10px |
| Slider track               | 3px tall       | 2px   | 2px  |
| ProgressBar                | 4px tall       | 2px   | —    |
| Table row                  | 22px           | 17px  | —    |
| MenuItem min-height        | 22px           | —     | —    |

## Font Sizes by Control Size

| Size    | Font             |
| ------- | ---------------- |
| Regular | 13px SF Pro Text |
| Small   | 11px             |
| Mini    | 9px              |

## Push Button

```css
.ns-button {
  height: 22px;
  padding: 0 10px;
  min-width: 64px;
  font-size: 13px;
  background: #ffffff;
  border-radius: 5px;
  box-shadow:
    0 0 0 0.5px rgba(0, 0, 0, 0.15),
    0 0.5px 1px rgba(0, 0, 0, 0.08);
  cursor: default;
}
.ns-button:hover {
  background: #f5f5f5;
}
.ns-button:active {
  background: #ebebeb;
  box-shadow:
    0 0 0 0.5px rgba(0, 0, 0, 0.18),
    inset 0 1px 2px rgba(0, 0, 0, 0.06);
}
.ns-button:disabled {
  opacity: 0.4;
  pointer-events: none;
}
```

## Default (Accent) Button

```css
.ns-button--default {
  background: #007aff;
  color: #ffffff;
  box-shadow:
    0 0 0 0.5px rgba(0, 90, 200, 0.6),
    0 0.5px 1px rgba(0, 0, 0, 0.15);
}
.ns-button--default:active {
  background: #005ed1;
}
```

## Borderless Button (toolbar)

```css
.ns-button-borderless {
  background: transparent;
  border: none;
  box-shadow: none;
  border-radius: 6px;
}
.ns-button-borderless:hover {
  background: rgba(0, 0, 0, 0.07);
}
.ns-button-borderless:active {
  background: rgba(0, 0, 0, 0.14);
}
```

## TextField

```css
.ns-textfield {
  height: 22px;
  padding: 0 6px;
  font-size: 13px;
  background: #ffffff;
  box-shadow:
    0 0 0 1px rgba(0, 0, 0, 0.25),
    inset 0 1px 2px rgba(0, 0, 0, 0.06);
}
.ns-textfield:focus-visible {
  box-shadow:
    0 0 0 1px rgba(0, 88, 208, 0.7),
    0 0 0 3px rgba(0, 122, 255, 0.45),
    inset 0 1px 2px rgba(0, 0, 0, 0.06);
}
```

## NSSwitch (Toggle)

```css
.ns-switch {
  width: 38px;
  height: 22px;
  border-radius: 11px;
  background: rgba(0, 0, 0, 0.15);
}
.ns-switch--on {
  background: #34c759;
}
/* Thumb: 18px circle, 2px inset, white with shadow */
```

## Focus Ring

System focus ring is a translucent glow, not a hard border:

```css
box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.5);
```

## Border Technique

macOS uses 0.5pt borders (1 physical pixel on Retina). Use `box-shadow` not `border`:

```css
box-shadow: 0 0 0 0.5px rgba(0, 0, 0, 0.15);
```

## Menu

```css
.ns-menu {
  padding: 4px 0;
  border-radius: 8px;
  box-shadow:
    0 0 0 0.5px rgba(0, 0, 0, 0.18),
    0 4px 22px rgba(0, 0, 0, 0.28);
}
.ns-menuitem {
  min-height: 22px;
  padding: 2px 12px 2px 23px;
}
.ns-menuitem:hover {
  background: #0057ff;
  color: #ffffff;
  border-radius: 4px;
  margin: 0 4px;
}
```
