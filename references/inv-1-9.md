# INV-1 … INV-9 with the exact rules

Each entry gives the pattern the checker matches and the rewrite that satisfies it.

## INV-1 — no raw interactive element outside `components/ui`

```ruby
RAW_ELEMENTS = [ /<button\b/, /<input\b/, /<select\b/, /<textarea\b/ ]
```

Applies to `pages/**`, `layouts/**`, `components/**` — everything except `components/ui/**`.

```tsx
// no
<button className="px-3 py-1.5 text-sm" onClick={save}>Simpan</button>

// yes
import { Button } from '@/components/ui/button'
<Button onClick={save}>Simpan</Button>
```

If the primitive does not exist, add it from the registry:

```bash
npx shadcn@latest add toggle-group
```

A hand-rolled control also bypasses the focus, disabled and `asChild` behaviour every other control
already has.

## INV-2 — focus stays visible

Forbidden:

```
focus:outline-none     focus-visible:outline-none     outline-none
```

Required in the stylesheet:

```css
:focus-visible {
  outline: 2px solid var(--ring);
  outline-offset: 2px;
}
```

**The text-field exemption.** A text field keeps its resting hairline and does not paint the ring:

```css
input:focus-visible,
textarea:focus-visible {
  outline: 1px solid var(--ring);
  outline-offset: 0;
}

/* The highlighted row of an open listbox or menu is announced by its fill. */
[role="option"]:focus-visible,
[role="menuitem"]:focus-visible {
  outline: none;
}
```

The select trigger is a Radix `<button>`, so the stylesheet exemption above cannot reach it — it pins
its own `focus-visible:outline-1`. Both halves (no ring in the component, ring in the stylesheet) are
checked so neither can drift back alone.

## INV-3 — no orphaned component

Every file in `components/ui/*.tsx` must appear in at least one import site anywhere under
`app/javascript`. Imports from inside the library count (`toggle-group` imports `toggle`).

Removing a component from the product means deleting the file. A component kept "for later" is a
component that ships untested, unstyled and unread.

## INV-4 — one signed-in shell

- Every page must import one of: `@/layouts/app-layout`, `@/layouts/dashboard-layout`,
  `@/layouts/auth-layout`, `@/layouts/marketing-layout`.
- Only those four files may render a `<nav>` together with a `<header>`.
- A layoutless page is allowed only for the shared error page (it renders without chrome when there is
  no session to render chrome for).

## INV-5 — no shadow, no ring

```ruby
SHADOW_UTILITIES = [ /\bshadow-[a-z0-9\[]/, /\bring-(?:\[|[0-9])/ ]
```

`ring-*` is a box-shadow under the hood, which is why it is grouped with shadows rather than with
focus. The focus indicator is an **outline**, which does not paint a box.

## INV-6 — no border utility

```ruby
BORDER_UTILITY = /(?<![\w-])border(?:-[a-z]+)?(?:-\d+)?(?:\[\d+(?:\.\d+)?px\])?(?:\/\d+)?(?![\w-])/
BORDER_ALLOWED = %w[border-0 border-collapse border-separate border-transparent border-spacing]
```

Legal on purpose:

- `border-0` — how the forms plugin's default border is switched off.
- `border-collapse`, `border-separate`, `border-spacing` — table layout, draws nothing.
- `border-transparent` — a resize handle or an offset.
- A **colour role** such as `bg-border` or `outline-border` is not a border utility and passes.

```tsx
// no
<Card className="border border-border">…</Card>
// yes
<Card className="bg-card">…</Card>
```

## INV-7 — seven type steps

```ruby
ILLEGAL_TEXT_SIZE = [ /text-\[[^\]]*\]/, /text-(?:3xl|4xl|5xl|6xl|7xl|8xl|9xl)\b/, /\bfontSize:/ ]
```

Exempt: the vendored chart primitive (`components/ui/chart.tsx`) sizes its own SVG ticks.

## INV-8 — four corner values

```ruby
ILLEGAL_RADIUS = [ /rounded-(?:xs|xl|2xl|3xl|4xl)(?![\w-])/, /rounded-\[[^\]]*\]/ ]
```

Allowed: `rounded-sm`, `rounded-md`, `rounded-lg`, `rounded-full`, `rounded-none`, plus the side
variants (`rounded-t-md`). All three token steps resolve to the same 5px value — the rule is about
staying on tokens, not about offering a scale.

## INV-9 — tabular figures, declared once

The stylesheet must contain:

```css
table { font-variant-numeric: tabular-nums; }
[data-numeric] { font-variant-numeric: tabular-nums; }
```

Nothing may opt out:

```ruby
/proportional-nums/   # violation anywhere under app/javascript/**/*.{css,tsx}
```

A rule that lives only in one table component is silently bypassed by the next table somebody writes
by hand.

## Running one rule in isolation

The scripts run whole-contract. During a large refactor, run them and filter:

```bash
bin/check-ui-components | grep INV-6
bin/check-theme | grep THEME-3
```

Both print `path:line: message` and exit non-zero, so they work unchanged as a CI step.
