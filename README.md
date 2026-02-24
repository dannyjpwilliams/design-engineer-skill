# Design Engineer

Craft-level UI principles for your AI coding agent. Teaches the difference between correct and *good* — layered shadows, perceptual color, spring physics, optical alignment, and the thousand invisible details that separate polished interfaces from generic ones.

Distilled from [Jakub Krehel](https://jakub.kr) and [Emil Kowalski](https://emilkowal.ski).

## Install

```bash
npx skills add dannyjpwilliams/design-engineer-skill
```

## What it does

When your agent builds, reviews, or refactors UI, this skill applies senior design-engineering judgment automatically. Instead of default shadows, generic easing, and mathematically-correct-but-visually-wrong alignment, your agent produces code that *looks* right.

Five principles guide every decision:

1. **Align to the eye, not the grid** — geometric correctness != perceptual correctness
2. **Animate with physics, not math** — springs over cubic-bezier, `bounce: 0` by default
3. **Sweat the invisible details** — everyone unconsciously feels them
4. **Restraint over eagerness** — the best motion is invisible
5. **Every value agrees on the same metaphor** — contradictory values break the illusion

## What it covers

- **Shadows** — three-layer transparent system (4-8% opacity), replacing borders
- **Color** — OKLCH for perceptual uniformity, shade scales without hue drift
- **Gradients** — oklab interpolation, color hints, blend-mode layering
- **Border radius** — concentric formula for nested elements
- **Optical alignment** — icon/text balance, SVG-level fixes, baseline-aware padding
- **Motion** — spring configs, duration hierarchy by user intent, gesture patterns
- **Micro-interactions** — button press feedback, icon swap animations, drag gestures
- **Performance** — compositor-only properties, will-change discipline, overflow-clip
- **Accessibility** — `prefers-reduced-motion`, focus states, keyboard equivalents
- **10-point review checklist** — systematic quality checks for any UI

## When it activates

The skill triggers automatically when your agent is:

- Building or styling UI components
- Adding animations, transitions, or hover/active/focus states
- Reviewing or refactoring frontend code
- Working with shadows, colors, gradients, spacing, typography, or border-radius
- Implementing drag interactions, layout animations, or icon transitions
- Asked to "think like a design engineer"

## Skill structure

```
SKILL.md                    Core principles, quick rules, review checklist
references/motion.md        Spring physics, gesture patterns, icon swaps, drag, shared layout
references/visual-craft.md  Shadow system, OKLCH color, gradients, optical alignment, performance
```

## License

MIT
