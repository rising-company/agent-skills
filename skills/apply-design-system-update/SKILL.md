---
name: apply-design-system-update
description: Use when a Rising Company design-system token (color, size, spacing, font) has changed in ~/rising-company/design-system and must be propagated to the consumer apps that copy tokens inline, or when the user invokes /apply-design-system-update.
---

# Apply Design System Update

The Rising design system is **not** a package. Consumer apps copy its tokens
inline, so every token change must be swept into each consumer by hand. Drift
is silent — a product keeps shipping old values until someone sweeps it.

## Core principle

**Grep by the old VALUE, not by the token name.** Variable names differ across
apps (`--color-danger`, `--accent-danger`, `danger:`), but the hex/value is the
same everywhere. Then **classify each hit** before replacing — not every match
is a token (see the judgment call below).

## Step 1 — Identify what changed

The canonical token surface inside `design-system/` is four files kept in sync:
`llms.txt`, `index.html`, `skills/rising-design/rising-design.md`,
`skills/rising-design/components.md`.

Get the exact old→new pairs from `git diff` (or ask the user). Example:
`#cc2200 → #ff4a24` (danger), `#ddaa30 → #f5b83d` (warning). Note: **additive**
changes (new tokens, new docs, new rules) need NO consumer sweep — only changed
*values* propagate.

## Step 2 — Find every occurrence

```bash
cd ~/rising-company
grep -rIl -i "OLDVALUE1\|OLDVALUE2" . \
  --exclude-dir=node_modules --exclude-dir=.next --exclude-dir=dist \
  --exclude-dir=.git --exclude-dir=build \
  | grep -v "/docs/" | grep -v "design-system/" | sort
# then re-run with -n (no -l) to see each line in context
```

Consumers (verify against the grep, this list drifts): **Next.js + Tailwind** —
`os`, `huddle`, `safe`, `venue-map` (tokens in `src/app/globals.css` as
`--color-*` and/or `tailwind.config.ts` as named keys; auth emails in
`supabase/templates/*.html`). **Plain CSS/HTML** — `www`, `wave-maker`,
`mesh-maker` (`demo/src/style.css` as `--accent-*`), `rv-diagrams` (raw inline
colors). **NOT consumers:** `resume`, `maple-candy-bakery`, `agent-handbook`,
`agent-skills`, `dotenv`. An app appears only if it actually uses that token.

## Step 3 — Classify each hit (the judgment call)

A value match is **not always a token**. Replace only semantic-token usages.
Leave coincidental literal colors that happen to equal the old hex.

| Replace (token usage) | Leave (literal/physical color) |
|---|---|
| `--color-danger: #cc2200` | A red object's color (`posWireMat = mat(0xcc2200…)` — a red cable) |
| `danger: "#cc2200"` in tailwind | A brass terminal `0xddaa30`, 3D material, chart series, brand art |
| Error/warning UI state styling | Anything where the hex means "this physical thing is red," not "this is our error color" |

When unsure, read the surrounding lines. If removing it from the token system
wouldn't change its meaning (a cable stays red), it's a literal — leave it and
flag it to the user. `rv-diagrams` is the canonical example: its danger/warning
hits are 3D material colors, not tokens.

## Step 4 — Sweep and verify

Sweep only the classified token files. A file-wide `sed` is safe **only** when
you confirmed every occurrence in that file is a token (check the `-n` grep — the
Next/Tailwind/demo token files typically hold exactly the definition lines).

```bash
files=( app1/src/app/globals.css app1/tailwind.config.ts … )   # token files only
for f in "${files[@]}"; do
  sed -i '' -e 's/OLDVALUE1/NEWVALUE1/g' -e 's/OLDVALUE2/NEWVALUE2/g' "$f"
done
grep -rIn -i "OLDVALUE1\|OLDVALUE2" "${files[@]}" || echo "clean"   # expect none
grep -rIn -i "NEWVALUE1\|NEWVALUE2" "${files[@]}"                   # confirm new
```

macOS `sed` needs `-i ''`. Match the existing format in each file (quoted in
tailwind, bare `#hex` in CSS, lowercase).

## Step 5 — Report, don't auto-commit

- Show the user the full list of edited files + any hits you deliberately left
  (with the reason). Let them eyeball before committing.
- For color changes, a side-by-side before/after render helps the user judge by
  eye — identity/vibe wins ties over strict contrast.
- **`os` develops on branch `test`, not `main`** — note the current branch.
- Don't commit/push unless asked; each app is its own repo.

## Common mistakes

- **Grepping by token name** → misses apps that use a different variable name.
- **Blind file-wide `sed` across the workspace** → corrupts literal colors
  (cables, 3D materials) and matches inside `node_modules`/`.next`/`docs`.
- **Sweeping additive changes** → new tokens/rules don't exist in consumers yet;
  there's nothing to replace. Only changed values propagate.
- **Forgetting `supabase/templates/*.html`** auth emails in the Next.js apps.
