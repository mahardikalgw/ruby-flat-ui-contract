# Flat UI contract

Apply and audit a flat, borderless, token-only UI contract in a React + Tailwind v4 app — no raw interactive elements, no border/shadow/ring utilities, a seven-step type scale, four corner values, a visible focus ring with a documented text-field exemption, and an oklch palette with no literal colours. Use when writing or reviewing UI, adding a shadcn component, changing the theme, or fixing a failing UI/theme gate.

One skill, one repository. Part of a twelve-skill pack extracted from a production Rails 8 +
Inertia + React helpdesk monolith — the opinionated layer no public Rails or shadcn skill can know.
Version `2.0.0` tracks the source constitution.

## Install

```jsonc
// skills-lock.json
{
  "version": 1,
  "skills": {
    "flat-ui-contract": {
      "source": "mahardikalgw/flat-ui-contract",
      "sourceType": "github",
      "skillPath": "SKILL.md"
    }
  }
}
```

Or clone and symlink:

```bash
git clone git@github.com:mahardikalgw/flat-ui-contract.git
ln -s "$PWD/flat-ui-contract" .claude/skills/flat-ui-contract
```

## When to activate

- Writing or editing any `.tsx` under `app/javascript/{pages,components,layouts}`.
- Adding a shadcn component from the registry.
- Changing a colour, radius, type step or focus style.
- `bin/check-ui-components` or `bin/check-theme` fails.
- Reviewing a PR that touches UI.

## What's inside

```
flat-ui-contract/
  SKILL.md
  references/inv-1-9.md
  references/theme-tokens.md
  scripts/check-theme
  scripts/check-ui-components
```

`SKILL.md` is a router (kept under 250 lines) — the depth lives in `references/`, and the
copy-in code lives in `assets/`.

## Validate

```bash
./bin/check-skill
```

Checks frontmatter, the activation section, link resolution, orphaned references, script
executability, leftover placeholders, and — when the source app is checked out next to this
repository — that vendored scripts still match it byte for byte. Runs in CI on every push.

## Related skills

- [`merge-gate-parity`](https://github.com/mahardikalgw/merge-gate-parity) — one gate list in every file, a script contract, and local/CI triage
- [`inertia-vertical-slice`](https://github.com/mahardikalgw/inertia-vertical-slice) — route → skinny controller → query/service → typed props → Inertia page → test

## Licence

MIT — see [LICENSE](LICENSE).
