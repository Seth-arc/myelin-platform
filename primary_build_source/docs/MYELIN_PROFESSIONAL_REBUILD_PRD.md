# Myelin Project - Product Requirements and Specifications (Professional Rebuild)

Document status: Draft for execution  
Version: 2.0  
Date: March 12, 2026  
Owner: Sethu (Founder, Myelin)  
Build partner: GPT-5.3-Codex (AI engineering copilot)  
Prepared by: Sethu and GPT-5.3-Codex  
Primary source artifacts:
- `ai-study-buddy-mockup.html`
- `module-foundation.html`, `module-1.html`, `module-2.html`, `module-3.html`, `module-4.html`
- `index.html`, `about.html`, `login.html`, `registration.html`, `emails.html`
- `dev_docs/PRODUCT_SPEC.md`
- `dev_docs/Product Decisions.md`
- `3D_simulation_work/*`

---

## 1. Executive Summary

Myelin is an AI-native, simulation-first training platform for South African engineering trades. The current prototype proves concept and learning flow quality but is not production-ready. This document defines the complete product requirements and technical specifications for a professional rebuild.

The rebuild goal is to deliver a secure, scalable, multi-tenant product that supports:
- Structured learning pathways and assessments.
- High-fidelity, standards-aligned 3D simulation (starting with the 7-step LOTO protocol).
- AI learning coach support (text and guided video experience).
- Competency evidence workflows (e-Portfolio, peer review, workplace logbook, statement of results).
- A prospect-navigable demo path where the core curriculum presents the full Myelin platform experience end-to-end.

### 1.1 Execution model for this PRD

- Single human decision owner: Sethu
- Single AI build partner: GPT-5.3-Codex

---

## 2. Product Vision and Positioning

### 2.1 Vision
Provide a measurable and safe path from theory to verified workplace competence for engineering trades learners through simulation, AI coaching, and evidence-based assessment.

### 2.2 Value Proposition
- Simulation realism with procedural assessment, not passive content only.
- AI support embedded inside learning context (not a generic chatbot detached from task state).
- Credential pipeline linking simulator performance, practical submission, peer review, mentor sign-off, and formal outcomes.
- Core curriculum baseline is a non-negotiable pillar for every partner; partner-specific pathways are additive extensions, not replacements.

### 2.3 Market Context
- Initial geographic focus: South Africa pilot deployments with expansion-ready architecture for Sub-Saharan Africa.
- Qualification alignment: SAQA/NQF/QCTO pathways with configurable standards mappings per country.
- Institutional customer model: pilot-to-annual contracting for single-campus, multi-site, and national-program buyers.
- Primary buyer segments: education ministries/departments, TVET networks/colleges, and public utility training academies.

---

## 3. Current State Assessment (Prototype Baseline)

### 3.1 What exists now
- Marketing and narrative pages (`index.html`, `about.html`).
- Authentication-like UI pages (`login.html`, `registration.html`) without backend auth.
- Main learner shell prototype with sidebar, multi-view app experience, dark mode, and WebGL simulator (`ai-study-buddy-mockup.html`).
- Standalone module pages mirroring module views.
- Email template mockups for onboarding and key lifecycle notifications.

### 3.2 Strengths
- Strong product narrative and coherent learning journey.
- High-quality simulation interaction prototype with sequential task gating.
- Cohesive UI direction and explicit competency flow.

### 3.3 Gaps to fix in rebuild
- No production backend, auth, RBAC, tenancy, APIs, or persistence.
- No real AI service integration or safety-guarded response layer.
- No file processing pipeline for submissions.
- No operational reporting, auditability, observability, or admin tooling.
- No robust compliance controls (POPIA implementation, retention automation, etc.).
- Minor prototype behavior bug: simulator completion points to non-existent `assignments` view.

---

## 4. Rebuild Objectives

### 4.1 Business Objectives
- Launch institutional pilot with production-grade reliability and compliance.
- Demonstrate measurable improvement in practical protocol adherence and assessment throughput.
- Establish foundation for additional trades and simulator modules.

### 4.2 Product Objectives
- Deliver end-to-end learner lifecycle from onboarding to statement of results.
- Enable lecturer and admin oversight with institution-level configuration.
- Support employer and mentor workflows for workplace evidence.
- Provide a guided prospect demo experience using the core curriculum flow to showcase full platform capability before pilot kickoff.

### 4.3 Technical Objectives
- Implement secure multi-tenant architecture with row-level data isolation.
- Support 5,000 concurrent user sessions across institutions for v1.
- Provide modular simulation framework to add new scenarios without re-platforming.

---

## 5. Personas and Roles

Primary platform roles:
- Student (learner)
- Lecturer
- Admin (institutional operations)
- Employer mentor
- Ministry/Department Viewer
- Procurement/Commercial Reviewer
- Regulator/Quality Assurance Auditor
- Super Admin (platform owner)

Secondary stakeholders:
- Training officer
- Assessor / moderator
- Sethu and GPT-5.3-Codex (build owners)

Key role capabilities:
- Student: consume content, complete simulation, submit evidence, perform peer review, track results.
- Lecturer: monitor cohorts, review submissions, intervene, configure assessment windows.
- Admin: manage programs, users, policies, institution settings, exports/reports.
- Employer mentor: validate workplace logbook entries and sign-off evidence.
- Ministry/Department Viewer: read-only cross-institution KPI and cohort outcomes dashboards.
- Procurement/Commercial Reviewer: view approved pilot scope, KPI scorecard outcomes, and commercial pack options.
- Regulator/Quality Assurance Auditor: access audit-ready evidence exports and standards-alignment reports.
- Super Admin: platform governance, tenancy, releases, compliance controls.

---

## 6. Scope Definition

### 6.1 In Scope for v1 (Production)
- Multi-tenant web platform (institution-aware).
- Authentication, SSO, RBAC.
- Learner pathway and module progression.
- Core curriculum baseline units are mandatory for every partner tenant.
- Prospect demo mode based on core curriculum baseline units with safe, non-production demo data.
- LOTO simulation production version with event tracking and deterministic assessment.
- Controlled simulation pathway pack behind feature flags for pilot expansion (`PV`, `Mobile Repair`, `PCB Rework`) where approved.
- AI learning coach (text-first production, guided media and controlled video mode).
- e-Portfolio upload and submission workflow.
- Peer review workflow with rubric scoring and moderation controls.
- Workplace logbook with mentor sign-off.
- Statement of results generation and export.
- Notification service (email and SMS in v1).
- Admin and lecturer console basics.
- Audit logs and compliance features.
- Pilot scorecard workflow (baseline capture, weekly KPI tracking, conversion decision evidence).
- Package entitlement controls for Pilot Pack, Operational Plan, and National Program Plan.

### 6.2 Out of Scope for v1
- Full offline mode (target v2).
- Native mobile apps (responsive web first).
- Third-party funding disbursement integrations.
- Open mentor self-registration.
- Real-time multi-user simulation in same scene.

### 6.3 Future Scope (v2+)
- Offline-first and low-bandwidth sync mode.
- Expanded simulation catalogue (additional trades and scenarios beyond the controlled pilot pathway pack).
- Advanced hand-tracking interaction modes for simulator.
- Push notifications.
- Deeper analytics and predictive risk scoring.

---

## 7. Information Architecture

Core product areas:
- Public site: Home, About, demo capture, prospect demo entry.
- Auth: login, activation, password reset.
- Learner app:
  - Dashboard
  - Learning Pathway
  - Modules
  - Virtual Simulator
  - Learning Coach
  - e-Portfolio
  - Peer Review
  - Workplace Logbook
  - Statement of Results
  - Profile and Settings
- Staff app:
  - Cohorts and learner progress
  - Review queues and moderation
  - Program and policy configuration
  - Institution reporting and exports
- Platform admin:
  - Tenants, role policy, feature flags
  - Audit and compliance controls
  - Incident and delivery tooling

Demo surfaces:
- Prospect demo shell: guided navigation through core curriculum pathway and key role views.
- Demo evidence pack view: sample KPI, intervention, and governance outputs generated from demo dataset.

---

## 8. End-to-End User Journeys

### 8.1 Learner Journey (primary)
1. Account activation and consent.
2. Enter pathway and complete module content.
3. Enter simulator and complete validated sequence.
4. Submit practical evidence in e-Portfolio.
5. Complete assigned peer reviews and receive feedback.
6. Add workplace logbook evidence and request mentor sign-off.
7. Receive statement of results when completion criteria are met.

### 8.2 Lecturer Journey
1. Review cohort progress and risk indicators.
2. Inspect simulation attempt history and assessment outcomes.
3. Grade or moderate submissions and peer reviews.
4. Trigger remediation activities and notifications.

### 8.3 Employer Mentor Journey
1. Receive invite and authenticate.
2. Review learner placement and logbook entries.
3. Approve/reject with comments and timestamped sign-off.

### 8.4 Prospective Client Demo Journey
1. Prospect enters guided demo environment from public site.
2. Prospect navigates core curriculum pathway with staged learner experience.
3. Prospect views AI support, simulation attempt trace, and instructor intervention flow.
4. Prospect reviews admin/governance dashboard and sample pilot KPI scorecard.
5. Prospect exports demo value snapshot and books pilot scoping session.

---

## 9. Functional Requirements

Requirement IDs:
- `FR-xxx` Product functionality
- `SIM-xxx` Simulation-specific
- `AI-xxx` AI coach-specific
- `ADM-xxx` Admin and operations

### 9.1 Access, Identity, and Security

| ID | Requirement |
|---|---|
| FR-001 | The system shall support email/password login and secure session management. |
| FR-002 | The system shall support SSO via OIDC and SAML 2.0 per institution. |
| FR-003 | The system shall enforce RBAC for Student, Lecturer, Admin, Employer Mentor, Ministry Viewer, Procurement Reviewer, Regulator Auditor, and Super Admin. |
| FR-004 | Employer mentor accounts shall be invite-only, institution-linked, and not open self-signup. |
| FR-005 | Consent capture for POPIA shall be mandatory before learner training interaction begins. |
| FR-006 | All privileged actions shall create immutable audit log entries. |
| FR-007 | Ministry, procurement, and regulator roles shall default to read-only access unless explicit override is approved and audited. |

### 9.2 Learning Pathway and Modules

| ID | Requirement |
|---|---|
| FR-010 | Learners shall see module roadmap, completion status, and lock/unlock states. |
| FR-011 | Progress shall be persisted at lesson, assessment, and practical levels. |
| FR-012 | Module prerequisites shall gate access to subsequent modules. |
| FR-013 | Content shall support mixed item types: video, quiz, simulation, practical task. |
| FR-014 | Lecturers/admins shall configure module sequencing and completion rules. |
| FR-015 | Transcript export shall reflect verified completion state only. |
| FR-016 | Every partner tenant shall include Myelin core curriculum baseline units as mandatory prerequisites; partner-specific modules may extend but not replace them. |
| FR-017 | Platform shall provide a prospect demo path that reuses the core curriculum flow and demonstrates learner, instructor, and admin experiences with demo data. |

### 9.3 Virtual Simulator (LOTO v1)

| ID | Requirement |
|---|---|
| SIM-001 | The simulator shall enforce strict ordered completion of the 7-step LOTO sequence. |
| SIM-002 | Interaction shall be object-driven with raycast hit-testing and explicit step state. |
| SIM-003 | Each step completion shall emit persisted assessment events (who, when, elapsed time, outcome). |
| SIM-004 | Wrong-order interactions shall produce immediate corrective feedback without step advancement. |
| SIM-005 | The simulator shall expose restart and camera reset controls. |
| SIM-006 | AI-guided instruction mode shall be toggleable during simulation. |
| SIM-007 | Successful completion shall unlock practical evidence submission state. |
| SIM-008 | Simulation attempt history shall be viewable by learner and lecturer. |
| SIM-009 | Simulation content shall be configurable for standards updates without hard-coded text changes. |
| SIM-010 | The simulator shall support quality-tier degradation policies based on detected runtime performance. |

### 9.4 Learning Coach (Nomvula)

| ID | Requirement |
|---|---|
| AI-001 | Coach shall provide context-aware guidance based on active module/simulator step. |
| AI-002 | Coach shall support text mode in v1 with controlled media references. |
| AI-003 | Coach responses in safety-critical context shall include verification disclaimer and standard references. |
| AI-004 | Conversation logs shall be stored with tenant isolation and retention policy. |
| AI-005 | Learners shall be able to switch between text and guided media mode in-session. |
| AI-006 | Unsafe or non-compliant guidance requests shall trigger refusal and safe alternative guidance. |

### 9.5 e-Portfolio

| ID | Requirement |
|---|---|
| FR-020 | Learners shall upload practical evidence files (MP4/WebM in v1, max 500 MB configurable). |
| FR-021 | Upload pipeline shall validate type, size, malware scan, and transcode status. |
| FR-022 | Submissions shall be versioned with immutable timestamps and submitter metadata. |
| FR-023 | Lecturers shall review, score, comment, and assign remediation. |
| FR-024 | Learners shall see status transitions: Draft, Submitted, Under Review, Passed, Rework Required. |

### 9.6 Peer Review

| ID | Requirement |
|---|---|
| FR-030 | The system shall assign peer reviews based on configurable cohort rules. |
| FR-031 | Rubric-based scoring shall require completion of all required criteria before submit. |
| FR-032 | Reviewer quality scoring and moderation controls shall be available to lecturers. |
| FR-033 | The platform shall prevent learners from reviewing their own submissions. |
| FR-034 | Peer feedback shall be visible to submission owners with moderation state. |

### 9.7 Workplace Logbook

| ID | Requirement |
|---|---|
| FR-040 | Learners shall create and manage workplace logbook entries tied to placement records. |
| FR-041 | Logbook entries shall support evidence attachments and activity classification. |
| FR-042 | Mentor sign-off workflow shall include approve/reject/comment and date/time identity proof. |
| FR-043 | Lecturer/admin shall monitor sign-off completion and outstanding actions. |

### 9.8 Statement of Results

| ID | Requirement |
|---|---|
| FR-050 | Result generation shall require completion of configured pathway and workplace criteria. |
| FR-051 | Result output shall support downloadable PDF and system record view. |
| FR-052 | Result documents shall include program metadata, learner identity fields, and verification hash/ID. |
| FR-053 | Any post-issuance changes shall create an amended version, preserving prior issued records. |

### 9.9 Notifications and Communications

| ID | Requirement |
|---|---|
| FR-060 | Event-driven notifications shall support email and SMS in v1. |
| FR-061 | Notification templates shall be centrally managed and institution-overridable where allowed. |
| FR-062 | Core triggers: onboarding, incomplete critical step, peer review assigned, result issued. |
| FR-063 | Delivery outcomes (sent, delivered, failed) shall be tracked and reportable. |

### 9.10 Profile and Settings

| ID | Requirement |
|---|---|
| FR-070 | Learners shall manage profile details, contact preferences, and accessibility toggles. |
| FR-071 | Security controls shall include password change and session/device management. |
| FR-072 | Preference changes shall be persisted and applied cross-session. |

### 9.11 Admin and Lecturer Console

| ID | Requirement |
|---|---|
| ADM-001 | Admin shall manage users, cohorts, programs, and role assignments. |
| ADM-002 | Admin shall configure institution policy options (funding warning/block, progression rules, template rules). |
| ADM-003 | Lecturer shall see learner progress dashboard with intervention actions. |
| ADM-004 | Admin shall access audit logs and policy change history. |
| ADM-005 | Reporting exports shall be available in CSV and scheduled delivery mode. |
| ADM-006 | Ministry Viewer shall access aggregated completion, competency, and intervention dashboards across scoped institutions. |
| ADM-007 | Procurement Reviewer shall access pilot scorecard evidence pack and pilot-to-annual conversion report. |
| ADM-008 | Regulator Auditor shall access standards mapping, evidence trace logs, and signed result provenance reports. |
| ADM-009 | Demo mode shall expose read-only exemplar dashboards for prospects without revealing any real tenant-identifiable data. |

---

## 10. Simulation Technical Specifications (LOTO v1)

### 10.1 Scene and Interaction Design
- Runtime: Browser-based WebGL 2 with graceful fallback behavior.
- Scene graph elements: nameplate, primary valve, hasp, padlock, tag, bleed valve, gauge, control panel.
- Input model: pointer events + keyboard alternatives for key actions.
- Step engine: strict deterministic state machine (`current_step = 1..7`).
- Validation: explicit per-step pass criteria and no implicit progression.

### 10.2 Performance Targets
- Desktop target: 60 fps at 1080p on mid-range hardware.
- Minimum acceptable on constrained devices: 30 fps with adaptive quality.
- Input-to-feedback target: <= 54 ms for core interactions.
- Startup target for simulator scene to first interaction: <= 5 seconds on broadband baseline.

### 10.3 Quality and Visual Standards
- PBR materials for interactive assets.
- Tone mapping and color consistency across devices.
- LOD and instancing strategy for complex scenes.
- Dynamic lighting tuned for legibility of interaction points.

### 10.4 Assessment and Telemetry Requirements
- Emit event record per step attempt:
  - `attempt_id`, `tenant_id`, `user_id`, `session_id`
  - `step_number`, `event_type`, `is_success`, `elapsed_ms`, `timestamp`
- Final attempt outcome:
  - `pass_fail`, total elapsed time, reset count, assist mode usage
- Store enough detail for replay-like diagnostics and lecturer review.

### 10.5 Future-Ready Simulation Extensions
- Config-driven step and object maps for non-LOTO scenarios.
- Optional hand-tracking pipeline integration as feature-flagged mode.
- Physics engine extension points for force, torque, and material constraints.

---

## 11. AI Coach Specifications

### 11.1 System Behavior
- Context inputs:
  - Active module and topic
  - Current simulator step and error state
  - Learner performance trends
- Output constraints:
  - Safety-first responses
  - No authoritative claims without referenced standard context
  - Response classes: explain, hint, corrective prompt, escalation

### 11.2 Safety and Governance
- Mandatory prompt safety layer for hazardous instruction boundaries.
- Red-team tested refusal policies for unsafe procedural shortcuts.
- Sensitive data filtering and role-scoped visibility.
- Human override path for disputed guidance.

### 11.3 Quality Metrics
- Helpfulness rating (learner feedback).
- Resolution rate without human intervention.
- Safety compliance rate (no policy violations).
- Latency p95 for response generation.

---

## 12. Data Model (Core Domain Entities)

Primary entities:
- `Tenant`
- `User`
- `RoleAssignment`
- `Programme`
- `Module`
- `LearningItem`
- `Enrollment`
- `SimulationAttempt`
- `SimulationStepEvent`
- `Submission`
- `Rubric`
- `PeerReview`
- `Placement`
- `LogbookEntry`
- `MentorSignoff`
- `ResultStatement`
- `Notification`
- `AuditLog`
- `ConsentRecord`

Data principles:
- Tenant-scoped identifiers on all business records.
- Soft-delete where needed with audit trail, not destructive deletion.
- Versioning for externally significant records (submissions, results, policy configs).
- 10-year retention for learner/application/financial/audit classes (as per stakeholder decisions).

---

## 13. API and Service Specifications

### 13.1 API Style
- REST for transactional operations.
- GraphQL endpoint for complex learner dashboards and mobile-optimized queries.
- Event bus for async workflows (notifications, transcoding, scoring jobs).

### 13.2 Representative API Domains
- Auth and identity: `/auth/*`, `/sso/*`
- Pathway and modules: `/programmes/*`, `/modules/*`, `/progress/*`
- Simulator: `/sim/attempts`, `/sim/events`, `/sim/config`
- Portfolio: `/submissions/*`, `/files/*`
- Peer review: `/peer-reviews/*`, `/rubrics/*`
- Logbook: `/placements/*`, `/logbook/*`, `/signoffs/*`
- Results: `/results/*`
- Admin: `/admin/users`, `/admin/policies`, `/admin/reports`
- Notifications: `/notifications/*`, `/templates/*`

### 13.3 Contract Requirements
- OpenAPI spec for REST endpoints.
- Schema versioning for GraphQL.
- Idempotency keys for submission and event ingestion endpoints.
- Tenant and actor identity in all request context.

---

## 14. Non-Functional Requirements

### 14.1 Performance
- Support 5,000 concurrent user sessions across institutions.
- p95 page transition <= 2.0 s on standard broadband.
- p95 API response <= 400 ms for non-reporting calls.
- Background job SLA:
  - video processing start <= 60 s
  - processing complete <= 10 min for standard upload target size

### 14.2 Availability and Reliability
- Monthly uptime target: 99.9%.
- Zero-data-loss objective for committed transactional writes.
- Backup and disaster recovery with documented RTO/RPO (sign-off required before go-live).

### 14.3 Security
- Encryption in transit (TLS 1.2+), encryption at rest.
- Password hashing with strong adaptive algorithm.
- Tenant isolation enforced at database layer (RLS required).
- Centralized secrets management and key rotation policy.
- OWASP ASVS-aligned secure development controls.

### 14.4 Privacy and Compliance
- POPIA consent tracking and revocation handling.
- Data minimization for AI context.
- Access logs and export trails for regulated records.
- Data subject access and correction workflows.
- Prospect demo mode must use synthetic or anonymized dataset only, with explicit prohibition on live tenant data exposure.

### 14.5 Accessibility
- WCAG 2.2 AA baseline for web application surfaces.
- Keyboard navigation for all core workflows.
- Captions support for video content.
- Reduced motion mode for intense animation contexts.

### 14.6 Localization
- English only for v1.
- Architecture must support future locale expansion without redesign.
- Content and AI response pipelines must support terminology dictionaries and language packs per institution/country in future phases.

---

## 15. Frontend and UX Specifications

### 15.1 Design System
- Define reusable component library for navigation, cards, progress indicators, forms, tables, and feedback states.
- Tokenized design variables (color, type, spacing, radii, elevation, motion).
- Dark/light themes with consistent token mapping.

### 15.2 Responsive Behavior
- Desktop-first for full simulator and management surfaces.
- Tablet support for learning and review workflows.
- Mobile support for lightweight learner tasks and notifications (full simulator optional by capability).

### 15.3 State and Navigation
- Replace inline `onclick` handlers with framework state routing and action dispatching.
- URL-addressable routes for all major views.
- Preserve scroll and filter state where user workflow benefits.
- Include a deterministic demo route map for prospects that starts at core curriculum and traverses full platform capabilities.

---

## 16. Content and Assessment Framework

### 16.1 Learning Content Structure
- Modules contain lessons, quizzes, simulations, and practical assessments.
- Completion policy supports weighted criteria and mandatory checkpoints.
- Content metadata includes duration, prerequisites, difficulty, and standards references.
- Core curriculum foundation units are mandatory in every partner pathway before partner-specific specialization units.

### 16.2 Assessment Rules
- Quiz scoring and pass thresholds configurable by institution/programme.
- Practical submissions require rubric-linked evaluation.
- Peer review participation can be a gating requirement where configured.

### 16.3 Standards Alignment
- Reference mappings from each learning activity to standards/outcomes.
- Audit report export for standards evidence.

---

## 17. Notification and Communication Specifications

Supported channels:
- Email (v1 required)
- SMS (v1 required)
- Push (v2 planned)

Template categories:
- Onboarding
- Protocol/critical step reminders
- Review assignments
- Feedback and grading updates
- Result issuance and completion

Delivery controls:
- Quiet hours by institution policy.
- Retry logic and failure handling.
- User preference and opt-in/opt-out enforcement by channel type.

---

## 18. Security and Compliance Detailed Controls

Minimum controls:
- RBAC and principle of least privilege.
- Row-level tenant isolation policies and test coverage.
- Comprehensive audit trail for:
  - login and auth events
  - policy and grade changes
  - data exports and result issuance
- Secure file handling:
  - malware scanning
  - signed URL access
  - short-lived access tokens
- AI safety controls:
  - policy filtering
  - response moderation
  - incident logging and review queue

---

## 19. Analytics, Reporting, and Observability

### 19.1 Product Analytics
- Funnel metrics:
  - activation to first simulator attempt
  - attempt to pass
  - pass to practical submission
  - submission to result issuance
- Learning quality metrics:
  - step-level error rates
  - remediation loop frequency
  - peer review quality index

### 19.2 Operational Observability
- Central logs, metrics, and traces.
- SLO dashboards for API latency, simulator load times, job queues.
- Alerting on tenant-impacting failures and degraded performance bands.

### 19.3 Compliance Reporting
- Institution-level progress and completion exports.
- Audit export packages for regulator/institution review.

---

## 20. Architecture Blueprint (Target)

### 20.1 High-Level Components
- Web frontend application.
- API gateway and auth service.
- Core domain services:
  - learning/pathway
  - simulation telemetry
  - portfolio/review
  - logbook/sign-off
  - results
  - notification
  - AI orchestration
- Relational database with tenant RLS.
- Object storage for media/evidence.
- Job queue and worker cluster for async tasks.
- Analytics pipeline and warehouse export path.

### 20.2 Deployment Principles
- Cloud-native deployment with environment isolation (dev/stage/prod).
- Zero-downtime migration patterns for schema and service deployment.
- Feature flags for staged rollout and pilot-specific controls.

---

## 21. Quality Assurance and Acceptance Strategy

### 21.1 Test Pyramid
- Unit tests: domain logic, scoring rules, policy engine.
- Integration tests: API + DB + queue + storage interactions.
- End-to-end tests: core learner and lecturer workflows.
- Simulator-specific automated checks:
  - sequence gating
  - event emission correctness
  - performance budget checks

### 21.2 Release Gates
- Security gate: SAST/DAST and dependency checks pass.
- Performance gate: NFR targets met in staging load tests.
- Compliance gate: audit logging and consent workflows validated.
- Curriculum integrity gate: all enabled partner configurations enforce mandatory core curriculum baseline units and mapped outcome traces.
- Product gate: acceptance criteria for all priority v1 stories signed off.

---

## 22. Delivery Plan

### Phase 0 - Foundations (4-6 weeks)
- Architecture setup, CI/CD, auth baseline, design system, tenant model.

### Phase 1 - Core Learner Platform (6-8 weeks)
- Pathway/modules, profile/settings, dashboard baseline, notifications skeleton.

### Phase 2 - Simulator and AI Core (8-10 weeks)
- Production LOTO simulator, telemetry, AI text guidance integration with safety controls.

### Phase 3 - Evidence and Outcomes (6-8 weeks)
- e-Portfolio, peer review, logbook, result statement generation.

### Phase 4 - Pilot Hardening (4-6 weeks)
- Load/perf hardening, analytics, security hardening, UAT, pilot launch.

---

## 23. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Simulation performance variance on low-end devices | Poor learner experience | Adaptive quality tiers, performance budgets, device capability checks |
| AI safety drift in critical guidance | Compliance and safety risk | Guardrails, policy tests, human escalation path |
| Multi-tenant data leakage risk | Critical legal and trust impact | RLS enforcement, automated isolation tests, security reviews |
| Video processing delays under load | Assessment bottlenecks | Queue autoscaling, backpressure controls, SLA monitoring |
| Scope expansion beyond v1 | Delivery delay | Strict release scope governance and feature flagging |

---

## 24. Open Decisions Requiring Product Owner Confirmation

1. Confirm v1 scope boundary between training platform modules and broader SIS/ERP ambitions captured in prior stakeholder notes.
2. Confirm formal list of regulator export formats required at pilot launch.
3. Confirm required level of AI video mode in v1 (full conversational video vs guided media mode).
4. Confirm statement-of-results legal formatting requirements per institution/regulator.
5. Confirm target pilot institutions and cutover timeline.

---

## 25. Prototype-to-Production Traceability

Mapped prototype views:
- `view-dashboard` -> Learner dashboard service and progress APIs.
- `view-pathway`, `view-module-*` -> LMS/module engine and content service.
- `view-sandbox` -> Simulation runtime + telemetry + attempt store.
- `view-buddy` -> AI orchestration, policy layer, chat UI.
- `view-portfolio` -> Submission pipeline + storage + review.
- `view-peer-review` -> rubric engine + allocation service.
- `view-logbook` -> placement and sign-off workflows.
- `view-results` -> result generation and document service.
- `view-profile` -> identity profile + preferences service.

Known prototype inconsistencies to resolve:
- Broken navigation target to `assignments` state after simulator completion.
- Theme storage key inconsistency (`myelin-theme` vs `theme` in module pages).
- Standalone module pages diverge from unified app state model.

---

## 26. Definition of Done for Professional Rebuild

The rebuild is considered complete when:
- All v1 functional requirements (`FR`, `SIM`, `AI`, `ADM`) are implemented and tested.
- NFR targets are demonstrated in staging and pilot readiness report.
- Security, compliance, and audit controls pass sign-off.
- Pilot institution onboarding and sample learner journeys run end-to-end without manual backend intervention.
- Operational runbooks, incident playbooks, and support handoff are complete.

---

## 27. Change Log

| Version | Date | Author | Notes |
|---|---|---|---|
| 2.0 | March 12, 2026 | Product + Engineering | New comprehensive PRD and technical specification for professional rebuild. |
| 2.1 | March 12, 2026 | Product + Engineering | Added pilot contract, KPI scorecard model, commercial entitlement map, compliance matrix, and simulation roadmap reconciliation. |

---

## 28. Pilot-to-Annual Execution Contract (8-12 Weeks)

Pilot execution follows this mandatory structure for ministry, TVET, and utility buyers.

| Phase | Weeks | Required Work | Exit Gate |
|---|---|---|---|
| Discovery and Baseline | 1-2 | Pathway mapping, baseline KPI capture, role and data-access mapping, pilot scorecard sign-off. | Signed pilot KPI scorecard and named owners for each KPI. |
| Integration and Setup | 3-4 | LMS/SIS integration, cohort provisioning, AI guardrail policy config, reporting wiring. | End-to-end dry run with no Sev-1/Sev-2 blockers. |
| Controlled Cohort Live | 5-8 | Weekly live operation, interventions, governance review, evidence capture. | Positive trend on target KPIs with stable incident/SLA profile. |
| Optimization and Conversion | 9-12 | Tuning, staff enablement, controls hardening, annual proposal drafting. | Pilot-to-annual decision approved or procurement decision date locked. |

### 28.1 Pilot Entry Prerequisites

- Signed statement of work and scorecard.
- Minimum one institution admin owner and one teaching owner assigned.
- Data processing and privacy terms approved.
- Escalation and incident contacts confirmed.

### 28.2 Pilot Exit and Conversion Rules

- At least two target KPIs must meet or exceed threshold for two consecutive weeks.
- No unresolved critical compliance incident at closeout.
- Final value report must include baseline vs current vs target for all committed KPIs.
- Conversion recommendation must include package entitlement and rollout phasing.

---

## 29. KPI Scorecard Contract

These KPI families are mandatory for pilot sign-off and conversion decisions.

| KPI Family | Metric Definition | Baseline Source | Target Window | Reporting Cadence |
|---|---|---|---|---|
| Learner Completion | % of enrolled learners reaching module/program completion in pilot scope. | Pre-pilot cohort records. | Weeks 5-12 | Weekly |
| Core Curriculum Coverage | % of learners completing required core curriculum baseline units in pilot scope. | Baseline program mapping and enrollment data. | Weeks 5-12 | Weekly |
| Competency Attainment | % of learners achieving defined practical/simulation competency threshold. | Assessment and simulation event baseline. | Weeks 5-12 | Weekly |
| Time-to-Competency | Median days from cohort start to competency threshold. | Historical cohort timeline. | Weeks 5-12 | Weekly |
| Instructor Capacity | Learners actively supported per lecturer plus intervention cycle time. | Lecturer workload baseline. | Weeks 5-12 | Weekly |
| Pilot Conversion Readiness | Composite readiness score from KPI attainment, SLA stability, and governance completion. | Pilot operations records. | Weeks 9-12 | Weekly |
| Demo-to-Pilot Conversion | % of qualified prospects who complete guided demo and enter pilot scoping. | Demo analytics baseline. | Rolling monthly | Weekly |

Scorecard rules:
- Every KPI must include owner, formula, data source, and minimum sample size.
- Baselines must be frozen at pilot kickoff and versioned.
- KPI formula changes during pilot require change control and sponsor sign-off.

---

## 30. Commercial Packaging and Entitlement Map

| Package | Typical Buyer | Included Entitlements | Controlled/Optional Entitlements |
|---|---|---|---|
| Pilot Pack | Single institution/division | Core curriculum foundation pack, core pathway delivery, LOTO simulation, AI learning coach, KPI scorecard, weekly governance review. | Additional simulation pathways behind feature flags; managed AgentOps add-on. |
| Operational Plan | Multi-site institution/network | Pilot Pack plus multi-site admin controls, expanded reporting, recurring optimization cadence. Core curriculum foundation pack remains mandatory across all sites. | Advanced integrations and additional AI role modules by approved scope. |
| National Program Plan | National ministry/agency or utility HQ | Operational Plan plus cross-institution governance views, regulator evidence packs, phased rollout controls. Core curriculum foundation pack remains mandatory across all participating institutions. | Country-specific localization packs and advanced policy automation modules. |

Entitlement control rules:
- Entitlements are enforced by tenant-level feature flags.
- Any high-risk AI automation remains approval-gated regardless of package.
- Package upgrades require migration checklist and governance sign-off.

---

## 31. Compliance and Data Localization Baseline Matrix

This matrix defines implementation controls and does not replace legal review.

| Geography | Primary Compliance Focus | Minimum Product Control | Approval Owner |
|---|---|---|---|
| South Africa | POPIA and institutional data governance | Consent capture, tenant isolation, audit exports, role-scoped access, retention enforcement. | Institution Admin + Super Admin |
| Rwanda (expansion-ready) | Public-sector and education data-handling constraints | Regional data-hosting decision record, cross-border export approval log, redaction for analytics exports. | Country Program Owner + Super Admin |
| Kenya (expansion-ready) | Public education and learner data controls | Policy pack selection by tenant, immutable export logs, regulator-ready reporting templates. | Institution Admin + Super Admin |
| Zambia (expansion-ready) | Procurement and learner record controls | Data classification labels, export approval workflow, retention policy lock by tenant. | Institution Admin + Super Admin |

Operational rules:
- No cross-border export of personally identifiable learner data without approved policy rule and audit log.
- Compliance policy packs are versioned and traceable by tenant and effective date.
- Regulator evidence exports must be reproducible from immutable audit records.

---

## 32. Simulation Pathway Roadmap Reconciliation

This roadmap resolves scope differences across PRD, simulation spec, and sales narrative.

| Release Stage | Required Simulation Scope | Delivery Control |
|---|---|---|
| v1 production gate | LOTO 7-step pathway is mandatory and release-blocking. | Always on for v1 institutions. |
| v1 pilot expansion | `PV`, `Mobile Repair`, `PCB Rework` pathways available as controlled packs. | Feature-flagged per pilot with explicit acceptance criteria. |
| v2+ | Additional utility-grade and sector-specific pathways (for example substation, water operations) after evidence from pilot packs. | Added via config-driven scenario onboarding and governance review. |

Alignment rule:
- The simulation build spec remains the technical source of truth for fidelity/performance criteria.
- The PRD remains the source of truth for release packaging and buyer-facing rollout commitments.
- Core curriculum baseline remains mandatory for every partner package and geography; specialization pathways are layered on top.
