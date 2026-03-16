# Myelin Professional Build - Features Document

Document status: Build planning baseline  
Version: 1.0  
Date: March 12, 2026  
Owner: Sethu (Founder, Myelin)  
Build partner: GPT-5.3-Codex (AI engineering copilot)  
Companion docs:
- `dev_docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`
- `3D_simulation_work/SIMULATION_DEVELOPMENT_GUIDANCE.md`
- `3D_simulation_work/threejs specs.md`
- `3D_simulation_work/mediapipe specs.md`
- `3D_simulation_work/Structured JSON Geometry Objects for 3D Simulation.md`
- `3D_simulation_work/Ultra-Granular JSON Geometry with Assembly Coordin.md`
- `3D_simulation_work/Additional Data for __Hyper-Realistic QCTO Simulat.md`

---

## 1. Purpose

This document defines the feature set for the Myelin professional build. It is structured for product, design, and engineering execution and includes the detailed simulation requirements from the `3D_simulation_work` specifications.

Execution model:

- Single human decision owner: Sethu
- Single AI build partner: GPT-5.3-Codex

---

## 2. Feature Delivery Principles

1. Safety-critical learning features are P0.
2. Multi-tenant security and auditability are mandatory at launch.
3. Simulator fidelity must be measurable, not subjective.
4. Browser-first delivery is required (no native install dependency for core usage).
5. Features must be configurable per institution where policy differences exist.

---

## 3. Product Feature Map

### 3.1 Legend
- Priority: `P0` required for production launch, `P1` shortly after launch, `P2` roadmap.
- Release target: `v1`, `v1.1`, `v2`.

### 3.2 Feature Catalog

| Feature ID | Feature Name | Description | Primary Roles | Priority | Release |
|---|---|---|---|---|---|
| CORE-001 | Multi-tenant foundation | Tenant-aware data model with strict institution isolation. | Super Admin, Admin | P0 | v1 |
| CORE-002 | Authentication and SSO | Email/password plus OIDC/SAML SSO by institution. | All | P0 | v1 |
| CORE-003 | RBAC and permissioning | Role-specific access controls for Student, Lecturer, Admin, Employer, Super Admin. | All | P0 | v1 |
| CORE-004 | Consent and privacy controls | POPIA consent capture, access logging, retention policy support. | Student, Admin | P0 | v1 |
| LEARN-001 | Pathway and module engine | Sequenced modules with prerequisites, completion states, and progress tracking. | Student, Lecturer | P0 | v1 |
| LEARN-002 | Module content delivery | Video, quiz, simulation, and practical content item support. | Student | P0 | v1 |
| LEARN-003 | Learner dashboard | Progress, next actions, reminders, and recent activity. | Student | P1 | v1.1 |
| SIM-001 | LOTO simulator runtime | 7-step interactive Lockout/Tagout simulation with deterministic step gating. | Student | P0 | v1 |
| SIM-002 | Simulator assessment telemetry | Step-level events, attempt outcomes, pass/fail evidence. | Student, Lecturer | P0 | v1 |
| SIM-003 | AI-guided simulation mode | Optional guided instructions while preserving strict step validation. | Student | P0 | v1 |
| AI-001 | Learning Coach text mode | Context-aware Q and A with safety-aligned guidance. | Student | P0 | v1 |
| AI-002 | Guided media mode | Curated visual explanations and contextual references. | Student | P0 | v1 |
| AI-003 | Live conversational mode | Feature-flagged voice + avatar/video coaching for approved tenants after readiness sign-off. | Student | P1 | v1 |
| PORT-001 | e-Portfolio submissions | Upload, process, and track practical video evidence. | Student, Lecturer | P0 | v1 |
| PORT-002 | Submission workflow states | Draft, submitted, review in progress, passed, rework required. | Student, Lecturer | P0 | v1 |
| PEER-001 | Peer review assignment engine | Assign anonymized peer reviews using configurable rules. | Student, Lecturer | P0 | v1 |
| PEER-002 | Rubric scoring console | Structured score and feedback capture with moderation support. | Student, Lecturer | P0 | v1 |
| WORK-001 | Workplace logbook | Capture workplace entries linked to placements and competencies. | Student, Employer, Lecturer | P0 | v1 |
| WORK-002 | Mentor sign-off workflow | Invite-only mentor approvals with comments and timestamps. | Employer, Lecturer | P0 | v1 |
| RES-001 | Statement of results | Generate and export verified outcome records. | Student, Admin | P0 | v1 |
| NOTIF-001 | Event-driven notifications | Email and SMS notifications for critical learner events. | All | P0 | v1 |
| ADMIN-001 | Institution configuration | Program rules, progression, policy toggles, and template management. | Admin | P0 | v1 |
| ADMIN-002 | Reporting and exports | Operational and learner outcome exports, including regulator-oriented datasets. | Admin, Lecturer | P0 | v1 |
| OPS-001 | Audit and observability | Audit log, metrics, traces, alerting, and incident visibility. | Super Admin, Admin | P0 | v1 |
| ACCESS-001 | Accessibility controls | Caption support, keyboard access, reduced motion options. | All | P0 | v1 |

---

## 4. Detailed Feature Specifications

## 4.1 Core Platform

### CORE-001 Multi-tenant foundation
- Description: Implement tenant-scoped storage, services, and access boundaries.
- Key requirements:
  - All business records include `tenant_id`.
  - Database-level row-level security is enforced.
  - Cross-tenant data access is blocked by default.
- Acceptance criteria:
  - Automated tests validate no cross-tenant reads/writes.
  - Admin exports include tenant scoping proof.

### CORE-002 Authentication and SSO
- Description: Support institutional and local login patterns.
- Key requirements:
  - Email/password with secure password policy and session handling.
  - OIDC and SAML 2.0 integration options.
- Acceptance criteria:
  - Institution can enable SSO without code change.
  - Session revocation and logout are enforced across devices.

### CORE-004 Consent and privacy controls
- Description: Implement POPIA-ready consent and data handling features.
- Key requirements:
  - Consent capture before practical/simulation data collection.
  - Data access logs and retention workflows.
- Acceptance criteria:
  - Consent records are versioned and auditable.
  - Privacy export request can be fulfilled per user and tenant.

---

## 4.2 Learning Experience

### LEARN-001 Pathway and module engine
- Description: Structured curriculum with gating and progress persistence.
- Key requirements:
  - Lock/unlock module logic based on prerequisites.
  - Mixed learning items and weighted completion.
- Acceptance criteria:
  - Progress is restored accurately after session restart.
  - Learner cannot enter locked modules unless override permission exists.

### LEARN-002 Module content delivery
- Description: Standardized lesson content experience across modules.
- Key requirements:
  - Video lesson support with metadata and completion signals.
  - Quiz and practical task item linking.
- Acceptance criteria:
  - Completion events are persisted for each content type.
  - Lecturer can view completion trace by learner and module.

---

## 4.3 AI Coach

### AI-001 Learning Coach text mode
- Description: Context-aware assistant integrated with module and simulator state.
- Key requirements:
  - Uses active topic, step, and recent learner errors as context.
  - Enforces safety constraints for protocol guidance.
- Acceptance criteria:
  - Unsafe shortcut requests are blocked and redirected to safe guidance.
  - Response history is tenant-scoped and role-protected.

### AI-002 Guided media mode
- Description: Coach can reference curated media snippets and standards links.
- Key requirements:
  - Referenced content linked to current lesson or simulator step.
  - Captions available where applicable.
- Acceptance criteria:
  - Media references are traceable to approved content sources.
  - Learner can jump from coach panel to relevant simulator/module context.

---

## 4.4 Evidence and Assessment

### PORT-001 e-Portfolio submissions
- Description: Learner practical evidence submission pipeline.
- Key requirements:
  - Upload format is MP4 only in v1.
  - File size limits configurable (default support for 500 MB in v1 policy).
  - Virus scanning and safe storage before review.
- Acceptance criteria:
  - Submission status lifecycle is accurate and auditable.
  - Failed processing cases show actionable error states.

### PEER-001 Peer review assignment engine
- Description: Assign and manage peer review tasks per cohort rules.
- Key requirements:
  - Self-review prevention.
  - Assignment balancing and anonymity settings.
- Acceptance criteria:
  - Required rubric fields must be completed before submit.
  - Lecturer can moderate and override peer review outcomes.

### WORK-002 Mentor sign-off workflow
- Description: Workplace mentor approves or rejects learner entries.
- Key requirements:
  - Invite-only mentor account model.
  - Structured sign-off record with comments and timestamps.
- Acceptance criteria:
  - Sign-off updates pathway evidence state.
  - Lecturer sees pending and overdue sign-offs.

---

## 4.5 Credentialing and Communication

### RES-001 Statement of results
- Description: Generate formal learner completion output.
- Key requirements:
  - Eligibility rules based on pathway completion and required workplace evidence.
  - Exportable result document with verification ID/hash.
- Acceptance criteria:
  - Results cannot be generated before completion rules are met.
  - Amendments create new versions, preserving prior records.

### NOTIF-001 Event-driven notifications
- Description: Multi-channel learner and stakeholder notifications.
- Key requirements:
  - Email and SMS channel support in v1.
  - Template-based events: onboarding, incomplete critical step, peer assignment, result issuance.
- Acceptance criteria:
  - Delivery status is tracked.
  - Retry strategy is configurable.

---

## 5. 3D Simulation Platform Features (from `3D_simulation_work`)

This section captures required and advanced simulator capabilities for the professional build.

## 5.1 Rendering and Runtime Quality

### SIM-RND-001 Browser rendering stack
- Requirement:
  - WebGL 2 is mandatory baseline.
  - WebGPU is optional with runtime feature detection and WebGL fallback.
- Acceptance:
  - Simulator runs in current Chrome, Firefox, Edge, Safari desktop versions.
  - Context loss recovery is implemented and tested.

### SIM-RND-002 Resolution and quality tiers
- Requirement:
  - Minimum tier: 1280x720, render scale 1.0, 1024 textures, 32-48 segment close-up geometry.
  - HD tier: 1920x1080, render scale 1.0-1.25, 2048 textures, 64-96 segment close-up geometry.
  - HD+ tier: 2560x1440, render scale 1.25-1.5, 2048-4096 textures, 96-128 segments.
  - UHD tier: 3840x2160, render scale 1.5-2.0, 4096 textures, 128+ segments.
- Acceptance:
  - Tier auto-selection based on capability is documented and deterministic.
  - Quality degradation occurs without breaking interaction correctness.

### SIM-RND-003 Performance targets
- Requirement:
  - Desktop target 60 fps, 16.67 ms frame budget.
  - Mobile/tablet target 30-60 fps.
  - Input-to-visual latency target <=54 ms for HD quality interaction.
  - Frame pacing variance target within +/-10 percent.
- Acceptance:
  - Performance test runs include low, mid, high tier profiles.
  - Tab-hidden rendering throttling uses Page Visibility behavior.

### SIM-RND-004 LOD and memory budgets
- Requirement:
  - Memory budgets: geometry 128 MB, texture 512 MB, physics 32 MB.
  - Performance tier guide:
    - high end: 60 fps, ~5000 objects, ~200 physics bodies
    - mid range: 45 fps, aggressive LOD, ~80 physics bodies
    - low end: 30 fps, simplified physics (kinematic-only profile)
- Acceptance:
  - Runtime counters expose object counts and memory pressure.
  - Fallback mode preserves step assessment fidelity.

---

## 5.2 Geometry, Materials, and Scene Blueprint Features

### SIM-GEO-001 Absolute coordinate scene blueprints
- Requirement:
  - Scene JSON supports absolute world coordinates and units in metres.
  - Precision support down to 0.0001 m for micro assemblies.
  - Parent-child assembly hierarchy must be explicit.
- Acceptance:
  - Import pipeline renders the same relative assembly geometry from structured and ultra-granular JSON datasets.

### SIM-GEO-002 Structured simulation blueprint library
- Requirement:
  - Include blueprint patterns from provided JSON specs:
    - Solar PV clinic microgrid assembly.
    - Mobile device repair (iPhone disassembly) assembly.
    - PCB rework technician micro-assembly.
- Acceptance:
  - Engine can instantiate each blueprint with collision-ready meshes and metadata.

### SIM-MAT-001 PBR material library
- Requirement:
  - Use physically based materials with industrial parameters (metalness/roughness/transmission/IOR/clearcoat).
  - Include baseline material definitions from provided datasets:
    - Zincalume corrugated iron.
    - Treated pine purlin.
    - PV panel glass.
    - Anodized aluminium extrusion.
    - Polished titanium frame.
    - OLED display layer.
    - FR4 substrate and solder mask.
- Acceptance:
  - Material presets are reusable by scenario config.
  - Reference parameters can be audited per object class.

### SIM-GEO-003 Micro-detail and imperfections layer
- Requirement:
  - Procedural imperfections supported, including:
    - rust pits, dents, and scratches on industrial surfaces
    - fingerprints and polish swirl marks
    - manufacturing artifacts (milling, chatter, porosity)
- Acceptance:
  - Detail layer can be toggled by quality tier.
  - Visual differences remain physically plausible under grazing light.

### SIM-ENV-001 HDRI and environmental context
- Requirement:
  - HDRI hierarchy support (8k/4k/2k/1k references) and IBL.
  - Dynamic weather for relevant scenarios (wind/rain) with configurable parameters.
  - Task lighting profiles for protocol-focused environments (for example LOTO setups).
- Acceptance:
  - Environment settings are data-driven by scene profile.
  - Lighting preserves readability of interactive targets.

---

## 5.3 Physics and Interaction Features

### SIM-PHY-001 Deterministic physics core
- Requirement:
  - Fixed timestep physics (1/60 or 1/120 sec profiles).
  - Rigid-body constraints and friction/restitution controls.
  - Async physics engine initialization to avoid blocking startup.
- Acceptance:
  - Same sequence of learner actions yields reproducible validation outcomes.

### SIM-PHY-002 LOTO-specific mechanics
- Requirement:
  - Valve rotation constraint around 90 degrees with tolerance controls.
  - Pressure simulation support using exponential bleed behavior for gauge logic.
  - Hasp/padlock/tag attachment logic with snap and alignment thresholds.
- Acceptance:
  - Interaction state transitions are blocked until prior step passes validation.
  - Camera motion does not bypass interaction gating.

### SIM-PHY-003 Interaction ergonomics and force realism
- Requirement:
  - Support force and grip constraints from provided specs:
    - hasp squeeze and alignment tolerance bands
    - torque resistance profiles
    - posture and fatigue penalties (advanced profile)
- Acceptance:
  - Advanced realism mode can be enabled via feature flag.
  - Standard mode keeps predictable usability for general learners.

### SIM-PHY-004 Tool and component physics dataset support
- Requirement:
  - Tool metadata includes torque targets, geometry, and interaction constraints.
  - Component metadata supports mass, center of mass, and material behavior.
- Acceptance:
  - Tool configuration is loaded from scenario data, not hard-coded logic.

---

## 5.4 Assessment and Validation Features

### SIM-VAL-001 Procedural step validation
- Requirement:
  - Required LOTO checks:
    - Step 2: valve rotation near pi/2 within configured tolerance.
    - Step 3: hasp distance to stem below threshold.
    - Step 6: pressure below zero-energy threshold (<0.1 bar).
    - Step 7: all required steps complete before test action pass.
- Acceptance:
  - Step pass/fail reasons are persisted and exposed in attempt review.

### SIM-VAL-002 Tolerance matrix support
- Requirement:
  - Scenario can define tolerance profiles by trade type (precision/heavy/overview).
- Acceptance:
  - Validation engine accepts pluggable tolerance profile definitions.

### SIM-VAL-003 Settling and completion windows
- Requirement:
  - Validation includes settle windows and velocity thresholds before marking completion.
- Acceptance:
  - No step is completed while object motion exceeds defined settle limits.

### SIM-VAL-004 Realism pass criteria instrumentation
- Requirement:
  - Capture optional realism metrics:
    - grip correctness
    - force feedback alignment
    - ergonomic and fatigue compliance
- Acceptance:
  - Metrics are available in staff analytics for pilot analysis.

---

## 5.5 Audio, Haptics, and Procedural Effects Features

### SIM-AUD-001 Spatial audio event framework
- Requirement:
  - Positional audio with distance attenuation and basic occlusion behavior.
  - Event profiles for contact and tool actions.
- Acceptance:
  - Core LOTO actions trigger distinguishable spatial cues.

### SIM-HAP-001 Haptic response framework
- Requirement:
  - Support gamepad/controller haptic patterns where available.
  - Event mapping includes intensity, frequency, and duration bands.
- Acceptance:
  - Haptics are optional and do not block task completion if unavailable.

### SIM-FX-001 Procedural failure and process effects
- Requirement:
  - Effect profiles from provided datasets are supported as pluggable modules:
    - DC arc flash
    - battery thermal runaway
    - conformal coating spray
    - trade-specific fluid/weld/smoke style effects
- Acceptance:
  - Effect modules can be enabled per scenario without core engine changes.

---

## 5.6 Optional MediaPipe and High-Precision Interaction Features

### SIM-MP-001 Camera and pose calibration pipeline (optional)
- Requirement:
  - Optional camera-based interaction mode includes calibration and world scale estimation.
  - Support checkerboard and reprojection quality controls from provided specs.
- Acceptance:
  - Calibration quality metrics are recorded before enabling precision mode.

### SIM-MP-002 Hand tracking precision profile (optional)
- Requirement:
  - Support 21-point hand tracking per hand with confidence thresholds and smoothing.
  - Optional full holistic model support up to 553 points (face + pose + hands).
- Acceptance:
  - Precision mode can be disabled without impacting standard pointer/keyboard mode.

### SIM-MP-003 Millimetre collision and grab confirmation (optional)
- Requirement:
  - Multi-stage grab confirmation:
    - collision detect
    - grip pattern validation
    - stiffness test
    - snap distance validation
  - Support capsule casts, ray hierarchy, and BVH-assisted collision paths.
- Acceptance:
  - Grab true-positive and false-positive rates are measurable in test harnesses.

### SIM-MP-004 Latency and fidelity targets (optional advanced)
- Requirement:
  - End-to-end camera-to-render latency target <=54 ms in advanced profile.
  - Validation metrics include spatial and temporal accuracy targets.
- Acceptance:
  - Benchmark outputs are produced per supported hardware profile.

---

## 6. LOTO v1 Feature Set (Production Commit)

The following simulator features are required for v1 launch:
- SIM-RND-001
- SIM-RND-002 (minimum and HD tiers)
- SIM-RND-003
- SIM-GEO-001 (for LOTO scene scope)
- SIM-MAT-001 (LOTO object material set)
- SIM-ENV-001 (task lighting plus static environment baseline)
- SIM-PHY-001
- SIM-PHY-002
- SIM-VAL-001
- SIM-VAL-003
- SIM-AUD-001 (core event cues)
- SIM-002 (assessment telemetry in product map)

Deferred to v1.1 or v2 by feature flag:
- SIM-GEO-002 full multi-trade blueprint library rollout
- SIM-PHY-003 advanced ergonomics/fatigue
- SIM-HAP-001 full haptic profile
- SIM-FX-001 advanced effects pack
- SIM-MP-001 through SIM-MP-004 advanced camera-based interaction mode

---

## 7. Feature Dependencies

| Feature | Depends On |
|---|---|
| SIM-001 LOTO simulator runtime | CORE-001, CORE-002, LEARN-001 |
| SIM-002 simulator telemetry | CORE-001, OPS-001 |
| AI-001 learning coach | CORE-002, LEARN-001 |
| PORT-001 e-Portfolio | CORE-002, CORE-004, OPS-001 |
| PEER-001 peer review assignment | LEARN-001, PORT-001 |
| WORK-002 mentor sign-off | CORE-002, CORE-003, WORK-001 |
| RES-001 statement of results | LEARN-001, PORT-002, WORK-002 |
| NOTIF-001 notifications | CORE-001, CORE-003 |

---

## 8. Acceptance and Readiness Checklist

### 8.1 Product Readiness
- All P0 features have signed acceptance criteria and test evidence.
- Role permissions validated for each critical workflow.
- End-to-end learner journey runs with no manual data intervention.

### 8.2 Simulation Readiness
- HD tier performance target met on baseline desktop hardware.
- LOTO validation checks pass with deterministic outcomes.
- Telemetry completeness verified for step and attempt events.
- Fallback behavior validated for lower-tier devices.

### 8.3 Compliance and Operations Readiness
- POPIA consent and access logging verified.
- Audit trails validated for grading, sign-off, and result generation.
- Alerting and runbooks available for production support.

---

## 9. Change Log

| Version | Date | Notes |
|---|---|---|
| 1.0 | March 12, 2026 | Initial professional build features document, including 3D simulation spec details from `3D_simulation_work`. |
