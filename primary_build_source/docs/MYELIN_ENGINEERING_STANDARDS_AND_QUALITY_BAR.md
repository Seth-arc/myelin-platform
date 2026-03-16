# Myelin Engineering Standards and Quality Bar

Document status: Execution baseline  
Date: March 15, 2026  
Owner: Sethu (Founder, Myelin)  
Audience: Current and future Myelin engineers, implementation partners, reviewers, and release approvers  
Scope: All production code, tests, infrastructure code, automation, schemas, prompts, simulator packages, AI orchestration, and operational tooling for the Myelin rebuild

Primary reference documents:
- `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`
- `docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md`
- `docs/v1_decisions.md`
- `docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`
- project skills for tenant isolation, telemetry, AI guardrails, transcript logging, testing, accessibility, routing, physics, renderer, and release gates

---

## 1. Purpose

This document defines the minimum engineering bar for Myelin.

It is not a style preference guide. It is the implementation contract for building a production-grade platform that is:

- safe for multi-tenant institutional use
- extensible by future teams
- auditable for regulator and buyer scrutiny
- deterministic in assessed simulation behavior
- supportable in staging and production

Where the PRD defines what Myelin must do, this document defines how Myelin code must be written, reviewed, tested, and released.

---

## 2. What Enterprise-Grade Means For Myelin

For this project, enterprise-grade engineering means:

- correctness over demo speed
- explicit contracts over implicit behavior
- modular boundaries over convenience imports
- repeatable delivery over heroics
- auditability over hidden side effects
- operational evidence over assumptions

Code is not considered high quality because it is clever, fast to write, or visually polished.

Code is considered high quality only if it is:

- understandable by engineers who did not originally write it
- safe to change in small increments
- testable at the right layer
- instrumented for diagnosis
- aligned to product, security, and compliance constraints

---

## 3. Non-Negotiable Engineering Principles

### 3.1 Specification-Backed Implementation

- Product behavior must trace back to the PRD, locked decisions, simulation spec, or approved follow-up decision record.
- Engineers must not invent new product rules, simulator thresholds, role names, tenant behavior, or AI response modes from intuition.
- If the source docs conflict, implementation stops until the conflict is resolved in source-of-truth docs.

### 3.2 Stable Boundaries

- The monorepo structure is part of the quality system, not an organizational suggestion.
- Apps own delivery surfaces.
- Shared packages own reusable domain logic and contracts.
- No app may become an unreviewed dumping ground for logic that should live in a package.

### 3.3 Explicit Contracts At Every Boundary

- Every external input must be validated.
- Every public module must expose typed inputs and outputs.
- Every queue job, API payload, AI context object, telemetry event, and configuration object must have a runtime schema.

### 3.4 Tenant Isolation Is Structural

- Tenant isolation is mandatory and must be enforced at the database layer and in application logic.
- A missing tenant guard is a release-blocking defect.
- No feature, report, export, admin tool, or background job may rely on developer discipline alone for isolation.

### 3.5 Determinism For Assessed Simulation

- Assessed simulation rules must be deterministic, reproducible, and testable.
- Simulator state, scoring, thresholds, and telemetry shape may not drift across implementations or environments.
- Renderer concerns must not silently change assessment behavior.

### 3.6 Safe AI As A Product Rule

- AI integration is a governed subsystem, not a generic chatbot add-on.
- Context binding, guardrails, transcript logging, escalation behavior, and response mode rules are mandatory.
- Any AI path that bypasses required safety or logging controls is non-compliant.

### 3.7 Small, Reversible Change Sets

- Prefer small pull requests and commit-sized changes.
- Large rewrites without migration strategy, test evidence, and rollback thinking are not acceptable.
- Refactors must preserve observable behavior unless the change is explicitly approved as behavior-changing work.

---

## 4. Architecture Standards

### 4.1 Approved Baseline

Myelin v1 engineering is standardized on:

- `pnpm` + `turbo` monorepo
- `TypeScript` as the default language
- `React + Vite` for the routed web app
- `NestJS + Fastify` for the backend API
- `BullMQ + Redis` for background jobs
- `Kysely + PostgreSQL` with RLS for the data layer
- `Three.js + cannon-es` for the simulator runtime

No team may substitute major stack components in active delivery without an approved architecture decision record.

### 4.2 Repository Dependency Rules

- `apps/*` may depend on `packages/*`.
- `apps/*` may not import from other apps.
- `packages/*` may not import from `apps/*`.
- Shared packages may depend on lower-level shared packages only where the dependency direction is obvious and documented.
- `packages/contracts` must remain dependency-light and portable.
- `packages/db` owns schema, migrations, RLS helpers, and query modules.
- Simulator packages must not depend on React UI concerns.

### 4.3 Module Responsibility Rules

- Controllers and route handlers coordinate; they do not own business logic.
- Domain services own business rules.
- Query modules own persistence logic.
- UI components own presentation and user interaction, not backend policy.
- The React app shell mounts the simulator; it does not own simulator internals.

### 4.4 Decision Records

- Any material architecture decision must be captured in an ADR before the implementation spreads across the codebase.
- ADRs are required for changes affecting stack selection, data model direction, AI runtime approach, auth model, storage model, or simulator execution model.

---

## 5. Code Standards

### 5.1 TypeScript Rules

- `strict` mode is required.
- `noImplicitAny` is required.
- `any` is banned unless the exact location is justified, isolated, and documented.
- Shared contracts must prefer `zod` schemas with inferred TypeScript types.
- Public APIs must expose typed return values. Implicit `unknown` or loosely shaped objects are not acceptable.

### 5.2 Function And Module Design

- Prefer small, single-purpose functions with obvious names.
- Prefer pure domain logic where possible.
- Side effects must be visible in the call path.
- Hidden mutable global state is not allowed.
- Utility modules must be cohesive; generic `helpers.ts` dumping grounds are not allowed.

### 5.3 Error Handling

- Errors must be explicit, typed where practical, and mapped to user-safe outcomes.
- Silent catch-and-ignore behavior is banned.
- Retry behavior must be deliberate and bounded.
- User-facing failures must be logged with enough context to diagnose the issue without leaking sensitive data.

### 5.4 Logging

- Structured logging is required in API and worker code.
- Use request, tenant, actor, and trace identifiers where available.
- `console.log` is acceptable for local bootstrap scripts and short-lived diagnostics only.
- Production paths must use the project logging standard.

### 5.5 Comments And Naming

- Names must explain intent without requiring surrounding guesswork.
- Comments should explain non-obvious reasoning, not restate code.
- TODO comments must include enough context to act on them later. Vague TODOs are not acceptable.
- Dead code and commented-out code blocks should be removed, not preserved as history.

---

## 6. Frontend Standards

### 6.1 App Structure

- Myelin is one routed application, not a collection of standalone static module pages.
- All major views must be URL-addressable.
- Route design must preserve user progress and support safe refresh/re-entry where required.

### 6.2 Component Design

- Presentational components should remain focused on rendering and interaction.
- Server state belongs in the server-state layer, not scattered across components.
- Business rules, access rules, and assessment logic must not be buried inside JSX trees.
- Reusable components must consume design tokens rather than hardcoded visual values.

### 6.3 Accessibility

- Core web surfaces must meet WCAG 2.2 Level A at minimum and target AA.
- Keyboard access is required for all core workflows, simulator HUD controls, and documented shortcuts.
- Focus behavior, reduced motion support, and caption requirements are part of the quality bar, not post-launch polish.

### 6.4 Frontend Anti-Patterns

- No inline event-handler business logic copied across screens.
- No direct fetch calls scattered throughout the UI when shared request layers exist.
- No component should directly shape tenant or role policy without going through the domain contract.
- No simulator correctness rule should depend on React rendering timing.

---

## 7. API And Backend Standards

### 7.1 API Design

- REST is the required v1 API surface.
- Endpoints must be versionable and documented.
- OpenAPI documentation is required for public API surfaces used by the web app, admin tooling, or external integrations.
- Idempotency keys are required for endpoints that ingest telemetry, create submissions, or may be retried by clients or workers.

### 7.2 Service Design

- Controllers must stay thin.
- Authentication, authorization, tenancy, validation, and business logic must be explicit layers.
- Service code must not rely on hidden request globals.
- Background work that can fail or retry must be moved to workers where appropriate.

### 7.3 Background Jobs

- Jobs must be idempotent.
- Jobs must record status transitions and failure causes.
- Retry policies must be defined, bounded, and observable.
- Dead-letter or equivalent failure handling must exist for critical workflows.

### 7.4 API Anti-Patterns

- No raw SQL in route handlers.
- No tenant-less query helpers.
- No response objects assembled by copy-paste across handlers.
- No privileged admin path may bypass audit logging.

---

## 8. Data, Database, And Security Standards

### 8.1 Data Modeling

- Core entities and relationships must follow the approved domain model.
- Business records must carry tenant scope.
- Soft-delete and versioning must be used where the product requires recovery or historical trace.
- Externally significant records must preserve historical versions.

### 8.2 Database Access

- `Kysely` is the primary data access layer.
- Database migrations must be forward-only, reviewed, and tested.
- Destructive or high-risk schema changes require rollback planning.
- Query modules must make tenant context obvious and hard to omit.

### 8.3 Tenant Isolation

- PostgreSQL RLS is mandatory for business data.
- Tests for tenant isolation are required for every new query surface that reads or writes tenant data.
- Cross-tenant reporting or regulator views require an explicit approved access model, not exceptions buried in code.

### 8.4 Security Controls

- Secrets must not be committed.
- Authentication and RBAC changes require focused review.
- File upload flows must validate type, size, malware scan, and access control behavior.
- Privileged actions must create immutable audit entries.
- Security-sensitive code paths require additional scrutiny before merge.

---

## 9. Simulator Engineering Standards

### 9.1 Runtime Boundary

- The simulator runtime must remain outside React render cycles.
- React hosts the canvas and surrounding controls only.
- Assessed behavior must live in simulator packages, not app-shell components.

### 9.2 Physics And Renderer Consistency

- `cannon-es` is the locked v1 physics engine.
- Renderer and physics baselines must follow the project's authoritative simulation rules.
- Performance optimizations must not change assessed behavior.

### 9.3 State Machine And Scoring

- The 7-step LOTO sequence must be enforced by a deterministic state machine.
- Wrong-order interactions must block progression and emit corrective feedback.
- Thresholds, step gating, retries, and scoring logic must come from approved source documents.
- Any threshold change is behavior-changing work and must be documented and tested.

### 9.4 Simulator Telemetry

- Step events and final attempt outcomes must use immutable, schema-validated payloads.
- Telemetry must be sufficient for replay-like diagnostics and lecturer review.
- Telemetry emission is part of simulator correctness, not optional analytics.

### 9.5 Simulator Quality Gates

- Sequence gating tests are mandatory.
- Telemetry contract tests are mandatory.
- Determinism tests are mandatory.
- Performance budget checks are mandatory before release sign-off.

---

## 10. AI Engineering Standards

### 10.1 Context Binding

- Every AI request must include the approved learner, simulation, and session context shape.
- Context must be minimized to what is required and must remain tenant-safe.
- Ad hoc prompt assembly without the approved context contract is not allowed.

### 10.2 Response Governance

- Response mode behavior must follow the approved text, guided media, and feature-flagged live mode rules.
- Safety-critical guidance must stay within the approved policy boundary.
- The model must not provide direct graded answers before submission.
- Unsafe real-world shortcut requests must refuse and redirect to safe guidance.

### 10.3 Transcript And Audit Requirements

- Transcript entries must be written before learner-visible rendering.
- Escalations, citations, and moderation-relevant metadata must be captured as required.
- AI logs must remain tenant-isolated and retention-aware.

### 10.4 AI Quality Gates

- Guardrail tests are mandatory.
- Prompt/context assembly tests are mandatory.
- Transcript logging tests are mandatory.
- Fallback behavior must be tested for provider failure and moderation events.

---

## 11. Testing Standards

### 11.1 Test Pyramid

Myelin follows a deliberate test pyramid:

- unit tests for domain logic, contracts, scoring, policy, and helper behavior
- integration tests for API, DB, queue, storage, auth, and audit behavior
- end-to-end tests for critical learner, lecturer, admin, and demo journeys
- simulator-specific tests for state sequencing, thresholds, telemetry, and performance-sensitive behavior

### 11.2 Mandatory Test Coverage Expectations

Initial minimum engineering gates:

- shared domain packages: `>= 80%` line coverage
- shared domain packages: `>= 70%` branch coverage
- security-critical modules: focused tests for all key paths
- tenant isolation query paths: positive and negative coverage for access rules
- simulator scoring and state machine modules: near-exhaustive behavioral coverage

Coverage numbers do not replace thoughtful tests. A high percentage with poor assertions does not meet the quality bar.

### 11.3 Required Test Cases By Risk Area

- every new API endpoint must have request validation and auth-path tests
- every tenant-scoped query path must have isolation tests
- every upload flow must have validation and failure-path tests
- every new role gate must have allow and deny tests
- every simulator step rule must have pass and wrong-order tests
- every AI orchestration change must have guardrail and transcript tests

### 11.4 Test Naming And Readability

- Test names must describe behavior, not implementation details.
- Tests must make setup, action, and expected outcome obvious.
- Snapshot tests should be used sparingly and never as a substitute for behavioral assertions on critical logic.

---

## 12. CI, Merge, And Release Standards

### 12.1 Required CI Checks For Merge

At minimum, the protected branch must require:

- lint
- typecheck
- unit test suite
- relevant integration suite
- build
- dependency vulnerability scan
- secret scan

Additional checks are required for affected areas:

- simulator tests when simulator packages or host wiring changes
- accessibility checks when learner-facing UI changes
- end-to-end smoke tests for major routed workflow changes
- migration checks when schema changes are introduced

### 12.2 Pull Request Standards

- No direct pushes to the protected main branch.
- PRs should be reviewable in one sitting.
- Large PRs require a reason and a structured review plan.
- PR descriptions must explain behavior changes, risks, test evidence, and rollout considerations.

### 12.3 Review Rules

- Security-sensitive code requires focused review.
- Data model and migration changes require DB-aware review.
- Simulator correctness changes require simulator-aware review.
- AI response or logging changes require policy-aware review.
- A green CI run is required but not sufficient; reviewers are expected to evaluate correctness and maintainability.

### 12.4 Release Gates

Before production release, Myelin must show evidence for:

- security controls
- tenant isolation validation
- audit log coverage
- consent and compliance flows
- critical user journey validation
- simulator correctness and performance
- AI safety and transcript behavior
- rollback readiness

---

## 13. Observability And Operational Standards

### 13.1 Logs, Metrics, And Traces

- API and worker flows must emit structured logs.
- Critical paths should emit measurable timings and error counts.
- Cross-service operations should carry trace or correlation identifiers.

### 13.2 Operational Readiness

- Features are not done if they cannot be diagnosed in staging or production.
- Critical workflows must expose enough evidence to support incident handling.
- Queue depth, failure rates, API latency, upload processing time, and simulator startup performance must be observable.

### 13.3 Runbooks

- Operationally significant subsystems require runbooks before release readiness.
- At minimum, runbooks are required for auth incidents, upload failures, queue failures, AI provider degradation, simulator failures, and cross-tenant exposure response.

---

## 14. Documentation And Team Scaling Standards

### 14.1 Codebase Documentation

- Every app and shared package must have a concise README covering purpose, entrypoints, main commands, and important constraints.
- Public interfaces and dangerous assumptions must be documented close to the code they affect.
- Source-of-truth docs must be updated when implementation materially changes the system.

### 14.2 Onboarding Friendliness

- Folder placement should make sense without oral tradition.
- Engineers joining later must be able to map product areas to repo locations quickly.
- Code that requires "tribal knowledge" to modify safely fails the maintainability bar.

### 14.3 Commit And Change Hygiene

- Commit messages should describe behavior and intent.
- Generated files should be minimized and justified.
- Formatting-only noise should be separated from behavior changes where possible.

---

## 15. Definition Of Done

A piece of work is done only when all of the following are true:

- the behavior is implemented in the correct layer
- the code follows approved architecture boundaries
- inputs and outputs are typed and validated
- error handling is explicit
- logs and audit behavior exist where required
- tests exist at the correct layer and pass
- affected docs are updated
- feature flags, migration steps, and rollout notes are handled where relevant
- the change is reviewable and diagnosable

For high-risk work, done also requires:

- tenant isolation evidence
- security review evidence
- accessibility evidence for affected UI
- simulator correctness evidence for assessed behavior
- AI guardrail and transcript evidence for agent changes

---

## 16. Explicitly Unacceptable Shortcuts

The following are below the Myelin quality bar:

- putting business logic directly in UI components because it is faster
- bypassing runtime validation because TypeScript "already covers it"
- adding tenant filters inconsistently instead of enforcing structural isolation
- writing simulator rules in the app shell instead of simulator packages
- changing thresholds, scoring, or physics behavior without source backing
- skipping transcript writes until after an AI response is shown
- introducing live-data access into demo pathways
- merging code that is not meaningfully tested
- using feature flags as a substitute for incomplete engineering
- leaving known risky behavior undocumented because "we will fix it later"

---

## 17. Adoption Checklist For The Rebuild Bootstrap

These standards should be enforced in the first implementation phase:

- create the monorepo structure that matches the approved app and package boundaries
- enable strict TypeScript configuration at the workspace level
- add ESLint and Prettier with project defaults
- add shared `zod` contract patterns early
- set up CI with blocking lint, typecheck, build, and test jobs
- add branch protection and PR templates
- add CODEOWNERS or equivalent reviewer routing
- add migration workflow and tenant-isolation test scaffolding before feature velocity increases
- add logging and trace conventions before the API surface expands
- define ADR templates and package README templates

If these controls are delayed, quality drift will arrive before the first meaningful release.

---

## 18. Final Standard

The standard for Myelin is not "good enough to demo."

The standard is:

- safe enough for institutions
- clear enough for future teams
- strict enough for regulated evidence workflows
- deterministic enough for assessed simulation
- observable enough for production support

Any code that does not meet that bar is incomplete, regardless of how quickly it was produced.
