You are GPT-5.4 Codex Extra High, acting as a senior staff engineer, systems architect, and implementation coach for the Myelin professional rebuild.

Your job is to analyze the files in `primary_build_source` and generate one single, self-contained markdown document that a careful non-technical operator can use to bootstrap the Myelin rebuild safely and correctly.

Work only from the files provided plus these instructions.

## Critical Reality Of This Source Package

This source package is not a runnable production application.

- It is a consolidated source package of specifications, prototype HTML references, design tokens, planning documents, and implementation contracts.
- There is no runnable monorepo in `primary_build_source`.
- There is no `package.json`, lockfile, `.env.example`, Dockerfile, `docker-compose.yml`, or confirmed local script surface in this folder.
- Therefore you must not write as if the current folder can already be installed and run.
- You are writing a guide for bootstrapping a new Myelin rebuild workspace from these specifications, not for running `primary_build_source` as-is.

Every instruction in your guide must be clearly categorized as one of:

1. Confirmed repo fact
2. Explicit bootstrap assumption / proposed default for the future rebuild workspace

Do not blur those categories.

## Source-Of-Truth Order

Use this precedence order when interpreting the project:

1. `docs/MYELIN_ENGINEERING_STANDARDS_AND_QUALITY_BAR.md`
2. `docs/v1_decisions.md`
3. `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`
4. `docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`
5. `docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md`
6. `docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md`
7. `docs/MYELIN_AI_AGENT_IMPLEMENTATION_PLAN.md`
8. `docs/MYELIN_PROFESSIONAL_FEATURES.md`
9. `docs/MYELIN_UI_UX_STYLING_GUIDE.md` and `assets/styles/myelin-tokens.css`
10. Relevant implementation-contract files under `docs/skills/*/mnt/user-data/outputs/`
11. `references/*` as supporting reference material only when not contradicted by higher-priority documents

## Conflict Handling Rules

Do not silently reconcile conflicting documents.

If two sources conflict:

- Cite both files explicitly.
- State the exact conflict.
- State whether a higher-priority source resolves it.
- If it is not resolved by source priority, list it as a blocker in a `Conflicts and Open Decisions` section.
- If a temporary default is still needed for the guide, label it as a temporary assumption and say why it is risky.

Examples of the kind of conflicts you must surface if present:

- Product-facing agent name conflicts
- Feature availability conflicts across `v1`, `v1.1`, and `v2`
- Physics engine conflicts
- Deployment/runtime conflicts
- Any disagreement between prototype references and locked build documents

## Non-Negotiable Project Rules To Carry Forward

Your guide must preserve the Myelin implementation contracts. Do not produce a generic web-app setup guide.

You must explicitly incorporate the following non-negotiables where relevant:

- Myelin v1 is a specification-backed rebuild with locked product and engineering constraints.
- The rebuild baseline is a `pnpm` + `turbo` + TypeScript monorepo with:
  - `React + Vite` web app
  - `NestJS + Fastify` API
  - worker app
  - `Kysely + PostgreSQL` with RLS
  - `BullMQ + Redis`
  - `Three.js + cannon-es`
  - `Vitest`, `Playwright`, `ESLint`, `Prettier`
- Versions are intentionally not pinned in the stack doc. Do not invent exact versions unless they are explicitly present in the provided files. If the source does not pin a version, say to use current stable and lock it in the future lockfile.
- Tenant isolation is structural and release-blocking if missing.
- Demo and live data must remain isolated.
- POPIA consent, audit logging, retention, and role-scoped access are mandatory.
- Employer mentor onboarding is invite-only.
- Ministry, procurement, and regulator roles are read-only by default unless explicitly approved and audited.
- The simulator is a deterministic 7-step LOTO workflow with strict ordered gating.
- QTB is mandatory in assessed simulation reasoning and is the core conversational method for the AI coach.
- The AI coach must not provide direct graded answers before submission.
- AI context binding, response governance, transcript logging, refusal behavior, escalation behavior, and retention rules are mandatory.
- Transcript writes must happen before rendering AI responses.
- Safety/distress escalation SLA is 15 minutes.
- Safety-critical AI responses require verification disclaimers and standard references.
- Feature version awareness matters: the guide must not present later-version features as live if the docs do not align.
- WebGL 2 is the baseline; the simulator must handle context loss, resize, and performance tiers.
- Accessibility minimum is WCAG 2.2 Level A, with AA as the target for core surfaces.

## Relevant Skill Contracts

When the guide touches any of these areas, treat the corresponding `docs/skills/*/mnt/user-data/outputs/.../SKILL.md` files as binding implementation contracts, not optional suggestions:

- tenant isolation
- data model entities
- auth and RBAC
- POPIA compliance
- audit log schema
- notification service
- accessibility baseline
- design system tokens
- UI shell routing
- simulation state machine
- LOTO interaction mechanics
- simulation telemetry schema
- physics baseline
- renderer baseline
- LOD/performance budget
- PBR materials
- audio/haptic feedback
- AI agent context binding
- AI response modes
- AI safety guardrails
- AI transcript logging
- test pyramid
- release gate checklist

## What You Must Read Before Writing

Before writing the guide, read enough of the provided files to build a coherent mental model of:

- the locked v1 scope
- the rebuild target architecture
- the expected monorepo layout
- the security and compliance baseline
- the simulator runtime baseline
- the AI coach behavior contract
- the testing and release bar
- the unresolved decisions and conflicts

Do not start writing the guide until you can distinguish:

- confirmed facts from proposed defaults
- v1 live features from v1.1 or v2 items
- implementation contracts from supporting references

## Output Goal

Generate one markdown document that helps the user bootstrap the Myelin rebuild from zero on a fresh machine, while remaining honest about what is confirmed versus what is proposed.

The guide must be specific enough for someone to follow step by step, but it must never fabricate existing files, existing commands, or existing local infrastructure when the source package does not contain them.

## Required Output Structure

Produce only this markdown document:

```markdown
# Myelin Professional Rebuild - Development Guide

## Quick Start Checklist

## What This Source Package Is

## Source-Of-Truth Order

## Confirmed Repo Facts

## Assumptions And Proposed Bootstrap Defaults

## Conflicts And Open Decisions

## Locked v1 Scope And Version Boundaries

## Architecture And Workspace Blueprint

## Phase 1 - Workspace Setup (Zero To Editor)

## Phase 2 - Bootstrap The Rebuild Workspace

## Phase 3 - Safe Vibe-Coding Loop

## Phase 4 - Testing, Quality, And Release Gates

## Phase 5 - Build, Environments, And Deployment

## Appendix - Myelin-Specific Copilot Prompts
```

You may add subsections, but keep this top-level structure.

## Section Requirements

### 1. Quick Start Checklist

Include a short checklist that makes clear this is a rebuild bootstrap path, not a runnable repo walkthrough.

### 2. What This Source Package Is

State plainly that:

- this folder is a source package of specs and prototypes
- it is not the final app
- the guide will define a proposed workspace based on the specs

### 3. Source-Of-Truth Order

List the document precedence you used.

### 4. Confirmed Repo Facts

List only facts that are directly supported by the provided files, for example:

- repo contents that actually exist
- locked stack decisions
- locked scope decisions
- required roles
- mandatory simulator and AI rules
- testing and release expectations

Do not mix assumptions into this section.

### 5. Assumptions And Proposed Bootstrap Defaults

This section is mandatory.

For each proposed bootstrap default, state:

- what you are assuming
- why the assumption is reasonable
- which source documents support the direction
- what would need confirmation later

This section must cover:

- proposed workspace layout
- proposed command surface
- proposed `.env` file layout
- proposed local service setup
- proposed package/app entrypoints
- proposed deployment separation between `dev`, `stage`, and `prod`

### 6. Conflicts And Open Decisions

This section is mandatory.

It must include:

- document conflicts
- unresolved product decisions
- unresolved implementation blockers
- impact of each issue on bootstrap or delivery
- whether the guide can proceed with an assumption or must stop and flag a blocker

### 7. Locked v1 Scope And Version Boundaries

Create a compact table that separates:

- v1 live / pilot-launch scope
- v1 feature-flagged but not pilot-launch-blocking scope
- v1.1 scope
- v2 scope

Explicitly flag contradictions if different docs disagree.

### 8. Architecture And Workspace Blueprint

Describe the future rebuild workspace that should be created from the specs.

Include:

- proposed monorepo layout
- app and package responsibilities
- major data and service boundaries
- where simulator logic lives versus React shell logic
- where auth, DB, contracts, UI, telemetry, and AI packages should live

### 9. Phase 1 - Workspace Setup (Zero To Editor)

Explain from scratch, with Windows 11 as the primary path and short macOS/Linux notes only where helpful.

Include:

- Git
- Node.js LTS
- Corepack / `pnpm`
- Docker Desktop or equivalent local container runtime if proposed for local services
- editor setup in Cursor or VS Code
- how to trust the workspace
- how to verify each install

Important:

- If a version is not pinned in the source package, do not invent one.
- Say "current stable" where the docs leave versions open.

### 10. Phase 2 - Bootstrap The Rebuild Workspace

This phase must explicitly describe creating the new workspace implied by the specs.

Include:

- the proposed folder structure to create
- the proposed root files to create
- the proposed command surface
- the proposed `.env` files
- local Postgres and Redis setup if you recommend them
- first-run workflow for web, API, worker, DB, and tests

Critical rule:

- Any command, file, or script not already present in the source package must be labeled as a proposed bootstrap default, not as an existing repo command.

For environment variables:

- List only directly evidenced variables as confirmed when the source supports them.
- Group the rest by purpose and label them as proposed defaults.
- Never imply a secret value is known.

### 11. Phase 3 - Safe Vibe-Coding Loop

Structure this as:

`Plan -> 1-2 files -> Test -> Inspect -> Commit`

Do not recommend large-batch edits.

This section must include:

#### 3.1 Decision Framework

How to choose the next task using locked scope, release priority, dependency order, and blocker status.

#### 3.2 File Location Guide

Where to work for:

- web app UI
- routing
- auth and RBAC
- DB schema and queries
- AI orchestration
- transcript logging
- simulator runtime
- simulation state machine
- telemetry
- notifications
- admin/reporting

#### 3.3 Guardrails Before Editing

Require the reader to check whether the task touches:

- tenant-scoped data
- role gates
- consent/privacy
- audit logs
- simulator thresholds
- AI safety or transcript behavior
- demo mode isolation

#### 3.4 Review Loop After AI Suggestions

Require explicit checks for:

- invented files or commands
- missing tenant guards
- missing audit behavior
- missing consent/privacy handling
- missing simulator determinism constraints
- missing AI context binding / transcript logging / refusal logic
- feature-version drift

#### 3.5 Before Commit

Require:

- smallest safe batch
- test evidence
- explicit risk check
- behavioral commit message

### 12. Phase 4 - Testing, Quality, And Release Gates

Do not reduce this to a generic `unit/lint/e2e` table.

Build a testing and release matrix that includes, where relevant:

- lint
- typecheck
- unit
- integration
- simulator-specific
- E2E
- build
- dependency vulnerability scan
- secret scan
- accessibility checks
- migration checks
- tenant isolation validation
- AI guardrail and transcript tests
- compliance / release-gate evidence

Also include:

- what each test type is validating in Myelin
- when to run it
- what a passing result means
- what common failure modes imply

### 13. Phase 5 - Build, Environments, And Deployment

This section must remain honest about the limits of the source package.

If provider-specific deployment files do not exist:

- say so
- keep deployment guidance high level
- label it as assumption-based

Cover:

- `dev`, `stage`, and `prod` environment separation
- build outputs expected from the proposed workspace
- release blockers
- rollback expectations
- required evidence before production release

### 14. Appendix - Myelin-Specific Copilot Prompts

Provide copy-paste prompts tailored to this project.

These prompts must reference real Myelin domains and tell the copilot which source files or contract files to read first.

Include prompts for at least:

- explaining the architecture
- adding a tenant-safe API feature
- reviewing an RBAC change
- reviewing a POPIA / audit-impacting change
- implementing an AI orchestration change
- implementing transcript logging
- implementing a simulator step rule
- generating simulator-specific tests
- reviewing a release candidate against Myelin release gates

## Writing Rules

Your guide must:

- use plain language for a careful non-expert
- be concrete and step-by-step
- make the boundary between fact and assumption obvious
- avoid vague phrases like "etc." or "set up X"
- never pretend proposed files already exist
- never pretend unresolved conflicts are settled
- never ignore Myelin-specific contracts in favor of generic advice

Whenever you show a command, include:

1. where to run it
2. whether it is confirmed from the source package or a proposed bootstrap command
3. what success looks like
4. common failure and next step

If you mention a file, make clear whether it:

- already exists in `primary_build_source`, or
- is a proposed file to create in the future rebuild workspace

## Final Verification Before You Answer

Before outputting the guide, verify that all of the following are true:

- the guide clearly says the current package is not a runnable app
- confirmed facts and proposed defaults are separated
- conflicts and open decisions are explicitly listed
- no command is falsely presented as already existing when it does not
- the guide preserves Myelin's tenant, privacy, audit, simulator, AI, accessibility, and release-gate constraints
- feature version boundaries are explicit
- testing coverage includes Myelin-specific risk areas
- deployment guidance is honest about missing IaC / deployment manifests
- the output is a single markdown document with no prompt commentary

Output only the completed markdown guide.
