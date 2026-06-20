---
name: migrate-from-supabase-to-convex
description: Use when migrating a Next.js (App Router) app from Supabase (Postgres + RLS + Supabase Auth) to Convex (data + Convex Auth), or when the user invokes /migrate-from-supabase-to-convex. Covers schema/function porting, Convex Auth magic links via your own SMTP, the convex-test auth pattern, the Next-16 /api/auth gotcha, data migration with re-link-by-email, and the production cutover.
---

# Migrate from Supabase to Convex

Battle-tested playbook for moving a Next.js App Router app off Supabase
(Postgres + RLS + Supabase Auth) onto Convex (tables + functions + Convex
Auth). Distilled from a real migration that shipped to production. The
**gotchas below are the whole point** — each one cost real debugging time and
several are invisible to unit tests (they only surface in E2E / production).

## Process

Run it as: **brainstorm → spec → plan → subagent-driven TDD execution →
E2E gate → production cutover**. Do NOT skip the E2E gate — three of the
worst bugs here (the `/api/auth` 404, the rate-limit lockout, the SMTP 504)
were invisible to the 37 passing unit tests and only appeared in the browser /
in production.

The backend port (schema, all functions, auth, email, re-link) is highly
parallelizable with one subagent per task + a review per task. The frontend
wiring and the cutover are sequential and need a human-driven E2E pass.

## Architecture mapping

| Supabase | Convex |
|---|---|
| Postgres tables + RLS | `convex/schema.ts` tables + auth checks **inside** each function |
| Row Level Security policies | `getAuthUserId(ctx)` + explicit ownership checks in queries/mutations |
| Server Actions calling supabase-js | Convex `query`/`mutation`/`action`; client `useMutation`; SSR `preloadQuery` + live `usePreloadedQuery` |
| Supabase Auth (magic link) | Convex Auth `Email` provider (magic link) + your own SMTP |
| `auth.users` + `user_metadata` | Convex Auth `users` table (extend `authTables`) |
| service-role client (bypass RLS) | trusted server context — but keep the httpOnly-cookie write path as a Next Server Action |
| Realtime (opt-in) | reactive queries by default — lean into it (live tallies, etc.) |

## The gotchas (read these first)

### 1. convex-test auth: `subject` must be a real user `_id`
`getAuthUserId(ctx)` is literally `identity.subject.split("|")[0]`. In
`convex-test`, `t.withIdentity({ email: "x" })` does **not** work — the auto
subject isn't a valid `Id<"users">`, so any `organizerId: userId` insert fails
validation. Agents reliably get this wrong and then "fix" it by corrupting
production code (e.g. scanning the users table and grabbing `users[0]` — a
security disaster). The correct pattern:

```ts
async function signIn(t, email) {
  const userId = await t.run((ctx) => ctx.db.insert("users", { email }));
  return { userId, as: t.withIdentity({ subject: userId }) };
}
```
Production code stays clean: `const userId = await getAuthUserId(ctx)`.

### 2. Next.js 16 + @convex-dev/auth: `POST /api/auth` returns 404
The library relies on `convexAuthNextjsMiddleware` proxying `POST /api/auth`
to Convex. Under Next 16 the middleware body-response path no longer works and
**every sign-in 404s**. Fix: add an explicit `src/app/api/auth/route.ts` that
re-implements `proxyAuthActionToConvex` with public APIs (`fetchAction` from
`convex/nextjs`, `cookies`/`headers` from `next/headers`). Critical invariants
to preserve: never return the real refresh token to the client (return a
`"dummy"`, store the real one in the httpOnly cookie); `__Host-` prefix +
`secure` off localhost only; CORS/origin check; `import "server-only"`. Call
the auth actions via `makeFunctionReference<"action">("auth:signIn")` (a bare
string fails the production typecheck even though dev/Turbopack tolerates it).

### 3. Magic-link email must NOT block `signIn` (SMTP → 504)
The `Email` provider's `sendVerificationRequest` runs inside `signIn`, which
the `/api/auth` route awaits. If you `await ctx.runAction(sendEmail)` and your
SMTP is slow to greet (improvMX intermittently delays ~30s), the whole HTTP
request 504s. **Schedule** the email instead so `signIn` returns immediately:

```ts
await ctx.runMutation(internal.rateLimit.checkLoginRate, { email, ip: "unknown" });
await ctx.scheduler.runAfter(0, internal.email.sendMagicLink, { email, url, token });
```
And harden the Node mail action: `connectionTimeout`/`greetingTimeout` ~10s,
`requireTLS` for STARTTLS on 587, fresh transport per attempt, 2–3 retries.
Nodemailer is Node-only → the action file must start with `"use node";`.

### 4. Rate limiting: there is no client IP inside Convex
Convex functions have no request headers. Don't pass a constant `ip:"unknown"`
into a shared bucket — it becomes a **global, shared-fate lockout** (one user's
attempts lock out everyone after the cap). Enforce the IP bucket only for a
*real* IP; the per-email cap is the effective guard. Real per-client IP limits
belong in the Next `/api/auth` layer (which has `headers()`).

### 5. Determinism: generate random tokens in an `action`, not a mutation
Mutations must be deterministic. Generate the anonymous-voter `editToken`
(or any CSPRNG value) in an `action` via `crypto.getRandomValues`, then persist
it through a mutation. Don't call CSPRNG inside a mutation.

### 6. Keep secrets server-only
Port a feature like an httpOnly per-resource cookie (an anonymous voter's
`editToken`) by keeping its **write path as a Next.js Server Action** that
reads/sets the cookie and calls Convex via `fetchMutation` — `useMutation` from
the browser can't manage httpOnly cookies. And strip such secrets from any
read query's projection (don't return `editToken` from the public tally query).

### 7. Vitest / tsconfig friction
- `convex-test` files use `import.meta.glob` (Vite-only) → **exclude
  `**/*.test.ts` from `convex/tsconfig.json`** or `npx convex dev` typecheck
  fails.
- Run `convex/**` tests under `edge-runtime`; in Vitest v4 use the `projects`
  config (the old `environmentMatchGlobs` was removed and breaks the Next build
  typecheck).
- **Exclude `scripts/` from the root (Next) tsconfig** — one-time migration
  tooling imports deps you intentionally don't ship (`@supabase/supabase-js`),
  which otherwise fails `next build`.

### 8. E2E needs a test-only auth bypass
Magic-link login can't be driven in a browser. Add a Convex Auth `Password`
provider gated by `ENABLE_TEST_AUTH==="true"` (Convex deployment env) plus a
`<DevSignIn>` form gated by `NEXT_PUBLIC_ENABLE_TEST_AUTH==="true"` that returns
`null` otherwise. Default-off → inert in production (verify it's absent from
prod env). React-19 forms submit reliably via `agent-browser press Enter`, not
button `click`.

## Schema notes (`convex/schema.ts`)

- Spread `...authTables` and re-declare `users` with your extra fields
  (`displayName`, `defaultTimezone`) — all base fields stay optional.
- Use **camelCase** (`startAt`, `huddleId`, …). Port shared TS types/utils to
  match; the pure algorithms (tally, timezone math) carry over unchanged.
- Convex sets `_id`/`_creationTime`, but to **preserve original timestamps**
  on migrated rows keep explicit `createdAt: v.number()` fields.
- Add a `legacyId: v.optional(v.string())` on every migrated table so the
  import can resolve old foreign keys, and index what re-link needs
  (`by_organizerEmail`, etc.).

## Data migration

1. **Export** (read-only, service-role key): dump tables, and join
   `auth.users` via the **Admin API** `supabase.auth.admin.listUsers()`
   (paginate) to attach the **organizer/voter email** to each row — this is
   what lets users re-claim their data by signing in. Preserve `editToken` and
   timestamps. Paginate `.range()` (PostgREST caps at ~1000) and surface, don't
   silently drop, rows whose organizer user is missing.
2. **Transform** → Convex shape with `legacyId`/`legacy*Id` FK refs.
3. **Seed** via an **idempotent** `internalMutation`: insert
   huddles→slots→voters→votes in dependency order, resolving FKs through
   `legacyId → _id` maps (populate the map from *existing* rows on re-runs so a
   second run is a no-op), patch `winningSlotId` after slots exist, upsert
   profiles by email. New owner rows get `organizerId: null`.
4. **Re-link on first sign-in** via `callbacks.afterUserCreatedOrUpdated`: seed
   the profile from `migratedProfiles[email]`, claim `huddles` where
   `organizerEmail == email && organizerId == null`, attach voters by
   `userEmail`. So **users are NOT migrated** — they re-sign-in and reclaim by
   email.

## Production cutover (order matters)

```bash
npx convex login                 # link the anonymous/local dev deploy to a cloud project
npx convex deploy -y             # push functions to PROD (prints the prod .convex.cloud URL)
npx @convex-dev/auth --prod --web-server-url https://<prod-domain>   # JWT keys + SITE_URL on prod
npx convex env set --prod SMTP_HOST … SMTP_PORT 587 SMTP_SECURE false SMTP_USER … SMTP_PASS … SMTP_FROM …
# Vercel: set NEXT_PUBLIC_CONVEX_URL=<prod convex url> (Production), deploy:
npx vercel --prod --yes
# Data: export from Supabase → run the seed mutation against PROD → validate counts.
```
- **Verify SMTP early** by invoking the email action directly
  (`npx convex run --prod email:sendMagicLink '{...}'`) before trusting it.
- Confirm `ENABLE_TEST_AUTH` is **absent** in prod (Convex + Vercel).
- Stop the local backend (`convex-local-backend` on :3210) before re-linking
  or it errors "local backend still running".

## Secrets handling (don't leak service keys)

- `vercel env pull` returns **Sensitive** env vars empty — you can't recover a
  service-role key that way.
- The Supabase CLI binary gets **SIGKILL'd on Apple Silicon** (codesign) —
  re-sign it: `codesign --force --sign - <path-to>/supabase`. Then
  `supabase projects api-keys … -o env --save…`/redirect to a file (never print).
- Never print a service-role/secret key into the transcript; write it to a
  gitignored file and source it. Gitignore the migration export
  (`migration/`) — it contains emails and `editToken`s.

## Is the Convex infra "as code"?

- **Yes**: schema, functions, `auth.ts`/`auth.config.ts`/`http.ts` live in
  `convex/` and are committed — that's the backend, fully reproducible.
- **No (by design)**: deployment env vars (SMTP, JWT keys, `SITE_URL`, feature
  flags) are set via `npx convex env set` on the deployment, and deploy keys
  live in Vercel — not in the repo. Codify the *build integration* (below);
  keep the secrets out.

## Preview / PR environments (Vercel + Convex) — verified end to end

Default Vercel previews are NOT functional for a Convex app (no
`NEXT_PUBLIC_CONVEX_URL`, no backend). To give every PR its own isolated Convex
backend:

1. **Integrated build** — commit `vercel.json`:
   ```json
   { "buildCommand": "npx convex deploy --cmd 'npm run build'" }
   ```
   Each Vercel build then deploys the matching Convex backend and auto-injects
   `NEXT_PUBLIC_CONVEX_URL` / `NEXT_PUBLIC_CONVEX_SITE_URL` into `npm run build`.
2. **Deploy keys** (`CONVEX_DEPLOY_KEY` in Vercel):
   - **Production scope** — a prod key:
     `npx convex deployment token create ci --prod --save-env <file>` (writes to
     a file instead of printing).
   - **Preview scope** — a **preview** key (prefix `preview:`). The **CLI cannot
     mint preview keys — generate it in the Convex dashboard** (Settings →
     Deploy Keys). Without it, preview builds fail fast with
     `✖ Vercel build environment detected but no Convex deployment configuration found`.
   - Vercel CLI gotcha: `vercel env add NAME preview --value '<v>' --yes`
     (older Vercel CLI, e.g. 52.x, ignores these flags — upgrade to latest).
3. **Verify** by reading the preview build log — a working run prints:
   `▌ [Preview] team:project:preview/<branch> └─ https://<name>.convex.cloud`
   and `✔ Deployed Convex functions to https://<name>.convex.cloud`.

### Preview deployments start EMPTY — auth crashes until you provision env

The deploy key gets you a preview *deployment*, but it has **no env vars and no
data**. Magic-link sign-in 500s with
`Missing environment variable SITE_URL` (in `redirectAbsoluteUrl` /
`handleEmailAndPhoneProvider`) until the preview deployment has auth config.
Provision a specific preview deployment by name (from the build log):
```bash
npx @convex-dev/auth --deployment-name <name> --web-server-url <preview-url>   # JWT keys + SITE_URL
npx convex env set --deployment <name> SMTP_HOST … SMTP_PASS …                  # email
```
- **Durable fix for all previews:** set **Preview default environment
  variables** in the Convex dashboard for the *static* values (`JWT_PRIVATE_KEY`,
  `JWKS`, `SMTP_*`) so every new preview inherits them.
- **`SITE_URL` is the awkward one** — it's per-preview (each Vercel preview has a
  different URL). Either set it per-branch (precise, manual), default it to a
  fixed URL (sign-in works but redirects land there, not the preview), or
  automate it from `VERCEL_BRANCH_URL` at build / via a `--preview-run` setup
  function for zero-touch preview auth.
- The **test-auth Password bypass** only renders on a preview if
  `NEXT_PUBLIC_ENABLE_TEST_AUTH` is set in the Vercel **Preview** scope at build
  time (it's a build-time `NEXT_PUBLIC_*` flag). Handy for exercising previews
  without email — but never set it (or `ENABLE_TEST_AUTH`) in Production.

## References in a real implementation

- `src/app/api/auth/route.ts` — the Next-16 auth proxy
- `convex/auth.ts` — magic-link provider, async email, re-link callback, env-gated test Password provider
- `convex/votes.ts` — security parity (slot-ownership guard, token-stripped tally, dedupe, token action)
- `convex/migrate.ts` + `scripts/export-supabase.mts` + `scripts/transform.ts` — the data pipeline
- `docs/runbooks/convex-cutover.md` + an ADR superseding the Supabase one
