# Myelin Rebuild Guide Plan

## Summary

- Write one markdown guide for bootstrapping the spec-defined Myelin rebuild from zero, not for treating the current folder as a runnable app.
- Ground the guide in the actual source package: `README.md`, the PRD, the recommended tech stack, the simulation baseline, the AI blueprint/implementation plan, the features doc, the UI guide, the test pyramid, and the release gate checklist.
- Separate every instruction into two buckets: confirmed repo facts and explicit bootstrap assumptions, because `primary_build_source` contains specs and prototypes but no `package.json`, lockfile, `.env.example`, Dockerfile, or runnable monorepo.

## Key Implementation Changes

- Frame the guide as a Windows 11-first setup with short macOS/Linux notes, since the user environment is Windows and the docs target browser-first delivery.
- Use the locked rebuild baseline from the docs:
  - `pnpm` + `turbo` + TypeScript monorepo
  - `apps/web` with React + Vite
  - `apps/api` with NestJS + Fastify
  - `apps/worker`
  - Postgres + Redis
  - Three.js + `cannon-es`
  - Vitest + Playwright + ESLint + Prettier
- Make the guide's bootstrap path assume a new workspace matching the recommended layout in `docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md`, including the main folders, likely entrypoints, and a small-batch workflow for vibe-coding.
- Define a clearly labeled assumed command contract for the future workspace:
  - `pnpm install`
  - `pnpm dev`
  - `pnpm build`
  - `pnpm lint`
  - `pnpm test`
  - `pnpm test:e2e`
  - app/package-level variants only where needed for clarity
- Define a clearly labeled assumed config contract for local development:
  - root `.env.example`
  - `apps/web/.env`
  - `apps/api/.env`
  - `apps/worker/.env`
  - local Postgres/Redis via Docker for first run
- Keep named env vars limited to what is directly evidenced in the source where possible, such as `NODE_ENV` and `EVIDENCE_BUCKET`; group the rest by purpose and label them as assumptions rather than presenting them as existing repo config.
- Structure Phase 3 around `Plan -> 1-2 files -> Test -> Commit`, with file-location guidance tied to the proposed rebuild folders and Myelin-specific constraints like tenant isolation, demo isolation, the 7-step LOTO flow, and AI safety boundaries.
- Build Phase 4 from the existing testing and release docs:
  - unit
  - integration
  - E2E
  - simulator-specific
  - lint
  - typecheck
  - build
  - pre-release security/performance/compliance gates
- Keep Phase 5 deployment guidance high level and assumption-labeled because no provider-specific IaC, Dockerfiles, or deploy manifests exist; use the docs' `dev` / `stage` / `prod` separation and rollback expectations.

## Public Interfaces And Conventions To Define In The Guide

- Proposed workspace layout and entrypoints for `apps/web`, `apps/api`, `apps/worker`, and shared packages.
- Proposed root script surface for install, dev, test, lint, build, and E2E.
- Proposed local service contract for web, API, Postgres, Redis, and optional storage/AI integrations.
- Proposed checklist format for each phase:
  - exact command
  - where to run it
  - success criteria
  - common failure and fix
  - git commit message after a working step

## Test Plan

- Verify every guide phase has ordered steps and at least one checklist.
- Verify every command is either backed by the source docs or explicitly marked as an assumption.
- Verify the testing matrix includes at least unit, integration, E2E, simulator-specific, lint/typecheck, and build/release validation.
- Verify the appendix includes eight copy-paste prompts tied to Myelin's real stack, folders, and constraints.
- Verify the final guide stays self-contained, with no placeholders, no fake existing files, and no unstated assumptions.

## Assumptions

- The guide will target the professional rebuild path, not the static prototype preview path.
- Windows 11 is the primary environment; macOS/Linux differences will be secondary notes.
- Because the repo has no runnable manifests yet, the guide will define sensible bootstrap defaults for scripts, env layout, and local services, and will label those defaults explicitly as assumptions.
- Local first run will assume Docker-backed Postgres and Redis, with production infrastructure kept generic unless a deployment target is later provided.
