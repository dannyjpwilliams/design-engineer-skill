# Visual Craft Reference

## Shadows

### Three-Layer Shadow System

```css
/* Default state */
box-shadow:
  0px 0px 0px 1px rgba(0, 0, 0, 0.06),   /* Layer 1: definition (shadow-as-border) */
  0px 1px 2px -1px rgba(0, 0, 0, 0.06),   /* Layer 2: near shadow (-1px spread contains it) */
  0px 2px 4px 0px rgba(0, 0, 0, 0.04);    /* Layer 3: ambient diffuse light */

/* Hover state -- only opacity changes, geometry stays identical */
box-shadow:
  0px 0px 0px 1px rgba(0, 0, 0, 0.08),
  0px 1px 2px -1px rgba(0, 0, 0, 0.08),
  0px 2px 4px 0px rgba(0, 0, 0, 0.06);
```

### Why Each Layer Exists

| Layer | Purpose | Key Detail |
|-------|---------|------------|
| 1: Definition | Crisp 1px outline replacing `border` | Zero offset, zero blur, 1px spread. Transparency makes it context-agnostic. |
| 2: Near shadow | Soft close-proximity depth | `-1px` negative spread contracts shadow inward. Without it: unwanted "glow." |
| 3: Ambient | Diffuse environmental light | Largest blur (4px), lowest opacity (0.04). Sells physical elevation. |

### Rules

- Opacity range: **4-8%**. Lower than you think. Higher = heavy, unrealistic.
- Hover: +0.02 opacity per layer. Same offsets, blur, spread. Element becomes more defined, not lifted.
- Color: always `rgba(0, 0, 0, ...)` (pure black at varying transparency). Works on any background.
- Transition `box-shadow` for smooth state changes.
- A single shadow layer looks flat. Three layers mimic real light: edge + penumbra + ambient.

### Why Not Borders

Borders are context-dependent (designed for one background). Shadows with transparency adapt to any background -- solid colors, gradients, images. This makes components portable.

## OKLCH Color

### Perceptual Uniformity

OKLCH ensures equal numeric steps = equal visual steps. HSL does not.

- **L** (Lightness): 0-1. Controls perceived brightness.
- **C** (Chroma): unbounded. Controls saturation/intensity.
- **H** (Hue): 0-360. The actual color.

Syntax: `oklch(0.55 0.18 260)` -- no commas, no degree unit.

### Uniform Palette Generation

Hold L and C constant, vary only hue:

```css
oklch(0.8 0.193 280)  /* purple */
oklch(0.8 0.193 220)  /* blue */
oklch(0.8 0.193 140)  /* green */
oklch(0.8 0.193 60)   /* orange */
oklch(0.8 0.193 0)    /* red */
```

0.193 chroma = safe vivid default. Above 0.3 risks gamut clipping.

### Shade Scale (No Hue Drift)

Vary only L, keep C and H locked:

```
L=0.25 (darkest) → L=0.35 → L=0.45 → L=0.55 → L=0.65 → L=0.75 → L=0.85 → L=0.92 (lightest)
```

~0.10 steps, slightly compressed at light end (perception is more sensitive there).
In HSL, lighter blues go purple and darker blues go muddy gray. OKLCH doesn't drift.

### Fallback Pattern

```css
--color-primary: #6366f1;
@supports (color: oklch(0 0 0)) {
  --color-primary: oklch(0.55 0.18 260);
}
```

### Key Takeaways

- OKLCH doesn't magically improve colors. It improves the *manipulation model*.
- Adding a new brand color = zero design review (uniform weight guaranteed by math).
- Dark mode = lightness inversion, no per-color adjustments.
- Contrast reasoning via the L axis alone.
- Tool: oklch.fyi for palette generation.

## Gradients

### Color Space Interpolation

```css
/* sRGB (default) -- muddy midpoints, avoid for vibrant colors */
linear-gradient(blue, red)

/* Oklab -- clean straight-line blends, perceptually uniform */
linear-gradient(in oklab, blue, red)

/* OKLCH -- circular hue path, can introduce unexpected intermediate colors */
linear-gradient(in oklch, blue, red)
```

- Use `oklab` for clean two-color blends.
- Use `oklch` when you want the hue arc (rainbows, color wheels).
- Tailwind 4 defaults to `oklab`.

### Color Hints (Midpoint Control)

Color hints shift the blend midpoint. They are NOT color stops.

```css
/* Red transitions faster, blue dominates */
linear-gradient(to right, red, 30%, blue)
```

### Gradient Layering

```css
background:
  linear-gradient(135deg, oklch(0.75 0.18 280), oklch(0.65 0.2 340)),
  conic-gradient(from 240deg at 50% 50%, oklch(0.7 0.12 200 / 0.3), transparent);
background-blend-mode: overlay, color-dodge;
```

Blend modes are per-layer. Treat `background` as a compositing stack (same model as Photoshop).

### Animation

Animate `background-position` or `background-size`, never color stops directly.
Color stops trigger repaint. Position/size are compositor-friendly.

## Concentric Border Radius

### Formula

```
outer_radius = inner_radius + padding
```

```css
:root {
  --inner-radius: 12px;
  --padding: 8px;
  --outer-radius: calc(var(--inner-radius) + var(--padding));
}
```

### When to Skip

- `border-radius: 0` -- no curvature to mismatch.
- `border-radius: 9999px` (pill) -- always looks balanced.

### Multiple Nesting Levels

Each layer compounds:
- Button: 8px
- Section: 8px + 4px padding = 12px
- Card: 12px + 8px padding = 20px

### Validation

Zoom in at corners. The gap between inner and outer edges should be uniform.
If it widens at corners, the radii are not concentric.

## Optical Alignment

### Principles

- Geometric alignment != visual alignment. Align to the eye, verify against the grid.
- Visual mass is area-based, not dimension-based. A circle has less mass than a same-height square.

### Icon + Text Buttons

Reduce padding on the icon side. Icons have less visual mass than text.
The exact reduction depends on the icon's weight.

### Icon Vertical Alignment

```css
/* Modern CSS shorthand */
translateY(calc(0.5em - 0.5lh))

/* In flex containers */
svg { flex-shrink: 0; }
.button { display: inline-flex; align-items: center; }
```

Fix alignment in the SVG viewBox when possible (one-time fix vs. per-usage CSS hacks).

### Typography

- Overshooting letters (T, V, W) and round letters (O, C, S) at line start: extend slightly beyond margin.
- Hanging punctuation: `hanging-punctuation: first last;`
- Card bottom padding: measure from baseline, not descender.

### Icon Shape Padding

- Elongated icons (arrows): touch the bounding frame edges.
- Square icons: generous internal padding.
- Goal: normalize perceived visual weight across differently-shaped icons.

## Performance (will-change)

### Compositor-Friendly Properties

Only these benefit from `will-change`:
`transform`, `opacity`, `filter`, `clip-path`, `mask`, `scroll-position`, `contents`

### Usage

```css
.animated { will-change: transform, opacity; }  /* Specific properties */
```

### Rules

- Never `will-change: all` (wastes resources, like `transition-property: all`).
- Never hint non-compositor properties (`top`, `left`, `width` -- they don't benefit).
- Remove `will-change` when not animating (keeps promoted layers alive, wastes memory).
- Safari shows the most visible improvement from `will-change`.

### Rendering Pipeline

```
Layout (CPU) → Paint (CPU+Memory) → Composite (GPU)
```

Compositor-friendly properties skip Layout and Paint entirely. That's why `transform` and `opacity` are fast -- GPU handles them in the Composite step alone.
