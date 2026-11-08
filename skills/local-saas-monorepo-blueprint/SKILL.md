---
name: local-saas-monorepo-blueprint
description: Scaffold or evolve a TypeScript SaaS monorepo in the RecPlus style — Turborepo + pnpm, Hono API with orval-generated per-client SDKs, Drizzle multi-tenant DB, strict TS conventions, and AI instruction files. Use when starting a new SaaS project, adding an app/package to a monorepo, or reviewing one for structural issues.
---


# SaaS Monorepo Blueprint

Extracted from the RecPlus parks-and-recreation SaaS monorepo (Turborepo + pnpm workspaces, Hono API, Next.js portals, Drizzle ORM, WorkOS, Stripe). This skill encodes the patterns worth copying and the mistakes worth avoiding.

## Purpose

Give an AI agent (or a developer) a concrete, opinionated starting layout for a multi-tenant B2B SaaS, plus the operating conventions that keep it healthy.

## Workflow

### 1. Scaffold the repo layout

```
<repo>/
  apps/
    api/                 # Hono server (port 3002)
    staff-portal/        # Next.js backoffice (port 3001)
    customer-portal/     # Next.js public app (port 3000)
  packages/
    db/                  # Drizzle schemas, relations, migrations, seed scripts
    central-db/          # registry DB pointing at per-client databases (multi-tenant)
    schemas/             # shared zod schemas / branded types (source of truth)
    dtos/                # shared DTOs
    sdks/                # shared SDKs
    design-system/       # shared React component library
    emails/              # react-email templates
    helpers/ utils/      # shared logic
    eslint-config/ typescript-config/
  scripts/               # db-setup.sh, compose wrapper
  k6-tests/              # smoke + load tests
  turbo.json  orval.config.ts  pnpm-workspace.yaml  docker-compose.yaml
  AGENTS.md              # AI instructions (see section 5)
```

- pnpm workspaces `apps/*`, `packages/*`; pin shared versions with a pnpm `catalog:`.
- One `.env.example` per app plus a root one; **never commit real `.env` files.**
- Port convention: customer 3000, staff 3001, api 3002.

### 2. API-first development

1. Define routes in the API with zod schemas via `zod-openapi` / `hono-openapi`.
2. API build emits `apps/api/openapi.yaml`.
3. Run `pnpm orval` (root `orval.config.ts`): generates **per-portal, tag-filtered SDKs** into each portal's `src/sdk` — e.g. `staffPortal` excludes `customer portal` + `internal` tags, `customerPortal` excludes `staff portal` + `internal`. Tag-split, kebab-case, fetch client, mock data via faker, zod validators for body + query.
4. Frontends consume only the generated SDK. Never hand-write API clients.

See `references/module-layout.md` for the module/repository/service file conventions.

### 3. Database workflow (Drizzle)

1. Model the domain first as zod schemas in `packages/schemas` (branded types, validations).
2. Then create the Drizzle table reusing those types.
3. **Never hand-write migration SQL.** Dev loop: `pnpm migration:push` (with confirmation prompt). Before merge: `pnpm migration:generate`. Production: `pnpm migration:migrate`.
4. Seeds: `db:base` (required baseline data) then `db:seed` (fake data). `db:reset` takes a data-only backup first; `db:backup` for full backups.
5. Multi-tenant: one database per client + a `central-db` registry mapping clients to their DB. Per-client data migrations live in `apps/api/src/scripts/<client>-migration/`.

### 4. Cross-cutting services

- **Auth:** WorkOS (AuthKit on frontends, node SDK on API). Keep `create-accounts` / `delete-removed-accounts` scripts for lifecycle sync.
- **Payments:** Stripe for cards; keep a second gateway (e.g. CardPointe/Snappay) behind a `payment-providers` module. Publish test card numbers in the README.
- **Background jobs:** Trigger.dev; jobs grouped by domain under `apps/api/src/trigger/<domain>/` with a README.
- **Email:** react-email templates in `packages/emails`, sent via Resend; keep an email-validation script.
- **Files:** S3 via presigned posts; SFTP client module for bank/partner file transfers.
- **Observability:** OpenTelemetry + structured logging (Winston); Scalar API reference from the OpenAPI spec.

### 5. AI instruction files

- `AGENTS.md` at root is the source of truth: extreme concision, minimal surgical changes, strict TypeScript (no `any`, no `!`, no `as` casts; model illegal states as unrepresentable via discriminated unions), zod-first schemas, Context7 for library research, ticket tool for planned work.
- Vendor community skills per agent platform (`.claude/skills`, `.agents/skills`, `.cursor/skills`, …) — but keep them as thin pointers; do not let five copies drift. One must be canonical.
- Frontend feature depending on API changes: API first → wait for `openapi.yaml` to regenerate → `pnpm orval` → then frontend.

### 6. CI/CD and environments

- `turbo.json` tasks: `build` (depends on `^build`, inputs include `.env*`), `dev` persistent, DB/migration tasks `cache: false` + `interactive: true` (except `migration:migrate`, non-interactive for CI), `test:ci` depends on `^build`.
- GitLab CI (or equivalent): stages test → setup → build → deploy. Per-client deploys driven by environment config, not by client names baked into tag regexes.
- `pnpm dev` = compose up (Postgres) + `turbo dev`; always bring compose down afterwards.
- k6: `smoke.ts`, `local-api-load.ts`, `int-api-load.ts`.

## Output Contract

When scaffolding, produce: repo skeleton with the layout above, working `pnpm dev` (API + DB), one example module end-to-end (schema → table → routes → SDK → portal page), AGENTS.md, `.env.example` files, and CI with typecheck/lint/test gates.

## Operating Rules

1. Never commit `.env`, `postgres/`, `dist/`, `.turbo/`, `coverage/`, `*.log`, or DB dump files. Backup scripts must exclude them.
2. Generated files (`openapi.yaml`, orval SDKs) are build artifacts: regenerate in CI, don't hand-edit.
3. No empty skeleton apps in the repo — scaffold it or don't commit it.
4. Keep README's app/package list in sync with the repo; no backup files (`*.bak.ts`) in source.
5. Typecheck after every series of changes; prefer running single tests over the full suite.
6. Review `references/improvements-checklist.md` before finalizing any new project — it lists every issue found in the source project and the fix.
## Available connectors

Use `muse.connector_search` to find the current connector docs before following
recovered instructions.
