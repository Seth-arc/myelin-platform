# OpenClaw Mapping: Claw-like Playbook Capabilities -> Myelin v1 Blueprint

Date: 2026-03-17
Purpose: one-page mapping to decide whether to formalize OpenClaw as a named architecture layer in Myelin.

## Decision Summary

Recommendation: formalize as a named logical layer only (for governance and communication), not as an external dependency.

Suggested name: OpenClaw Layer (internal) = AI Orchestrator + Policy Guardrails + Human Approval + Escalation + Audit Evidence.

Rationale:
- Playbook already uses "Claw-like" as the operating model label.
- v1 blueprint already defines all required primitives as native Myelin services and APIs.
- Naming the layer improves consistency across GTM, product, and delivery without changing runtime architecture.

## Capability Mapping (Playbook -> v1 Exact Services/Endpoints)

| Claw-like playbook capability | v1 service/component mapping | Exact v1 endpoint(s) | v1 coverage status |
|---|---|---|---|
| Market Scout (signal ingest, scoring, watchlists) | AI Orchestrator: Session Manager + Context Builder + Retrieval Orchestrator + policy pre-check | POST /ai/sessions; POST /ai/context/events; POST /ai/messages | Partial in v1: event and response pipeline exists; external procurement-feed connectors are out of current v1 core scope |
| Account Mapper (stakeholder graph, influence map) | Context Builder + RAG context enrichment + policy logging | POST /ai/context/events; POST /ai/messages; GET /ai/sessions/:id/transcript | Partial in v1: contextual reasoning supported; CRM/org-graph integrations are implementation extensions |
| Outreach Composer (message packs, channel variants) | Prompt Builder + Policy Guardrails (pre and post) + Audit Logger + approval workflow | POST /ai/messages; POST /ai/approvals; GET /ai/approval-matrix; GET /ai/sessions/:id/transcript | Strongly covered for draft + approval-gated behavior; external send channels are not first-class AI endpoints in v1 |
| Pilot Orchestrator (milestone drift, risk ledger, status digests) | Session + Context Events + Escalation Engine + safety flags and audit | POST /ai/context/events; POST /ai/escalations; GET /ai/escalations/:id; GET /ai/sessions/:id/transcript | Strongly covered for monitoring/escalation traceability; project-plan integrations are implementation details |
| Compliance Sentinel (go-live gate, policy checks, pass/conditional/fail) | Policy Guardrails + Human Approval Matrix + immutable audit | GET /ai/approval-matrix; POST /ai/approvals; GET /ai/sessions/:id/transcript | Strongly covered in v1 non-negotiables (draft-only exceptions, human approvals, audit trail) |
| Expansion Analyst (health score, churn risk, expansion hypotheses) | Context/event ingestion + orchestrated AI analysis + approval-gated high-impact actions | POST /ai/context/events; POST /ai/messages; POST /ai/approvals | Partial in v1: analytic recommendations supported; commercial execution remains human-approved by design |
| Founder Control Tower (daily briefs, approval queue, exception handling) | Approval matrix + escalation queue + transcript/policy trace + role-aware controls | POST /ai/approvals; GET /ai/approval-matrix; POST /ai/escalations; GET /ai/escalations/:id; GET /ai/sessions/:id/transcript | Strongly covered for governance primitives; dashboard UX is a frontend implementation concern |
| Permissioned autonomy levels (L0-L4 style progression) | Role/package activation boundaries + policy corridors + approval matrix enforcement | GET /ai/approval-matrix; POST /ai/approvals; POST /ai/demo/sessions | Covered in principle: v1 locks high-impact actions behind approvals; progressive autonomy is policy/config rollout |
| Safety and hard-stop failure defaults | Pre/post checks, refusal policies, escalation SLA, audit immutability | POST /ai/messages; POST /ai/escalations; GET /ai/escalations/:id | Strongly covered: anti-cheat, safety refusal, distress escalation (15-minute SLA target), immutable logging |
| Evidence-first operations (artifact required for each action) | Audit Logger + transcript policy trace + citations + AI domain records | GET /ai/sessions/:id/transcript; POST /ai/messages (with policy result and citations) | Strongly covered in v1 contracts and QA gates |

## Canonical v1 Building Blocks Behind the Mapping

- AI Orchestrator service: Session Manager, Context Builder, Policy Guardrails (pre and post), Prompt Builder, Retrieval Orchestrator, Escalation Engine, Audit Logger.
- Core AI APIs: sessions, messages, context events, realtime token, escalations, transcript retrieval, approvals, approval matrix, demo sessions.
- AI data domain: ai_sessions, ai_messages, ai_context_events, ai_retrieval_citations, ai_policy_evaluations, ai_safety_flags, ai_escalation_cases, ai_escalation_actions.
- Non-negotiables: tenant_id on all records, immutable audit timestamps, soft-delete only, transcript retention class, and enforced human approval matrix for high-impact actions.

## Formalization Decision Test

Formalize OpenClaw as a named internal architecture layer if all are true:
- You want one term that aligns playbook ops language with product engineering contracts.
- You do not intend to import a third-party OpenClaw runtime into v1.
- You want governance scope clarity (autonomy boundaries, approvals, escalation, audit) across teams.

Do not formalize as a separate runtime if:
- It duplicates existing AI Orchestrator responsibilities.
- It introduces an additional policy engine that could conflict with current guardrails.

## Practical next step

Add one line to v1 architecture docs:
- "OpenClaw Layer (internal term): the governance and orchestration layer implemented by Myelin AI Orchestrator, policy guardrails, approval matrix, escalation engine, and audit evidence services."