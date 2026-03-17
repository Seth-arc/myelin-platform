# Myelin Professional Rebuild - Development Guide

## Quick Start Checklist

- [ ] Treat this as a rebuild bootstrap guide, not a runnable repo walkthrough.
- [ ] Read the source-of-truth documents in the order listed below.
- [ ] Separate confirmed facts from proposed defaults before executing any step.
- [ ] Confirm conflict items in the Conflicts And Open Decisions section.
- [ ] Create a new monorepo workspace around primary_build_source.
- [ ] Implement in small batches: Plan -> 1-2 files -> Test -> Inspect -> Commit.
- [ ] Do not release without tenant isolation, auditability, simulator gates, and AI safety controls.

## What This Source Package Is

primary_build_source is a consolidated source package of specifications, implementation contracts, design tokens, and prototype references. It is not a runnable production application.

Confirmed facts:
- It contains docs, references, and style tokens.
- It does not currently include a runnable monorepo surface such as root package manifests, lockfiles, runnable app scripts, or deployment manifests.

Guide intent:
- This document defines a safe bootstrap path for a new rebuild workspace derived from the source package.
- Every build command and generated file in Phases 1-5 is a proposed bootstrap default unless explicitly marked as confirmed from existing files.

## Source-Of-Truth Order

Use this precedence when documents conflict:

1. docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md
2. docs/v1_decisions.md
3. docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md
4. docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md
5. docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md
6. docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md
7. docs/MYELIN_AI_AGENT_IMPLEMENTATION_PLAN.md
8. docs/MYELIN_PROFESSIONAL_FEATURES.md
9. docs/MYELIN_UI_UX_STYLING_GUIDE.md and assets/styles/myelin-tokens.css
10. docs/skills/*/mnt/user-data/outputs/*/SKILL.md
11. references/* as supporting material only

## Confirmed Repo Facts

Repository structure and status:
- primary_build_source exists and is documentation-first.
- references/ui_baseline contains prototype HTML files for feature reference.
- assets/styles/myelin-tokens.css exists as a design-token source.

Locked architecture direction:
- Rebuild baseline is pnpm + turbo + TypeScript monorepo.
- App split is web (React + Vite), API (NestJS + Fastify), and worker.
- Data/runtime baseline includes Kysely + Postgres, BullMQ + Redis, Three.js + cannon-es.

Locked v1 constraints:
- v1 curriculum scope is FND-TT-01 and FND-IW-01.
- Build model is one routed app, not standalone module pages.
- LOTO simulation is deterministic 7-step ordered gating.
- Wrong-order actions must block progression and warn the learner.
- Required route correction after simulator completion is /app/submissions/new.
- Theme key must be myelin-theme.

Tenant and compliance facts:
- Tenant isolation is structural and release blocking.
- POPIA consent, role-scoped access, and audit coverage are mandatory.
- Demo data and live tenant data must remain isolated.

AI safety and behavior facts:
- QTB is mandatory for assessed simulation reasoning and AI instructional behavior.
- AI must not provide direct graded answers before submission.
- Safety/distress escalation SLA is 15 minutes.
- AI transcript retention class is distinct from broader platform legal retention classes.
- Transcript logging must follow write-before-render behavior.

Quality and release facts:
- Test pyramid, release gates, and operational readiness are mandatory.
- Observability is required before release for critical workflows.
- Runbooks are required before release readiness for operationally significant subsystems.

## Assumptions And Proposed Bootstrap Defaults

This section defines proposed defaults for creating the future rebuild workspace. These are not current repo facts.

### Proposed Workspace Layout (Assumption)

Assumption:
- Create a new workspace root with primary_build_source retained as read-only source material.

Reasonable because:
- Tech stack doc defines a monorepo-oriented target structure.

Supported by:
- docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md
- docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md

Needs confirmation:
- Final naming for package boundaries and whether sim-loto remains separate from sim-core in v1.

Proposed layout:

```text
myelin-rebuild/
  primary_build_source/
  apps/
    web/
      src/
      package.json
      tsconfig.json
      vite.config.ts
      .env
    api/
      src/
      package.json
      tsconfig.json
      .env
    worker/
      src/
      package.json
      tsconfig.json
      .env
  packages/
    contracts/
      src/
      package.json
      tsconfig.json
    db/
      src/
      package.json
      tsconfig.json
    ui/
      src/
      package.json
      tsconfig.json
    sim-core/
      src/
      package.json
      tsconfig.json
    sim-renderer/
      src/
      package.json
      tsconfig.json
    sim-physics/
      src/
      package.json
      tsconfig.json
    sim-loto/
      src/
      package.json
      tsconfig.json
    sim-telemetry/
      src/
      package.json
      tsconfig.json
    ai-orchestration/
      src/
      package.json
      tsconfig.json
  infra/
    docker/
      docker-compose.dev.yml
  scripts/
  .vscode/
    settings.json
    extensions.json
  .env.example
  .gitignore
  README.md
  package.json
  pnpm-lock.yaml
  pnpm-workspace.yaml
  turbo.json
  tsconfig.base.json
```

### Proposed Command Surface (Assumption)

Assumption:
- Define root scripts for dev/build/test/lint/typecheck plus migration/test variants.

Reasonable because:
- Docs require full quality gates and phased delivery.

Supported by:
- docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md

Needs confirmation:
- Exact script names and CI mapping.

Proposed scripts:
- pnpm dev
- pnpm build
- pnpm lint
- pnpm typecheck
- pnpm test
- pnpm test:integration
- pnpm test:sim
- pnpm test:e2e
- pnpm db:migrate
- pnpm db:seed

### Proposed Environment File Layout (Assumption)

Assumption:
- Add root .env.example plus app-level env files for web/api/worker.

Reasonable because:
- Multi-surface architecture and role isolation require clear per-surface config.

Supported by:
- docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md

Needs confirmation:
- Which integration env variables are required at bootstrap vs later phases.

Confirmed variable direction:
- Use NODE_ENV conventions.
- Keep evidence-storage direction and retention-aware behavior.

Confirmed variable references from source docs:

| Variable / Concept | Source-backed status | Source direction |
|---|---|---|
| NODE_ENV | Confirmed convention | Runtime environment separation across dev/stage/prod |
| EVIDENCE_BUCKET (or equivalent evidence storage bucket) | Confirmed concept | Evidence storage is required for submissions/evidence workflows |
| AI transcript retention class | Confirmed policy concept | AI chats/transcripts use a distinct retention class (2 years in AI blueprint) |
| Safety escalation SLA timer | Confirmed policy concept | Distress/safety escalation handling targets 15 minutes |

Proposed grouped defaults:
- API runtime: PORT, DATABASE_URL, REDIS_URL.
- Web runtime: VITE_API_BASE_URL.
- Security/auth placeholders: cookie and key references.
- AI/provider placeholders: vendor keys left blank.
- Observability placeholders: DSN or endpoint values left blank.

### Proposed Local Service Setup (Assumption)

Assumption:
- Local Postgres and Redis via Docker compose for first-run development.

Reasonable because:
- Required persistence and queue infrastructure are part of v1 baseline.

Supported by:
- docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md

Needs confirmation:
- Any mandatory local object storage emulator for evidence pipeline testing.

### Proposed Entrypoints (Assumption)

Assumption:
- apps/web/src/main.tsx, apps/api/src/main.ts, apps/worker/src/main.ts.

Reasonable because:
- Matches standard framework conventions and monorepo responsibility split.

Supported by:
- docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md

Needs confirmation:
- Final app module structure and package aliases.

### Proposed Environment Separation (Assumption)

Assumption:
- dev, stage, prod environments with hard data and secret boundaries.

Reasonable because:
- Compliance, pilot governance, and release gate model require progressive hardening.

Supported by:
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md
- docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md

Needs confirmation:
- Hosting provider, IaC, secrets manager, and region/data-localization deployment choices.

## Conflicts And Open Decisions

This section lists cross-doc conflicts and unresolved blockers.

### C1: Upload Policy Conflict (Resolved by Priority)

Conflict:
- PRD includes MP4/WebM in v1 language.
- Canonical/v1 decision set locks v1 to MP4 only.

Sources:
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md (FR-020)
- docs/MYELIN_CANONICAL_DECISIONS_TO_LOCK_BEFORE_BUILD.md
- docs/v1_decisions.md

Resolution:
- Follow MP4-only for v1 by higher-priority lock documents.

Impact:
- UI accept lists, backend validators, and transcode assumptions must align to MP4-only.

Proceed/block:
- Proceed with MP4-only. Treat WebM as out-of-scope until explicitly re-approved.

### C2: AI Mode Timeline Conflict (Partially Resolved)

Conflict:
- Canonical decisions place guided media as v1 and live conversational as feature-flagged/non-pilot-blocking.
- AI blueprint lists all required AI modes as done criteria in v1.

Sources:
- docs/v1_decisions.md
- docs/MYELIN_CANONICAL_DECISIONS_TO_LOCK_BEFORE_BUILD.md
- docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md

Resolution:
- Use v1_decisions for pilot launch gate: text + guided media required, live conversational gated by readiness and tenant approval.

Impact:
- Scope planning and release messaging must distinguish pilot-launch blocker vs in-v1 gated rollout.

Proceed/block:
- Proceed with staged rollout model.

### C3: Notification Controls vs Scope Precision (Open)

Conflict:
- PRD requires quiet hours, retry policy, and preference enforcement.
- Some implementation docs focus on channel availability and may under-specify quiet-hour enforcement details.

Sources:
- docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md
- docs/MYELIN_PROFESSIONAL_FEATURES.md

Resolution status:
- Not fully resolved in implementation detail.

Impact:
- Notification service may ship without institution-policy timing safeguards.

Proceed/block:
- Proceed with explicit v1 requirement: email + SMS plus quiet hours and preference controls.
- Track exact scheduler and timezone policy as a blocker for release gate evidence.

### C4: Retention-Class Granularity (Open)

Conflict:
- v1 decisions mention 10-year retention broadly.
- AI docs specify 2-year retention for chats/transcripts with separate legal retention classes for compliance records.

Sources:
- docs/v1_decisions.md
- docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md
- docs/skills/* transcript and compliance contracts

Resolution status:
- Needs formal retention matrix harmonization.

Impact:
- Risk of over-retention or policy violations.

Proceed/block:
- Proceed only with explicit retention class table in implementation design.
- Block release if retention classes are flattened into one rule.

### C5: Production Topology Open Items (Blocker for final deployment)

Open items:
- Final NVIDIA hosting/runtime topology.
- Browser certification cadence and target hardware profile for realtime QA.

Sources:
- docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md

Impact:
- Affects performance, cost, and live-conversational readiness sign-off.

Proceed/block:
- Proceed with local and staging bootstrap work.
- Block production-live rollout decisions until these are signed.

## Locked v1 Scope And Version Boundaries

| Scope Band | Included | Status Notes |
|---|---|---|
| v1 live / pilot-launch | Core routed app, FND-TT-01 and FND-IW-01, deterministic 7-step LOTO, text + guided media AI, tenant isolation, POPIA consent, auditability, email/SMS notifications | Release blocking |
| v1 feature-flagged non-pilot-blocking | Live conversational AI for approved tenants after safety/latency/transcript/ops sign-off | In v1 scope but gated |
| v1.1 candidate | Additional guided media depth, selected extensions per package and governance review | Requires decision record |
| v2 and later | Push notifications, broader curriculum expansion, advanced optional interactions and longer roadmap items | Out of pilot-launch scope |

Contradiction notes:
- AI mode phrasing differs across docs; pilot-launch boundary follows v1_decisions.
- Upload format phrasing differs across docs; v1 upload contract follows MP4-only lock.

## Architecture And Workspace Blueprint

### Proposed Monorepo Responsibilities

Apps:
- apps/web: routed UX shell, simulation host surface, AI chat surfaces, learner/lecturer/admin views.
- apps/api: REST APIs, auth/rbac, policy gates, uploads, telemetry ingest, reporting domain endpoints.
- apps/worker: async jobs for notifications, media processing, scoring tasks, retention jobs.

Packages:
- packages/contracts: shared schemas and API contracts.
- packages/db: schema/migrations/query modules and row-level security helpers.
- packages/ui: shared components aligned with token system.
- packages/sim-core: simulation state machine and scoring gates.
- packages/sim-renderer: Three.js scene/runtime glue.
- packages/sim-physics: cannon-es setup and deterministic interaction constraints.
- packages/sim-loto: LOTO-specific mechanics and step thresholds.
- packages/sim-telemetry: immutable event schema and replay/evidence helpers.
- packages/ai-orchestration: context binding, prompt assembly, response governance, policy checks.

### Major Boundaries

- React shell vs simulator core:
  - React owns layout, orchestration, and lifecycle hooks.
  - Simulation packages own state machine, thresholds, and deterministic behavior.
- API vs worker:
  - API handles synchronous transactional and policy decisions.
  - Worker handles retriable async jobs and SLA-driven operations.
- Contracts-first boundary:
  - All cross-app payloads must flow through packages/contracts.

### Data and Service Boundaries

- Tenant context must be explicit at API and DB boundaries.
- Demo namespace must stay separate from live tables/records and transcript stores.
- AI logging and escalation must create audit-grade records with traceable linkage.

## Prototype-To-Production Traceability

Prototype files are implementation references, not canonical runtime architecture.

Suggested traceability map:
- references/ui_baseline/index.html and module pages -> routed learner application surfaces in apps/web.
- references/ui_baseline/admin-baseline.html -> admin console feature modules.
- references/ui_baseline/lecturer-baseline.html -> lecturer dashboards, review flows.
- references/ui_baseline/ai-study-buddy-mockup.html -> AI chat panel and mode controls.
- references/ui_baseline/emails.html -> notification template baseline.

Known drift to avoid copying:
- Standalone module page model should not be reproduced in final routed architecture.
- Route drift after simulation completion must end at /app/submissions/new.
- Theme key drift must be resolved to myelin-theme.

## Pilot Governance, KPI Scorecard, And Entitlements

Pilot governance constraints that affect delivery:
- KPI families are mandatory for pilot sign-off.
- Baselines must be frozen at pilot kickoff and formula changes must be controlled.
- Reporting cadence is weekly in pilot operations.
- Pilot-to-annual decisions require KPI trend evidence and SLA stability.

Mandatory KPI families:
- Learner completion
- Core curriculum coverage
- Competency attainment
- Time-to-competency
- Instructor capacity
- Pilot conversion readiness
- Demo-to-pilot conversion

Commercial and entitlement implications:
- Entitlements are tenant-level feature-flag controlled.
- High-risk AI automation remains approval-gated regardless of package.
- Package upgrades require migration checklist and governance sign-off.

Implementation-order impact:
- Build telemetry/reporting contracts early, not after feature completion.
- Preserve evidence provenance for KPI and governance workflows.

## Phase 1 - Workspace Setup (Zero To Editor)

Windows 11 is the primary path. macOS/Linux equivalents are noted briefly where useful.

### Step 1: Install Git

Where to run:
- New PowerShell terminal.

Command type:
- Proposed bootstrap verification command.

Command:

```powershell
git --version
```

Success looks like:
- A Git version is printed.

Common failure and next step:
- Command not found: reinstall Git for Windows and reopen terminal.

### Step 2: Install Node.js LTS and enable Corepack/pnpm

Where to run:
- New PowerShell terminal.

Command type:
- Proposed bootstrap setup commands.

Commands:

```powershell
node -v
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v
```

Success looks like:
- node and pnpm versions print.

Common failure and next step:
- pnpm missing: rerun corepack commands and reopen terminal.

### Step 3: Install Docker Desktop (or equivalent container runtime)

Where to run:
- PowerShell after Docker starts.

Command type:
- Proposed bootstrap verification commands.

Commands:

```powershell
docker --version
docker info
```

Success looks like:
- Docker responds with engine details.

Common failure and next step:
- Docker daemon unavailable: start Docker Desktop and complete setup prompts.

### Step 4: Install Editor (VS Code or Cursor)

Where to run:
- GUI/editor setup.

Command type:
- Proposed bootstrap tooling setup.

Actions:
- Open workspace root.
- Trust workspace.
- Enable linting/formatting/test explorer extensions.

Success looks like:
- Editor opens workspace and recommendations load.

Common failure and next step:
- Extension drift: add workspace recommendations under .vscode/extensions.json in the future workspace.

### Step 5: Create workspace root and copy source package

Where to run:
- PowerShell.

Command type:
- Proposed bootstrap command.

Commands:

```powershell
mkdir C:\Projects\myelin-rebuild
Set-Location C:\Projects\myelin-rebuild
```

Success looks like:
- Current directory is the new workspace.

Common failure and next step:
- Permission errors: choose a writable path under your user profile.

### Short macOS/Linux Notes

- macOS/Linux shell equivalents:
  - Use `mkdir -p` for nested directory creation.
  - Use `cd` instead of Set-Location.
- Corepack, pnpm, and Docker verification commands are the same.
- Docker Desktop alternatives on Linux are acceptable if Docker CLI compatibility is maintained.

## Phase 2 - Bootstrap The Rebuild Workspace

All files/commands below are proposed bootstrap defaults unless marked confirmed.

### 2.1 Create folder structure

Where to run:
- Workspace root terminal.

Command type:
- Proposed bootstrap command.

Commands:

```powershell
mkdir apps, packages, infra
mkdir apps\web, apps\api, apps\worker
mkdir packages\contracts, packages\db, packages\ui, packages\sim-core, packages\sim-renderer, packages\sim-physics, packages\sim-loto, packages\sim-telemetry, packages\ai-orchestration
mkdir infra\docker
```

Success looks like:
- Folders exist and are visible in editor.

Common failure and next step:
- Existing directories: continue; this is idempotent.

### 2.2 Create root manifests and workspace config

Where to run:
- Workspace root terminal and editor.

Command type:
- Proposed bootstrap commands/files.

Commands:

```powershell
pnpm init
```

Proposed files to create:
- package.json
- pnpm-workspace.yaml
- turbo.json
- tsconfig.base.json
- .gitignore

Success looks like:
- Workspace recognizes apps/* and packages/*.

Common failure and next step:
- Mixed lockfiles: keep pnpm-lock.yaml only.

### 2.3 Install baseline tooling

Where to run:
- Workspace root terminal.

Command type:
- Proposed bootstrap commands.

Commands:

```powershell
pnpm add -Dw turbo typescript vite vitest @playwright/test eslint prettier @types/node tsx
pnpm exec playwright install
```

Success looks like:
- Dependencies install and Playwright browsers are provisioned.

Common failure and next step:
- Browser install issues: retry after network/firewall check.

### 2.4 Scaffold web/api/worker baseline

Where to run:
- Workspace root terminal plus editor.

Command type:
- Proposed bootstrap actions.

Notes:
- Create minimal web/app worker start paths first.
- Keep first pass behavior minimal: app shell, API health route, worker startup log.

Success looks like:
- Web runs locally.
- API health endpoint returns status.
- Worker process starts without immediate crash.

Common failure and next step:
- Port collisions: rebind local ports and update env.

### 2.5 Add local Postgres and Redis

Where to run:
- Workspace root terminal.

Command type:
- Proposed bootstrap command.

Commands:

```powershell
docker compose -f infra/docker/docker-compose.dev.yml up -d
docker ps
```

Success looks like:
- Postgres and Redis containers run persistently.

Common failure and next step:
- Port conflict on 5432/6379: remap host ports and update env values.

### 2.6 Add env files

Where to run:
- Editor/filesystem.

Command type:
- Proposed bootstrap files.

Proposed files:
- .env.example
- apps/web/.env
- apps/api/.env
- apps/worker/.env

Rules:
- Keep secret values blank/placeholders in tracked files.
- Use purpose-grouped variables when not directly locked by source docs.

### 2.7 First run and quality baseline

Where to run:
- Workspace root terminal.

Command type:
- Proposed bootstrap commands.

Commands:

```powershell
pnpm dev
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

Success looks like:
- Services run and quality checks pass.

Common failure and next step:
- Missing script wiring: align root and package script names first, then rerun.

## Phase 3 - Safe Vibe-Coding Loop

Workflow contract:
- Plan -> 1-2 files -> Test -> Inspect -> Commit.
- Avoid large-batch edits and broad refactors unless scheduled and gated.

### 3.1 Decision Framework

Choose next task by order:
1. Locked v1 scope and release blockers.
2. Dependency order (contracts, data boundaries, then UX).
3. Highest-risk controls (tenant isolation, safety, determinism).
4. Smallest safe reversible change.

### 3.2 File Location Guide

Where to work by concern:
- Web app UI and routing: apps/web and packages/ui.
- Auth and RBAC: apps/api auth modules + packages/contracts + packages/db.
- DB schema/queries and RLS: packages/db.
- AI orchestration and policy checks: packages/ai-orchestration + API AI modules.
- Transcript logging and escalation: API AI modules + transcript/escalation tables.
- Simulator runtime and state machine: packages/sim-renderer, sim-physics, sim-core, sim-loto.
- Telemetry and evidence contracts: packages/sim-telemetry + API ingest/reporting.
- Notifications: worker notification jobs + API preference/policy endpoints.
- Admin/reporting: API reporting services + web admin surfaces.

### 3.3 Guardrails Before Editing

Before any change, explicitly check whether it touches:
- Tenant-scoped records or cross-tenant access paths.
- Role gates and approval-gated actions.
- Consent/privacy/retention behavior.
- Audit/event trail requirements.
- Simulator thresholds, order gating, and deterministic rules.
- AI context binding, refusal logic, transcript write-before-render.
- Demo/live data isolation.

### 3.4 Review Loop After AI Suggestions

Reject or revise any suggestion that introduces:
- Invented commands or files presented as existing.
- Missing tenant context checks or RLS assumptions.
- Missing audit or consent handling.
- Missing simulator deterministic guardrails.
- Missing AI safety controls (context binding, refusal, transcript, escalation, prompt injection defense).
- Feature-version drift (v1 vs v1.1/v2 confusion).

### 3.5 Before Commit

Require:
- Smallest safe batch.
- Relevant tests passed.
- Explicit risk statement recorded in PR/commit notes.
- Behavioral commit message (what changed for users/system behavior).

## Phase 4 - Testing, Quality, And Release Gates

This matrix is Myelin-specific and includes release-risk controls.

| Test Area | Typical Command Surface (Proposed) | Validates | Run When | Passing Result |
|---|---|---|---|---|
| Lint | pnpm lint | Style consistency and anti-pattern checks | Every commit | No blocking lint errors |
| Typecheck | pnpm typecheck | Contract/type safety across boundaries | Every commit | No type errors |
| Unit | pnpm test | Local logic correctness | Each logic change | Deterministic pass |
| Integration | pnpm test:integration | API + DB + queue behavior | Before merge | Domain flows pass with real services |
| Simulator-specific | pnpm test:sim | Step order, thresholds, telemetry correctness, perf assertions | Before simulator release and staging | LOTO and telemetry gates pass |
| E2E | pnpm test:e2e | Core learner/lecturer/admin journeys | Before staging/pilot demo | Journeys complete successfully |
| Build | pnpm build | Production artifact integrity | Before release candidate | Clean build outputs |
| Dependency vulnerability scan | CI scanner/tooling | Supply-chain risk | Each PR and release cut | No unapproved critical findings |
| Secret scan | CI scanner/tooling | Credential leakage prevention | Each PR and release cut | No secrets detected |
| Accessibility checks | automated + manual scripted checks | WCAG baseline behavior, keyboard, reduced motion, captions | Before merge on core UI paths | Meets A minimum and AA target on core surfaces |
| Migration checks | migration dry-run + rollback test | Schema safety and upgrade path | Before merge to release branch | Forward + rollback validated |
| Tenant isolation validation | targeted integration tests | RLS and tenant boundary correctness | Before release candidate | No cross-tenant exposure paths |
| AI guardrail tests | policy test suite | refusal, safety redirect, no-cheat behavior | Before release candidate | Guardrails enforced consistently |
| AI transcript and escalation tests | transcript/escalation tests | write-before-render, policyFlag incident logging, SLA timers | Before release candidate | Writes precede render, incidents logged |
| Compliance evidence pack | release checklist artifacts | POPIA controls, audit coverage, retention class behavior | Release gate | Evidence complete and approved |

Common failure interpretation:
- Isolation failures are release blockers.
- Retention-class mismatch is a compliance blocker.
- Simulator determinism drift is a curriculum integrity blocker.
- Missing AI safety logs/escalations is an operational blocker.

## Phase 5 - Build, Environments, And Deployment

Honesty boundary:
- Provider-specific deployment manifests are not present in the source package.
- Deployment guidance here is assumption-based and must be finalized in implementation design.

### Environment Separation

- dev: local feature development with synthetic/test data.
- stage: production-like validation and release gate evidence generation.
- prod: live institutional workloads with strict secrets, observability, rollback, and governance controls.

Rules:
- Never point demo pathways at live tenant data.
- Never store production secrets in repo-tracked files.
- Never promote builds that have unresolved compliance or isolation findings.

### Build and Release Expectations

Expected build outputs (proposed):
- apps/web static output
- apps/api server output
- apps/worker runnable output

Release blockers:
- tenant isolation gaps
- missing consent/audit controls
- simulator gate failures
- AI safety/incident/escalation non-compliance
- incomplete observability and runbooks

Rollback expectations:
- Maintain previous-release redeploy path.
- Validate rollback scripts/procedures in stage before production release.

## Observability, Incident Handling, And Runbooks

Observability is delivery scope, not post-launch polish.

### Required Signals

API and worker:
- Structured logs with correlation IDs.
- Error counts, latency, throughput, queue depth, retry/failure rates.

Upload and evidence pipeline:
- Validation, malware, transcode, and processing stage metrics.
- Traceable status transitions for learner-visible states.

Simulator path:
- Startup timing, performance tier selection, FPS bucket metrics, deterministic failure reasons.

AI path:
- Mode used, policy flags, refusal events, escalation triggers, transcript write status.
- Safety-critical response markers and review linkage.

### Incident Handling Expectations

- Any AI response with policyFlag also creates incident-review log entry.
- Distress/safety escalations must meet 15-minute SLA.
- Cross-tenant exposure requires immediate containment workflow.
- Incident artifacts must be exportable for governance review.

### Minimum Runbooks Required Before Release Readiness

- Auth and session incident runbook.
- Upload failure and evidence recovery runbook.
- Queue backlog/failure runbook.
- AI provider degradation and fallback runbook.
- AI safety escalation triage runbook.
- Simulator failure and degraded-mode runbook.
- Cross-tenant exposure response runbook.
- Previous-release redeploy and rollback runbook.

### Operational Readiness vs Feature Completeness

A feature is not release-ready if:
- it cannot be diagnosed quickly in stage/prod,
- it has no runbook,
- or it lacks evidence for compliance and SLA controls.

## Appendix - Myelin-Specific Copilot Prompts

1. Explain architecture with source precedence

```text
Explain the Myelin rebuild architecture using this source order first: docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md, docs/v1_decisions.md, docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md, docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md. Map each responsibility to apps/* and packages/* folders. Flag anything that is an assumption.
```

2. Add tenant-safe API feature

```text
Implement [FEATURE] in apps/api with tenant isolation and audit coverage. Read docs/v1_decisions.md and docs/skills/2/mnt/user-data/outputs/multi-tenant-isolation/SKILL.md first. Touch at most 2 files per step and list tests after each step.
```

3. Review RBAC change

```text
Review this RBAC change against docs/skills/2/SKILL.md and docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md. Findings first. Confirm read-only default behavior for ministry/procurement/regulator roles unless explicit approved exceptions exist.
```

4. Review POPIA and audit-impacting change

```text
Audit this change for POPIA consent, retention classes, and audit logging requirements using docs/v1_decisions.md, docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md, and relevant docs/skills outputs. List violations and exact fixes.
```

5. Implement AI orchestration change

```text
Implement [AI CHANGE] with context binding, refusal behavior, escalation logic, and prompt-injection defense. Read docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md plus docs/skills/3/mnt/user-data/outputs/agent-context-binding/SKILL.md and docs/skills/4/SKILL.md first.
```

6. Implement transcript logging

```text
Implement transcript logging with write-before-render and policyFlag incident linkage. Use docs/skills/3/mnt/user-data/outputs/agent-transcript-logging/SKILL.md and docs/skills/4/SKILL.md. Do not render responses until transcript commit succeeds.
```

7. Implement simulator step rule

```text
Implement LOTO step [N] rule in simulator packages only (not UI shell business logic). Read docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md and docs/skills/2/mnt/user-data/outputs/simulation-state-machine/SKILL.md first. Include deterministic threshold tests.
```

8. Generate simulator-specific tests

```text
Generate simulator tests for ordered gating, wrong-order blocking, pass/fail thresholds, telemetry integrity, and deterministic replay using packages/sim-core, sim-loto, and sim-telemetry contracts.
```

9. Review release candidate against gates

```text
Review this release candidate against Myelin release gates: tenant isolation, POPIA, auditability, simulator correctness, AI safety, observability, and runbook readiness. Use docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md and docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md as primary sources. Return blocker list first.
```
