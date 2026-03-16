# Myelin UI/UX and Styling Guide

Document status: Design system baseline for professional build  
Version: 1.0  
Date: March 12, 2026  
Owner: Sethu (Founder, Myelin)  
Build partner: GPT-5.3-Codex (AI engineering copilot)  
Applies to:
- Public marketing (`index.html`, `about.html`)
- Authentication (`login.html`, `registration.html`)
- Learner application (`ai-study-buddy-mockup.html`, `module-*`)

---

## 1. Purpose

This guide defines the visual, interaction, and implementation standards for Myelin so all product surfaces are consistent, maintainable, and accessible.

Execution model:

- Single human decision owner: Sethu
- Single AI build partner: GPT-5.3-Codex

Primary goals:
1. One recognizable Myelin visual language across all pages.
2. Predictable component behavior in all states.
3. Shared token system for light and dark modes.
4. Reduced style drift between Tailwind and custom CSS surfaces.

---

## 2. Design Principles

1. Industrial clarity over decorative complexity.
2. Safety-first emphasis for simulation and assessment workflows.
3. High contrast hierarchy for critical actions.
4. Motion supports comprehension, not novelty.
5. Every interactive element has visible feedback and focus state.

---

## 3. Experience Layers

Myelin has three UX layers that must feel related but purpose-specific:

1. Marketing Layer
- Cinematic, high contrast, aspirational.
- Used for storytelling and demo capture.

2. Access Layer
- Focused, low-cognitive-load forms.
- Used for login, activation, reset, and consent.

3. Learning Layer
- Structured, utility-forward workspace.
- Used for pathway, simulator, assessment, evidence, and records.

Consistency rule:
- Typography, color meaning, iconography, and interaction patterns must stay consistent across all three layers even if composition differs.

---

## 4. Canonical Design Tokens

Use these tokens as the source of truth in all new UI work. Do not hardcode one-off values unless explicitly approved.

### 4.1 Color Tokens

#### Light theme
- `--bg-base: #f8fafc`
- `--bg-surface: #ffffff`
- `--border-color: rgba(15, 23, 42, 0.08)`
- `--text-heading: #0f172a`
- `--text-body: #475569`
- `--text-muted: #8492a6`
- `--brand-main: #0A1C30`
- `--brand-accent: #0A1C30`
- `--brand-accent-hover: #1e293b`
- `--brand-light: #f1f5f9`
- `--success: #10b981`
- `--success-bg: #ecfdf5`
- `--warning: #f59e0b`
- `--danger: #ef4444`

#### Dark theme
- `--bg-base: #050505`
- `--bg-surface: rgba(5, 5, 5, 0.8)`
- `--border-color: rgba(255, 255, 255, 0.05)`
- `--text-heading: #FFFFFF`
- `--text-body: #E5E5E5`
- `--text-muted: #A3A3A3`
- `--brand-main: #FFFFFF`
- `--brand-accent: #FFFFFF`
- `--brand-accent-hover: #E5E5E5`
- `--brand-light: rgba(255, 255, 255, 0.05)`
- `--success-bg: rgba(16, 185, 129, 0.1)`

### 4.2 Radius Tokens
- `--radius-md: 10px`
- `--radius-lg: 16px`
- `--radius-xl: 20px`

### 4.3 Elevation Tokens
- `--shadow-subtle: 0 4px 20px rgba(15, 23, 42, 0.04)`
- `--shadow-float: 0 20px 40px rgba(15, 23, 42, 0.08)`
- Dark variants:
  - `--shadow-subtle: 0 4px 20px rgba(0, 0, 0, 0.5)`
  - `--shadow-float: 0 20px 40px rgba(0, 0, 0, 0.8)`

### 4.4 Typography Tokens
- Base family: `Inter`
- Mono/accent family: `JetBrains Mono` (marketing highlights, tiny metadata, system labels)
- Base body size: `14px` (application), `16px` acceptable for long-form marketing copy
- Tracking:
  - Dense headings: `tracking-tighter`
  - Utility labels: uppercase with widened tracking

---

## 5. Typography System

### 5.1 Scale
- Display Hero: 64-144px equivalent (marketing only)
- H1: 32-48px
- H2: 24-32px
- H3: 18-22px
- Body: 14-16px
- Helper/meta: 10-12px

### 5.2 Usage Rules
1. Use `text-gradient` only for major marketing/value statements.
2. Use plain solid headings for application work surfaces and all safety-critical content.
3. Use mono style only for telemetry, tags, metadata, and system control labels.
4. Avoid more than two font families in one surface.

---

## 6. Spacing and Layout

### 6.1 Spacing Scale
Use an 8px rhythm with 4px sub-steps:
- 4, 8, 12, 16, 24, 32, 40, 48, 64

### 6.2 Application Shell
- Sidebar width: `280px`
- Topbar height: `70px`
- Main workspace padding: `2rem`
- View sections: vertical scrolling inside view, not on full body

### 6.3 Content Width
- Form surfaces: max 480-640px
- Dashboard/content surfaces: max 1200-1440px depending on data density
- Full-width immersive sections only for simulator and major marketing visuals

### 6.4 Grid
- Marketing: 12-column responsive grid via Tailwind utilities.
- Application: fixed shell + fluid content columns with minimum readable widths.

---

## 7. Component Standards

### 7.1 Buttons

### Variants
- Primary: brand-accent background, white text in light mode, black text in dark mode.
- Outline: surface background + border.
- Success: success color for positive completion actions only.
- Destructive: danger color reserved for cancel/end session/high-risk actions.

### Shared behavior
- Border radius: `var(--radius-md)` default, pill (`99px`) for hero and high-prominence CTAs.
- Transition: `0.2s` to `0.3s`, include hover and active states.
- Active press: scale to ~0.98 optional.
- Disabled: reduced opacity, no elevation, no pointer cursor.

### Size presets
- Sm: height 32px, text 12px
- Md: height 40px, text 14px
- Lg: height 48px, text 16px

### 7.2 Inputs and Form Controls
- Input background should remain low-contrast translucent dark in auth screens and solid surface in app screens.
- Focus state must include both border contrast and visible ring or equivalent glow.
- Labels use uppercase micro utility style for forms.
- Error state:
  - border uses danger color
  - helper text below field
  - no color-only error indication; include icon/text cue

### 7.3 Navigation
- Sidebar nav item states:
  - default: muted text
  - hover: elevated contrast + soft fill
  - active: brand-light background + accent text
- Ensure one active section marker only.
- Breadcrumbs must reflect current route and module context.

### 7.4 Panels and Cards
- `minimal-panel` is the default app card container.
- Required properties:
  - `background: var(--bg-surface)`
  - `border: 1px solid var(--border-color)`
  - `border-radius: var(--radius-xl)`
  - `box-shadow: var(--shadow-subtle)`

### 7.5 Progress and Status
- Progress bars:
  - track uses muted background
  - fill uses success color unless representing risk
- Task state tokens:
  - active: accent
  - completed: success
  - blocked/locked: muted and non-interactive affordance

### 7.6 Toggles
- Track and knob must clearly indicate on/off without color-only dependency.
- Provide an accompanying textual label.
- Use success for enabled state by default in settings contexts.

### 7.7 Modals and Overlays
- Backdrop: semi-opaque dark layer with blur.
- Content panel: high contrast surface, clear close control, esc and click-outside behavior (if non-critical modal).
- Entrance animation should not exceed 300ms.

### 7.8 Toasts and Alerts
- Success and error toasts use icon + text.
- Duration default: 2.5s for passive confirmation.
- Critical alerts stay persistent until dismissed or resolved.

---

## 8. Simulation-Specific UI Rules

### 8.1 Critical Hierarchy
1. Current step instruction
2. Interactive object feedback
3. Step completion checklist
4. Global controls (reset, camera, guide mode)

### 8.2 Step UX
- Only one current step can be active.
- Completed step must switch to check icon and locked history state.
- Wrong interaction attempt must trigger immediate corrective feedback.

### 8.3 Panel Layout
- 3D canvas left, task/assessment panel right.
- Keep right panel width stable to prevent layout shifts during step changes.

### 8.4 Safety Messaging
- AI guidance disclaimers must be visible near coach interactions in simulation context.
- Avoid ambiguous copy on pass/fail criteria; use explicit thresholds where possible.

---

## 9. Motion and Interaction

### 9.1 Motion Principles
- Use motion to explain state change and attention transitions.
- Avoid non-functional looping animations in dense work views.

### 9.2 Timing
- Fast interactions: 120-180ms
- Standard transitions: 200-300ms
- Large scene/context transitions: 400-1200ms

### 9.3 Easing
- Use smooth ease-out for entry.
- Use ease-in-out for camera and scene shifts.
- Keep motion curves consistent across modules.

### 9.4 Reduced Motion
- Respect reduced-motion preference by disabling non-essential animation.
- Preserve core state transitions without visual overload.

---

## 10. Accessibility Standards

### 10.1 Contrast
- Text and interactive elements must meet WCAG 2.2 Level A requirements in v1 and should meet WCAG 2.2 AA contrast targets on all core surfaces.
- Avoid low-contrast muted text for required instructions.

### 10.2 Keyboard Support
- All key workflows must be keyboard navigable:
  - auth
  - navigation
  - settings
  - submission and review actions
- Focus indicators must be visible on all controls.

### 10.3 Semantic Structure
- Use semantic headings and landmark regions.
- Forms require explicit label associations.
- Icons used alone must include accessible labels.

### 10.4 Media
- Captions available for instructional and AI-generated media where possible.
- Provide transcript/summary fallback for critical learning content.

---

## 11. Responsive Behavior

### 11.1 Breakpoint Guidance
- Mobile: <768px
- Tablet: 768-1023px
- Desktop: >=1024px
- Large desktop: >=1440px

### 11.2 Behavior by Surface
1. Marketing
- Stacked content on mobile.
- Keep hero readability and CTA visibility above fold.

2. Auth
- Single-column forms.
- Avoid clipped card edges and horizontal scroll.

3. Learning app
- Preserve functional access on tablet and larger.
- On small screens, prioritize task lists and forms over dense dual-column panels.
- Simulator may degrade to guided or restricted mode on unsupported devices.

---

## 12. Content and Microcopy Guidelines

### 12.1 Tone
- Professional, direct, instructional.
- Avoid hype language in operational workflow UI.

### 12.2 Safety-Critical Copy
- Use explicit action verbs and object names.
- Include why an action matters when learner safety is relevant.
- For errors, include cause + next action.

### 12.3 Labeling Conventions
- Navigation labels use title case.
- Form labels use concise nouns.
- Buttons use clear verbs: `Submit`, `Resume`, `Start`, `Download`.

---

## 13. Implementation Consistency Rules

### 13.1 One token source
- Define and import a single token source for:
  - color
  - typography
  - spacing
  - radius
  - shadow
  - motion

### 13.2 Tailwind and custom CSS alignment
- If Tailwind is used, extend Tailwind theme with Myelin tokens.
- Do not mix arbitrary one-off Tailwind values with near-identical custom CSS tokens.

### 13.3 Theme persistence
- Use one storage key for theme across all pages:
  - recommended key: `myelin-theme`
- Remove legacy duplicate keys to prevent mode inconsistency.

### 13.4 Avoid inline style drift
- Replace repeated inline style blocks with reusable classes/components.
- Inline styles allowed only for one-off dynamic values not representable by class token.

### 13.5 Component ownership
- Each reusable component must have:
  - canonical class or component definition
  - state variants
  - accessibility notes
  - usage examples

---

## 14. QA Checklist for UI Consistency

Run this checklist before each release:

1. Theme check
- Light and dark tokens apply consistently on all pages.

2. Navigation check
- Active state is correct and breadcrumbs match destination.

3. Component check
- Buttons, inputs, panels, toggles match tokenized styles.

4. Accessibility check
- Keyboard navigation works for all critical workflows.
- Focus and contrast are acceptable.

5. Responsive check
- No horizontal overflow on mobile auth and marketing pages.
- Core app interactions remain usable at tablet widths.

6. Motion check
- Transitions are smooth, consistent, and not distracting.

7. Copy check
- Safety and instruction text is clear and actionable.

---

## 15. Immediate Standardization Actions

1. Create shared token file and apply it to marketing, auth, and app pages.
2. Normalize theme storage key to `myelin-theme` across all HTML files.
3. Refactor repeated inline styles into shared utility classes.
4. Define canonical button/input/card components and replace local variants.
5. Add a design review gate to pull requests touching UI.

---

## 16. Versioning

| Version | Date | Notes |
|---|---|---|
| 1.0 | March 12, 2026 | Initial comprehensive UI/UX and styling guide for the professional build. |
