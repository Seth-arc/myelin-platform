# Myelin TMS — Product Specification

**Document version:** 1.0  
**Source:** Myelin Build / ai-study-buddy-mockup.html  
**Last updated:** February 2026  

---

## 1. Product Overview

**Myelin TMS** (Training Management System) is an **AI-driven simulation training platform** for **South African Engineering Trades**. It combines high-fidelity WebGL 3D simulations, an AI Learning Coach, structured learning pathways, and evidence-based assessment (e-Portfolio, peer review) so learners can master industrial safety protocols—especially **LOTO (Lockout/Tagout)**—in a safe, recursive sandbox before moving to real-world practice and mentor sign-off.

### 1.1 Vision Statement

A high-fidelity WebGL ecosystem for South African Engineering Trades: master LOTO and related protocols in a safe, recursive 3D sandbox, with a persistent AI Learning Coach and a clear path from simulation to verified skills and e-Portfolio submissions.

### 1.2 Core Value Propositions

| Pillar | Description |
|--------|-------------|
| **High Fidelity** | 60FPS WebGL simulations, optimized for desktop and constrained connectivity. |
| **AI-Native** | Persistent AI Learning Coach (e.g. “Nomvula”) for real-time protocol verification and conceptual guidance. |
| **Verified Skills** | Seamless path from 3D practice → video e-Portfolio submissions → mentor sign-off and Statement of Results. |

---

## 2. Target Users & Context

- **Primary:** Engineering Trades learners (e.g. NQF Level 4, QCTO-registered qualifications such as SAQA 61595 — Engineering Trades Electrical, 125 credits).
- **Institutions:** TVET colleges (e.g. Tshwane South TVET, Pretoria West Campus).
- **Workplace:** Employer placements (e.g. BMW Group Plant Rosslyn) for logbook hours and mentor sign-off.
- **Regulatory:** Aligned with South African standards (SAQA, QCTO), POPIA for data consent, and international safety standards (e.g. ISO 14118).

---

## 3. Platform Structure

The application is a **single-page learner experience** with a fixed sidebar navigation and a main workspace. The top bar shows programme context (e.g. Programmes → Mobile Device Repair Level 4 → Module 3) and global actions (notifications, WebGL status).

### 3.1 Navigation (Sidebar)

| Item | View ID | Purpose |
|------|---------|---------|
| Dashboard | `view-dashboard` | Progress overview and recent activity. |
| Learning Pathway | `view-pathway` | Structured modules and overall progress (e.g. 45%). |
| Virtual Simulator | `view-sandbox` | 3D LOTO (and future) simulations. |
| Workplace Logbook | `view-logbook` | Placements, hours, mentor sign-offs. |
| e-Portfolio | `view-portfolio` | Assignments, video submissions, past submissions. |
| Peer Review | `view-peer-review` | Review and grade peers’ video submissions. |
| Learning Coach | `view-buddy` | AI coach (text + video chat). |
| Statement of Results | `view-results` | Official transcripts and unit standards. |

**Footer:** Learner profile summary (avatar, name, “Learner Account”) and entry to **Learner Profile & Settings** (`view-profile`).

---

## 4. Feature Specifications

### 4.1 Dashboard

- **Purpose:** Overview of learning progress and recent activity.
- **Content (from mockup):** Placeholder for “Dashboard Analytics” — progress metrics, cohort analytics, recent activity.
- **Behaviour:** Single view; no secondary actions in the current mockup.

### 4.2 Learning Pathway

- **Purpose:** Show the structured curriculum and per-module completion.
- **Display:** Programme name (e.g. “Engineering Trades Level 4”), overall progress (e.g. 45%).
- **Actions:** “Download Transcript” (outline button).
- **Modules (example from mockup):**
  - **Module 1:** Introduction to Industrial Workspaces — Videos (4/4), Quizzes (2/2), Practical (1/1). Completed.
  - **Module 2:** Hand Tools and Power Tools Precision — In progress.
  - **Module 3:** Advanced Safety Protocol Sandbox — Current; primary CTA “Enter Virtual Simulator”, secondary “Ask Learning Coach”.
  - **Module 4:** Equipment Maintenance — Locked/upcoming; muted styling.
- **Behaviour:** Each module can show completion state, activity counts, and CTAs to Simulator or Learning Coach where relevant.

### 4.3 Virtual Simulator (3D Sandbox)

- **Purpose:** Practise the **7-step LOTO protocol** (ISO 14118) in a high-fidelity WebGL environment.
- **Layout:** Main 3D canvas (left) + task panel (right, ~380px).
- **Header:** Title “Advanced LOTO Simulation (7-Step Protocol)”, subtitle describing the environment; “Restart Protocol” button.
- **Canvas behaviour:**
  - Three.js scene (pumps, piping, valves, nameplate, hasp, padlock, tag, bleed valve, gauge, control panel).
  - OrbitControls for camera; crosshair and hover tooltip for interactable parts.
  - Step-by-step instruction overlay (e.g. “Step 1: Inspect the primary equipment Nameplate…”).
  - Action toasts for feedback (“Equipment TRB-094 Verified”, “Primary Flow Isolated”, etc.).
  - Toolbar: Interact, Rotate View, Reset Camera.
- **Task panel:**
  - Title “Standard LOTO Protocol” with badge “ISO-14118”.
  - “Complete in exact sequence to pass.”
  - **AI Walkthrough** toggle (outline button).
  - **7 steps:**  
    1) Verify Equipment Nameplate — Locate & click technical specification plate.  
    2) Isolate Primary Flow — Turn large green gate valve 90° clockwise.  
    3) Apply Scissor Hasp — Attach multi-lock isolation hasp to stem.  
    4) Apply Personal Lock — Attach personal safety padlock to hasp.  
    5) Attach Do-Not-Operate Tag — Affix warning tag to padlock.  
    6) Bleed Residual Energy — Open bleed valve; pressure to 0.  
    7) Verify Zero Energy (Test) — Press START on control panel to confirm lockdown.  
  - On completion: “System Safely Locked Out” message; “Record Official Video Submission” CTA (enabled after success).
- **Technical notes:** GSAP for camera/object animations; step state drives UI (active/completed) and instruction text; success state enables e-Portfolio recording flow.

### 4.4 Workplace Logbook

- **Purpose:** Track employer placements and mentor sign-offs.
- **Header:** “Workplace Logbook”, subtitle “Track your employer placements and mentor sign-offs”; “Add Entry” (primary).
- **Empty state:** “No Active Placements” — “Link an employer placement to begin logging hours and requesting mentor sign-offs.”

### 4.5 e-Portfolio

- **Purpose:** Compile and submit evidence, assignments, and practical demonstrations.
- **Header:** “e-Portfolio”, subtitle “Compile and submit evidence, assignments, and practical demonstrations.”
- **Current requirement (example):** “Module 3 LOTO Procedure” — record a continuous video of the 7-step LOTO procedure as in the 3D Sandbox.
- **Submission options:**
  - Upload: Drag-and-drop or browse; MP4 or WebM; max 500MB.
  - Or “Record directly from Webcam/Device”.
- **Past submissions:** List with thumbnail, title (e.g. “Module 2: Tool Handling Assessment”), date, status (e.g. “Passed”), “View Peer Feedback (n)”.

### 4.6 Peer Review Console

- **Purpose:** Review and grade peers’ video submissions to reinforce learning.
- **Header:** “Peer Review Console”, subtitle; reviewer score (e.g. 4.8/5.0).
- **Layout:** Two columns — (1) video player for current submission (e.g. “Anon Student_842 - Mod 2 Assignment”), (2) Assessment Rubric.
- **Rubric (example):** Criteria such as “Correct Tool Selection”, “Sequence Adherence” with options e.g. Fail (0) / Pass (1).
- **Action:** “Submit Peer Evaluation” (primary).
- **Section:** “Your submissions awaiting peer review” (list of learner’s own submissions pending review).

### 4.7 Learning Coach (AI Study Buddy)

- **Persona:** “Nomvula” — Learning Coach; avatar and optional video.
- **Modes:** **Text** and **Video** (toggle in header).
- **Text mode:**
  - Chat feed: AI messages (with optional embedded “AI Generated Explanation” video clips, e.g. “Fluid Dynamics: Trapped Pressure Vectors”).
  - User messages; context-aware (e.g. “Current Topic: LOTO Procedures”).
  - Input: attachment, textarea “Ask the AI Instructor a question about the protocol…”, send.
  - Disclaimer: “AI can make mistakes. Verify critical safety information against standard operating procedures.”
- **Video mode:**
  - Pre-call: Coach avatar, “Ready to discuss LOTO Procedures”, “Start Call”.
  - In-call: Full-screen AI avatar video (or placeholder), user PiP, call controls (mute, video, captions, End).
- **Context panel (sidebar):**
  - “Current Context Focus” — e.g. Advanced LOTO Simulator, “Jump to Simulation”.
  - “Referenced Theory” — links e.g. ISO-14118 Safety Standard (PDF), “Fluid Dynamics: Trapped Pressure” (AI-generated video).
- **Behaviour:** Coach can ask conceptual questions (e.g. why verify nameplate), confirm answers, and surface explanatory content; video mode for richer dialogue.

### 4.8 Statement of Results

- **Purpose:** Official transcripts and completed unit standards.
- **Header:** “Statement of Results”, subtitle “Official transcripts and completed unit standards”; “Download PDF” (outline).
- **Empty state:** “Results Pending” — “Complete your learning pathway and workplace logbook to generate an official Statement of Results.”

### 4.9 Learner Profile & Settings

- **Purpose:** Manage account, institution/placement, learning environment, and preferences.
- **Profile header:** Avatar (with “Change Photo”), name (e.g. Thabo Lekota), badges: Institution (e.g. Tshwane South TVET), Workplace (e.g. BMW Group Plant Rosslyn).
- **Sections:**
  - **Personal Details:** First/Last name, Work/Study email (Verified), SA ID, DOB, mobile, preferred language (e.g. English / Sesotho); Edit.
  - **Academic Programme:** Current qualification (e.g. Engineering Trades Electrical), SAQA ID, QCTO, credits; NQF Level, Cohort/Intake, Campus/Site.
  - **Security & Privacy:** Change password, POPIA consent; Update / Active.
  - **Learning Environment:** Active Learning Coach (e.g. Nomvula, description); Connectivity Mode (e.g. Good / High Data); Primary Device (e.g. Laptop/PC); Configure.
  - **Communication & Access:** Email notifications (assignments, peer reviews, feedback); SMS (urgent reminders, OTPs); toggles.
  - **Accessibility:** Auto-captions on videos; Learning Coach audio responses (text-to-speech); toggles.

---

## 5. User Flows (Summary)

1. **Learn & practise:** Dashboard / Pathway → choose module → Virtual Simulator → complete 7-step LOTO (optional: AI Walkthrough, Learning Coach).
2. **Get help:** Learning Coach (text or video) — ask questions, receive explanations and links to theory/simulator.
3. **Prove competence:** After simulator success → e-Portfolio → record/upload LOTO video → submit.
4. **Peer learning:** Peer Review → watch peer submission → complete rubric → submit evaluation; view feedback on own submissions.
5. **Workplace evidence:** Logbook → add placement → log hours → request mentor sign-offs.
6. **Credentials:** Complete pathway + logbook → Statement of Results → Download PDF.
7. **Account:** Profile & Settings → edit details, security, notifications, accessibility, learning environment.

---

## 6. Technical Stack (From Mockup)

- **Front-end:** Single HTML document; Inter (Google Fonts); Phosphor Icons.
- **3D:** Three.js (r128), OrbitControls; GSAP for animations.
- **Styling:** Inline CSS with CSS variables (--bg-base, --brand-accent, --radius-*, etc.); no Tailwind in the mockup file.
- **Interactivity:** Vanilla JS; view switching via `switchView(viewName)`; Three.js init, raycasting, step state, reset, camera controls.

---

## 7. Standards & Compliance

- **Safety:** LOTO procedure aligned with **ISO 14118** (referenced in UI).
- **Qualifications:** SAQA IDs, QCTO registration, NQF levels, credit values.
- **Privacy:** POPIA consent and data processing for learner records (mentioned in Profile).
- **Accessibility:** Captions on videos, optional TTS for Learning Coach; placeholder for further a11y requirements.

---

## 8. Out of Scope / Placeholders (Current Mockup)

- Dashboard and Logbook content are placeholder (empty states).
- Statement of Results is “Results Pending” until pathway and logbook are complete.
- Peer review rubric and submission list are illustrative.
- Learning Coach is UI + static copy; no real AI/LLM or video-call backend.
- “Download Transcript” and “Download PDF” have no backend.
- Notifications (bell) and WebGL indicator are UI-only.
- No authentication, authorization, or multi-tenant logic in the mockup.

---

## 9. Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Feb 2026 | Initial product spec derived from ai-study-buddy-mockup.html. |

---

*This spec describes the Myelin TMS platform as represented in the Myelin Build ai-study-buddy-mockup.html prototype. Implementation may refine or extend these features.*
