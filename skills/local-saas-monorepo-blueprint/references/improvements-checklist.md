# Improvements Checklist — issues found in RecPlus, fixes for future projects

All findings are read-only observations from the `parks-and-rec` Drive snapshot (2026-10-12). The source project was NOT modified.

## Bugs

1. **`TUROBO_UI=0` typo in root `package.json`** — the `data-migration:workos-accounts` and `validate-emails` scripts set `TUROBO_UI=0` instead of `TURBO_UI=0`. The misspelled variable is a no-op, so turbo falls back to default UI behavior. Fix: correct the spelling; add a CI check that greps scripts for common env-var typos (or a smoke test invoking each script with `--help`).

## Security / hygiene

2. **Real `.env` files in the repo** — root plus `apps/api`, `apps/customer-portal`, `apps/staff-portal` each carry a `.env` with real-looking values. `.gitignore` covers `.env*`, so git is clean, but any folder copy/backup leaks secrets. Fix: never write real values into the repo; use a secrets manager (Doppler, Infisical, AWS Secrets Manager) or per-developer `.env.local` outside versioned paths; add `gitleaks` to CI.
3. **Live Postgres data directory shipped with the project** — `postgres/` contains a full cluster copy including `postmaster.pid` (copied from a running database). Fix: backup/archive scripts must exclude data directories; database backups should be `pg_dump` artifacts, not raw data-dir copies.
4. **Generated and ephemeral files in the snapshot** — `dist/`, `.turbo/`, `coverage/`, `.trigger/`, `apps/api/.cursor/`, `combined.log`, `error.log`, `openapi.yaml` (regenerable), a 0-byte `full-dump-*.sql`. Fix: archive scripts should exclude `dist .turbo coverage .next *.log *-dump-*.sql`; regenerate `openapi.yaml` + SDKs in CI instead of storing them.

## Codebase health

5. **Empty skeleton apps** — `apps/admin-portal/` and `apps/user-portal/` contain nothing but `.turbo/`. Fix: scaffold it or don't commit it; dead folders confuse onboarding and AI agents alike.
6. **README drift** — lists only 3 apps and 10 packages; the repo has 5 apps and 11 packages. Also the `# Parks anc Rec` title typo. Fix: keep the app/package inventory generated or checked in CI (a one-line script comparing README list vs. filesystem).
7. **`schema.bak.ts` committed** in `packages/db/src/` next to `schema.ts`. Fix: delete; git history is the backup.
8. **Hollow `CLAUDE.md`** — 0 bytes, while `AGENTS.md` holds the real instructions and five platform skill dirs (`.claude`, `.agents`, `.cursor`, `.opencode`, `.codex`) each vendor the same 15 community skills. Fix: one canonical instruction file; the rest symlink or import it; don't let five copies drift.
9. **Deploy compose is dev-in-disguise** — `docker-compose-deploy.yaml` bind-mounts the whole repo into the container and builds the `builder` target, with PM2 inside. Fix: multi-stage Dockerfile producing a lean runtime image; no bind mounts at deploy; healthchecks on every service.
10. **Per-client deploys keyed on git tag names** — `.gitlab-ci.yml` matches tags like `myrtlebeach-*`, `manchester-tn-*` to pick runners/regions. Works for 5 clients, doesn't scale. Fix: environment-based deploys (one pipeline, per-client env config/tenant id).
11. **`dev` script cleanup is fragile** — `turbo dev; ./scripts/universal-compose.sh down` only runs `down` if turbo exits cleanly; a SIGKILL leaves Postgres running. Fix: trap EXIT, or document `pnpm dev:down`.
12. **Inconsistent SDK generation** — orval sets `useMutation: false`, so mutations are hand-rolled while queries are generated. Fix: pick one (prefer generated) and document why if there's an exception.
13. **`migration:push` + `migration-hardsync` in dev** is pragmatic but can silently diverge schema state across developers. Fix: CI dry-run that diffs `push` state vs. generated migrations on every PR touching `packages/db`.

## Keep (strongest patterns, worth copying verbatim)

- API-first: zod-openapi → `openapi.yaml` → orval **tag-filtered per-portal SDKs** (staff vs. customer vs. internal). This is the best pattern in the repo.
- zod schemas in `packages/schemas` as the source of truth, Drizzle tables built from them.
- Multi-tenant via `central-db` registry + per-client databases, with per-client migration scripts.
- Turbo task graph with `interactive`/`cache:false` correctly set for DB tasks; `.env*` in build inputs.
- AGENTS.md conventions: no `any` / no `!` / no `as`, ADTs for illegal states, minimal surgical changes, typecheck-after-change loop.
- k6 smoke + load tests checked in next to the code.
