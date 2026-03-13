# Myelin AI Agent v1 Implementation Plan

Document status: Execution plan  
Date: March 12, 2026  
Plan owner: Sethu (Founder, Myelin; final go/no-go owner)  
Build partner: GPT-5.3-Codex (AI engineering copilot)

---

## 1. Plan Purpose

Convert the AI blueprint into an execution-ready plan with clear owners and weekly milestones for:

- Demo mode using NVIDIA Developer API key
- Production migration path using NVIDIA AI Enterprise

Execution model for this plan:

- One human owner: Sethu
- One AI build partner: GPT-5.3-Codex

This plan is aligned with:

- `primary_build_source/docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md`
- `primary_build_source/docs/v1_decisions.md`
- `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`

---

## 2. Locked v1 Delivery Scope

- Curriculum scope: `FND-TT-01` and `FND-IW-01` only
- Required AI modes in v1:
  - Text
  - Guided media
  - True live conversational video
- No direct graded answers before submission
- Safety/distress escalation owner: admin
- Escalation SLA: 15 minutes
- AI chat retention: 2 years

---

## 3. Task Ownership (Two-Party Model)

| Workstream | Accountable Owner | Build Partner | Key Deliverables |
| --- | --- | --- | --- |
| Product and scope control | Sethu | GPT-5.3-Codex | Scope control, acceptance gates, go/no-go |
| Architecture and technical direction | Sethu | GPT-5.3-Codex | Final service architecture, design sign-off |
| AI orchestration and policy engine | Sethu | GPT-5.3-Codex | Orchestrator, guardrails, policy enforcement |
| Backend platform and APIs | Sethu | GPT-5.3-Codex | Session/message/context/escalation APIs |
| RAG and curriculum grounding | Sethu | GPT-5.3-Codex | Indexed knowledge base, citation-ready retrieval |
| Realtime voice/video integration | Sethu | GPT-5.3-Codex | Live conversational channel, failover to text |
| Frontend AI surfaces | Sethu | GPT-5.3-Codex | In-app AI panel, guided media, session controls |
| Safety operations and escalation | Sethu | GPT-5.3-Codex | Escalation queue, SLA timers, runbooks |
| Security and compliance | Sethu | GPT-5.3-Codex | Tenant isolation tests, audit logging, secrets |
| QA and release validation | Sethu | GPT-5.3-Codex | Test packs, gate evidence, release report |
| Infrastructure and deployment | Sethu | GPT-5.3-Codex | Environments, monitoring, rollback readiness |

---

## 4. Week-by-Week Milestones

### Phase A: Demo Build (NVIDIA Developer API Key)

| Week | Milestone | Owners | Exit Criteria |
| --- | --- | --- | --- |
| Week 1 | Kickoff and baseline freeze | Sethu + GPT-5.3-Codex | Scope and non-negotiables signed; backlog approved |
| Week 2 | Foundation setup | Sethu + GPT-5.3-Codex | AI service scaffolded; tenant-aware schema created; auth and audit base in place |
| Week 3 | Text AI and RAG baseline | Sethu + GPT-5.3-Codex | Text mode working with approved corpus and citations |
| Week 4 | Policy and safety guardrails | Sethu + GPT-5.3-Codex | No-answer-to-graded enforcement passes; unsafe request refusal passes |
| Week 5 | Guided media and simulation context | Sethu + GPT-5.3-Codex | AI responses adapt to module and simulation step; approved media linking active |
| Week 6 | Realtime conversational MVP | Sethu + GPT-5.3-Codex | Live voice interaction active in staging; text fallback works |
| Week 7 | Live video mode and resilience | Sethu + GPT-5.3-Codex | Video/avatar surface integrated; reconnect/degraded states tested |
| Week 8 | Escalation operations and demo hardening | Sethu + GPT-5.3-Codex | 15-minute SLA drills pass; transcripts and audit traces complete; demo flow passes end-to-end |

### Phase B: Production Migration (NVIDIA AI Enterprise)

| Week | Milestone | Owners | Exit Criteria |
| --- | --- | --- | --- |
| Week 9 | Production target architecture lock | Sethu + GPT-5.3-Codex | AI Enterprise target topology approved; migration runbook signed |
| Week 10 | Stage deployment on AI Enterprise stack | Sethu + GPT-5.3-Codex | Stage stack live with production-like configs and monitoring |
| Week 11 | Compatibility and performance qualification | Sethu + GPT-5.3-Codex | Functional parity confirmed; latency/reliability targets met |
| Week 12 | Cutover rehearsal and release readiness | Sethu + GPT-5.3-Codex | Rehearsed cutover complete; previous-release redeploy tested; go/no-go decision made |

---

## 5. Detailed Task Breakdown by Stream

### 5.1 AI Orchestrator and Guardrails

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Implement session lifecycle (`/ai/sessions`, `/ai/messages`)
- Build pre-check and post-check policy pipeline
- Enforce graded-answer refusal
- Enforce unsafe guidance refusal and professional-help redirect
- Emit policy decision logs for audit

### 5.2 Backend APIs and Data Model

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Implement context and escalation endpoints
- Create AI domain tables with strict `tenant_id` scoping
- Add immutable audit timestamps and actor IDs
- Implement soft-delete model (recoverable + audited)
- Enforce 2-year chat retention job policy

### 5.3 RAG and Curriculum Controls

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Ingest approved v1 corpus only (`FND-TT-01`, `FND-IW-01`)
- Tag by module/unit/assessment/safety metadata
- Block non-approved sources
- Return citations for instructional claims

### 5.4 Realtime Voice and Video

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Build realtime token service and client handshake
- Integrate ASR/TTS path for conversational turns
- Integrate video/avatar rendering path
- Implement graceful degradation to text-only mode

### 5.5 Frontend Integration

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Embed AI panel in routed app pages
- Add guided media panel and state-aware prompts
- Implement realtime states: connecting, active, degraded, disconnected
- Add controls: mute, captions, device switch, end session

### 5.6 Safety Escalation and Operations

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Build escalation case queue and status model
- Configure admin notifications (email, SMS, push)
- Add SLA timers and breach alerts (`T+12`, `T+15`)
- Publish triage runbook and handoff SOP

### 5.7 Security, Compliance, and Isolation

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Enforce least-privilege RBAC and scoped service access
- Validate tenant isolation via automated tests
- Ensure encryption in transit and at rest
- Validate audit immutability and secret rotation

### 5.8 QA and Gate Validation

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Run safety/anti-cheat/distress test packs
- Run realtime stability and fallback tests
- Run cross-tenant isolation tests
- Certify full learner-to-admin flow for launch evidence

---

## 6. Milestone Gate Criteria

| Gate | Required Evidence | Owner |
| --- | --- | --- |
| Gate A (end Week 4) | Text AI + policy core stable in staging | Sethu + GPT-5.3-Codex |
| Gate B (end Week 8) | Demo mode end-to-end pass with escalation SLA pass | Sethu + GPT-5.3-Codex |
| Gate C (end Week 11) | AI Enterprise stage parity + performance pass | Sethu + GPT-5.3-Codex |
| Gate D (end Week 12) | Cutover rehearsal and rollback test pass | Sethu + GPT-5.3-Codex |

---

## 7. Production Migration Checklist (AI Enterprise)

Accountable owner: Sethu  
Build partner: GPT-5.3-Codex

- Provision production-grade AI Enterprise environment
- Configure network, IAM, secret stores, and observability
- Deploy equivalent model/runtime interfaces used in demo
- Validate API contract parity against demo integration
- Migrate staging traffic and run load/performance tests
- Run security and isolation checks in production-like settings
- Execute cutover rehearsal
- Confirm previous-release redeploy rollback path
- Obtain final go/no-go approval from Sethu

---

## 8. Risks and Mitigations

| Risk | Impact | Owner | Mitigation |
| --- | --- | --- | --- |
| Realtime latency too high for natural conversation | Poor learner experience | Sethu + GPT-5.3-Codex | Aggressive fallback to text; network and media profiling |
| Policy false negatives (unsafe or cheating leakage) | Safety/compliance breach | Sethu + GPT-5.3-Codex | Expand adversarial tests; stricter post-generation filters |
| Cross-tenant data leakage | Critical security incident | Sethu + GPT-5.3-Codex | Mandatory isolation tests in CI and pre-release gates |
| Escalation SLA misses | Operational failure | Sethu + GPT-5.3-Codex | Timer alerts, on-call coverage, drill-based readiness |
| Vendor migration drift from demo to production stack | Delayed launch | Sethu + GPT-5.3-Codex | Contract-parity tests and staged cutover rehearsal |

---

## 9. Definition of Done

The implementation plan is complete when:

- All Week 1-12 milestones are closed with evidence
- Required AI modes are live and validated
- Safety and anti-cheat policies pass critical tests
- Escalation SLA meets 15-minute requirement in staging drills
- Tenant isolation, audit, retention, and rollback checks pass
- Sethu signs final go/no-go
