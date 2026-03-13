# Myelin v1 Recommended Tech Stack

Document status: Recommended implementation baseline  
Date: March 13, 2026  
Audience: Sethu and implementation partners  
Scope: v1 browser platform, simulator runtime, and supporting backend services

---

## 1. Purpose

This document turns the current simulator and platform requirements into one concrete v1 stack recommendation.

The repository currently contains specifications and prototype assets, not a runnable production application. There is no `package.json`, `pyproject.toml`, `requirements.txt`, or `Dockerfile` in `primary_build_source`, so the stack below is a proposed build baseline derived from:

- `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`
- `docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`
- `references/simulation/simulation_development_guide.md`
- `references/product/PRODUCT_SPEC.md`
- project skills that lock renderer, physics, telemetry, and tenant isolation behavior

---

## 2. Recommended v1 Stack

### 2.1 Monorepo and Tooling

- Package manager: `pnpm`
- Monorepo orchestration: `turbo`
- Language: `TypeScript`
- Build tool for web: `Vite`
- Testing baseline: `Vitest`, `Playwright`
- Linting and formatting: `ESLint`, `Prettier`

Why:

- `pnpm` keeps workspace installs fast and deterministic.
- `turbo` fits a multi-app platform with shared simulator, telemetry, auth, and DB packages.
- `TypeScript` is the safest choice for shared contracts across web, API, workers, and telemetry.

### 2.2 Frontend Application

- App shell: `React`
- Routing: `react-router-dom`
- Server state: `@tanstack/react-query`
- UI styling: shared design tokens CSS plus component-scoped styles
- Form/schema validation: `zod`

Why:

- The product requires one routed application rather than separate static module pages.
- The simulator needs to live inside a broader learner/staff/admin shell, which React handles well.
- The specs do not justify SSR complexity for the simulator core, so `Vite` is the simpler fit than `Next.js`.

### 2.3 Simulator Runtime

- 3D engine: `three`
- Controls/loaders/post-processing: Three.js examples modules
- Physics engine: `cannon-es`
- Motion/camera polish: `gsap`
- IndexedDB backup: `idb`
- Optional PWA/cache support: `vite-plugin-pwa`
- Optional hand tracking: `@mediapipe/tasks-vision`

Why:

- Three.js is the clear implementation center of gravity across the product spec, prototype, and renderer baseline.
- The current physics skill locks `cannon-es` for reproducible scoring behavior.
- The simulator must support WebGL 2 as the production baseline, with WebGPU only as a progressive enhancement.
- IndexedDB and Cache API support match the simulation guidance for offline resilience and local backup.

### 2.4 Backend Platform

- API framework: `NestJS`
- HTTP adapter: `Fastify`
- Database access: `Kysely` + `pg`
- Auth/SSO integration: `WorkOS`
- Queue/workers: `BullMQ` + `ioredis`
- Object storage access: `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner`
- Schema validation: `zod`
- Logging: `pino`

Why:

- The PRD calls for a web frontend, auth service, domain services, queue workers, storage, and analytics paths.
- Postgres RLS needs explicit SQL control. `Kysely` is a better fit than a high-abstraction ORM for `SET LOCAL app.current_tenant_id`, append-only event tables, and tenant-scoped query discipline.
- `NestJS` keeps the API organized without forcing microservices too early.
- `Fastify` gives a better performance profile than Express for a session-heavy multi-tenant platform.

### 2.5 Infrastructure

- Database: `PostgreSQL`
- Cache/queue broker: `Redis`
- Object storage: S3-compatible storage
- CDN/static asset delivery: CloudFront or equivalent
- Background media tooling: `FFmpeg`
- Malware scanning for uploads: `ClamAV` or managed equivalent
- Observability: `OpenTelemetry`, `Sentry`

Why:

- Postgres with RLS is a hard requirement in the product decisions and isolation rules.
- Redis-backed queues are the simplest fit for notifications, exports, upload processing, telemetry fan-out, and evidence jobs.
- Media submission and evidence workflows require background processing and safe storage.

---

## 3. Stack Decisions to Lock

### 3.1 Use Three.js, not React Three Fiber

Recommendation:

- Use raw `three` in a dedicated simulator package.
- Mount the canvas inside the React shell, but keep the assessed simulation runtime outside React render cycles.

Reason:

- The simulator needs strict control over renderer initialization, resize handling, context loss recovery, post-processing, timing, and physics synchronization.

### 3.2 Use Cannon-ES for v1 Physics

Recommendation:

- Standardize v1 on `cannon-es`.

Reason:

- The broad simulation guidance mentions Rapier in several places, but the local physics baseline skill explicitly locks `cannon-es` and states that the torque curve and pressure model are calibrated to it.
- Mixing physics engines across docs and implementation will create avoidable scoring drift and debugging overhead.

Action:

- Update the older simulation references so the v1 build documents no longer conflict on physics engine choice.

### 3.3 Use Kysely, not Prisma

Recommendation:

- Use `Kysely` as the primary data access layer.

Reason:

- This platform requires:
  - database-enforced RLS
  - explicit tenant context on every query
  - append-only event tables
  - audit-heavy exports
  - careful SQL control for reporting and evidence reconstruction

### 3.4 Avoid Next.js for v1

Recommendation:

- Use `React + Vite` for the main app.

Reason:

- The simulator is a client-heavy WebGL application.
- SSR and app-router complexity add little value for the assessed runtime and increase operational surface area.

---

## 4. Proposed Monorepo Layout

```text
myelin/
  apps/
    web/
      src/
        app/
        routes/
        features/
        simulator-host/
        styles/
      public/
      index.html
      vite.config.ts
    api/
      src/
        modules/
          auth/
          users/
          learning/
          simulations/
          telemetry/
          portfolio/
          logbook/
          results/
          notifications/
          admin/
          ai/
        main.ts
      test/
    worker/
      src/
        jobs/
          notifications/
          media/
          telemetry/
          exports/
        main.ts
  packages/
    sim-core/
      src/
        state-machine/
        input/
        scoring/
        session/
    sim-renderer/
      src/
        renderer/
        materials/
        postprocessing/
        loaders/
        environment/
        lod/
    sim-physics/
      src/
        world/
        bodies/
        constraints/
        sync/
    sim-loto/
      src/
        scenario/
        objects/
        interactions/
        thresholds/
    sim-telemetry/
      src/
        events/
        emitters/
        replay/
        evidence/
    ui/
      src/
        components/
        tokens/
        hooks/
    contracts/
      src/
        api/
        telemetry/
        auth/
        domain/
    db/
      src/
        schema/
        migrations/
        queries/
        rls/
    auth/
      src/
        rbac/
        sessions/
        sso/
    storage/
      src/
        uploads/
        signed-urls/
        object-keys/
    queue/
      src/
        producers/
        consumers/
        job-types/
    config-eslint/
    config-ts/
  infra/
    docker/
    terraform/
    sql/
  docs/
  pnpm-workspace.yaml
  turbo.json
  package.json
  tsconfig.base.json
```

### 4.1 Workspace Responsibilities

`apps/web`

- Learner, lecturer, admin, and demo routes
- React shell
- simulator host page
- profile, notifications, pathway, e-portfolio, peer review, logbook, results

`apps/api`

- Auth/session APIs
- Tenant-aware domain APIs
- Simulation session creation and telemetry ingestion
- Admin/export/reporting endpoints
- AI orchestration entry points

`apps/worker`

- Notification dispatch
- Upload scan/transcode pipelines
- Export generation
- telemetry batching and secondary processing

`packages/sim-core`

- Canonical state machine
- input mode abstraction
- scoring and session rules

`packages/sim-renderer`

- Three.js renderer baseline
- HDRI loading
- post-processing
- LOD and performance helpers

`packages/sim-physics`

- Cannon-ES world setup
- constraints and body sync
- shared materials and world stepping

`packages/sim-loto`

- LOTO v1 scenario logic
- step thresholds
- interaction objects
- scenario-specific assets and metadata

`packages/sim-telemetry`

- immutable event payloads
- event emitters
- replay helpers
- evidence export reconstruction support

`packages/contracts`

- shared `zod` schemas and TypeScript types for API, telemetry, RBAC, and session payloads

`packages/db`

- Kysely schema and migrations
- RLS helpers
- explicit query modules

---

## 5. Proposed Dependency List

Versions are intentionally not pinned in this document. Use current stable releases when bootstrapping the workspace, then lock them in `pnpm-lock.yaml`.

### 5.1 Root Workspace Dependencies

Runtime:

- `typescript`
- `zod`

Dev:

- `turbo`
- `vite`
- `vitest`
- `playwright`
- `eslint`
- `prettier`
- `@types/node`
- `tsx`

### 5.2 `apps/web`

Runtime:

- `react`
- `react-dom`
- `react-router-dom`
- `@tanstack/react-query`
- `three`
- `cannon-es`
- `gsap`
- `idb`

Optional runtime:

- `@mediapipe/tasks-vision`
- `vite-plugin-pwa`
- `@sentry/react`

Dev:

- `@types/react`
- `@types/react-dom`
- `@types/three`

### 5.3 `apps/api`

Runtime:

- `@nestjs/common`
- `@nestjs/core`
- `@nestjs/platform-fastify`
- `@nestjs/config`
- `@fastify/cookie`
- `@fastify/multipart`
- `kysely`
- `pg`
- `pino`
- `workos`
- `bullmq`
- `ioredis`
- `@aws-sdk/client-s3`
- `@aws-sdk/s3-request-presigner`
- `@opentelemetry/api`
- `@sentry/node`

Optional runtime:

- `graphql`
- `@nestjs/graphql`
- `@apollo/server`

Why optional:

- GraphQL is in scope in the product decisions, but it does not need to block the initial REST-first build. It can be added once the domain model stabilizes.

### 5.4 `apps/worker`

Runtime:

- `bullmq`
- `ioredis`
- `pino`
- `pg`
- `kysely`
- `@aws-sdk/client-s3`
- `fluent-ffmpeg` or direct `ffmpeg` process wrapper
- `@opentelemetry/api`
- `@sentry/node`

System dependencies:

- `ffmpeg`
- malware scanning service or `clamd`

### 5.5 `packages/sim-renderer`

Runtime:

- `three`
- `postprocessing` or Three.js native post-processing utilities
- `meshoptimizer`

Notes:

- Keep renderer-specific dependencies here so the web app imports a stable simulator surface rather than low-level rendering glue.

### 5.6 `packages/sim-physics`

Runtime:

- `cannon-es`

### 5.7 `packages/sim-telemetry`

Runtime:

- `zod`
- `uuid`

### 5.8 `packages/contracts`

Runtime:

- `zod`

### 5.9 `packages/db`

Runtime:

- `kysely`
- `pg`
- `uuid`

Dev:

- migration tooling of choice compatible with Kysely

### 5.10 `packages/auth`

Runtime:

- `workos`
- `zod`

### 5.11 `packages/storage`

Runtime:

- `@aws-sdk/client-s3`
- `@aws-sdk/s3-request-presigner`
- `mime`

---

## 6. Non-NPM Dependencies to Plan For

These are required by the architecture even though they do not live in `package.json`.

- PostgreSQL
- Redis
- S3-compatible object storage
- FFmpeg
- malware scanning service
- CDN for static assets and heavy simulator files
- TLS-terminated HTTPS delivery
- CI/CD runner with Playwright browser support

---

## 7. Initial Build Sequence

1. Bootstrap `pnpm` monorepo with `apps/web`, `apps/api`, and `apps/worker`.
2. Create `packages/contracts`, `packages/db`, and `packages/ui`.
3. Stand up Postgres with RLS helpers and Redis-backed queue.
4. Implement auth/session and tenant propagation before business logic.
5. Build simulator packages in this order:
   - `sim-core`
   - `sim-renderer`
   - `sim-physics`
   - `sim-telemetry`
   - `sim-loto`
6. Mount the simulator into the React shell.
7. Add upload/storage/worker pipelines.
8. Add admin/reporting/export surfaces.

---

## 8. Final Recommendation

The v1 stack should be:

- `pnpm + turbo + TypeScript`
- `React + Vite` for the routed web app
- `Three.js + cannon-es` for the assessed simulator
- `NestJS + Fastify` for the backend API
- `Kysely + PostgreSQL RLS` for the data layer
- `BullMQ + Redis` for asynchronous jobs
- `S3-compatible object storage` for media and evidence

This is the most defensible implementation path for the current Myelin specs. It aligns with the browser-first simulator requirements, the strict multi-tenant model, the append-only telemetry requirements, and the need to add future simulation scenarios without re-platforming.
