# Simulation Development Guidance — Hyper-Realistic QCTO Training

**Purpose:** Standards and guidelines for **HD-quality**, hyper-realistic, QCTO/SETA-aligned 3D training simulations delivered **online in the browser**. Defines resolution tiers, interactivity requirements, and reference specifications for web-based rendering (WebGL/WebGPU), physics, and input—no install, runs on standard browsers.  
**Source:** 3D_simulation_work folder.  
**Audience:** Web developers and architects building browser-based training sims (e.g. Three.js, Babylon.js, PlayCanvas; JavaScript/TypeScript).

---

## Table of Contents

1. [HD Quality, Resolution & Interactivity Standards](#1-hd-quality-resolution--interactivity-standards)
2. [Tech Stack Guidelines (Agnostic)](#2-tech-stack-guidelines-agnostic)
3. [Implementation Priority Order](#3-implementation-priority-order)
4. [Geometry & Assembly Specifications](#4-geometry--assembly-specifications)
5. [PBR Materials & Photochemical Hyper-Realism](#5-pbr-materials--photochemical-hyper-realism)
6. [Surface Imperfections & Micro-Detail](#6-surface-imperfections--micro-detail)
7. [Environment, HDRI & Dynamic Weather](#7-environment-hdri--dynamic-weather)
8. [Physics, Constraints & Interaction](#8-physics-constraints--interaction)
9. [Acoustic Signatures & Haptic Feedback](#9-acoustic-signatures--haptic-feedback)
10. [Procedural Effects & Failure States](#10-procedural-effects--failure-states)
11. [Hand, Pose & High-Precision Input Pipelines](#11-hand-pose--high-precision-input-pipelines)
12. [Rendering & Post-Processing Standards](#12-rendering--post-processing-standards)
13. [Validation, Assessment & Performance](#13-validation-assessment--performance)
14. [Trade-Specific Physics & Hypnotics](#14-trade-specific-physics--hypnotics)
15. [Quick Reference Checklist](#15-quick-reference-checklist)

---

## 1. HD Quality, Resolution & Interactivity Standards

These standards apply to **browser-based** simulations. Implementations **SHALL** meet the minimum tier for the target deployment; **SHOULD** aim for HD where typical desktop/laptop browsers allow. All targets assume delivery via HTTPS and standard web APIs (WebGL 2, WebGPU where available).

### 1.1 Resolution Tiers

| Tier | Viewport resolution (min) | Render scale | Texture atlas (albedo/normal) | Geometry LOD (close-up) | Use case |
|------|----------------------------|--------------|-------------------------------|--------------------------|----------|
| **Minimum** | 1280×720 (720p) | 1.0 | 1024 | 32–48 segments (cylinder/torus) | Low-end browsers, small windows |
| **HD** | 1920×1080 (1080p) | 1.0–1.25 | 2048 | 64–96 segments | Standard browser (desktop/tablet) |
| **HD+** | 2560×1440 (1440p) | 1.25–1.5 | 2048–4096 | 96–128 segments | Fullscreen, high-DPI |
| **UHD** | 3840×2160 (4K) | 1.5–2.0 | 4096 | 128+ segments | Fullscreen 4K, kiosk/broadcast |

- **Rule:** Primary interactive objects (within 0.5 m equivalent) **SHALL** use the segment counts for the chosen tier; background or distant objects **MAY** use the next tier down.
- **Browser viewport:** Use `window.innerWidth` / `innerHeight` or canvas client size; **SHALL** handle resize and devicePixelRatio (e.g. 1.25×–1.5× on retina) without breaking layout or quality tier.

### 1.2 Frame Rate & Fluency

| Context | Target frame rate | Max frame time | Notes |
|---------|-------------------|----------------|--------|
| **Browser (desktop, primary)** | 60 fps | 16.67 ms | Primary target; use requestAnimationFrame |
| **Browser (mobile / tablet)** | 30–60 fps | 16.67–33.3 ms | Detect capability; **MAY** cap at 30 fps to save battery |
| **Background tab** | Paused or low | — | **SHALL** reduce or pause render when tab not visible (Page Visibility API) |

- **Standard:** Simulations **SHALL** sustain 60 fps on mid-range desktop browsers (Chrome, Firefox, Edge, Safari) at HD viewport; **SHOULD** degrade gracefully (e.g. lower LOD, disable heavy post-process) when frame budget is exceeded.
- **Frame pacing:** **SHOULD** keep frame-time variance within ±10% of the mean; **MAY** use adaptive quality (e.g. reduce shadow resolution, particle count) to maintain rate.

### 1.3 Color & Output

- **Color space:** **SHALL** use a linear workflow with sRGB (or equivalent) output for final display; HDR output **MAY** be used where supported (e.g. HDR10, Dolby Vision for kiosks).
- **Bit depth:** **SHALL** render and composite at minimum 8 bpc; **SHOULD** use 10+ bpc for HDR or wide-gamut pipelines.
- **Tone mapping:** **SHALL** apply a consistent, physically plausible tone map (e.g. ACES, Reinhard) for HD+ and above; **MAY** expose exposure as a user or scene setting.

### 1.4 Interactivity Standards (Browser)

| Requirement | Minimum | Target (HD) | Notes |
|-------------|---------|-------------|--------|
| **Input-to-visual latency** | ≤100 ms | ≤54 ms | From pointer/touch/key event to frame showing result |
| **Click / tap response** | ≤2 frames | 1 frame | Button, grab, or tool activation |
| **Pointer / touch continuity** | No visible stalls | 1:1 or smoothed 1:1 | Cursor/finger to object motion; use pointer events |
| **Haptic / audio feedback** | On key events | On contact + release | Web Audio API; Gamepad Haptics if available |

- **Input modalities:** **SHALL** support **pointer events** (mouse + touch in one model) and **keyboard** for primary actions. **SHOULD** support **Gamepad API** for optional controller; **MAY** support **getUserMedia**-based hand/pose (e.g. MediaPipe in-browser) for enhanced interaction.
- **Feedback:** **SHALL** provide immediate visual feedback for every primary action; **SHOULD** use **Web Audio API** for spatialized or positional sound; **MAY** use **navigator.vibrate** and Gamepad vibration where supported.
- **Accessibility:** **SHOULD** support keyboard navigation and optional captions; **MAY** support reduced motion or simplified physics; ensure focus management and ARIA where UI overlays the canvas.

### 1.5 Quality Bar (HD Tier, Browser)

For a **browser-based** simulation to be considered **HD quality** in line with this guidance:

1. **Resolution:** Canvas/viewport ≥1080p at 1.0× scale (or devicePixelRatio-adjusted); runs in Chrome, Firefox, Edge, Safari (current versions).
2. **Frame rate:** Sustained ≥60 fps on mid-range desktop in the primary browser; **SHALL** use `requestAnimationFrame` and respect tab visibility.
3. **Materials:** PBR (metalness/roughness or equivalent) with environment reflection; no unlit flat shading for key assets.
4. **Interactivity:** Input-to-frame latency ≤54 ms when using camera-based tracking; ≤2 frames for pointer/tap/key actions; pointer + keyboard **SHALL** be supported.
5. **Assessment:** Procedural or scripted validation of steps (e.g. LOTO sequence, tool use) with defined tolerances and pass/fail criteria.

---

## 2. Tech Stack Guidelines (Browser-Based)

Implementations **SHALL** run in a standard web browser (no native install or plugins). Use **WebGL 2** as the minimum rendering API; **WebGPU** **MAY** be used where available with a WebGL fallback. The following are requirements and recommendations for online delivery.

### 2.1 Rendering (Web)

- **SHALL** use **WebGL 2** (or **WebGPU** with feature detection and fallback) for 3D; **SHALL** support a scene graph, camera, and lights with PBR-style materials and environment lighting (IBL/HDRI or probes).
- **SHOULD** use a browser-oriented 3D library (e.g. **Three.js**, **Babylon.js**, **PlayCanvas**) that handles context loss, resize, and devicePixelRatio; **SHOULD** support shadow mapping and post-processing (AA, bloom, tone mapping) for HD+.
- **SHALL** handle **WebGL context loss** (e.g. `webglcontextlost` / `webglcontextrestored`) and recover or prompt reload; **SHOULD** use instancing, LOD, and culling to meet 60 fps in-browser.

### 2.2 Physics (WASM or JS)

- **SHALL** support rigid-body dynamics and constraints (hinge, fixed, spring) for mechanical fidelity; **SHALL** run in the browser (no native binaries except WASM).
- **SHOULD** use a fixed timestep (1/60 s or 1/120 s) and deterministic behavior for assessment; **SHOULD** prefer **WASM-based** engines (e.g. **Rapier**, **Ammo.js**) for stability and performance, or pure JS (e.g. **Cannon-es**) with lighter scenes.
- **SHALL** load physics engine asynchronously (e.g. dynamic import or WASM init) so the main thread and startup remain responsive.

### 2.3 Input & Interaction (Web APIs)

- **SHALL** use **Pointer Events** (mouse, touch, pen) for primary interaction; **SHALL** map to 3D (raycast or collision) with units in metres.
- **SHOULD** support **KeyboardEvent** for accessibility and shortcuts; **SHOULD** support **Gamepad API** for optional controller; **MAY** use **getUserMedia** + a browser-compatible hand/pose library (e.g. **MediaPipe in JavaScript**) for hand tracking—latency and accuracy **SHALL** meet §1.4.
- **SHALL** respect **user gesture** requirements where the platform mandates them (e.g. audio playback, fullscreen, camera access).

### 2.4 Audio (Web Audio API)

- **SHALL** use **Web Audio API** for spatialized or positional sound for key events (tool contact, alarms); **SHOULD** support distance attenuation and, where needed, simple occlusion.
- **SHALL** not autoplay audio without user interaction (follow browser autoplay policy); reference values in §9 (frequency, duration, decay) apply as design targets.

### 2.5 Data & Assets (Web Delivery)

- **SHALL** use consistent units (metres, seconds, kilograms) for geometry, physics, and animation; **SHOULD** use **glTF/GLB** or engine-native formats that stream or lazy-load.
- **SHOULD** serve assets over HTTPS with appropriate caching (e.g. long-lived cache for immutable assets); **MAY** use IndexedDB or Cache API for offline/progressive use. **SHALL** avoid blocking the main thread with large synchronous asset parsing.

---

---

## 3. Implementation Priority Order

| Phase | Focus | Outcome |
|-------|--------|--------|
| 1 | **PBR materials** | Instant ~80% realism; metalness, roughness, IOR, clearcoat per object. |
| 2 | **HDRI + surface imperfections** | Environment context; scratches, rust pits, fingerprints, polish swirls. |
| 3 | **Physics properties** | Mass, CoM, friction, rotational constraints; tool interaction authenticity. |
| 4 | **Audio / haptics** | Multi-modal immersion; fundamental freq, decay, optional haptic impulse. |
| 5 | **Procedural effects** | Failure-state drama: arc flash, thermal runaway, conformal spray. |
| 6 | **High-precision input (optional)** | Millimeter-level collision; hand/pose or controller pipeline; grip/ergonomics model. |

**LOTO-specific build order:** Geometry (core shapes + PBR) → Interactions (raycast or equivalent snap + angular validation) → Physics (rigid-body constraints) → Polish (micro-details, pressure sim, sequence validation).

---

## 4. Geometry & Assembly Specifications

### 4.1 Principles

- **Absolute coordinates** — Every object positioned relative to world origin `[0,0,0]`; gravity vector e.g. `[0, -9.81, 0]`. Units **SHALL** be consistent (metres recommended).
- **Exact dimensions** — Manufacturer-spec geometry down to **0.0001 m** (0.1 mm) where tolerances matter (e.g. BGA ball 0.00025 m, solder mask, ZIF lift force).
- **Assembly hierarchies** — Explicit parent-child; profile paths for extrusions; position, rotation, and offset in metres.
- **Collision volumes** — Geometry **SHALL** support the interaction model in use: e.g. raycasting, capsule sweeps, or skeleton/hand collision (fingertip rays, capsule cast, convex hull/BVH) for high-precision input.

### 4.2 Segmentation (Hyper-Realism)

| Distance / use | Cylinder | Torus | Sphere | Extrude |
|----------------|----------|-------|--------|---------|
| **High detail 0–0.5 m** | 128 | 96 | [64, 48] | steps 8, bevelSegments 6 |
| **Medium ~2 m** | 64 | 48 | [32, 24] | steps 4, bevelSegments 3 |

- **Primary (80% visual weight):** High-segment cylinders/tori for valve bodies and handwheels (64+ segments for HD tier).
- **Secondary (15%):** Legible text or decals for nameplate (e.g. ISO-14118:2017), laser-etched serials.
- **Tertiary (5%):** Instanced geometry for bolt heads; tube/spline for weld beads; vertex displacement or normal detail for contaminants.

### 4.3 Manufacturing Detail (Threads & Fasteners)

- **Threading:** pitch `0.0018` m, depth `0.0006` m, repeat 24, normalStrength `2.2`.
- **Bolted assemblies:** bolt count 12; hex head across flats `0.013` m, across corners `0.015` m; thread profile **ISO 898-1 class 8.8**.
- **M8 panel bolt:** shank `[0.008, 0.05]`, thread_pitch `0.00125`, head across_flats `0.013`, height `0.006`.

### 4.4 Example Assembly Snippets (Reference Specifications)

**Solar PV clinic:** `global_scene`: world_origin, roof_pitch_angle 22°, wind_direction `[1,0,0.8]`. Purlin extrude_beam `[3.6, 0.1, 0.05]`; corrugated_sheet `[3.0, 0.84, 0.00047]`, corrugation_profile `[0.012, 0.076, 8]`. Panel frame_outer rounded_box `[1.05, 1.82, 0.035]`, corner_radius `0.008`, segments `[32,64,8]`; glass_face thin_box `[1.03, 1.77, 0.0035]` with transmission 0.92, IOR 1.52; cell_grid 72 cells layout [12,6], cell_geometry `[0.0396, 0.156, 0.0002]`, busbars width `0.0015`, tabs 156. Panel mass 22 kg, com_offset_local `[0, 0.18, 0]`. Mid_clamp body `[0.04, 0.03, 0.025]`; M8 bolt position and head specs as above. Torque wrench target_torque **12 Nm**, handle `[0.22, 0.038, 0.038]`, head `[0.045, 0.035, 0.025]`.

**iPhone 16 Pro Max:** world_origin `[0, 0.9, 0]`. Titanium frame fillet_box `[0.077, 0.162, 0.0085]`, corner_radius `0.012`, wall_thickness `0.0012`; metalness 0.96, roughness 0.08. Display glass_oled_stack `[0.073, 0.158, 0.0028]`; true_tone_flex 12 pins, pitch `0.0005`; display_connector_zif slot `[0.012, 0.003, 0.0008]`, lift_tab `[0.004, 0.002, 0.001]`. Logic board PCB `[0.110, 0.040, 0.0012]`, 8 layers; BGA PMIC `[0.006, 0.006, 0.0009]`, pitch 0.0005, 144 balls. Battery pouch `[0.115, 0.055, 0.0045]`; VHH adhesive islands 6, pull_tab `[0.095, 0.008, 0.001]`. USB-C port_shell `[0.010, 0.006, 0.003]`, opening `[0.0084, 0.0026]`; pogo_pins 4, dia `0.0006`, spring_travel `0.0012`, spring_k 0.35. Phillips 000 driver tip `[0.006, 0.004, 0.002]`, handle `[0.12, 0.012, 0.012]`, torque_limit **0.17 Nm**; curved tweezers tip_width `0.0006`, tip_length `0.010`, handle_span `0.095`.

**Ventilator PCB:** 10-layer HDI; substrate `[0.200, 0.160, 0.0018]`; fiducials dia `0.001` at four corners. BGA 624-ball: pitch `0.0004`, ball_dia `0.00025`, post_reflow_height `0.00022`, pad_dia `0.00028`. Hot-air nozzle 4 mm outlet, temp setpoint 261°C, accuracy 2°C, air_flow 12; squeegee width 0.075, thickness 0.0005, angle 60°. Conformal coating orifice `0.0008`, cone_angle 45°, droplet_size **40 µm**, surface_tension 0.032 N/m. BGA tweezer placement: dual_tip_precision `0.0003`, solder_ball_contact `0.00012`, placement_tolerance **0.000015 m**.

---

## 5. PBR Materials & Photochemical Hyper-Realism

Implementations **SHALL** use a PBR workflow (metalness/roughness or specular/glossiness); values below are reference targets for any engine.

### 5.1 Per-Object PBR Specs (Reference Values)

**Solar PV:**

| Material | base_color (RGB) | metallic | roughness | normal_scale | clearcoat | IOR | transmission | thickness |
|----------|------------------|----------|-----------|---------------|-----------|-----|--------------|-----------|
| zincalume_corrugated_iron | [0.65,0.65,0.68] | 0.92 | 0.18 | 1.2 | 0 | 1.33 | — | — |
| treated_pine_purlin | [0.45,0.32,0.22] | 0.02 | 0.75 | 2.1 | 0.15 | — | — | — |
| pv_panel_glass | [0.95,0.96,0.98] | 0 | 0.02 | — | — | 1.52 | 0.92 | 0.0035 |
| aluminium_extrusion_anodized | [0.82,0.85,0.88] | 0.95 | 0.12 | — | 0.25 | — | — | — |

**iPhone / PCB:**

| Material | base_color / emissive | metallic | roughness | clearcoat | anisotropy | IOR | transmission |
|----------|----------------------|----------|-----------|------------|------------|-----|--------------|
| titanium_frame_polished | [0.85,0.82,0.78] | 0.98 | 0.08 | 0.45 | 0.8 | — | — |
| oled_display_layer | emissive [0.02,0.05,0.12] | — | 0.01 | — | — | 1.51 | 0.88 |
| fr4_pcb_substrate | [0.18,0.22,0.28] | 0 | 0.65 | — | — | — | — |
| solder_mask_green | [0.12,0.45,0.22] | 0 | 0.35 | 0.12 | — | — | — |

### 5.2 Measured Industrial Material Database (Reference)

- **AISI_304_stainless:** albedo [0.56,0.57,0.56], metallic 0.98, roughness [0.02, 0.08], clearcoat 0.95, clearcoatRoughness 0.01, anisotropy 0.85, ior 2.42, F0 match albedo.
- **brass_C36000:** albedo [0.71,0.65,0.41], metallic 0.94, roughness [0.10, 0.22], clearcoat 0.88, sheen 0.05, edgeF0 [0.95,0.91,0.82].
- **ABS_polycarbonate:** albedo [0.12,0.14,0.18], metallic 0, roughness 0.96, transmission 0.92, ior 1.49, thickness [0.018, 0.028], attenuationDistance 0.12, attenuationColor [0.98,0.97,0.95].
- **neoprene_60A:** albedo [0.11,0.13,0.12], roughness 0.99, normalScale [2.8, 2.8], clearcoat 0.25, sheen 0.4, sheenTint [0.9,0.82,0.75].

### 5.3 Microfacet BRDF (Hyper-Realism)

- **GGX Trowbridge–Reitz:** alphaRemap from roughness; visibility **Smith GGX correlated**; Fresnel **Schlick optimized**; multiScatter `(1 - reflectance) / 1.5`.
- **Measured microfacet:** milled_aluminum alphaG 0.12, alphaD 0.08; sandblasted_steel 0.28 / 0.35; polished_brass 0.045 / 0.032.

### 5.4 Subsurface Scattering (Plastics & Skin)

- **ABS:** scatterProfile backward_scatter 0.7, scatterDistance 0.015, scatterColor [0.95,0.96,0.97], phaseFunction 0.6.
- **Polycarbonate:** forward_scatter 0.4, scatterDistance 0.028, scatterColor [0.98,0.97,0.94].
- **Skin (if hands visible):** epidermis thickness 0.0012, scatter 0.85; dermis 0.0021, scatter 0.92.

### 5.5 Environment Map Hierarchy

- **Primary 8k:** industrial_welding_bay_8k.hdr.
- **Secondary 4k:** machine_shop_overhead_4k.hdr.
- **Ambient 2k:** workshop_diffuse_2k.hdr.
- **Specular 1k:** chrome_reflection_1k.hdr.
- **LOD streaming:** mipBias -0.7, anisotropy 16; pre_filtered mipmap_count 9, specular_levels [0, 0.33, 0.66, 1.0].

---

## 6. Surface Imperfections & Micro-Detail

### 6.1 Procedural Imperfections (Reference Counts & Sizes)

**Corrugated iron (5-year wear):**

- Scratch depths [0.00005, 0.0002] m.
- Rust pits: count 2450, size_range [0.0008, 0.003] m, depth 0.00012 m.
- Dents: count 18, diameter_range [0.008, 0.025] m, depth_range [0.0003, 0.0008] m.

**PV panel frame anodized:**

- Fingerprints: count 12, oval_size [0.008, 0.004] m, opacity [0.15, 0.35].
- Scratches: count 42, length_range [0.002, 0.012] m, width 0.0001 m, depth 0.00002 m.

**iPhone titanium case:**

- Micro_abrasions: density 1250, size 8e-6 m, depth 2e-6 m.
- Polish_swirl_marks: pattern_radius 0.0012 m, spacing 0.0008 m.

### 6.2 Manufacturing Artifacts (Procedural)

- **Milling marks:** direction tool_path_vector, depth 0.0003 m, spacing 0.0012 m, normal_strength 2.1.
- **Tool chatter:** frequency 120 Hz, amplitude 0.00015 m, phase_variation 0.3.
- **Casting porosity:** cluster_size 0.002 m, density 0.03, displacement -0.0002 m.
- **Ejector pin marks:** diameter 0.0021 m, depth 0.0004 m, count_per_cm² 4.

### 6.3 Wear Patterns

- **Finger oil transfer:** roughness_reduction 0.018 per grab, sheen_increase 0.12, fade_curve exponential 300 s, visible under grazing light.
- **Thread galling:** material_transfer aluminum_to_steel 0.001 m, friction_increase 0.22, localized_roughness 0.75.
- **Contaminants:** cutting_fluid residue grease_bloom roughness 0.15; dust ao_multiplier 1.2 at edges.

### 6.4 Contact Shadows (Micro)

- **Bias table (m):** small_gaps_0.1mm 0.00008; bolts_nuts 0.00012; gasket_seams 0.0002; weld_beads 0.00015.
- **Normal offset:** dynamic = geometry_thickness × 0.1, min 0.0003, max 0.002.
- **Penumbra:** industrial_fluorescent 0.018; spot_lighting 0.008; area_light_1m² 0.025.
- **Dynamic update:** 60 Hz; bias 0.0001, normal_offset 0.001, penumbra 0.02 for general contact shadows.

### 6.5 Visual Interaction Feedback

- **Fingerprints/oil:** transfer_rate 0.01 roughness reduction per grab, fade_time 300 s, visible_under_grazing_light true.
- **Deformation:** plastic_flex vertex_displacement 0.3 mm; metal_compression yield_mark_radius 2 mm.
- **Sweat/oil on valve:** roughness_reduction 0.02 per grab; hand_slip_risk when torque > 12 Nm with wet grip.

---

## 7. Environment, HDRI & Dynamic Weather

### 7.1 HDRI & Bounce Lighting

- **Khayelitsha clinic (outdoor):** hdri_base `overcast_cape_town_4k.hdr`.
- **Sandton boardroom (indoor):** hdri_base `luxury_office_daylight_8k.hdr`; bounce_lighting ceiling_fluorescent [5000 K, 0.92], desk_laminate [0.85, 0.12].

### 7.2 Dynamic Wind (Outdoor PV)

- **intensity_range:** [0.25, 1.2] kN/m².
- **gust_frequency:** 0.8 Hz.
- **direction_vector:** [1.0, 0.0, 0.8].
- **Weather:** rain_particles count 3500, size_distribution [50e-6, 200e-6] m, terminal_velocity [2.5, 8.0] m/s, density 1.22 kg/m³; sky_conditions cloud_coverage 0.85, precipitation_rate 4.2 mm/hr.

### 7.3 Task Lighting (ISO-14118 Style)

- Overhead fluorescents + localized task lighting; position lights to match LOTO diagram conventions. Optional: **gaze-driven** task_lighting auto_brighten_gaze_target.

---

## 8. Physics, Constraints & Interaction

Any physics engine used for precision trades **SHOULD** meet the following; values are reference targets.

### 8.1 Physics Engine (Reference Configuration)

- **Gravity:** [0, -9.81, 0] m/s² (or equivalent in engine units).
- **Timestep:** 1/60 s or 1/120 s fixed; **SHALL** be consistent for reproducibility.
- **Solver:** Sufficient iterations for stable constraints (e.g. 32 position, 8 velocity); allow sleep with low threshold for performance.
- **CCD (optional):** Recommended for fast or small objects; 1 substep, threshold ~0.01.
- **Colliders:** Use convex or hull colliders for precision; trimesh for static environment only where supported without instability.

### 8.2 Physics Properties (Per Object)

- **pv_panel_440W:** mass 22 kg, com_offset [0, 0.18, 0], moments_inertia [28.5, 1.2, 29.1], drag_coefficient 1.05, wind_load_area 1.91 m².
- **mc4_connector_male:** mass 0.028 kg, friction_static 0.42, friction_dynamic 0.35, restitution 0.12, **crimp_force_required 45 kg**.
- **iphone_frame_titanium:** density 4506 kg/m³, youngs_modulus 110e9 Pa, poissons_ratio 0.34, yield_strength 880e6 Pa.
- **bga_solder_ball_sac305:** diameter 0.00025 m, volume 8.18e-12 m³, density 7400 kg/m³, melting_point 490.15 K, surface_tension 0.51 N/m.

### 8.3 Valve Assembly (Mechanical Fitter / LOTO)

- **Stem torque curve (Nm):** [8.2, 3.1, 11.8] — initial, running, final snap.
- **Packing friction:** 2.4 Nm; seat_load 4500 N.
- **Rotational constraint:** axis [0,0,1], min 0, max π/2 + 0.01, stiffness 1e6, damping 1e4.
- **Validation:** rotation ∈ [π/2 ± 0.05] rad (90° ± ~3°); snap detection within 0.01 rad of target.
- **Pressure simulation:** gauge needle pressure = 10×exp(-bleedValveAngle×0.5) (10 bar → 0); Step 6 validate pressure < 0.1 bar.

### 8.4 Grab / Manipulation (Input-Agnostic)

- **Reach / raycast:** maxDistance 0.4 m, grabRadius 0.03 m, forceMax 25 N, smoothness 0.95 (or equivalent for controller/hand).
- **Collision-based grab (if used):** overlapThreshold 0.015 m; palm sphere radius 0.08 m; finger capsules 5× radius 0.012 m.
- **Grip strength:** linear ramp 0→1 over 0.2 s; release: inherit_velocity 0.8, damping 0.7.
- **Finger forces (N), for hand models:** thumb 12, index 18, middle 20, ring 15, pinky 10.
- **Joint limits (°), for hand models:** mcp_flex [0,90], pip_flex [0,110], dip_flex [0,80], abduction [-20,20].

### 8.5 LOTO Step-by-Step Interaction Physics

**Step 2 – Valve handle:**

- Grip: palm_wrap + thumb_oppose; wrist_pronation 45° required.
- Torque: gradual_ramp 0.5 s, max 15 Nm; resistance profile high (8) → running (3) → final snap (12) Nm.
- Slip risk: torque > 12 Nm with wet grip.

**Step 3 – Hasp:**

- Two_hand_requirement true; squeeze_force 25 N thumb_index (validation 45 N for full squeeze); scissor_opening 45° max; alignment_tolerance ±3 mm stem center.
- Sequence: squeeze → position → release; alignment_assist magnetic_preview 10 N; snap_confirmation auditory + haptic + visual triple.
- Auditory: squeeze plastic_creak 400 Hz, release spring_twang 1200 Hz.

**Step 4 – Padlock:**

- Shackle clearance_required 0.008 m; squeeze_deform 0.5 mm plastic flex.
- Key insertion: alignment_tolerance 0.002 m, insertion_force 2.5 N, tumbler_feedback 3 clicks sequential.
- Sequence: squeeze → align → insert → twist → release; two_hand_manipulation true; orientation upright_lock_body; fumble_factor small_shackle 8 mm gap.

### 8.6 Tool Physics Database

- **Crescent wrench 8 inch:** mass 0.42 kg, com_offset [-0.08, 0, 0], inertia [0.002, 0.0008, 0.0025]; jaw adjuster_thread 0.0016 m, max_opening 0.032 m, slippage torque×0.12 when angle > 3°.
- **Phillips #2 screwdriver:** tip cross_angle 55°, depth 0.006 m, width 0.004 m; cam_out_force 5.2 N at 5°; handle torque max 12 Nm over 120° wrist.
- **Pliers:** squeeze_force 45 N, pivot_friction 0.3.
- **Torque wrench M8:** target_torque 12 Nm; handle [0.22, 0.038, 0.038], head [0.045, 0.035, 0.025].

### 8.7 Friction Table (Trade: Mechanical)

- metal_metal 0.75; metal_plastic 0.65; metal_rubber 0.92.
- Mass profiles: valve_handle mass 2.5 kg, inertia [0.1, 0.1, 0.2]; wrench mass 1.2 kg, com [0, 0, -0.15]; bolt mass 0.08 kg, shape cylinder.

### 8.8 Hasp Attachment

- Raycast-to-snap: `hasp.position.lerp(stem.worldToLocal(0,0,0.1), 0.1)` with damping 0.9 tween for mechanical “clunk”. Step 3 validate hasp distance to stem < 0.02 m.

---

## 9. Acoustic Signatures & Haptic Feedback

Audio and haptic **SHALL** use spatialized or positional feedback where supported; reference values below apply to any middleware or engine.

### 9.1 Acoustic (Fundamental Freq, Duration, Decay)

| Event | fundamental_freq (Hz) | duration (s) | decay | harmonics / notes |
|-------|------------------------|--------------|-------|---------------------|
| mc4_connector_snap | 1850 | 0.045 | 0.12 | 3.7, 7.4, 11.1 kHz |
| torque_wrench_click_12Nm | 4200 | 0.032 | 0.08 | impulse_ramp haptic |
| phillips_screwdriver_strip | 2800 | 0.12 | — | noise_floor -24 dB |
| valve_turn | — | — | — | scraping_metal_fm_sweep |
| hasp_squeeze | 400 (creak), 1200 (release) | — | — | plastic_compression 2nd harmonic |
| padlock_click | 2000 | — | — | precise_metal_detent |
| magnetic_snap | 80 | — | — | electromagnetic_thump |
| wrench_tighten | 1200 | — | — | metal_thread |

### 9.2 Haptic (Intensity, Frequency, Duration)

- **mc4_snap:** intensity 0.65, frequency 1850 Hz, duration 0.08 s.
- **Torque wrench click:** intensity 0.85, waveform impulse_ramp.
- **Phillips strip:** frequency [120, 450] Hz, intensity 0.45.
- **Frequency response bands:** metal_contact [200, 800] Hz; plastic_flex [100, 300] Hz; magnetic_snap [40, 60] Hz; valve_torque variable 2–20 Hz.
- **Force patterns:** grab_resistance rigid 25 N limit; rotation_torque proportional angle/velocity; collision_impact impulse_based_velocity; magnetic_attraction smooth_pull 50 N peak.
- **Thermal (optional):** metal_conduction slow_warmup 42°C; plastic ambient.

### 9.3 Spatialized Audio

- Contact sounds position_based_occlusion; occlusion_filter muffled_lowpass_distance; reverb_zones industrial_metal_tank.
- **Material impact table:** steel_steel freq [800, 2500] Hz, decay 0.3; plastic_metal [200, 800], decay 0.6; rubber_concrete [80, 250], decay 1.2.
- **Spatial:** lowpass_distance 440 Hz at 5 m; air_absorption 1.2 dB/m high freq.
- **Surface scan (gamepad):** metal_hard freq [280, 820], decay 0.18; plastic_med [110, 320], decay 0.42; rubber_soft [55, 135], decay 1.1.

### 9.4 Haptic Force Emulation (Gamepad / Controller)

- **Collision impulse:** intensity `sqrt(impulse_mag)*0.45`, duration `min(0.12, impulse_mag*0.02)`, frequency clamp(80, impulse_mag*120, 250).
- **Continuous resistance:** torque_feedback `torque/18.0 * 0.85`, frequency 2–28 Hz torque_proportional, waveform sawtooth_damped.

---

## 10. Procedural Effects & Failure States

### 10.1 DC Arc Flash

- **Plasma blob:** size_range [0.002, 0.008] m, temperature 6000 K, lifetime 0.012 s, particles 450.
- **Shockwave:** radius 0.018 m, velocity 1450 m/s.
- **Electrical arc (generic):** voltage_breakdown 3 kV/mm air, arc_temperature 6000 K, emissive_scale voltage/1000; breakdown_field 3e6 V/m, shockwave 0.02 s radius 0.1 m.

### 10.2 Battery Thermal Runaway

- **Vent gases:** particles 2800, velocity [5.2, 12.0] m/s, temperature 950 K.
- **Cell rupture:** debris_count 24, size_range [0.0008, 0.0025] m.

### 10.3 Conformal Coating Spray

- **Droplet_size:** 40e-6 m (40 µm); cone_angle 45°; evaporation_rate 1.2 mm/s; surface_tension 0.032 N/m.
- **Selective nozzle:** orifice 0.0008 m, cone 45°, droplet 0.00004 m.

### 10.4 Welding (Trade)

- **Molten metal:** viscosity 0.006 Pa·s, surface_tension 1.8 N/m, cooling_rate 200°C/s, thermal_emissive temp/2000.
- **Spatter:** ejection_velocity [2, 8] m/s, cooling_trajectory true, stick_probability 0.3; lifespan [0.8, 2.1] s, velocity [2.1, 8.4], cooling_emissive 6000K→1200K, bounce_restitution 0.12, trail_length 8.

### 10.5 Fluid (Plumbing)

- **Pipe leak jet:** bernoulli_velocity sqrt(2*pressure/rho), cone_angle 28°, surface_tension 0.073 N/m, splash restitution 0.3, spread 45°.
- **Cutting fluid mist:** droplet_size [0.0002, 0.001] m, evaporation linear 3 s, refractive_index 1.33.

### 10.6 Smoke / Thermal

- **Smoke:** buoyancy 9.81×1.3, turbulence perlin 0.02 scale, curl_noise true, temperature_gradient 800 K surface → 300 K core.
- **Blackbody:** planck_curve_resolution 256, temperature_range [300, 8000] K; cold_steel 300, operating_valve 320, weld_pool 2200, arc_core 6000 K.
- **Grip warmth:** 42°C surface over 10 s; friction_heating localized roughness decrease 0.08.

---

## 11. Hand, Pose & High-Precision Input Pipelines (Browser)

When using **in-browser camera-based hand/pose tracking** (e.g. **MediaPipe** via JavaScript, or similar running on **getUserMedia**), the following reference specifications **SHOULD** be met for HD-quality interaction. Implementations using only pointer, touch, or gamepad **MAY** skip this section but **SHALL** still meet §1.4. **SHALL** request camera only after user gesture and handle permission denial gracefully.

### 11.1 Holistic Model (Reference: 553 Points)

Typical point counts for full-body + hands + face (e.g. pose + left/right hand + face mesh); exact counts depend on the tracking library (e.g. MediaPipe, ARKit, custom).

- **Pose:** 33 landmarks (nose, eyes, ears, mouth, shoulders, elbows, wrists, hips, knees, ankles, etc.).
- **Hands:** 21 per hand × 2 = 42.
- **Face:** 478 (full mesh); critical regions: eye [33,160,158,133,153,144], mouth [13,14,78,81,291], jawline, cheekbones, forehead.
- **Total:** 33 + 42 + 478 = **553**.

### 11.2 Hand Pipeline Config (Reference)

- **model_complexity:** 1; max_num_hands 2; min_detection_confidence 0.88; min_tracking_confidence 0.94; smooth_landmarks true; enable_segmentation true; min_hands_presence 0.75; hand_presence_threshold 0.5.

### 11.3 Landmark 3D Accuracy Targets (mm / °)

- wrist_0 pos <3.2 mm, rot <2.1°; thumb_mcp_1 <2.8 mm, flex <3.2°; thumb_tip_4 <1.4 mm, flex <2.1°; index_tip_8 <1.2 mm, flex <1.9°; palm_center_9 <2.4 mm.

### 11.4 Camera Calibration (Ultra-Precision)

- **Checkerboard:** 9×6, square 0.0254 m (1 inch), printed <0.1 mm jitter; min/max corners 48/54; corner_subpix QUADS; rms_reprojection_target 0.25, max_reproj_error 0.8.
- **Distortion:** radial_tangential 5-param (k1,k2,p1,p2,k3); Levenberg–Marquardt; convergence_eps 1e-9.
- **Rolling shutter:** scan TOP_TO_BOTTOM; scan_time_per_line 1/120/720 (720p 120 fps); gyro integration if 6-axis IMU available.

### 11.5 Scale Metrology (World Space)

- **Primary:** biacromial_width landmarks 11–12 = 0.42 m ± 0.03 m (fusion weight 0.72).
- **Secondary:** hand_palm_width 0_5_17 = 0.085 m ± 0.008 m (weight 0.18).
- **Tertiary:** eye_inner 33_362 = 0.033 m ± 0.003 m (weight 0.10).
- **Recalibration:** threshold 0.018 m (4.3 cm drift); update_interval 1000 ms.
- **Anthropometric:** biacromial mean 0.418 m, std 0.032, 95% [0.355, 0.481]; hand_length mean 0.189 m (landmark 0→20); middle_finger 0.078 m (8→6).

### 9.6 Intersection Detection (Millimetre)

- **Fingertip ray:** origin landmark_tip_xyz, direction tip→pip normalized, length 0.035 m, threshold 0.006 m, recursive true.
- **Capsule cast:** radius 0.0132 m, height 0.028 m, segments 12; sweep_distance 0.025 m; intersection_margin 0.002 m; backface_cull false.
- **Volume:** hand_mesh 21-capsule union or convex hull; BVH SAH 12-leaf min; epsilon 0.0015 m.
- **Hand capsules (84 total):** phalanx radius 0.0115 m (or 0.0118), length [0.022, 0.045] m; metacarpal radius 0.0142 m, length 0.070 m.

### 11.7 Grab Confirmation Pipeline

- Phase1: capsule_intersect_any.
- Phase2: grip_pattern match power / precision / hook.
- Phase3: stiffness_test push 5 N pull 5 N 0.1 s.
- Phase4: snap_distance <3.2 mm joint stable.
- **Timeout:** 0.35 s.

### 11.8 Physics Attachment (After Grab)

- **Servo spring:** stiffness 8500, damping 220, target_distance 0.0015 m, max_force 32 N.
- **Torque constraints:** max_torque 16.2 Nm, axis_lock [0.95, 0.95, 1.0], slippage_friction 0.68.

### 11.9 Grip Physiology

- **MVC (N) / endurance (s):** thumb_ip 12.4/48, index_distal 17.8/62, middle_mcp 21.3/78, ring_pip 14.6/58, little_dip 9.8/42.
- **Fatigue (Hill):** force_velocity eccentric 1.2× isometric; fatigue_rate 0.85^time_constant; tremor_threshold >82% MVC for 22 s; tremor 8.2 Hz, 0.62 mm peak.
- **Slippage:** dry_skin_metal 0.78, sweat_skin_metal 0.58, oil_contaminated 0.34, glove_nitrile 0.92.

### 11.10 IK / Gesture Precision (Reference Targets)

- **MC4 crimp:** thumb_tip_4 world_precision 0.0009 m, index_tip_8 0.0007 m, required_distance 0.011 m, force_vector [0, -45, 0] N.
- **Phillips screwdriver:** tip_landmark 8, endpoint_precision 0.0004 m, wrist_16 orientation tolerance 2.5°, torque_axis z_local.
- **BGA tweezer:** dual_tip_precision 0.0003 m, solder_ball_contact 0.00012 m, placement_tolerance 0.000015 m.

### 11.11 Latency Budget (60 fps)

- camera_grab ≤6.8 ms; mediapipe_inference ≤22 ms (M1/iPhone15); landmark_projection ≤2.1 ms; scale_transform ≤1.2 ms; collision_dispatch ≤3.8 ms; physics_step ≤4.2 ms (120 Hz); render_submit ≤14 ms; **total ≤54 ms**.
- **Temporal:** extrapolation_frames 2, velocity dt=1/60, acceleration_limit 15; jitter: landmark_ema alpha 0.74 beta 0.68, pose_slerp 0.82, velocity_smoothing lowpass 8 Hz.

### 11.12 Validation Metrics (Absolute)

- **Spatial:** fingertip_rms ≤7.8 mm world; wrist_pose ≤2.4°; palm_center ≤4.2 mm; inter_phalanx_angle ≤2.1°.
- **Temporal:** frame_jitter_rms ≤1.8 mm; drift_30s ≤12 mm; velocity_tracking_error ≤18 cm/s.
- **Interaction:** grab true_positive ≥97.2%, false_positive ≤0.8%; torque_prediction R² 0.94; slippage_force_accuracy ±12%.

---

## 12. Rendering & Post-Processing Standards (Web)

Rendering **SHALL** meet the resolution and frame-rate tiers in §1 and run in the browser on **WebGL 2** (or WebGPU with fallback). The following are reference settings for web engines (e.g. Three.js, Babylon.js, PlayCanvas).

### 12.1 Renderer Config (Web)

- **Context:** **WebGL 2** with `alpha: false`, `antialias: true`, `powerPreference: 'high-performance'`; or **WebGPU** with equivalent options and **WebGL 2 fallback** when WebGPU is unavailable.
- **Output:** sRGB (or equivalent) encoding; **SHALL** handle `devicePixelRatio` for sharp rendering on high-DPI screens.
- **Lighting:** Physically correct lights and tone mapping (e.g. ACES Filmic, exposure 1.0) for HD+.
- **Shadows:** Shadow mapping with soft shadows (e.g. PCF); resolution ≥2048 (HD) or 4096 (UHD); configurable bias to avoid artifacts.

### 12.2 Resolution Pipeline

- **Render target:** scale 1.5, samples 8, encoding sRGBEncoding.
- **Texture atlas:** albedo 2048, normal 2048, pbr 2048, displacement 1024.

### 12.3 Post-Processing (Reference Pipeline)

**Standard stack:**

- SMAAEffect(threshold 0.05); FXAAEffect(); BloomEffect(threshold 0.92, radius 0.7); DepthOfField(fstop 1.8); ChromaticAberration(offset [0.001, 0.002]); FilmGrain(intensity 0.14); ACESFilmic(exposure 1.1).

**Cinematic pipeline (full):**

1. **SMAA:** edgeThreshold 0.02, maxSearchSteps 32, maxSearchStepsDiag 16.
2. **TemporalAA:** samples 8, feedback 0.92, accumulation 0.7.
3. **BloomPass:** threshold 0.88, strength 1.2, radius 0.85, passes 7.
4. **BokehDoF:** fstop 1.4, focalDistance 1.2, focusRange 0.8, hex_blades 9, maxBlur 0.015.
5. **LensDistortion:** distortion [0.12, -0.28], vignette 0.78, chromatic [0.0015, -0.0011].
6. **FilmGrain35mm:** intensity 0.16, size 0.42, lumaInfluence 0.7.
7. **ColorMatrix:** saturation 1.12; matrix 4×4 (e.g. 1.05,0.02,-0.03; -0.01,1.08,0.01; 0.02,-0.04,1.06).
8. **ACESFilmicToneMap:** exposure 1.18, whitePoint 1.2.

### 12.4 Lens Optics (Optional)

- **Prime 50 mm f/1.4:** focalLength 50, fStop 1.4, bokehShape 9_blade_rounded, distortion [0.012, -0.028], chromaticAberration [0.0012, -0.0008], vignette 0.75.
- **Industrial 24–105 f/2.8:** focalLength 70, fStop 2.8, 7_blade, distortion [0.008, -0.015].
- **Motion blur (handheld):** shutter 1/60 s, samples 16, maxSampleLength 0.015, jitter 0.8.

### 12.5 LOD & Memory

- **LOD0 mobile:** faces <500, texture 256.
- **LOD1 tablet:** faces <2000, texture 512.
- **LOD2 desktop:** faces <8000, texture 1024.
- **LOD3 VR:** faces <20000, texture 2048.
- **Memory budget:** geometry 128 MB, texture 512 MB, physics 32 MB.
- **Performance tiers:** high_end 60 fps, 5000 objects, 200 physics bodies; mid_range 45 fps, 80 bodies, LOD aggressive; low_end 30 fps, kinematic_only, KTX2/basis.

---

## 13. Validation, Assessment & Performance

### 13.1 ISO-14118 LOTO Compliance

- **Step 2:** rotation.z ∈ [π/2 ± 0.05] rad (90° ± ~3°).
- **Step 3:** hasp distance to stem < 0.02 m.
- **Step 6:** pressure < 0.1 bar (zero energy).
- **Step 7:** allStepsComplete && startButtonPressed.

### 13.2 Tolerance Matrix (By Trade)

| Trade | rot (rad) | pos (m) | force / other |
|-------|-----------|---------|----------------|
| precision (welding, electrical) | 0.087 | 0.005 | force 0.5; voltage 1; bead 0.001, penetration 0.002 |
| heavy (mechanical, plumbing) | 0.17 | 0.02 | — |
| overview (safety) | 0.52 | 0.05 | — |

### 13.3 Settling Validation

- **Linear velocity:** <0.02 m/s.
- **Angular velocity:** <0.05 rad/s (or 0.1 in some specs).
- **Acceleration:** <0.1 m/s².
- **Settle window:** 0.5–0.8 s before marking step complete.

### 13.4 Realism Pass Criteria (LOTO)

- **Grip:** thumb_opposition for valve; two_finger_minimum hasp squeeze; shackle_clearance; pronation_feedback wrist angle.
- **Force feedback:** torque 8–15 Nm range; collision impulse velocity proportional; magnetic_snap 50 N peak; thermal_conduction metal 42°C.
- **Ergonomics:** awkward_posture strength penalty; fatigue after 90 s; tool_clearance collision response; natural_hand_release physics.

### 13.5 Gaze & Attention (Optional)

- **Eye tracking:** foveal_region 0.5° diameter; peripheral 60° motion; head_gaze_weight 70%; task_lighting auto_brighten_gaze_target.
- **Guidance:** subtle_highlights emissive_pulse 0.3 Hz; next_step_preview wireframe_glow 20% opacity; error_correction pulsing_red faulty component.

### 13.6 Ergo & Fatigue

- **Posture penalties:** awkward_wrist -15% grip after 30 s; thumb_overextension tremor 8 Hz 0.5 s; prone strength_reduction 25%.
- **Fatigue:** endurance_limit 90 s continuous torque; recovery 2× exertion duration; tremor_onset force >80% max for 15 s.
- **Tool clearance:** knuckle_collision halt; palm_interference reduced_precision 50%.

### 13.7 Hyper-Realism Validation (Expert Scrutiny)

- **Surface:** directional specular at grazing angles; F0 Fresnel; microfacet distribution; multi-scatter metals.
- **Shadow:** contact at bolt gaps 0.1 mm; penumbra fluorescent correct; no bias artifacts.
- **Optics:** hexagonal bokeh; chromatic/lens distortion; film grain uniform.
- **Physics visuals:** weld spatter cooling; fluid jet Bernoulli; thermal blackbody; smoke buoyancy/turbulence.
- **Expert distance:** mechanical_engineer 0.5 m; industrial_designer 1.0 m; maintenance_technician 0.3 m.

---

## 14. Trade-Specific Physics & Hypnotics

### 14.1 Electrical

- **Arc:** voltage_breakdown 3 kV/mm air, temperature 6000 K; Paschen V = f(P×d); breakdown_field 3e6 V/m; shockwave 0.02 s radius 0.1 m.
- **Crimp:** deformation yield 250 MPa, contact_resistance 0.1 mΩ, pullout_force 450 N.
- **Wire bending:** stiffness 1500, damping 0.95, plastic_deform strain > 0.02.
- **Visual:** arc_flash_glow_bloom; **Audio:** plasma_hiss 8 kHz.

### 14.2 Welding

- **Molten:** viscosity 0.006, surface_tension 1.8, cooling 200°C/s, thermal_emissive temp/2000.
- **Spatter:** velocity [2, 8], stick_probability 0.3; cooling trajectory.
- **Visual:** molten_pool_subsurface; **Audio:** plasma_hiss 8 kHz.

### 14.3 Plumbing

- **Fluid:** Darcy–Weisbach pipe flow; Bernoulli jet leak; Hagen–Poiseuille pressure drop; cone_angle 28°, surface_tension 0.073.
- **Pipe stress:** youngs_modulus 200e9 Pa, yield_strength 250e6 Pa, poissons 0.3, thermal_expansion 17.3e-6, safety_factor 1.5.
- **Thread gall:** stick_slip 0.15, lubrication_reduction 0.4.
- **Visual:** fluid_splash_particles; **Audio:** pipe_burst 200 Hz.

### 14.4 HVAC

- **Airflow:** turbulence k-epsilon; velocity streamlines; pressure color_mapped.
- **Duct flex:** bending_stiffness 1200; vibration_modes [20, 45, 120] Hz.
- **Visual:** air_velocity_streamlines; **Audio:** duct_vibration 45 Hz.

### 14.5 Mechanical / LOTO

- **Visual:** metal_stress_deform; **Audio:** wrench_tighten 1200 Hz, valve scraping FM sweep, hasp plastic/spring, padlock 2 kHz, magnetic 80 Hz.

---

## 15. Quick Reference Checklist

- [ ] **World & gravity** defined; assembly coordinates in metres; roof_pitch / wind where relevant. **Browser:** HTTPS delivery; consistent units; no blocking main thread on load.
- [ ] **Geometry:** dimensions and segments per table (128/96/64 for close-up); instancing for bolts/repeats; threading pitch 0.0018, hex 0.013/0.015; manufacturing artifacts (milling, chatter, porosity, ejector pins).
- [ ] **PBR:** per-object from material tables (zincalume, pine, PV glass, aluminium, titanium, OLED, FR4, solder mask; AISI304, brass, ABS, neoprene); microfacet where needed; subsurface for plastics/skin; HDRI hierarchy 8k/4k/2k/1k.
- [ ] **Surface imperfections:** counts and sizes (rust pits, dents, fingerprints, scratches, micro_abrasions, swirl_marks); wear (finger oil 0.018 roughness, 300 s fade; thread gall; contaminants); contact shadow bias table and penumbra.
- [ ] **Environment:** HDRI + bounce lighting; dynamic wind/rain for outdoor PV (gust 0.8 Hz, rain 3500 particles, 4.2 mm/hr); task lighting ISO-14118 style.
- [ ] **Physics:** Fixed timestep (e.g. 1/60 or 1/120 s), stable constraints; mass, CoM, friction, restitution per object; valve torque curve [8.2, 3.1, 11.8], constraint π/2 ± 0.01; pressure 10*exp(-angle*0.5), Step 6 <0.1 bar; grab reach 0.4 m / radius 0.03 m (or palm 0.08 / finger 0.012 for hand); LOTO step 2/3/4 grip and force specs; tool DB (wrench, Phillips, pliers, torque 12 Nm).
- [ ] **Acoustic/haptic:** fundamental freq and duration per event (MC4 1850 Hz, torque 4200 Hz, Phillips 2800 Hz; valve/hasp/padlock/magnetic); haptic intensity/frequency/duration and surface scan bands; spatial occlusion and material impact table.
- [ ] **Procedural effects:** arc (6000 K, 450 particles, shockwave 1450 m/s); thermal runaway (950 K, 2800 particles); conformal 40 µm, 0.032 N/m; weld spatter; fluid jet Bernoulli; smoke buoyancy; blackbody table; grip warmth 42°C.
- [ ] **Hand/pose tracking (if used):** getUserMedia + in-browser pipeline (e.g. MediaPipe); meet §1.4 latency (≤54 ms); scale calibration and collision per §11; camera only after user gesture.
- [ ] **Rendering (browser):** WebGL 2 (or WebGPU + fallback); requestAnimationFrame; Page Visibility to pause when tab hidden; handle context loss; §1 resolution and frame rate; PBR + env lighting; shadows; post-process; LOD and memory per §1.1 / §12.5.
- [ ] **Validation:** ISO-14118 step 2/3/6/7; tolerance matrix by trade; settling vel/angular/settle_window; grip/force/ergo pass criteria; hyperrealism surface/shadow/optics/physics and expert distances.
- [ ] **Trade-specific:** electrical arc/crimp/wire; welding molten/spatter; plumbing fluid/pipe stress/gall; HVAC airflow/duct; mechanical friction/mass/tool; hypnotics (visual + audio) per trade.

---

*This guidance focuses on **online, browser-based** development: HD quality, resolution, and interactivity standards for simulations delivered over the web (WebGL/WebGPU, no install). Use it for SETA-accreditable, production-grade QCTO training sims that run in standard browsers on desktop and tablet.*
