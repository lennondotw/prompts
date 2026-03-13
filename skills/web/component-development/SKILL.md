---
name: component-development
description: UI component development workflow for building and verifying React components from Figma designs. Use when creating new components, implementing designs from Figma, building UI primitives, or iterating on visual fidelity. Covers the full cycle from Figma extraction through browser verification, Storybook stories, and testing.
---

# Component Development

## Workflow

### Phase 1: Design Extraction

1. Fetch Figma node via MCP (`get_design_context`, `get_screenshot`).
2. Take a screenshot first — confirm the overall visual: layout structure, hierarchy, proportions, and color balance before diving into individual values.
3. Record every spec: font family, weight, size, line-height, letter-spacing, text-align, text-transform, text-decoration, color (with opacity), background (solid, gradient), padding, margin, gap, width/height (including min/max constraints), border-radius, border-width, box-shadow, opacity, overflow, and icon sizing.
4. Map Figma tokens to project design system tokens where they exist; use raw values only when no token matches.

### Phase 2: Implementation

Write the component following these principles:

**Design fidelity** — Faithfully convey the design's _intent_, not pixel-for-pixel reproduction of static values. Round obviously unreasonable fractional values that are Figma artifacts (e.g. `12.1px` → `12px`). Adapt static layouts to responsive/interactive reality. The design is a communication, not a specification.

**Shallow DOM** — Use modern CSS layout (`flex`, `grid`) directly. Avoid wrapper divs that exist only for styling. Every DOM node should carry semantic or layout purpose.

**Border & stroke strategy** — Choose deliberately:

| Technique                                         | Layout impact         | When to use                                    |
| ------------------------------------------------- | --------------------- | ---------------------------------------------- |
| `border`                                          | Yes (shrinks content) | When border is part of the sizing model        |
| `outline` + negative `outline-offset`             | No                    | Non-layout-affecting focus rings, selection    |
| `box-shadow` (inset or outset)                    | No                    | Decorative strokes, glow, elevation            |
| Absolute overlay `div` with `pointer-events-none` | No                    | Complex multi-layer strokes on rounded corners |

Maintain border-radius concentricity: `inner-radius = outer-radius − border-width`.

**Typography** — Set every parameter explicitly: `font-family`, `font-weight`, `font-size`, `line-height`, `letter-spacing`, `color`. Do not rely on inherited or default values for components.

**Color** — Use exact values from Figma including alpha. Prefer `rgba()` or Tailwind opacity modifiers (`text-black/70`) over approximation. Add `dark:` variants for colors that lose contrast on dark backgrounds.

**Touch targets** — For buttons and clickable elements with small visual footprints, use an absolutely positioned `::after` pseudo-element (or a child div) with negative inset to expand the interactive area to at least 44×44px without affecting layout.

**Animations** — Use Motion (previously Framer Motion). Always call `useReducedMotion()` and skip or simplify animation when true. This also ensures deterministic screenshot testing.

### Phase 3: Stories

Write Storybook stories covering every meaningful variant before visual verification.

- One story per variant/state.
- A single composite story that elegantly lays out all variants — just the components themselves, no extra labels or decorations.
- Use **seeded/deterministic data** — no `Math.random()` or `Date.now()` in stories. Use hardcoded values or a seeded PRNG.

### Phase 4: Browser Verification

**Always verify every component in the browser before considering it done.**

1. Open the story's iframe URL directly (e.g. `http://localhost:6006/iframe.html?id=components-button--default`) to get the isolated component without Storybook chrome.
2. Inspect the rendered DOM — confirm every CSS property matches the Figma spec:
   - Font: family, weight, size, line-height, letter-spacing.
   - Color: foreground, background, border, opacity.
   - Spacing: padding, margin, gap — verify computed values, not just class names.
   - Dimensions: width, height, border-radius.
3. Take a screenshot for visual comparison against the Figma screenshot.
4. If anything is off, fix it and re-verify. Do not proceed until the component matches.

### Phase 5: Testing

Only after browser verification confirms the component is correct:

1. Write unit tests (React Testing Library) covering rendering, props, accessibility, visual styles.
2. Update snapshots (`--update` flag).
3. Add visual regression (screenshot) tests if the project uses them.
4. Run tests and confirm all pass.

### Phase 6: Commit

Only after all tests pass and browser verification is complete:

1. Run `pnpm lint && pnpm format`.
2. Commit with conventional commit message.

## Decomposition

For large or complex components, decompose into smaller, self-contained sub-components first. Build each sub-component through the full cycle (implementation → stories → browser verification → tests) independently. Only compose them into the parent component after each sub-component is verified and stable across all states.

## Separation of Concerns

For complex components, split into two layers:

| Layer            | Responsibility                                                                                            | Tests                                                 |
| ---------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| **Presentation** | Visual rendering, styling, layout. Props are primitives and ReactNode. No data fetching, no side effects. | Storybook stories, snapshot tests, screenshot tests   |
| **Container**    | Data fetching, state management, event handling. Composes presentation components.                        | Unit tests for logic, integration tests for data flow |

Presentation components must render identically given the same props — no internal randomness, no time-dependent rendering.

## Anti-Patterns

- **Do not commit snapshots/screenshots before browser verification** — they lock in wrong output.
- **Do not guess font or color values** — always extract from Figma or inspect in browser.
- **Do not add animation without `useReducedMotion` guard** — breaks screenshot tests and a11y.
- **Do not use non-deterministic random data in stories** — breaks visual regression.
- **Do not add empty wrapper divs** — if a div has only one class for one child, the class belongs on the child.
- **Do not use `border` when you need zero layout impact** — use `outline`, `box-shadow`, or an overlay instead.
