# Myelin Canonical Decisions To Lock Before Build

Document status: Proposed canonical pre-build lock  
Date: March 15, 2026  
Owner: Sethu (decision owner)  
Audience: Sethu and implementation partners

---

## 1. Purpose

This document resolves the four build-blocking ambiguities identified while reviewing `MYELIN_VIBE_CODING_DEVELOPMENT_GUIDE.md` against the source package:

1. AI mode scope in v1
2. Upload file format contract
3. Accessibility and keyboard interaction baseline
4. Notifications and GraphQL scope for the initial build

The goal is not to rewrite the whole spec set. The goal is to establish one canonical interpretation that implementation can follow immediately, then list the exact edits needed in the conflicting source files.

---

## 2. Canonical Decisions Summary

| Area | Canonical decision to lock now | Implementation effect |
|---|---|---|
| AI modes | Pilot-launch v1 must support `TEXT` and `GUIDED_MEDIA`. `LIVE_CONVERSATIONAL` remains in v1 scope but is feature-flagged and not pilot-launch blocking. | Build text and guided media first. Keep live conversational behind tenant flags and readiness gates. |
| Upload format | Learner evidence upload is `MP4` only in v1. | Restrict frontend accept lists, backend validation, malware scanning, and transcode assumptions to MP4. |
| Accessibility | v1 release gate is WCAG 2.2 Level A minimum. WCAG 2.2 AA is the target for all core surfaces and required before government/ministry-grade deployment. Keyboard access is required for all core workflows and simulator HUD controls, but sequential Tab-based control of the 3D canvas is not required. | Align QA gates, HUD design, and simulator shortcut behavior. |
| Notifications | v1 pilot launch supports `email` and `SMS`. Push is deferred until after pilot launch as a feature-flagged extension. | Do not build push infrastructure into the bootstrap milestone. |
| GraphQL | GraphQL is not a bootstrap or pilot-launch blocker. Initial build is REST-first. GraphQL may be added later after the domain model and dashboard query patterns stabilize. | Keep the first API surface simple and avoid schema/resolver overhead in the first build wave. |

---

## 3. Detailed Decisions And Exact Source Edits

### D1. AI Mode Scope In v1

#### Canonical decision

- `TEXT` and `GUIDED_MEDIA` are required for pilot-launch v1.
- `LIVE_CONVERSATIONAL` remains part of the broader v1 codebase direction, but it is feature-flagged and not pilot-launch blocking.
- `LIVE_CONVERSATIONAL` may only be enabled for approved tenants after separate safety, latency, transcript, and operational readiness sign-off.

#### Why this is the right lock

- It preserves the locked NVIDIA-first direction and the three-mode architecture without making realtime voice/video a day-one blocker.
- It aligns with the PRD's "text-first production" wording.
- It stays compatible with the existing feature-flag model in the agent-response skill.
- It lets the build proceed with a clean phased implementation order:
  1. `TEXT`
  2. `GUIDED_MEDIA`
  3. `LIVE_CONVERSATIONAL`

#### Exact edits to apply

1. File: `docs/v1_decisions.md`

Search for:

```md
- Required in v1:
  - text mode
  - guided media mode
  - true live conversational video mode
```

Replace with:

```md
- Required at pilot launch in v1:
  - text mode
  - guided media mode
- Live conversational video mode remains in v1 scope but is feature-flagged and not pilot-launch blocking.
- Live conversational video mode may be enabled only for approved tenants after separate safety, latency, transcript, and operational readiness sign-off.
```

2. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Search for:

```md
- AI learning coach (text-first production, guided media and controlled video mode).
```

Replace with:

```md
- AI learning coach (text mode and guided media required for v1 launch; live conversational video available as a controlled feature-flagged rollout after readiness sign-off).
```

3. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Search for:

```md
3. Confirm required level of AI video mode in v1 (full conversational video vs guided media mode).
```

Replace with:

```md
3. AI video scope for v1 is locked: live conversational mode is feature-flagged and not pilot-launch blocking.
```

4. File: `docs/MYELIN_PROFESSIONAL_FEATURES.md`

Search for:

```md
| AI-002 | Guided media mode | Curated visual explanations and contextual references. | Student | P1 | v1.1 |
| AI-003 | Advanced video interaction mode | Extended conversational/video coaching experience. | Student | P2 | v2 |
```

Replace with:

```md
| AI-002 | Guided media mode | Curated visual explanations and contextual references. | Student | P0 | v1 |
| AI-003 | Live conversational mode | Feature-flagged voice + avatar/video coaching for approved tenants after readiness sign-off. | Student | P1 | v1 |
```

5. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Insert this row after `AI-005` in the Learning Coach requirements table:

```md
| AI-007 | Live conversational mode may be enabled by feature flag for approved tenants after separate safety, latency, transcript, and operational readiness sign-off. |
```

6. File: `docs/skills/3/mnt/user-data/outputs/agent-response-modes/SKILL.md`

Search for:

```md
| Live conversational | `LIVE_CONVERSATIONAL` | Feature-flagged, Operational/National tier only | Voice + avatar/video surface; real-time interaction |
```

Replace with:

```md
| Live conversational | `LIVE_CONVERSATIONAL` | Feature-flagged, Operational/National tier only; not pilot-launch blocking | Voice + avatar/video surface; real-time interaction after readiness sign-off |
```

7. File: `docs/skills/3/mnt/user-data/outputs/agent-response-modes/SKILL.md`

Search for:

```md
Voice-enabled interaction with an avatar/video surface. This mode is a controlled feature — not available at Pilot tier by default.
```

Replace with:

```md
Voice-enabled interaction with an avatar/video surface. This mode is a controlled v1 feature — not pilot-launch blocking and not available at Pilot tier by default.
```

---

### D2. Upload File Format Contract

#### Canonical decision

- Learner practical evidence upload is `MP4` only in v1.
- `WEBM` is out of scope unless explicitly reapproved through a later change.
- The backend may transcode or normalize internal derivatives as needed, but the accepted learner submission format is still `MP4`.

#### Why this is the right lock

- It matches the locked v1 decisions file.
- It reduces frontend validation complexity.
- It simplifies malware scanning, media review, and transcode assumptions.
- It removes ambiguity from the prototype copy and data model.

#### Exact edits to apply

1. File: `docs/MYELIN_PROFESSIONAL_FEATURES.md`

Search for:

```md
  - Upload formats include MP4 and WebM.
```

Replace with:

```md
  - Upload format is MP4 only in v1.
```

2. File: `docs/skills/2/mnt/user-data/outputs/data-model-entities/SKILL.md`

Search for:

```md
- `fileType` enum: MP4 | WEBM
```

Replace with:

```md
- `fileType` enum: MP4
```

3. File: `references/ui_baseline/ai-study-buddy-mockup.html`

Search for:

```html
<p style="font-size: 0.85rem; color: var(--text-muted);">Drag and drop MP4 or WebM files
                                here, or click to browse. Maximum file size: 500MB.</p>
```

Replace with:

```html
<p style="font-size: 0.85rem; color: var(--text-muted);">Drag and drop MP4 files
                                here, or click to browse. Maximum file size: 500MB.</p>
```

4. File: `docs/v1_decisions.md`

No content change required. Keep this line as the canonical source:

```md
- Allowed submission type: MP4 only.
```

Add this note directly below it:

```md
- WebM is not accepted in v1 unless this document is explicitly amended.
```

---

### D3. Accessibility And Keyboard Interaction Baseline

#### Canonical decision

- v1 release gate is WCAG 2.2 Level A minimum.
- WCAG 2.2 AA is the target for all core product surfaces and is required before government/ministry-grade deployment.
- Keyboard access is required for all core workflows, simulator HUD controls, and documented simulator shortcuts.
- Sequential Tab-based control of the 3D canvas is not required.
- The existing `keyboard-only completion requirement: not required` wording should be replaced because it is too easy to misread as "keyboard support optional."

#### Why this is the right lock

- It aligns the PRD, UI guide, release gate, and accessibility skill into one practical release bar.
- It preserves the simulator interaction model already described in the accessibility skill.
- It prevents a team from interpreting keyboard support too narrowly.

#### Exact edits to apply

1. File: `docs/v1_decisions.md`

Search for:

```md
- Keyboard-only completion requirement: not required.
```

Replace with:

```md
- Sequential Tab-based completion of the 3D canvas is not required.
- Keyboard access is required for all core workflows, simulator HUD controls, and documented simulator shortcuts.
```

2. File: `docs/v1_decisions.md`

Search for:

```md
- Accessibility baseline: WCAG 2.1 AA.
```

Replace with:

```md
- Accessibility release bar for v1: WCAG 2.2 Level A minimum.
- Accessibility target for all core surfaces: WCAG 2.2 AA.
- Government/ministry-grade deployment requires WCAG 2.2 AA sign-off.
```

3. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Search for:

```md
- WCAG 2.2 AA baseline for web application surfaces.
- Keyboard navigation for all core workflows.
- Captions support for video content.
- Reduced motion mode for intense animation contexts.
```

Replace with:

```md
- WCAG 2.2 Level A is the minimum v1 release gate for web application surfaces.
- WCAG 2.2 AA is the target for all core surfaces and required before government/ministry-grade deployment.
- Keyboard access is required for all core workflows, simulator HUD controls, and documented simulator shortcuts.
- Captions support for video content.
- Reduced motion mode for intense animation contexts.
```

4. File: `docs/MYELIN_UI_UX_STYLING_GUIDE.md`

Search for:

```md
- Text and interactive elements should meet WCAG 2.2 AA contrast ratios.
```

Replace with:

```md
- Text and interactive elements must meet WCAG 2.2 Level A requirements in v1 and should meet WCAG 2.2 AA contrast targets on all core surfaces.
```

5. File: `docs/skills/4/mnt/user-data/outputs/release-gate-checklist/SKILL.md`

Search for:

```md
- Advisory gate for v1: no WCAG 2.2 Level A violations (Level AA is the target milestone for v2)
```

Replace with:

```md
- Advisory gate for v1: zero WCAG 2.2 Level A violations.
- WCAG 2.2 AA remains the target for all core surfaces and becomes mandatory before government/ministry-grade deployment.
```

---

### D4. Notifications And GraphQL Scope For The Initial Build

#### Canonical decision

Notifications:

- v1 pilot launch supports `email` and `SMS`.
- Push is deferred until after pilot launch as a feature-flagged extension.
- Any admin escalation flow in v1 must work through email and SMS without requiring push infrastructure.

GraphQL:

- GraphQL is not a bootstrap or pilot-launch blocker.
- The initial build is REST-first.
- GraphQL may be added later if complex dashboard or mobile query needs justify it after the REST domain model stabilizes.

#### Why this is the right lock

- It aligns the first implementation wave with the recommended stack document.
- It reduces backend and mobile-query complexity during bootstrap.
- It keeps notification delivery focused on the channels already reflected in the main PRD and features document.

#### Exact edits to apply

1. File: `docs/v1_decisions.md`

Search for:

```md
- v1 channels:
  - email
  - SMS
  - push
```

Replace with:

```md
- v1 pilot-launch channels:
  - email
  - SMS
- Push notifications are deferred until after pilot launch as a feature-flagged extension.
```

2. File: `references/product/Product Decisions.md`

Search for:

```md
| **GraphQL** | In scope for v1 (for complex/mobile queries). |
```

Replace with:

```md
| **GraphQL** | Not required for bootstrap or pilot launch. Reassess after the REST domain model and dashboard query patterns stabilize. |
```

3. File: `references/product/Product Decisions.md`

Search for:

```md
| **Notifications** | Email/SMS/push in scope for v1 (e.g. early warning, reminders to learners/employers). |
```

Replace with:

```md
| **Notifications** | Email and SMS required for v1 pilot launch. Push deferred until after pilot launch as a feature-flagged extension. |
```

4. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Search for:

```md
- GraphQL endpoint for complex learner dashboards and mobile-optimized queries.
```

Replace with:

```md
- REST is the required v1 API surface. GraphQL may be added later for complex learner dashboards and mobile-optimized queries after the REST domain model stabilizes.
```

5. File: `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`

Search for:

```md
- Schema versioning for GraphQL.
```

Replace with:

```md
- If GraphQL is added later, apply explicit schema versioning and parity checks against the REST contract.
```

6. File: `docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md`

Search for:

```md
        +--> Notification Service (email/SMS/push)
```

Replace with:

```md
        +--> Notification Service (email/SMS in v1; push optional post-pilot)
```

7. File: `docs/MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md`

Search for:

```md
- Admin notification fanout via email, SMS, push.
```

Replace with:

```md
- Admin notification fanout via email and SMS in v1; push optional post-pilot.
```

8. File: `docs/MYELIN_AI_AGENT_IMPLEMENTATION_PLAN.md`

Search for:

```md
- Configure admin notifications (email, SMS, push)
```

Replace with:

```md
- Configure admin notifications (email and SMS in v1; push optional post-pilot)
```

---

## 4. Recommended Edit Order

Apply the file changes in this order so the highest-level source of truth becomes consistent first:

1. `docs/v1_decisions.md`
2. `docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md`
3. `docs/MYELIN_PROFESSIONAL_FEATURES.md`
4. `references/product/Product Decisions.md`
5. AI mode and notification skills/docs
6. Data-model and UI reference files

---

## 5. Follow-Up Cleanup After These Locks

These are not part of the four canonical decisions above, but they should be cleaned up immediately after the decisions are locked because they will still confuse copilots and new engineers:

- Replace stale `dev_docs/` and `3D_simulation_work/` path references in the newer Myelin docs with the actual `primary_build_source/docs/` and `primary_build_source/references/simulation/` paths.
- Mark `references/product/Product Decisions.md` clearly as legacy or superseded if it remains in the repo.
- Update `MYELIN_VIBE_CODING_DEVELOPMENT_GUIDE.md` after the source-of-truth docs are edited so its assumptions match the locked decisions.

---

## 6. Definition Of Done For This Decision Lock

This decision-lock exercise is complete when:

- The four decisions above are approved.
- The listed source file edits are applied.
- No remaining source file contradicts the locked decisions on AI scope, upload formats, accessibility baseline, notification channels, or GraphQL bootstrap scope.
- `MYELIN_VIBE_CODING_DEVELOPMENT_GUIDE.md` is refreshed against the updated source docs.
