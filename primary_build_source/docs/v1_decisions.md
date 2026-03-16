# Myelin v1 Decisions

Document status: Locked decisions for v1 execution
Date: March 12, 2026
Decision owner: Sethu (Founder, Myelin)
Build partner: GPT-5.3-Codex (AI engineering copilot)
Audience: Sethu and GPT-5.3-Codex

---

## 1. Purpose

This document records confirmed v1 decisions from product direction discussions.
It is the decision baseline for implementation and release readiness.

### 1.1 Execution model (locked)

- Single human owner for decisions and approvals: Sethu
- Single AI implementation partner for build execution: GPT-5.3-Codex

---

## 2. v1 Product Goal

v1 must support:

- Investor demos with a chosen testing audience.
- Full end-to-end experience of the core curriculum.
- Full simulation experience.
- Full AI engagement experience.

---

## 3. Scope Boundary (Locked)

- v1 learning scope is only:
  - `FND-TT-01`
  - `FND-IW-01`
- No extra modules are included in v1.
- Build experience is one routed application (not standalone module pages).

---

## 4. Simulation Decisions (Locked)

- Final simulation source of truth:
  - `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md`
- Focus is the core curriculum only.
- QTB input is mandatory in every assessed simulation step.
- Wrong-order actions must:
  - block progression
  - warn the learner
- Pass/fail standard for modules and simulations: `80%`.
- Retry rule:
  - learner gets 3 retries
  - after 3 failed retries, escalate to lecturer engagement
  - lecturer can allow additional retry
- Sequential Tab-based completion of the 3D canvas is not required.
- Keyboard access is required for all core workflows, simulator HUD controls, and documented simulator shortcuts.
- Device support for full simulator:
  - desktops
  - laptops
  - Macs
- Browser support:
  - latest Chrome
  - latest Edge
  - latest Firefox
  - latest Safari (Mac)
- FPS and performance thresholds follow simulation docs.

---

## 5. AI Coach Decisions (Locked)

- Required at pilot launch in v1:
  - text mode
  - guided media mode
- Live conversational video mode remains in v1 scope but is feature-flagged and not pilot-launch blocking.
- Live conversational video mode may be enabled only for approved tenants after separate safety, latency, transcript, and operational readiness sign-off.
- AI must never provide direct answers to graded questions before submission.
- If learner asks unsafe real-world guidance:
  - AI must advise against unsafe behavior
  - AI must instruct learner to seek professional/supervisor guidance
- Escalation owner: admin.
- Escalation SLA for safety/distress: within 15 minutes.
- AI chat log retention: 2 years.

### 5.1 AI Runtime Direction

- Preferred direction: NVIDIA-based live conversational stack.
- Technical implementation options will be selected in engineering design with NVIDIA-first preference.

---

## 6. Learning and Assessment Decisions (Locked)

- All assessments are mandatory for module completion.
- Learner is blocked from progressing if module activities and assessments are incomplete.
- Scores are visible to:
  - learners
  - lecturers
  - admins
- Evidence required to issue results:
  - completion of module activities
  - completion of assessments

---

## 7. Data, Privacy, and Security Decisions (Locked)

- Signup fields:
  - all fields currently defined in reference files are required.
- POPIA consent is required at signup.
- If consent is revoked later:
  - learner loses platform access.
- Data retention:
  - all data retained for 10 years.
- Deletion policy:
  - admin-only deletion permissions
  - soft-delete only (recoverable + audited), no hard delete by default
- Tenant isolation and security controls:
  - apply industry-standard controls and validation baselines.
- Cross-tenant exposure response:
  - admin locks access immediately.

---

## 8. Uploads, Files, and Media Decisions (Locked)

- Allowed submission type: MP4 only.
- WebM is not accepted in v1 unless this document is explicitly amended.
- Upload cap: fixed 500 MB.
- Upload/scan/transcode failure handling:
  - notify admin
  - give learner alternative submission path
- Target review readiness for uploaded videos: 2 to 5 hours.
- Low-bandwidth behavior:
  - learner can submit without losing progress.
- Re-submission versioning: required.
- Storage direction for v1: cloud object storage.

---

## 9. Notifications and Communications Decisions (Locked)

- v1 pilot-launch channels:
  - email
  - SMS
- Push notifications are deferred until after pilot launch as a feature-flagged extension.
- Required v1 triggers:
  - signup welcome
  - course added to profile
  - module complete
  - submission approved
  - submission under lecturer review
  - lecturer feedback provided
- College-level template editing: not allowed.
- Quiet hours: not required in v1.
- User channel opt-out:
  - managed in profile settings.
- Retry window target for failed sends:
  - 5 to 10 minutes.
- Delivery metrics and retry/failure behavior:
  - follow industry-standard operational baselines.

---

## 10. Frontend, UX, and Brand Decisions (Locked)

- Final product name: `Myelin`.
- Remove `ITS Construct` branding from login, registration, and email surfaces.
- Technical theme storage key: `myelin-theme`.
- Language in v1: English only.
- Accessibility release bar for v1: WCAG 2.2 Level A minimum.
- Accessibility target for all core surfaces: WCAG 2.2 AA.
- Government/ministry-grade deployment requires WCAG 2.2 AA sign-off.

---

## 11. Performance, QA, and Release Decisions (Locked)

- First simulator interaction load target: 5 to 10 seconds.
- Browser support matrix: latest supported versions only.
- Launch approval evidence:
  - full end-to-end user flow validation.
- Mid-range desktop definition:
  - use industry-standard baseline.
- Release blockers vs acceptable known issues:
  - follow industry-standard release governance.
- Final go/no-go owner: Sethu.
- Rollback strategy if pilot fails:
  - previous release redeploy.

---

## 12. Operational Note

Where this document says "industry-standard," architecture and security design documents must define exact technical controls, SLAs, and test evidence before release gate sign-off.
