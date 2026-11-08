# API Module Layout (RecPlus conventions)

`apps/api/src/` top level:

```
core/
  app/        # app composition, route mounting
  domain/     # domain models / business rules
  infra/      # core infrastructure adapters
infra/
  http/       # HTTP-level concerns (middleware, error mapping)
shared/
  config.ts, di-container.ts (@needle-di/core), exceptions.ts,
  logger.ts, mappers.ts, strings.ts, types.ts
  dtos/  value-objects/
modules/
  <domain>/                 # billing, bookings, members, payments, programs, ...
    <sub-resource>/         # e.g. payments/invoices
      <name>.repository.ts  # data access only (Drizzle)
      <name>.service.ts     # business logic, injected via @needle-di
    <domain>.routes.ts      # Hono routes, zod-validated, tagged for OpenAPI
scripts/                    # ops: db-init, per-client migrations, backfills
trigger/                    # Trigger.dev jobs, grouped by domain + README
tracing.ts  index.ts
```

## Conventions

- **Repository + Service split:** every sub-resource gets `<name>.repository.ts` (Drizzle queries only) and `<name>.service.ts` (business logic). Routes stay thin: validate (zod) → call service → return DTO.
- **Routes file per domain:** `<domain>.routes.ts` mounts all sub-resource routes and carries the OpenAPI tags (`admin`/`staff portal`, `customer portal`, `internal`) that drive orval SDK filtering.
- **DI:** services/repositories registered in `shared/di-container.ts` with `@needle-di/core`; constructor injection, no globals.
- **Value objects:** shared value-objects live in `shared/value-objects/`; DTOs in `shared/dtos/`.
- **Trigger jobs:** one directory per domain under `src/trigger/` (`payments/`, `memberships/`, `refunds/`, …), each with its own README describing the job's purpose and schedule.
- **Ops scripts:** one-off scripts under `src/scripts/` invoked via root package.json (`backfill:*`, `cleanup:*`, `data-migration:<client>`). Each script must be idempotent and resumable; record progress so reruns skip done work.
- **OpenAPI tags discipline:** every route gets exactly one audience tag (`staff portal` | `customer portal` | `internal`). `internal` routes are excluded from all generated SDKs.
