# Myelin AI Agent v1 Build Blueprint

Document status: Execution blueprint
Date: March 12, 2026
Owner: Sethu (Founder, Myelin)
Build partner: GPT-5.3-Codex (AI engineering copilot)
Audience: Sethu and GPT-5.3-Codex

---

## 1. Purpose

Define the implementation blueprint for the Myelin AI Agent in v1 so Sethu and GPT-5.3-Codex can build, test, and release a production-capable agent that supports:

- Core curriculum learning support (`FND-TT-01`, `FND-IW-01`)
- Simulation-context guidance
- True live conversational AI mode
- Safety governance and admin escalation
- Prospect-facing guided demo flow built on the core curriculum baseline

This blueprint aligns with:

- `primary_build_source/docs/v1_decisions.md`
- `primary_build_source/references/ai_agent/tau_agent_handbook.md`
- `primary_build_source/references/core_curriculum/foundation_curriculum_v2.md`
- `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`

### 1.1 Execution model for this blueprint

- Single human decision owner: Sethu
- Single AI build partner: GPT-5.3-Codex
- No separate delivery team is assumed in this document

---

## 2. Locked v1 Constraints

The agent implementation must enforce these decisions:

- AI modes required: text, guided media, true live conversational video.
- No direct answers to graded assessments before submission.
- Unsafe real-world guidance requests must be refused and redirected to human/professional guidance.
- Safety/distress escalation SLA: 15 minutes.
- Escalation owner: admin.
- Human decision authority remains with educators/admins; AI may draft or recommend but may not finalize high-impact decisions.
- AI chat retention: 2 years.
- Core curriculum scope only: `FND-TT-01`, `FND-IW-01`.
- Core curriculum baseline is mandatory across all partner tenants/packages; partner-specific adaptations are extension-only.
- Demo mode must use demo dataset boundaries only and must never expose live tenant-identifiable data.
- QTB behavior and plain-language pedagogy from Tau handbook are mandatory.

---

## 3. Capability Scope (v1)

### 3.1 Must-have capabilities

- Context-aware instructional chat tied to:
  - active module
  - active simulation step
  - recent learner errors/retries
- Guided media references (approved content links only).
- Live conversational interaction (voice + avatar/video surface).
- Policy-safe responses for:
  - grading/cheating attempts
  - safety-critical requests
  - distress signals
- Admin escalation queue with timestamp, SLA tracking, and outcome state.
- Full transcript logging, citations, and audit trace per session.
- Guided demo narration mode that explains learner, instructor, and admin workflow moments in the core curriculum flow.

### 3.2 Out of scope in v1

- Open-domain generic assistant behavior outside approved curriculum.
- Autonomous grading authority.
- Safety authority replacement for on-site professionals.
- Free-form sales chatbot behavior detached from approved curriculum and platform workflow.

### 3.3 Institutional Agent Role Map (v1-aligned)

| Agent Role | v1 Behavior | Autonomy Level | Human Owner |
|---|---|---|---|
| Learner Coach | In-session explanations, hints, remediation prompts, and safe fallback guidance. | Assistive only | Lecturer/Admin |
| Instructor Copilot | Lesson adaptation suggestions, feedback drafts, intervention suggestions. | Assistive only | Lecturer |
| Assessment Builder | Draft question/rubric suggestions for lecturer review. | Draft only | Lecturer |
| Cohort Risk Agent | Early-warning flags for dropout/performance risk using approved signals. | Alerting only | Lecturer/Admin |
| Certification Sentinel | Deadline/compliance reminders and readiness checks for completion workflows. | Alerting only | Admin |
| Content Localizer | Terminology and level adaptation suggestions within approved curriculum corpus. | Draft only | Lecturer/Admin |

### 3.4 Role Activation Boundaries by Package

- Pilot Pack: Learner Coach, Instructor Copilot, Cohort Risk Agent (alert-only), Certification Sentinel, all anchored to core curriculum baseline units.
- Operational Plan: Pilot Pack roles plus Assessment Builder drafts for approved modules.
- National Program Plan: Operational Plan roles plus expanded role-scoped dashboards; all high-impact actions remain approval-gated.
- Prospect Demo Mode: read-only guided experience through core curriculum baseline with synthetic/anonymized demo signals only.

---

## 4. Target Architecture

### 4.1 Logical architecture

```text
Client (Web App)
  |- Text Chat UI
  |- Guided Media Panel
  |- Live Voice/Video UI (WebRTC)
  |- Simulation Context Emitter
        |
        v
API Gateway / BFF
        |
        v
AI Orchestrator Service
  |- Session Manager
  |- Context Builder
  |- Policy Guardrails (pre + post)
  |- Prompt Builder (QTB + role style)
  |- Retrieval Orchestrator (RAG)
  |- Escalation Engine
  |- Audit Logger
        |
        +--> Knowledge Index (curriculum + rubrics + SOP refs)
        +--> LLM Inference Service
        +--> Speech Services (ASR/TTS)
        +--> Avatar/Video Render Service
        +--> Notification Service (email/SMS/push)
        +--> Core Platform DB (tenant, user, role, policy)
```

### 4.2 Deployment model

- Multi-tenant, tenant-isolated service layer.
- Environment separation: `dev`, `stage`, `prod`.
- Feature flags per tenant for staged rollout and pilot control.
- Previous-release redeploy path prepared for rollback.

---

## 5. NVIDIA-First Live Conversational Options

You requested NVIDIA-leaning architecture. Use one of these:

### Option A: Full NVIDIA stack (recommended target)

- LLM inference: NVIDIA NIM LLM service.
- Speech: NVIDIA Speech NIM (ASR + TTS).
- Avatar/video animation: NVIDIA ACE Audio2Face-3D service.
- Orchestration and policy remain in Myelin backend.

Pros:

- Single-vendor acceleration path.
- Strong real-time speech + avatar alignment.
- Cleaner path for future GPU optimization.

Tradeoff:

- Higher integration complexity in first implementation wave.

### Option B: NVIDIA realtime media + external LLM

- Speech + avatar via NVIDIA stack.
- LLM via current best-fit provider.

Pros:

- Faster model iteration and safety tuning.
- Lower early lock-in risk.

Tradeoff:

- Multi-vendor operations and observability complexity.

### Option C: Two-stage rollout (demo-first)

- Stage 1: live voice conversation + text UI.
- Stage 2: full avatar/video enhancement.

Pros:

- Fastest demo readiness.
- Lower initial latency risk.

Tradeoff:

- Live video fidelity deferred by one release increment.

---

## 6. Service Contracts

### 6.1 Core APIs (minimum)

- `POST /ai/sessions`
  - Create session with tenant/user/context.
- `POST /ai/messages`
  - Text turn request/response with policy result and citations.
- `POST /ai/context/events`
  - Push simulation/module events to active session.
- `POST /ai/realtime/token`
  - Issue short-lived token for realtime voice/video channel.
- `POST /ai/escalations`
  - Create manual or auto escalation case.
- `GET /ai/escalations/:id`
  - Track status, owner, timestamps.
- `GET /ai/sessions/:id/transcript`
  - Retrieve authorized transcript and policy trace.
- `POST /ai/approvals`
  - Record explicit human approval/rejection for high-impact AI-proposed actions.
- `GET /ai/approval-matrix`
  - Retrieve active approval policy rules by tenant/package.
- `POST /ai/demo/sessions`
  - Start guided demo session scoped to core curriculum demo dataset.

### 6.2 Event schema (minimum fields)

- `tenant_id`
- `user_id`
- `role`
- `session_id`
- `module_id`
- `simulation_id`
- `simulation_step`
- `event_type`
- `event_timestamp`
- `attempt_count`
- `error_code` (if applicable)

---

## 7. Data Model (AI domain)

Create these tables/collections with tenant scoping:

- `ai_sessions`
- `ai_messages`
- `ai_context_events`
- `ai_retrieval_citations`
- `ai_policy_evaluations`
- `ai_safety_flags`
- `ai_escalation_cases`
- `ai_escalation_actions`

Mandatory attributes:

- `tenant_id` on every record.
- immutable `created_at`.
- actor identity (`user_id` or service principal).
- retention class label.

Retention policy:

- AI chats/transcripts: 2 years.
- Escalation and compliance audit records: follow platform legal retention policy.

Deletion model:

- Soft-delete only.
- Full audit trail for create/update/delete actions.

---

## 8. Knowledge and RAG Pipeline

### 8.1 Approved corpus (v1)

- Tau handbook behavior and pedagogy rules.
- Foundation curriculum (`FND-TT-01`, `FND-IW-01` only).
- Simulation rubric/pass logic linked to current module/step.
- Approved safety and policy snippets for refusal/escalation templates.
- Partner-specific content overlays only when explicitly mapped to foundation curriculum outcomes.

### 8.2 Ingestion requirements

- Chunk by semantic section and module/unit metadata.
- Attach tags: `module`, `unit`, `assessment_type`, `safety_level`, `audience`.
- Version content snapshots for traceable updates.
- Block unapproved sources by default.
- Reject ingestion of partner content that does not include core-curriculum outcome mapping metadata.

### 8.3 Response requirements

- Return citation metadata for every instructional claim in critical contexts.
- If retrieval confidence is low:
  - avoid definitive claim
  - provide safe fallback
  - recommend human guidance path

---

## 9. Guardrails and Safety Engine

### 9.1 Pre-generation checks

- Detect grading-cheat intent.
- Detect unsafe procedural shortcuts.
- Detect distress language.
- Detect out-of-scope requests.

### 9.2 Post-generation checks

- No direct graded answer leakage.
- Safety language must be explicit (no hedged unsafe approval).
- Must include escalation cue when required.
- Must remain plain-language and QTB-aligned.

### 9.3 Escalation policy

Auto-create escalation case when:

- Distress signals meet threshold.
- Real-world safety conflict appears high-risk.
- Repeated learner stuck pattern exceeds configured limit.

SLA behavior:

- Escalation case timer starts at case creation.
- Admin notification fanout via email, SMS, push.
- Breach alert at `T+12 min`; SLA breach at `T+15 min`.

### 9.4 Human Approval Matrix (Non-Negotiable)

| Action Type | AI Permission | Required Human Approval |
|---|---|---|
| Graded assessment answer or marking decision | Not allowed | Lecturer |
| Learner progression override | Draft recommendation only | Lecturer/Admin |
| Compliance or policy exception | Draft recommendation only | Admin |
| Certification readiness sign-off | Draft recommendation only | Admin |
| KPI scorecard target changes during pilot | Draft recommendation only | Admin + Program Sponsor |
| Commercial pricing/discount or contract terms | Not allowed | Procurement/Commercial Owner |
| Sensitive data export outside default scope | Not allowed | Admin + Compliance Owner |

---

## 10. Frontend Integration Requirements

- One routed app state model (no fragmented standalone behavior).
- Session-aware AI panel embedded in:
  - module pages
  - simulation surfaces
- Role-aware AI controls for:
  - learner workflows
  - lecturer intervention workflows
  - admin escalation and approval queue workflows
- Demo-aware AI controls for:
  - prospect walkthrough prompts
  - guardrail-safe transitions between learner/instructor/admin demo views
- Realtime UI states:
  - connecting
  - active
  - degraded/fallback to text
  - disconnected/retry
- Show safety disclaimer in simulation context.
- Provide user controls:
  - mute
  - captions
  - device switch
  - end session

---

## 11. Security and Compliance Baseline

Use these baseline controls:

- RBAC with least privilege.
- Tenant isolation at DB layer with RLS and automated isolation tests.
- Encryption in transit and at rest.
- Signed short-lived access tokens for realtime and media endpoints.
- Secret management with rotation.
- Immutable audit logging for policy and escalation actions.
- Security testing gates: SAST, DAST, dependency scanning.

Incident response baseline:

- On cross-tenant exposure suspicion:
  - lock affected access paths immediately
  - preserve logs
  - notify admin owner
  - run incident workflow and containment.

---

## 12. Performance and Reliability Targets

Text mode targets (proposed baseline):

- p95 first response token: <= 2.0 s
- p95 full response: <= 8.0 s

Realtime mode targets (proposed baseline):

- conversational turn latency target: <= 1.5 s perceived
- graceful fallback to text mode on degradation

Platform reliability:

- Escalation delivery reliability >= 99.9%
- transcript write success >= 99.9%

Note:

- Final thresholds should be validated against pilot hardware and finalized in QA gate criteria.

---

## 13. QA and Acceptance Plan

### 13.1 Test packs

- Curriculum accuracy tests by module/unit.
- Safety refusal tests for unsafe requests.
- Anti-cheating tests for graded prompts.
- Distress detection and escalation timing tests.
- Realtime stability tests (network jitter, mic/camera failures).
- Cross-tenant isolation tests.
- Transcript retention and soft-delete audit tests.
- Approval matrix enforcement tests for all blocked and draft-only action classes.
- Core-curriculum invariance tests (partner configuration cannot disable or bypass mandatory foundation units).
- Demo data isolation tests (prospect sessions cannot query or infer live tenant-specific records).
- Guided demo completion tests (prospect can traverse full platform storyline from core curriculum entry point).

### 13.2 Release acceptance gates

- 100% pass for critical safety policy scenarios.
- 0 tolerance for direct graded answer leakage.
- Escalation workflow meets 15-minute SLA in staging drills.
- Full learner flow pass:
  - module context -> simulation attempt -> AI help -> escalation (if triggered) -> admin action trace.

---

## 14. Delivery Plan (Execution)

### Phase 0: Foundations (1-2 weeks)

- Service scaffolding, data schema, tenant isolation, audit model.

### Phase 1: Text + RAG + policy core (2-3 weeks)

- Text coach, context ingestion, guardrails, citations, transcript logging.

### Phase 2: Guided media + simulation context hardening (1-2 weeks)

- Deep linking to approved media and simulation state-aware prompts.

### Phase 3: Live conversational mode (2-4 weeks)

- Realtime ASR/TTS/avatar integration (NVIDIA-first option).
- Fallback and resilience controls.

### Phase 4: Safety operations + pilot hardening (1-2 weeks)

- Admin escalation console, SLA monitoring, incident drills, load/perf tuning.

### Phase 5: Role governance and conversion evidence (1 week)

- Enable package-specific role activation flags.
- Validate approval-matrix behavior under pilot and annual-plan settings.
- Produce pilot decision evidence pack with traceable approval logs.

---

## 15. Runbooks Required Before Pilot

- AI safety incident runbook.
- Escalation triage runbook (15-minute SLA).
- Realtime service degradation/fallback runbook.
- Model/policy rollback runbook.
- Previous-release redeploy runbook (aligned with platform rollback decision).

---

## 16. Build-Blocking Open Items

These are still required as concrete implementation values:

- Exact "latest browser" certification policy (how often validated).
- Concrete mid-range desktop hardware profile for agent realtime QA.
- Final vendor and hosting topology for NVIDIA services in production.

---

## 17. Definition of Done (AI Agent v1)

AI Agent v1 is done when:

- All required AI modes are live (text, guided media, true live conversational video).
- Policy engine enforces no-cheat and safety rules consistently.
- Escalation workflow meets 15-minute SLA in production-like tests.
- Curriculum-grounded responses are traceable and citation-backed.
- Security, audit, retention, and tenant isolation requirements pass sign-off.
- Human approval matrix is enforced with audited approvals for all high-impact actions.
- Core curriculum baseline enforcement passes for every enabled partner/package configuration.
- Full user flow works for demo and pilot audience with acceptable performance.
- Prospect demo flow runs end-to-end from core curriculum entry with approved guardrails and no live tenant data exposure.

---

## 18. Pilot KPI and Decision Evidence Contract

The AI layer must feed pilot-to-annual decision evidence with traceable provenance.

Required outputs:
- Weekly KPI support signals (completion trend, intervention volume, risk alerts, escalation SLA adherence).
- Weekly core-curriculum coverage signal (baseline units started/completed by partner cohort).
- Role-specific activity logs (what AI suggested, what human approved, what executed).
- Pilot closeout evidence bundle with baseline vs current vs target KPI snapshots.
- Demo funnel signal (started demo, completed walkthrough, requested pilot scoping).

Non-negotiable rule:
- AI-generated KPI narrative is advisory; the final KPI decision record is human-authored and signed.
