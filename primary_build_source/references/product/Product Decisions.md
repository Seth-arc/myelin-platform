# Product Decisions (TVET/FET SIS)

This document records product and technical decisions from stakeholder Q&A. It should be treated as the source of truth for scope, positioning, and behaviour until superseded by a formal change.

---

## 1. Product & positioning

| Topic | Decision |
|-------|----------|
| **Product name** | Placeholder only (e.g. "TVETFlow SIS/ERP"). Final name TBD. |
| **Geographic scope** | South Africa only for now. |
| **Licensing model** | Per campus. |
| **Delivery model** | Single product (we build and sell one product under our brand). |

---

## 2. Scope & MVP

| Topic | Decision |
|-------|----------|
| **First release scope** | All modules (Admissions through Workplace Learning, Funding, Reporting, etc.) with dummy data and demo-ready flows. Some areas may be "demo quality" (e.g. validations, performance) rather than fully production-hard. |
| **LMS** | Part of the same product (SIS + LMS in one platform). Targeted at technical/practical TVET/FET. Detailed scope to be discussed later. Do not reference Moodle or other third-party LMS in positioning. |
| **RPL and credit transfer** | Full workflow in v1. |
| **Offline / low bandwidth** | v2 only. |

---

## 3. Users, roles & access

| Topic | Decision |
|-------|----------|
| **Roles** | Admin, Lecturer, Employer, Super Admin, Student. |
| **Learner self-service (v1)** | View plus: update contact details, submit applications, etc. |
| **Employer mentor onboarding** | Invite-only by college, or employer linked to college. No open self-registration. |

---

## 4. Data, compliance & reporting

| Topic | Decision |
|-------|----------|
| **TVETMIS/DHET/SETA formats** | System designed so formats can be reverse-engineered from samples, then maintained via **config-only** (no code changes for format updates). |
| **SAQA/NQF codes** | Maintained in the product (not imported/synced from external source). |
| **Data residency** | No requirement that data must stay in SA or a specific region. |
| **Retention & archival** | 10 years for learner, application, financial and audit data. |

---

## 5. Funding

| Topic | Decision |
|-------|----------|
| **No funding at registration** | Behaviour **configurable per institution** (block or warn). **Product default = warn** (registration can proceed with warning until institution configures funding rules and optionally switches to block). |
| **External funding systems (MVP)** | Funding tracked internally only. No NSFAS or other disbursement system integration in v1. |

---

## 6. Workplace learning

| Topic | Decision |
|-------|----------|
| **Employer accreditation** | Attested by both college and SETA. Stored as **structured data**. |
| **Logbook templates** | Defined by the platform (not only college/SETA/programme). |
| **Mentor identity** | Preferably employees of the employer; externals (e.g. SETA assessors) allowed. |

---

## 7. Technical

| Topic | Decision |
|-------|----------|
| **Multi-tenant isolation** | Postgres **row-level security (RLS)** required (not application-level scoping only). |
| **SSO** | Required for v1. Support as **generic as possible** (e.g. SAML 2.0 / OIDC so any compliant IdP can be used). |
| **GraphQL** | Not required for bootstrap or pilot launch. Reassess after the REST domain model and dashboard query patterns stabilize. |
| **File storage** | Still open (no constraint specified). |
| **Notifications** | Email and SMS required for v1 pilot launch. Push deferred until after pilot launch as a feature-flagged extension. |

---

## 8. Non-functional & operations

| Topic | Decision |
|-------|----------|
| **Concurrent users** | Target: **5,000 concurrent users across all institutions** for v1. "Concurrent" = sessions. |
| **Backup / DR** | Follow best practice (concrete RTO/RPO to be documented for sign-off). |
| **Localisation** | English only for v1. No multi-language UI or RTL requirement. |

---

## 9. Process & governance

| Topic | Decision |
|-------|----------|
| **Product ownership** | Lead client has final say. |
| **Regulatory format changes** | TVETMIS/DHET/SETA format changes handled via **config-only** changes (no code release required). |
| **Pilot / beta** | All Agricultural Colleges in the Eastern Cape. |

---

## 10. Edge cases (workflows to be documented in spec)

| Topic | Decision |
|-------|----------|
| **Transfers** | Workflow fully defined in spec (who approves, what moves, what gets archived). |
| **Withdrawals & refunds** | In scope for funding module in v1; refund rules and reporting to be specified. |
| **Sensitive statuses** | Special handling required for: **Deceased**; **Unenrolled**; **Dropped out**. Document in spec: reporting, access, visibility. |

---

*Last updated from stakeholder Q&A. When updating, add a short note and date at the bottom.*
