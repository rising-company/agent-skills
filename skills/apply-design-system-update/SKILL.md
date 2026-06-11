---
name: apply-design-system-update
description: Use when a design-system token (color, size, spacing, font) has changed and the change must be propagated to downstream apps that copy tokens inline rather than importing a shared package, or when the user invokes /apply-design-system-update.
---

# Apply Design System Update

Some design systems are **not** shipped as a package — downstream apps copy the
tokens inline (CSS variables, Tailwind config, raw values). When that's the case,
every token change must be swept into each consumer by hand. Drift is silent: an
app keeps shipping old values until someone sweeps it.

## Core principle

**Grep by the old VALUE, not by the token name.** Variable names differ across
apps (`--color-danger`, `--accent-danger`, `danger:`), but the value (e.g. a hex)
is the same everywhere. Then **classify each hit** before replacing — not every
match is a token (see the judgment call below).

## Step 1 — Identify what changed

Get the exact old→new pairs from the design-system `git diff` (or ask the user).
Example: `#cc2200 → #ff4a24`. Note: **additive** changes (new tokens, new docs,
new rules) need NO consumer sweep — only changed *values* propagate.

## Step 2 — Find every occurrence

Search the workspace by value, excluding build artifacts and the design-system
source itself:

```bash
grep -rIl -i "OLDVALUE1\|OLDVALUE2" . \
  --exclude-dir=node_modules --exclude-dir=.next --exclude-dir=dist \
  --exclude-dir=.git --exclude-dir=build \
  | grep -v "/docs/" | sort
# then re-run with -n (no -l) to see each line in context
```

Build the consumer list from the grep results, not from memory — the set of apps
and the file each keeps tokens in drifts over time. Common locations: CSS custom
properties (`--color-*` / `--accent-*` in a global stylesheet), a Tailwind config
(named theme keys), raw inline values in plain HTML/CSS, and any email templates.
An app only matters if it actually uses the changed value.

## Step 3 — Classify each hit (the judgment call)

A value match is **not always a token**. Replace only semantic-token usages.
Leave coincidental literal colors that happen to equal the old value.

| Replace (token usage) | Leave (literal / physical color) |
|---|---|
| `--color-danger: #cc2200` | A red object's color (e.g. a 3D material for a red cable) |
| `danger: "#cc2200"` in a Tailwind config | A chart series color, brand artwork, an illustration |
| Error/warning UI state styling | Anything where the value means "this thing is red," not "this is our error color" |

When unsure, read the surrounding lines. If removing it from the token system
wouldn't change its meaning (a cable stays red), it's a literal — leave it and
flag it to the user. 3D-diagram / illustration pages are the classic trap: their
colors describe physical objects, not UI state.

## Step 4 — Sweep and verify

Sweep only the classified token files. A file-wide `sed` is safe **only** when
you confirmed every occurrence in that file is a token (check the `-n` grep).

```bash
files=( path/to/globals.css path/to/tailwind.config.ts )   # token files only
for f in "${files[@]}"; do
  sed -i '' -e 's/OLDVALUE1/NEWVALUE1/g' -e 's/OLDVALUE2/NEWVALUE2/g' "$f"
done
grep -rIn -i "OLDVALUE1\|OLDVALUE2" "${files[@]}" || echo "clean"   # expect none
grep -rIn -i "NEWVALUE1\|NEWVALUE2" "${files[@]}"                   # confirm new
```

macOS `sed` needs `-i ''`. Match the existing format in each file (quoted in a
JS/TS config, bare value in CSS, lowercase).

## Step 5 — Report, don't auto-commit

- Show the user the full list of edited files + any hits you deliberately left
  (with the reason). Let them eyeball before committing.
- For color changes, a side-by-side before/after render helps the user judge by
  eye.
- Note the current branch of each repo — an app may be mid-feature on a branch
  other than its default; isolate the token change onto its own branch so
  unrelated in-progress commits don't ride along.
- Don't commit/push unless asked; each app may be its own repo.

## Common mistakes

- **Grepping by token name** → misses apps that use a different variable name.
- **Blind file-wide `sed` across the workspace** → corrupts literal colors
  (illustrations, 3D materials) and matches inside build dirs.
- **Sweeping additive changes** → new tokens/rules don't exist in consumers yet;
  there's nothing to replace. Only changed values propagate.
- **Forgetting non-obvious surfaces** → e.g. email templates, not just app UI.
