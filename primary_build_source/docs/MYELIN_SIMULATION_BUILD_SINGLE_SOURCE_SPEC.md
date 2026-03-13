# Myelin Simulation Build Single Source Spec

Document status: Primary build baseline
Version: 1.0
Date: March 12, 2026
Owner: Sethu (Founder, Myelin)
Build partner: GPT-5.3-Codex (AI engineering copilot)
Audience: Sethu and GPT-5.3-Codex
Applies to: Browser-based professional simulation build

---

## 1. Purpose

This document is the single execution baseline for simulation design and engineering. It consolidates the simulation reference set into one implementable source so Sethu and GPT-5.3-Codex can build with shared targets, tolerances, and acceptance gates.
The same baseline also defines how simulation components participate in a prospect-facing demo path built on the core curriculum experience.

### 1.1 Execution model for this spec

- Single human decision owner: Sethu
- Single AI build partner: GPT-5.3-Codex

---

## 2. Scope

This spec covers:
- Visual quality and UI/UX expectations for simulation scenes.
- Runtime architecture and technical implementation requirements.
- Physics, interaction, and validation rules for assessment integrity.
- Performance, memory, and compatibility targets for browser deployment.
- Data contract expectations for simulation geometry and interaction metadata.
- Simulation telemetry outputs required for pilot KPI scorecards and audit evidence.
- Core curriculum outcome mapping requirements for every enabled simulation scenario across partner tenants.
- Simulation behavior required for a guided prospect demo journey using safe demo data.

Out of scope:
- Institutional LMS workflows and non-simulation product modules.
- Native app-only capabilities.
- Commercial packaging and pilot contract policy decisions (owned by PRD).
- Production sales CRM workflows beyond simulation-linked demo telemetry.

---

## 3. Source Map (Consolidated)

Primary source files in `primary_build_source/references/simulation/`:
- `SIMULATION_DEVELOPMENT_GUIDANCE.md`: Core standards, tiers, runtime, validation.
- `threejs specs.md`: Three.js-oriented realism and LOTO interaction examples.
- `mediapipe specs.md`: Optional high-precision hand tracking pipeline and metrics.
- `Structured JSON Geometry Objects for 3D Simulation.md`: Structured scenario geometry datasets.
- `Ultra-Granular JSON Geometry with Assembly Coordin.md`: Ultra-precise assembly coordinates and hierarchy.
- `Additional Data for __Hyper-Realistic QCTO Simulat.md`: PBR, micro-detail, environment, audio/haptic, effects data.

---

## 4. Non-Negotiable Build Baseline (Must)

### 4.1 Platform and Delivery

- Must run in modern browsers with no native install requirement.
- Must use WebGL 2 as baseline.
- May use WebGPU only with feature detection and WebGL 2 fallback.
- Must support Chrome, Firefox, Edge, and Safari current versions.
- Must handle resize, device pixel ratio changes, and graphics context loss.
- Must pause or reduce rendering when tab is hidden (Page Visibility API).

### 4.2 Resolution and Performance Tiers

| Tier | Viewport (min) | Texture target | Close-up geometry target |
|---|---|---|---|
| Minimum | 1280x720 | 1024 | 32 to 48 segments |
| HD | 1920x1080 | 2048 | 64 to 96 segments |
| HD+ | 2560x1440 | 2048 to 4096 | 96 to 128 segments |
| UHD | 3840x2160 | 4096 | 128+ segments |

Performance baseline:
- Desktop target: 60 fps (16.67 ms frame budget).
- Mobile/tablet target: 30 to 60 fps with quality scaling.
- Input-to-visual latency target: <=54 ms (hard max <=100 ms).
- Primary action response: <=2 frames.

### 4.3 LOD and Memory Budget

- LOD0 mobile: <500 faces, 256 textures.
- LOD1 tablet: <2000 faces, 512 textures.
- LOD2 desktop: <8000 faces, 1024 textures.
- LOD3 VR/high detail: <20000 faces, 2048 textures.
- Memory ceilings: geometry 128 MB, textures 512 MB, physics 32 MB.
Performance tier targets:
- High end: 60 fps, up to 5000 objects, 200 physics bodies.
- Mid range: 45 fps target, aggressive LOD.
- Low end: 30 fps target, kinematic-only fallback where required.

---

## 5. Designer Requirements

### 5.1 Visual Realism Direction

- Use PBR-first workflow for all primary objects.
- Apply industrial material realism values for metal, glass, plastic, PCB, rubber.
- Include micro-detail where visible in close-up views: scratches, rust pits, swirl marks, fingerprints, wear transfer.
- Use physically plausible tone mapping and lighting.
- No flat-shaded key training assets in production scenes.

Mandatory material and visual inputs:
- PBR channels: base color, metallic, roughness, normal, clearcoat, IOR, transmission where relevant.
- Environment hierarchy: 8k/4k/2k/1k HDRI levels with fallback.
- Contact shadow quality and bias tuning to avoid floating artifacts.
- Trade-specific visuals for failure states (arc flash, thermal runaway, fluid, smoke).

### 5.2 Environment and Atmosphere

- Include scene-appropriate bounce lighting and task lighting.
- For outdoor PV contexts, support dynamic wind/rain parameters from reference data.
- Keep visual feedback readable under both bright and low-contrast scene conditions.

### 5.3 Interaction UX Requirements

LOTO interaction experience must visibly communicate:
- Step 2 valve turn progression and end-state snap.
- Step 3 hasp alignment, squeeze requirements, and lock confirmation.
- Step 4 padlock sequence and orientation validity.
- Step 6 zero-energy pressure confirmation.
- Step 7 completion + start action gating.

Feedback channels:
- Visual feedback on hover, grab, alignment, constraint hit, and success.
- Audio signatures tied to material and mechanism (for example 400 Hz creak, 1200 Hz spring, 1850 Hz connector snap, 4200 Hz torque click).
- Haptic signals when supported.

### 5.4 Accessibility and Overlay Design

- Keyboard support for core interactions is required.
- Reduced-motion option should be available for comfort.
- Ensure clear focus states for any HTML overlay controls.
- Keep instructional overlays concise and synchronized with simulation state.

### 5.5 Design Deliverables (Required)

- Scene style board and material mapping table.
- Interaction state map for each assessed step.
- HUD/overlay wireframes for guidance, errors, and completion states.
- Feedback matrix mapping events to visual/audio/haptic responses.
- Annotated tolerance visualization for high-risk interactions.

---

## 6. Engineering Requirements

### 6.1 Runtime and Rendering

- WebGL2 renderer config must include antialiasing and high-performance preference.
- Enforce linear workflow with sRGB output.
- Support tone mapping and shadows (HD target shadow map >=2048; UHD target 4096).
- Use instancing, culling, and LOD switching to hold frame budget.
- Use non-blocking loading for heavy assets and physics initialization.

Recommended post-processing baseline:
- AA (SMAA or FXAA), bloom, depth of field, chromatic aberration (subtle), film grain (subtle), ACES tone mapping.
- Adaptive quality controls when frame pacing degrades.

### 6.2 Physics and Interaction Core

Physics baseline:
- Gravity [0, -9.81, 0].
- Fixed timestep 1/60 or 1/120 for reproducibility.
- Stable constraint solver settings and optional CCD for fast/small objects.

LOTO mechanics baseline:
- Valve torque curve (Nm): [8.2, 3.1, 11.8].
- Rotational validation: pi/2 +/- 0.05 rad for step pass.
- Pressure model: `pressure = 10 * exp(-angle * 0.5)`.
- Step 6 pass threshold: pressure <0.1 bar.
- Step 3 pass threshold: hasp distance to stem <0.02 m.

Grab/manipulation baseline:
- Reach 0.4 m, grab radius 0.03 m, max force 25 N.
- Collision grab option: overlap 0.015 m, palm radius 0.08 m, finger capsule radius 0.012 m.
- Grip ramp 0 to 1 over 0.2 s with controlled release damping.

### 6.3 Data Contract Requirements

All simulation content data should support:
- Absolute world coordinates.
- Exact dimensions down to 0.0001 m where relevant.
- Parent-child assembly hierarchy.
- Material metadata for PBR rendering.
- Physics metadata (mass, CoM, friction, restitution, torque/force limits).
- Interaction metadata (snap points, tolerance windows, tool constraints).
- Collision-ready structures for ray/capsule/BVH workflows.

Scenario scope by release stage:

| Stage | Required Scenario Set | Delivery Control | Core Curriculum Constraint |
|---|---|---|---|
| v1 production gate | LOTO 7-step pathway (release-blocking). | Always enabled. | Must map to mandatory core curriculum outcomes before release and power demo navigation storyline. |
| v1 controlled pilot expansion | Solar PV installer, Mobile device repair, PCB rework. | Feature-flagged per institution/pilot with explicit acceptance criteria. | Must include documented mapping to core curriculum outcomes plus specialization outcomes. |
| v2+ | Additional utility and sector pathways. | Added through config-driven onboarding and QA certification. | No scenario activation without approved core-curriculum linkage and standards trace. |

### 6.4 Optional High-Precision Hand Tracking Mode

When camera-based tracking is enabled:
- Require explicit user gesture before requesting camera access.
Reference tracking thresholds:
- min detection confidence 0.88.
- min tracking confidence 0.94.
- max hands 2.
- End-to-end latency target <=54 ms.
Interaction fidelity thresholds:
- Grab true positive >=97.2%.
- Grab false positive <=0.8%.
- Fingertip RMS error <=7.8 mm.

If these thresholds are not met, fallback to pointer/keyboard mode and continue assessment safely.

### 6.5 Telemetry and Assessment Logging

- Log each assessed step with timestamp, tolerances, and pass/fail.
- Persist attempt traces for review and QA replay.
- Track performance metrics (fps, frame-time variance, interaction latency).
- Track input mode (pointer, gamepad, camera) for diagnostics.

### 6.6 Institutional Evidence Contract (Pilot and Governance)

Simulation telemetry must produce institution-ready evidence payloads:
- Step-level competency traces with baseline and current outcomes.
- Cohort-level completion and time-to-competency summaries.
- Intervention markers (where learners failed, retried, and recovered).
- Audit export pack including scenario version, scoring rules version, and signed timestamps.
- Core-curriculum mapping coverage (which required foundation outcomes were exercised and passed).
- Demo usage traces for prospect walkthrough (entry point, scenario completion, transition to KPI/governance view).

Evidence integrity requirements:
- Every assessed event must be traceable to scenario version and learner/session identifiers.
- KPI aggregates must be reproducible from immutable event logs.
- Cross-institution dashboards must use tenant-scoped aggregation only.
- No scenario may be enabled for a partner tenant unless its scoring events are mapped to required core curriculum outcome IDs.
- Prospect demo analytics must be stored separately from live tenant learning analytics.

### 6.7 Demo Experience Contract (Core Curriculum Showcase)

Simulation demo behavior requirements:
- Prospects must be able to complete a deterministic core-curriculum simulation walkthrough without needing partner-tenant credentials.
- Demo run must surface representative feedback, scoring, and intervention signals identical in structure to production flows.
- Demo mode must prevent access to real learner attempts, real institution identifiers, and live operational alerts.
- Demo route must end with generated sample KPI and governance evidence views for pilot discussion.

---

## 7. Acceptance Criteria

### 7.1 Assessment Correctness Gates

ISO-14118 LOTO checks:
- Step 2: rotation within pi/2 +/- 0.05 rad.
- Step 3: hasp distance <0.02 m.
- Step 6: pressure <0.1 bar.
- Step 7: all required steps complete and start action performed.

### 7.2 Performance Gates

- HD profile sustains 60 fps on mid-range desktop in primary browser.
- Input-to-visual latency <=54 ms in target conditions.
- Memory budgets stay within defined limits.
- No blocking behavior during normal asset loading transitions.

### 7.3 Realism and Interaction Gates

- PBR and environment lighting applied to all primary assets.
- Surface imperfection layers visible in close-up quality review.
- Audio/haptic cues match key interaction events.
- Constraint and force behavior follows documented ranges.

### 7.4 QA Sign-Off Artifacts

- Test checklist mapped to this spec sections.
- Recorded validation runs for LOTO sequence.
- Cross-browser verification evidence.
- Performance captures for minimum, HD, and degraded fallback modes.
- Feature-flag certification records for each controlled pilot expansion pathway.
- Evidence export validation showing KPI reproducibility from raw simulation events.
- Scenario-to-core-curriculum mapping report per enabled partner tenant.
- Recorded prospect demo walkthrough showing full core curriculum storyline and evidence handoff.

---

## 8. Build Execution Phases

1. Phase 1 - Core visual baseline: geometry + PBR + HDRI.
2. Phase 2 - Core interaction baseline: snap logic, constraints, step gating (LOTO mandatory path).
3. Phase 3 - Physics and assessment hardening: reproducibility + tolerance checks.
4. Phase 4 - Feedback and immersion: audio, haptics, micro-detail polish.
5. Phase 5 - Controlled pathway expansion: `PV`, `Mobile Repair`, `PCB Rework` behind feature flags with core-curriculum mapping validation.
6. Phase 6 - Optional precision mode: camera-based hand tracking integration.
7. Phase 7 - Performance, evidence, and certification prep: optimization + full QA evidence + pilot KPI export validation.
8. Phase 8 - Demo readiness and certification: core-curriculum walkthrough validation + demo data isolation checks.

---

## 9. Definition of Done

The simulation build is production-ready when:
- All must-have baseline requirements in Section 4 are met.
- Designer and engineering deliverables in Sections 5 and 6 are complete.
- Acceptance criteria in Section 7 pass in QA.
- LOTO pathway is executable with validated outcomes and release-blocking gates passed.
- Controlled pilot pathways (`PV`, `Mobile Repair`, `PCB Rework`) are executable when enabled and certified.
- Simulation evidence exports are reproducible and accepted for pilot KPI reporting.
- Every enabled scenario has validated linkage to mandatory core curriculum outcomes for the active partner tenant.
- Core-curriculum demo flow for prospects passes walkthrough, telemetry, and data-isolation validation.
