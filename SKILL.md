---
name: flat-ui-contract
description: Apply and audit a flat, borderless, token-only UI contract in a React + Tailwind v4 app — no raw interactive elements, no border/shadow/ring utilities, a seven-step type scale, four corner values, a visible focus ring with a documented text-field exemption, and an oklch palette with no literal colours. Use when writing or reviewing UI, adding a shadcn component, changing the theme, or fixing a failing UI/theme gate.
version: 2.0.0
---

# Flat UI contract

The product separates surfaces with **tone and spacing**, never with a line or a shadow. That single
decision generates every rule below, and two scripts enforce them. Read the rules, then run the scripts
— the script tells you what you broke, this file tells you how to fix it without a second attempt.

## When to activate

- Writing or editing any `.tsx` under `app/javascript/{pages,components,layouts}`.
- Adding a shadcn component from the registry.
- Changing a colour, radius, type step or focus style.
- `bin/check-ui-components` or `bin/check-theme` fails.
- Reviewing a PR that touches UI.

## The nine component invariants

| # | Rule | Fails when |
|---|---|---|
| INV-1 | No raw `<button>`, `<input>`, `<select>`, `<textarea>` outside `components/ui` | Anything hand-rolls an interactive control |
| INV-2 | Focus stays visible: no `outline-none`; every control keeps a `:focus-visible` outline — **text fields keep their resting 1px hairline instead of a ring** | A keyboard user loses their position |
| INV-3 | Every `components/ui` component has ≥ 1 import site | A dead component ships |
| INV-4 | Every page imports one of four sanctioned layouts; only layout files render nav/header | A second chrome implementation |
| INV-5 | No `shadow-*`, no `ring-*` (both are box-shadows) | Depth is faked |
| INV-6 | No border utilities; `border-0`, `border-collapse`, `border-transparent`, `border-spacing`, and colour roles like `bg-border` stay legal | A boxed look returns |
| INV-7 | Seven type steps, nothing above `text-2xl`, no `text-[…]`, no inline `fontSize` | Arbitrary sizes drift in |
| INV-8 | Corners only `rounded-sm\|md\|lg\|full\|none`; no `rounded-xl`, `rounded-[…]` | Off-token corners |
| INV-9 | Tabular figures declared once on `table` and `[data-numeric]`; nothing opts out with `proportional-nums` | Numeric columns stop aligning |

## The three theme invariants

| # | Rule |
|---|---|
| THEME-1 | Every colour declared in `:root` is mapped in `@theme inline`, and every `--color-*` mapping resolves to a declared role |
| THEME-2 | No literal colour exists outside the palette block: no hex, `rgb()`, `hsl()`, `bg-white`/`bg-black`, no Tailwind palette classes — in app code **or** in static pages |
| THEME-3 | Every recorded foreground/background pairing meets its WCAG threshold, recomputed from the live `oklch()` values |

Adding a colour means adding a **role** (`--positive`, `--warning`), mapping it, and recording its
pairing in the checker's matrix with the threshold you expect. Never relax a threshold to make a change
pass.

## How to build a surface, since you cannot use borders

| Need | Use |
|---|---|
| Card / panel | White fill on the canvas; the canvas, not the border, makes it read as raised |
| Table header, chip, hover, track | The "well" fill (`bg-secondary`, `bg-muted`) |
| Grouping inside a card | Spacing, plus a section heading at a smaller step |
| A floating menu over white | The one legitimate hairline (`--border`), because the menu has no canvas beneath it |
| Focus | A visible **outline** ring (`outline-2 outline-offset-2`), never a box-shadow |
| Text field rest state | A 1px `outline` in `--input`, kept while focused |
| Depth | Tone: canvas → card → well. Three steps is all a screen needs |

## Type scale

Seven steps only, mapped in `application.css`:

```
text-2xs  caption, meta, uppercase table headers
text-xs   helper text, badges
text-sm   body, table cells, form labels
text-base section copy
text-lg   card titles, panel headings
text-xl   page titles
text-2xl  the single largest heading on a page
```

`text-2xl` is the ceiling. A hero number that needs to be bigger uses a chart or a KPI card, not a
larger font size. Arbitrary values (`text-[11px]`) are how a screen drifts — the checker flags them
because they were used 22 times before the rule existed.

## Repair order when a gate fails

1. **Raw element (INV-1)** → replace with the `components/ui` equivalent. If none exists, add it from
   the shadcn registry; do not hand-roll.
2. **Border (INV-6)** → delete it and add the well fill or spacing that does the job. A `border-b` on a
   table row becomes `bg-secondary/60` on the header and nothing on the body.
3. **Shadow/ring (INV-5)** → delete it. If it was a focus ring, it becomes an outline.
4. **Type step (INV-7)** → pick the nearest step from the scale; if nothing fits, the layout is wrong,
   not the scale.
5. **Corner (INV-8)** → `rounded-sm|md|lg`; `rounded-full` for avatars, pills and icon buttons.
6. **Colour (THEME-2)** → find the role that already means this (`--positive` for a success tint) or
   add a role; never inline a hex.
7. **Focus (INV-2)** → restore the `:focus-visible` outline in the stylesheet; only text fields are
   exempt and they keep their 1px outline.

## Run the gates

```bash
bin/check-ui-components      # INV-1..INV-9
bin/check-theme              # THEME-1..THEME-3, recomputes contrast
```

Both are copied into `scripts/` so another project can adopt them:

```bash
cp skills/flat-ui-contract/scripts/check-ui-components bin/
cp skills/flat-ui-contract/scripts/check-theme bin/
chmod +x bin/check-ui-components bin/check-theme
```

Then wire them into both CI definitions — see `merge-gate-parity`.

## References

- [references/inv-1-9.md](references/inv-1-9.md) — each invariant with the exact regex and a fix example
- [references/theme-tokens.md](references/theme-tokens.md) — palette block, roles, mapping, contrast matrix
- [scripts/check-ui-components](scripts/check-ui-components) — the component gate
- [scripts/check-theme](scripts/check-theme) — the palette and contrast gate

