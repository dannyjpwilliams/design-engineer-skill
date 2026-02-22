# Motion & Animation Reference

## Spring Physics

Always prefer springs over CSS easing for interactive motion. Springs model real-world deceleration.

### Standard Spring Configs

```tsx
// Default for most interactions
transition={{ type: "spring", duration: 0.3, bounce: 0 }}

// Shared layout animations (slightly longer, tiny bounce for life)
transition={{ type: "spring", duration: 0.55, bounce: 0.1 }}

// Strict transitions (no overshoot tolerated)
transition={{ type: "spring", duration: 0.55, bounce: 0 }}
```

`bounce: 0` = critically damped. The element arrives and stops. No oscillation.
Only add bounce deliberately (onboarding, celebrations) -- never as a default.

## Easing Selection

When CSS (not springs), choose deliberately:

| Context | Use | Avoid |
|---------|-----|-------|
| Enter/appear | `ease-out` | `ease-in` |
| Exit/disappear | `ease-out` | `ease-in` |
| Object movement | `ease-in-out` | `linear` |
| Continuous rotation | `linear` | everything else |
| Polish/energy | custom `cubic-bezier()` | built-in keywords |

Built-in CSS easing keywords are too gentle. Use custom curves from easing.dev or easings.co.

## Button Press Feedback

```css
button {
  transition: transform 0.125s ease-out;
}
button:active {
  transform: scale(0.97);
}
```

0.97 scale, 125ms. Subtle enough to not distort, strong enough to register.

## Icon Swap Animation

Three properties together: opacity + scale + blur.

### Framer Motion approach

```tsx
<AnimatePresence mode="popLayout" initial={false}>
  <motion.div
    key={isCopied ? "check" : "copy"}
    initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
    exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    transition={{ type: "spring", duration: 0.3, bounce: 0 }}
  >
    {isCopied ? <CheckIcon /> : <CopyIcon />}
  </motion.div>
</AnimatePresence>
```

- `mode="popLayout"` prevents layout shift during swap
- `initial={false}` prevents animation on first mount
- `key` drives the swap via mount/unmount
- Scale 0.25 (not 0): enough to register, not theatrical
- Blur 4px: mimics coming into focus (biomimicry)

### CSS-only approach

Both icons exist in DOM, overlaid. Toggle visibility via classes:

```tsx
<div className="relative">
  <div className={`absolute transition-all duration-300 ease-in-out
    will-change-[scale,opacity,filter]
    ${active ? "scale-100 opacity-100 blur-0" : "blur-xs scale-[0.25] opacity-0"}`}>
    <ActiveIcon />
  </div>
  <div className={`transition-all duration-300 ease-in-out
    will-change-[scale,opacity,filter]
    ${active ? "blur-xs scale-[0.25] opacity-0" : "scale-100 opacity-100 blur-0"}`}>
    <DefaultIcon />
  </div>
</div>
```

## Shared Layout Animations

Declare "what," not "how." Render different instances, let FLIP handle the spatial math.

```tsx
<motion.div layoutId="unique-string"
  transition={{ type: "spring", duration: 0.55, bounce: 0.1 }}
/>
```

### Critical Rules

1. **`layoutId` elements go OUTSIDE `AnimatePresence`**. Inside = competing animations = chaos.
2. Each `layoutId` must be unique per animation group.
3. Never manually calculate positions. Render the element in its new position; FLIP does the rest.

### Card-to-Modal Pattern

Use `layoutId` for spatial morph + `AnimatePresence` for overlay + Radix Dialog for a11y.
Three systems composed: each does one thing well.

## Gesture Animations

### Scale Value Hierarchy

```
Tap/Drag:  scale(0.8)   -- "pressed down" metaphor
Pan:       scale(0.9)   -- continuous action, less dramatic
Focus:     scale(1.1)   -- "highlighted" metaphor
In View:   scale(1.1)   -- "appearing" metaphor
```

Shrink = user pressing (push metaphor). Grow = element calling attention (emergence metaphor).

### Motion Gesture Props

```tsx
// Hover (ambient, longest duration)
<motion.div whileHover={{ rotate: 180 }}
  transition={{ type: "spring", duration: 0.8, bounce: 0 }} />

// Tap (quick tactile feedback)
<motion.div whileTap={{ scale: 0.8 }}
  transition={{ type: "spring", duration: 0.5, bounce: 0 }} />

// Drag (constrained to parent)
<motion.div drag dragConstraints={constraintsRef} whileDrag={{ scale: 0.8 }}
  transition={{ type: "spring", duration: 0.5, bounce: 0 }} />

// Focus (most subtle -- keyboard nav)
<motion.button whileFocus={{ scale: 1.1 }}
  transition={{ type: "spring", duration: 0.3, bounce: 0 }} />

// In View (scroll reveal with blur)
<motion.div initial={{ filter: "blur(8px)" }}
  whileInView={{ scale: 1.1, rotate: 45, filter: "blur(0px)" }}
  viewport={{ amount: 0.5 }}
  transition={{ type: "spring", duration: 0.6, bounce: 0 }} />
```

## Drag Gesture Pattern

### Precision dragging (momentum off)

```tsx
<motion.div
  drag="x"
  dragConstraints={{ left: -116, right: 116 }}
  dragElastic={0.05}
  dragMomentum={false}
/>
```

- `dragMomentum={false}` -- the critical property. Removes velocity-based fling.
- `dragElastic={0.05}` -- near-zero overshoot. Just enough "give" to feel physical.
- Commitment threshold at 50% of boundary (58px of 116px).
- Progressive reveal: actions appear at increasing drag distances (44px, then 88px).

### Coherent physics

Every value must agree: momentum off + bounce off + low elasticity = "user is in control."
One contradictory value breaks the illusion.

## Tooltips

First tooltip: normal delay. Subsequent tooltips while one is showing: **instant** (zero delay, zero transition).

```css
[data-instant] { transition-duration: 0ms; }
```

Supported by Radix UI and Base UI via `data-instant` attribute.

## Accessibility

- Always respect `prefers-reduced-motion`. Replace spatial motion with short fades (~0.2s).
- Focus animations: subtle scale only (1.1). Never rotation, blur, or multi-property.
- `whileFocus` follows `:focus-visible` logic (keyboard only, not mouse clicks).
- Every gesture must have a keyboard equivalent.

## Quality Checks

- Test animations in slow motion to catch timing conflicts invisible at full speed.
- Review with fresh eyes the next day. In-the-moment judgment is unreliable for motion tuning.
- Ask: "How often will the user trigger this?" High-frequency = minimize or remove animation.
- Never start from `scale(0)`. Use 0.25+ (objects retain visible shape even at rest).
- Custom `cubic-bezier()` > built-in keywords. Always.
