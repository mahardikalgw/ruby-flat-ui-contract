# Palette, roles and contrast

One stylesheet owns every colour: `app/javascript/entrypoints/application.css`. Two blocks, one
direction: roles are declared in `:root`, then mapped into Tailwind in `@theme inline`.

## Structure

```css
@import 'tailwindcss';
@import 'tw-animate-css';

:root {
  /* roles — the only place a colour value may exist */
  --background: oklch(0.976 0.0025 264);   /* canvas, the single non-white surface */
  --card: oklch(1 0 0);                    /* every surface: card, table, rail, top bar */
  --foreground: oklch(0.2077 0.0148 264);
  --muted-foreground: oklch(0.4914 0.014 264);
  --secondary: oklch(0.9425 0.0035 264);   /* wells: chips, table header, hover, track */
  --primary: oklch(0.429 0.1455 18.39);
  --primary-foreground: oklch(1 0 0);
  --destructive: oklch(0.5397 0.1829 31.22);
  --positive: oklch(0.4769 0.0881 153.61);
  --warning: oklch(0.551 0.1277 57.1);
  --border: oklch(0.9225 0.003 264);       /* hairlines only: floating menu, form state */
  --input: oklch(0.6391 0.003 264);
  --ring: oklch(0.429 0.1455 18.39);
  --radius: 0.3125rem;                     /* 5px */
}

@theme inline {
  --color-background: var(--background);
  --color-card: var(--card);
  --color-foreground: var(--foreground);
  --color-muted-foreground: var(--muted-foreground);
  --color-secondary: var(--secondary);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-destructive: var(--destructive);
  --color-positive: var(--positive);
  --color-warning: var(--warning);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --radius-sm: var(--radius);
  --radius-md: var(--radius);
  --radius-lg: var(--radius);
}
```

Rules:

- **oklch only.** A hex value is not a role, and mixing colour spaces makes two surfaces that should
  match drift by a perceptible amount.
- **A role per meaning, not per value.** `--positive` and `--destructive` are meanings; `--chart-3` is a
  series. A new screen reuses a meaning.
- **A paired foreground.** Every fill that carries text declares its own `*-foreground`, so contrast is
  a property of the palette rather than of each call site.
- **Light theme only** unless a theme is specified. A dark theme doubles the matrix below and is a
  feature, not a refactor.

## Contrast matrix

THEME-3 is not advisory: every pair is recomputed from the live `oklch()` values, composited the way
the browser paints them (tints over white, alpha text over its own background).

| Foreground | Background | Min ratio | Meaning |
|---|---|---|---|
| `--foreground` | `--background` | 4.5 | body copy on canvas |
| `--foreground` | `--card` | 4.5 | body copy on a surface |
| `--muted-foreground` | `--card` | 4.5 | helper text, table headers |
| `--muted-foreground` | `--secondary` | 4.5 | labels on a well |
| `--secondary-foreground` | `--secondary` | 4.5 | secondary button label |
| `--primary-foreground` | `--primary` | 4.5 | primary button label |
| `--destructive-foreground` | `--destructive` | 4.5 | destructive button label |
| `--positive-foreground` | `--positive` @18% | 4.5 | positive badge text |
| `--warning-foreground` | `--warning` @15% | 4.5 | warning alert text |
| `--negative-foreground` | `--negative` @16% | 4.5 | negative badge text |
| `--muted-foreground` @70% | `--card` | 3.0 | disabled nav item (large/decorative) |

Changing a value means re-measuring every row it appears in. Changing a threshold to make a change pass
is the one edit that turns the gate into decoration.

## Adding a colour

1. Add the role to `:root` with a short comment stating what it means.
2. Map `--color-<role>` in `@theme inline`.
3. Add every pairing that uses it to the checker's matrix, with the threshold you are claiming.
4. Run `bin/check-theme` and fix the value, never the threshold.
5. Add a badge or alert variant that consumes it, in `components/ui`, so the role has one consumer.

## Static pages and the icon

`public/*.html` and `public/icon.svg` cannot read CSS variables. The checker allows them a tiny fixed
palette (`#F7F7F8`, `#191A1C`, `#8E1F2F`, `#E6E6EA`) and the icon one brand colour. Anything else in a
static page is a literal colour that will not follow a theme change.

## Semantic chart colours

```css
--chart-1: oklch(0.429 0.1455 18.39);   /* identity */
--chart-2: oklch(0.4775 0.0783 185.45);  /* teal */
--chart-3: oklch(0.6111 0.1204 79.93);   /* gold */
--chart-4: oklch(0.457 0.0262 256.8);    /* slate */
--chart-5: oklch(0.5528 0.104 117.26);   /* olive */
```

A chart series is a value, not a meaning — but it still comes from the palette, so a screen never
introduces a sixth hue because a chart needed one.
