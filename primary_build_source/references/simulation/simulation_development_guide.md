# Simulation Development Guide
## Artisan Training Platform — Foundation Curriculum
### FND-TT-01 & FND-IW-01 | Browser-Based 3D Training Simulations
#### Technical Specification v1.0 | For Development Teams

---

> **Scope and Purpose**
>
> This guide is the definitive technical reference for all five 3D simulation exercises across the two Foundation modules. It translates curriculum learning objectives, assessment requirements, and pedagogical design into fully specified development instructions — covering scene geometry, PBR materials, physics configuration, interaction systems, audio-haptic feedback, MediaPipe hand-tracking integration, assessment validation logic, and quality gates.
>
> Every value in this document is production-grade. Numbers are sourced from manufacturer specifications, anthropometric databases, physical first principles, and the ISO/SETA/QCTO compliance standards applicable to SETA-accredited vocational training simulations.
>
> **Target runtime:** Browser-based, WebGL 2 / WebGPU, delivered over HTTPS. No install. Sustained 60 fps at 1920×1080 on mid-range desktop (Chrome, Firefox, Edge, Safari current).

---

## Contents

- [Part 1 — Architecture and Shared Systems](#part-1)
- [Part 2 — FND-TT-01 Sim 01: The Workshop Power Failure](#part-2)
- [Part 3 — FND-TT-01 Sim 02: The Dead Panel](#part-3)
- [Part 4 — FND-IW-01 Sim 01: First Day on Site](#part-4)
- [Part 5 — FND-IW-01 Sim 02: Suiting Up](#part-5)
- [Part 6 — FND-IW-01 Sim 03: Setting Out and Fixing](#part-6)
- [Part 7 — Assessment and Validation Engine](#part-7)
- [Part 8 — Performance, LOD, and Delivery](#part-8)
- [Part 9 — Quality Gates and Sign-Off Checklist](#part-9)

---

<a name="part-1"></a>
# PART 1 — ARCHITECTURE AND SHARED SYSTEMS

---

## 1.1 Technology Stack

### Rendering
- **Primary:** Three.js (r156+) with WebGL 2 context
- **Context config:** `alpha: false`, `antialias: true`, `powerPreference: 'high-performance'`
- **Output encoding:** sRGB; `renderer.outputColorSpace = THREE.SRGBColorSpace`
- **Tone mapping:** `THREE.ACESFilmicToneMapping`, exposure `1.18`
- **Shadow maps:** `THREE.PCFSoftShadowMap`; shadow map resolution 2048 (HD) / 4096 (HD+); bias `-0.0001`; normalBias `0.001`
- **WebGPU:** Feature-detect and use where available; maintain WebGL 2 fallback branch
- **devicePixelRatio:** `Math.min(window.devicePixelRatio, 2)`; handle resize via ResizeObserver
- **Tab visibility:** `document.addEventListener('visibilitychange', ...)` — pause `requestAnimationFrame` loop when hidden (Page Visibility API)
- **Context loss:** Handle `webglcontextlost` / `webglcontextrestored`; prompt reload on unrecoverable loss

### Physics
- **Engine:** Rapier.js (WASM) — async init via dynamic import; run on fixed timestep 1/60 s
- **Solver:** 32 position iterations, 8 velocity iterations
- **CCD:** Enabled for objects with velocity > 0.5 m/s or mass < 0.1 kg
- **Sleep threshold:** linear 0.01 m/s, angular 0.02 rad/s

### Input
- **Primary:** Pointer Events (mouse + touch unified model) mapped to 3D raycasts in metres
- **Keyboard:** Full keyboard navigation for all assessment interactions (WCAG 2.1 AA)
- **Optional – Hand Tracking:** MediaPipe Holistic (in-browser JavaScript pipeline) via `getUserMedia`; camera permission requested only after user gesture; graceful degradation to pointer if denied
- **Optional – Gamepad:** Gamepad API for haptic feedback delivery; no core interaction dependency

### Audio
- **Engine:** Web Audio API; all sound triggered only after user gesture (autoplay policy compliance)
- **Spatialization:** `PannerNode` with `panningModel: 'HRTF'`, distance model `inverse`, refDistance 1.0, rolloffFactor 1.2
- **Occlusion:** Low-pass filter (cutoff 440 Hz at 5 m) applied when listener is occluded from source by geometry

### Assets
- **Format:** glTF 2.0 / GLB for all geometry assets; KTX2/Basis Universal texture compression for LOD1 and below
- **Delivery:** HTTPS, long-lived cache headers for immutable assets; lazy-load secondary assets post-first-render
- **Offline:** Core scene geometry and textures pre-cached via Cache API for offline resilience

---

## 1.2 Renderer and Post-Processing Pipeline

All simulations share the same post-processing stack. Sequence and parameters:

```
1. SMAA           edgeThreshold: 0.02 | maxSearchSteps: 32 | maxSearchStepsDiag: 16
2. TemporalAA     samples: 8          | feedback: 0.92       | accumulation: 0.7
3. BloomPass      threshold: 0.88     | strength: 1.2        | radius: 0.85 | passes: 7
4. BokehDoF       fstop: 1.4          | focalDistance: 1.2   | focusRange: 0.8
                  hex_blades: 9       | maxBlur: 0.015
5. LensDistortion distortion: [0.12, -0.28] | vignette: 0.78
                  chromatic: [0.0015, -0.0011]
6. FilmGrain      intensity: 0.16     | size: 0.42           | lumaInfluence: 0.7
7. ColorMatrix    saturation: 1.12
                  matrix: [1.05,0.02,-0.03,0 | -0.01,1.08,0.01,0 |
                           0.02,-0.04,1.06,0 | 0,0,0,1]
8. ACESFilmic     exposure: 1.18      | whitePoint: 1.2
```

Camera model for interactive scenes: `industrial_24_105_f2_8` — focalLength 70 mm, fStop 2.8, 7-blade bokeh, distortion `[0.008, -0.015]`.

---

## 1.3 PBR Material Standard

All interactive objects use `THREE.MeshPhysicalMaterial`. Reference material database shared across all five simulations:

### Structural Materials

| Identifier | base_color (RGB) | metalness | roughness | normal_scale | clearcoat | IOR | Notes |
|-----------|-----------------|-----------|-----------|--------------|-----------|-----|-------|
| `zincalume_corrugated` | [0.65,0.65,0.68] | 0.92 | 0.18 | 1.2 | 0.0 | 1.33 | Corrugation via displacementMap |
| `treated_pine` | [0.45,0.32,0.22] | 0.02 | 0.75 | 2.1 | 0.15 | — | anisotropy: 0.6 |
| `bare_concrete` | [0.52,0.50,0.48] | 0.0 | 0.88 | 3.2 | 0.0 | — | AO baked |
| `painted_brick_aged` | [0.58,0.46,0.38] | 0.0 | 0.82 | 2.8 | 0.0 | — | Peeling clearcoat: 0.06 |
| `epoxy_floor` | [0.72,0.74,0.68] | 0.0 | 0.12 | 0.8 | 0.35 | 1.46 | |
| `galvanised_steel` | [0.72,0.74,0.78] | 0.88 | 0.22 | 1.4 | 0.1 | — | Spangle normals |
| `aisi304_stainless` | [0.56,0.57,0.56] | 0.98 | 0.04 | — | 0.95 | 2.42 | anisotropy: 0.85 |
| `aluminium_anodized` | [0.82,0.85,0.88] | 0.95 | 0.12 | — | 0.25 | — | sheen: 0.1 |

### Electrical / Safety Materials

| Identifier | base_color (RGB) | metalness | roughness | transmission | IOR | Notes |
|-----------|-----------------|-----------|-----------|--------------|-----|-------|
| `pv_glass` | [0.95,0.96,0.98] | 0.0 | 0.02 | 0.92 | 1.52 | thickness: 0.0035 |
| `abs_polycarbonate` | [0.12,0.14,0.18] | 0.0 | 0.96 | 0.92 | 1.49 | SSS: scatter 0.85 |
| `neoprene_60A` | [0.11,0.13,0.12] | 0.0 | 0.99 | 0.0 | — | normalScale: [2.8,2.8] |
| `brass_c36000` | [0.71,0.65,0.41] | 0.94 | 0.16 | 0.0 | — | clearcoat: 0.88 |
| `hi_vis_yellow` | [0.92,0.85,0.0] | 0.0 | 0.78 | 0.0 | — | emissive boost in shadow |
| `hi_vis_orange` | [0.96,0.42,0.0] | 0.0 | 0.76 | 0.0 | — | retroreflective strip: separate mesh |
| `safety_helmet_white` | [0.94,0.94,0.94] | 0.0 | 0.42 | 0.0 | — | clearcoat: 0.18 |
| `nitrile_glove_blue` | [0.18,0.28,0.62] | 0.0 | 0.88 | 0.0 | — | SSS active |
| `leather_boot` | [0.22,0.16,0.11] | 0.0 | 0.72 | 0.0 | — | anisotropy: 0.3 |

### Microfacet BRDF Configuration (applied globally)

- **Distribution:** GGX Trowbridge–Reitz; alphaRemap from roughness²
- **Visibility:** Smith GGX correlated
- **Fresnel:** Schlick optimised
- **Multi-scatter:** `(1 - reflectance) / 1.5`
- **Measured overrides:** milled_aluminum alphaG 0.12/alphaD 0.08; sandblasted_steel 0.28/0.35; polished_brass 0.045/0.032

### Subsurface Scattering (plastics and PPE)

- **ABS/polycarbonate PPE:** scatterProfile backward 0.7; scatterDistance 0.015 m; scatterColor [0.95,0.96,0.97]
- **Nitrile gloves:** forward scatter 0.4; scatterDistance 0.028 m; scatterColor [0.18,0.28,0.65]
- **Skin (if learner hands rendered):** epidermis 0.0012 m scatter 0.85; dermis 0.0021 m scatter 0.92

---

## 1.4 Surface Imperfections (Procedural — Applied Per Material)

All interactive surfaces carry procedurally generated wear. Apply via `normalMap` + `displacementMap` generated from noise functions at material creation time.

**Manufacturing artifacts (universal):**
```
milling_marks:     direction: tool_path_vector | depth: 0.0003 m
                   spacing: 0.0012 m | normalStrength: 2.1
tool_chatter:      frequency: 120 Hz | amplitude: 0.00015 m | phase_variation: 0.3
casting_porosity:  cluster_size: 0.002 m | density: 0.03 | displacement: -0.0002 m
ejector_pin_marks: diameter: 0.0021 m | depth: 0.0004 m | count: 4/cm²
```

**Wear patterns (triggered by interaction):**
```
finger_oil_transfer:  roughness_reduction: 0.018 per grab | sheen_increase: 0.12
                      fade_curve: exponential_300s | visible_under_grazing: true
thread_galling:       material_transfer: aluminum→steel 0.001 m
                      friction_increase: 0.22 | localised_roughness: 0.75
surface_contaminants: cutting_fluid_residue roughness 0.15 bloom
                      dust_accumulation AO multiplier 1.2 at edges
```

**Per-surface imperfection counts:**

| Surface | Rust pits | Dents | Scratches | Fingerprints |
|---------|-----------|-------|-----------|-------------|
| `zincalume_5yr` | 2450 (0.0008–0.003 m dia, depth 0.00012 m) | 18 (dia 8–25 mm, depth 0.3–0.8 mm) | — | — |
| `aluminium_anodized_frame` | — | — | 42 (len 2–12 mm) | 12 (oval 8×4 mm, opacity 0.15–0.35) |
| Tool handles | — | 6–12 (dia 3–8 mm) | 18–35 | 8–24 |

---

## 1.5 HDRI Hierarchy and Environment Lighting

```
Primary (outdoor):   overcast_cape_town_4k.hdr   — Sims 01 (TT), 02 (TT)
Primary (indoor):    industrial_workshop_4k.hdr   — Sims 01, 02, 03 (IW)
Secondary:           machine_shop_overhead_4k.hdr
Ambient:             workshop_diffuse_2k.hdr
Specular:            chrome_reflection_1k.hdr
```

**LOD streaming config:** `mipBias: -0.7`, `anisotropy: 16`; pre-filtered mipmap count 9; specular levels `[0, 0.33, 0.66, 1.0]`.

**Task lighting (all indoor sims):** Overhead fluorescent strips at 5000 K / 0.92 intensity; directional task spotlights on key interaction zones auto-brighten 20% when gaze dwell > 1.2 s on target (optional gaze-driven feature).

---

## 1.6 The QTB Clipboard Widget

The QTB Clipboard is a persistent, non-diegetic UI overlay that appears in every simulation. It is the primary formative data-collection tool and feeds the assessment record. It must be implemented consistently across all five simulations.

**Layout:** Fixed to bottom-right of viewport. Three input areas labelled Q, T, B with line-height sufficient for 3–4 sentences each. `textarea` elements with character limits (500 per field). A timestamp is auto-applied on first keypress per field. A "Lock & Continue" button submits the entry and prevents editing.

**Persistence:** QTB entries POST to the platform assessment API on submission; stored server-side against learner session. Client-side: IndexedDB backup to prevent data loss on navigation.

**Assessment integration:** The submitted T field is parsed for presence of physics-grounded observation language. The Q field is parsed for challenge verbs (`assumed`, `not confirmed`, `could be`, `before testing`). The B field is parsed for reasoning connectives (`therefore`, `which means`, `so the fault`, `because`). Absence of these patterns in any field triggers a soft-flag in the assessment record for facilitator review.

**Hint integration:** If a learner has not touched a QTB field after 180 s of simulation time, a Level 1 hint fires: *"Stop and apply the T step: what do you know is physically true right now — from what you can see or measure?"*

---

## 1.7 MediaPipe Hand Tracking Pipeline

Hand tracking is an optional enhancement layer. All simulations **must** be fully functional via pointer events. When a camera is available and permission is granted, the MediaPipe pipeline activates.

### Model Configuration
```json
{
  "model_complexity": 1,
  "max_num_hands": 2,
  "min_detection_confidence": 0.88,
  "min_tracking_confidence": 0.94,
  "static_image_mode": false,
  "smooth_landmarks": true,
  "enable_segmentation": true,
  "min_hands_presence": 0.75,
  "hand_presence_threshold": 0.5
}
```

### Holistic Point Coverage (553 points total)
- **Pose:** 33 landmarks — `pose[11,12]` (shoulders) used as primary scale anchor; `pose[15,16]` (wrists) used for reach validation
- **Hands:** 21 landmarks × 2 = 42 — full 21-DOF kinematic chain per hand
- **Face:** 478 landmarks — used for attention/gaze analysis only; not used for interaction collision

### Scale Metrology (World Space Calibration)
```
Primary   (weight 0.72): biacromial landmarks 11→12 = 0.42 m ± 0.03 m
Secondary (weight 0.18): hand_palm landmarks 0_5_17 = 0.085 m ± 0.008 m
Tertiary  (weight 0.10): inner_eye landmarks 33_362 = 0.033 m ± 0.003 m
Recalibration threshold: 0.018 m drift → re-run scale fusion
Update interval: 1000 ms
```

### Landmark Accuracy Targets

| Landmark | Position error | Rotation error |
|----------|---------------|----------------|
| wrist_0 | < 3.2 mm | < 2.1° |
| thumb_mcp_1 | < 2.8 mm | < 3.2° |
| thumb_tip_4 | < 1.4 mm | < 2.1° |
| index_tip_8 | < 1.2 mm | < 1.9° |
| palm_center_9 | < 2.4 mm | — |

### Collision Geometry (84 capsules per hand)
```
Phalanges:    proximal 14, middle 14, distal 14 capsules
              radius: 0.0118 m | length: [0.022, 0.045] m
Metacarpals:  5 capsules
              radius: 0.0142 m | length: 0.070 m
Collision:    backface_cull false | margin: 0.0012 m | segments: 12
```

### Intersection Detection Hierarchy
```
Level 1 – Fingertip ray:
  origin: landmark_tip_xyz
  direction: tip→pip normalized
  length: 0.035 m | threshold: 0.006 m | recursive: true

Level 2 – Capsule sweep:
  radius: 0.0132 m | height: 0.028 m | segments: 12
  sweep_distance: 0.025 m | intersection_margin: 0.002 m

Level 3 – Volume intersect:
  hand_mesh: 21-capsule union → convexHull
  BVH: SAH 12-leaf minimum | epsilon: 0.0015 m
```

### Grab Confirmation Pipeline
```
Phase 1: capsule_intersect_any
Phase 2: grip_pattern → match power | precision | hook | tool_grasp
Phase 3: stiffness_test → push 5 N, pull 5 N over 0.1 s
Phase 4: snap_distance < 3.2 mm joint stable
Timeout: 0.35 s
```

### Physics Attachment (post-grab)
```
servo_spring:  stiffness 8500 | damping 220 | target_distance 0.0015 m | max_force 32 N
torque:        max_torque 16.2 Nm | axis_lock [0.95,0.95,1.0] | slippage_friction 0.68
```

### Grip Physiology Model
```
MVC / endurance:  thumb_ip 12.4 N / 48 s
                  index_distal 17.8 N / 62 s
                  middle_mcp 21.3 N / 78 s
                  ring_pip 14.6 N / 58 s
                  little_dip 9.8 N / 42 s

Fatigue (Hill):   fatigue_rate 0.85^time_constant
                  tremor onset > 82% MVC for 22 s
                  tremor: 8.2 Hz, 0.62 mm peak

Slippage:         dry_skin_metal 0.78 | sweat_skin_metal 0.58
                  oil_contaminated 0.34 | glove_nitrile 0.92
```

### Latency Budget (60 fps — total ≤ 54 ms)
```
camera_grab:          ≤ 6.8 ms
mediapipe_inference:  ≤ 22.0 ms
landmark_projection:  ≤ 2.1 ms
scale_transform:      ≤ 1.2 ms
collision_dispatch:   ≤ 3.8 ms
physics_step:         ≤ 4.2 ms (at 120 Hz substep)
render_submit:        ≤ 14.0 ms
total:                ≤ 54.0 ms
```

### Jitter Stabilisation
```
landmark_ema:     alpha 0.74, beta 0.68
pose_slerp:       0.82
velocity_smooth:  low-pass at 8 Hz
temporal predict: extrapolation_frames 2 | Kalman gain: confidence-weighted
```

---

## 1.8 Haptic Feedback System

Delivered via Gamepad API where available. Fallback: audio-only feedback on the same trigger.

### Frequency Response Bands
```
metal_contact:    [200, 800] Hz
plastic_flex:     [100, 300] Hz
magnetic_snap:    [40,   60] Hz
valve_torque:     variable 2–20 Hz (proportional to resistance)
rubber_soft:      [55,  135] Hz | decay 1.1
```

### Force Patterns
```
grab_resistance:   rigid 25 N limit
rotation_torque:   proportional to angle × angular_velocity
collision_impact:  impulse-based, velocity-proportional
magnetic_snap:     smooth pull to 50 N peak
thermal feedback:  metal_conduction slow warmup 42°C over 10 s
                   plastic: ambient maintained
```

### Impulse-to-Gamepad Mapping
```
intensity:    sqrt(impulse_mag) × 0.45
duration:     min(0.12 s, impulse_mag × 0.02)
frequency:    clamp(80, impulse_mag × 120, 250)

continuous resistance:
  torque_feedback: torque/18.0 × 0.85
  freq modulation: 2–28 Hz torque-proportional
  waveform:        sawtooth_damped
```

---

<a name="part-2"></a>
# PART 2 — FND-TT-01 SIM 01: THE WORKSHOP POWER FAILURE

---

## 2.1 Simulation Brief

**Module:** FND-TT-01 | **Unit:** 1.2 (IPO) and 1.3 (Symptoms vs. Root Causes)
**Assessment weight:** 20% of module total | **Pass threshold:** 65%
**Notional time:** 45–60 minutes | **Resubmission:** One (if 55–64%)

**Pedagogical purpose:** The learner must apply the IPO model to map a simple electrical distribution system, then use the 5 Whys and System Map Clipboard to trace a supply-chain fault from symptom (no lighting) to root cause, entering QTB reasoning at each decision point.

**Scenario:** A township electrical workshop. The overhead fluorescent strip lights are off. The learner must walk the supply chain from the main DB through to the light fittings, identify the fault location using structured elimination, and complete a QTB entry at each test point before taking action.

---

## 2.2 Scene Geometry

### Global Scene
```json
{
  "world_origin": [0, 0, 0],
  "gravity": [0, -9.81, 0],
  "room_dimensions": [8.0, 3.2, 6.0],
  "floor_level": 0.0
}
```

### Structural Elements

**Floor — bare concrete:**
```
dimensions:  [8.0, 0.12, 6.0]
segments:    [64, 1, 48]
position:    [0, -0.06, 0]
material:    bare_concrete
surface:     3200 procedural cracks (length 0.02–0.18 m, depth 0.0002 m)
             oil stain patches: 8 (radius 0.08–0.22 m, roughness +0.35)
```

**Walls — painted brick (aged):**
```
wall_north:  [8.0, 3.2, 0.25] | position [0, 1.6, -3.0]
wall_south:  [8.0, 3.2, 0.25] | position [0, 1.6, 3.0]
wall_east:   [0.25, 3.2, 6.0] | position [4.0, 1.6, 0]
wall_west:   [0.25, 3.2, 6.0] | position [-4.0, 1.6, 0]
material:    painted_brick_aged
segments:    [96, 48, 1] per wall face
```

**Ceiling — painted plaster:**
```
dimensions:  [8.0, 0.08, 6.0]
position:    [0, 3.2, 0]
material:    base_color [0.88,0.87,0.84] | roughness 0.72 | metalness 0.0
```

### Workshop Equipment (non-interactive, PBR-dressed scene dressing)

```
bench_01:     [1.8, 0.88, 0.7] | position [-2.5, 0.44, -2.0] | material: steel_galvanised
bench_02:     [1.8, 0.88, 0.7] | position [2.5, 0.44, -2.0]  | material: steel_galvanised
lathe_body:   [1.2, 0.95, 0.6] | position [-3.0, 0.475, 1.5] | material: aisi304_stainless
drill_press:  [0.45, 1.85, 0.5]| position [3.2, 0.925, 1.0]  | material: painted_grey
tool_cabinet: [0.9, 1.4, 0.5]  | position [-3.4, 0.7, 0.0]  | material: steel_galvanised
```

All scene dressing objects: LOD2 desktop (faces < 8000), collider type convex hull, physics: kinematic (immovable), no physics interactions except as ray-stop geometry.

### Electrical Distribution System (interactive)

This is the core interactive geometry. Every component in the supply chain must be individually collider-enabled and independently visible from at least one viewing angle in the scene.

**Main Distribution Board (DB):**
```json
{
  "enclosure": {
    "dimensions": [0.6, 0.8, 0.18],
    "position":   [-3.85, 1.8, -1.5],
    "rotation":   [0, 1.5708, 0],
    "material":   "abs_polycarbonate",
    "door": {
      "dimensions": [0.58, 0.78, 0.008],
      "hinge_axis": "left_vertical",
      "open_angle": 1.5708,
      "physics":   "hinge_constraint"
    }
  },
  "main_breaker": {
    "type":       "MCB_63A",
    "dimensions": [0.018, 0.085, 0.065],
    "position":   [-3.84, 2.0, -1.5],
    "state":      "ON",
    "toggle":     true,
    "segments":   [32, 24, 16]
  },
  "circuit_breakers": [
    {"id": "CB_lighting",  "rating": "16A", "position": [-3.84, 1.95, -1.52], "state": "ON"},
    {"id": "CB_outlets",   "rating": "16A", "position": [-3.84, 1.90, -1.52], "state": "ON"},
    {"id": "CB_equipment", "rating": "32A", "position": [-3.84, 1.85, -1.52], "state": "ON"}
  ],
  "neutral_bar":  {"position": [-3.84, 1.75, -1.52], "lugs": 12},
  "earth_bar":    {"position": [-3.84, 1.70, -1.52], "lugs": 12},
  "fault_state":  "CB_lighting tripped — visible as lever in DOWN position"
}
```

**Cable Run (from DB to first junction box):**
```json
{
  "conduit_pvc_20mm": {
    "path": "spline from [-3.85,2.1,-1.5] along wall to [-3.85,3.0,-1.5] then [0.0,3.0,-2.8]",
    "outer_dia":   0.022,
    "wall_thickness": 0.002,
    "material":    "abs_polycarbonate",
    "cleat_spacing": 0.6,
    "cleat_geometry": {"dimensions": [0.04,0.018,0.025], "material": "abs_polycarbonate"}
  },
  "cable_2_5mm2_3core": {
    "visible_at_terminations": true,
    "conductor_dia": 0.0018,
    "outer_dia":     0.009,
    "insulation_colours": ["red","black","green_yellow"]
  }
}
```

**Junction Box JB-01:**
```json
{
  "enclosure":  {"dimensions": [0.1, 0.1, 0.06], "position": [0.0, 3.0, -2.8]},
  "cover":      {"hinge_axis": "top", "open_angle": 1.5708, "physics": "hinge_constraint"},
  "terminals":  {"count": 6, "type": "Wago_lever_221"},
  "fault_note": "No fault here — correct terminations"
}
```

**Junction Box JB-02:**
```json
{
  "enclosure":  {"dimensions": [0.1, 0.1, 0.06], "position": [2.5, 3.0, -2.8]},
  "terminals":  {"count": 6, "type": "Wago_lever_221"},
  "fault_note": "No fault here"
}
```

**Fluorescent Light Fittings (4x — ceiling-mounted):**
```json
[
  {"id": "light_01", "position": [-2.0, 3.15, 0.0],  "dimensions": [1.2, 0.12, 0.14], "state": "OFF"},
  {"id": "light_02", "position": [ 0.0, 3.15, 0.0],  "dimensions": [1.2, 0.12, 0.14], "state": "OFF"},
  {"id": "light_03", "position": [ 2.0, 3.15, 0.0],  "dimensions": [1.2, 0.12, 0.14], "state": "OFF"},
  {"id": "light_04", "position": [-2.0, 3.15, -2.0], "dimensions": [1.2, 0.12, 0.14], "state": "OFF"}
]
```

Each fitting: material `aged_fluorescent` (base_color [0.82,0.80,0.75], roughness 0.65, metalness 0.12); tube material transmission 0.85, emissive [0.98,0.95,0.88] when active. Diffuser: polycarbonate, roughness 0.45, transmission 0.78.

**Fault embedded:** Circuit breaker `CB_lighting` in the main DB is in the tripped/OFF position. Visual indicator: lever visually DOWN (normally UP = ON). The fault is observable only once the DB door is opened. No other fault exists in the system — the chain is otherwise intact.

---

## 2.3 System Map Clipboard

In this simulation the QTB Clipboard is supplemented by a **System Map Clipboard** — a second persistent UI overlay. The System Map is a blank canvas where the learner sketches the IPO chain by clicking to add nodes and dragging to connect them.

**Nodes available:** Source / Process / Output / Sub-process / Test-point
**Connection type:** Directional arrow
**Annotation:** Each node supports a text label (max 60 characters)
**Save:** Exported as SVG to assessment record on "Submit Map" button

The System Map is not scored for aesthetic quality — it is scored for whether the learner has correctly represented the energy flow path and identified the fault location node.

---

## 2.4 Interaction Model

### Navigation
- First-person walkthrough; WASD + mouse look; pointer-lock on click
- Interaction range: 2.5 m radius from learner camera position
- Interactable objects highlight on hover: `emissiveIntensity: 0.15`, pulse 0.5 Hz
- Tooltip on hover: component name + "Click to interact"

### DB Door
- Click/tap to toggle open/close
- Physics: hinge constraint, axis [0,1,0], damping 8.0, stiffness 0.0
- Audio on open: metal creak 320 Hz, duration 0.4 s, decay 0.8
- QTB prompt fires 3 s after door opens: *"What can you physically confirm about the state of each circuit breaker? Write your T step before you take any action."*

### Circuit Breaker Interaction
- Click to toggle only AFTER QTB T field contains minimum 20 characters AND System Map has at least one node placed
- Attempting to interact without completing QTB: soft block + hint *"Apply the T step first — record what you can observe before you change anything."*
- On correct identification (CB_lighting toggled from OFF to ON): lights activate; emissive bloom fires on all 4 fittings; spatial audio from each fitting position (ballast_hum 120 Hz, tube_flicker_hiss 18 kHz, decay 2.0 s)
- On incorrect action (toggling wrong CB): feedback: *"That circuit remains energised — the lights are still off. What does that tell you about where the fault is?"*

### Multimeter (available as grabbable tool)
```json
{
  "model":      "digital_multimeter_fluke_style",
  "position":   [2.8, 0.9, -1.9],
  "dimensions": [0.18, 0.09, 0.03],
  "material":   "abs_polycarbonate | display: emissive",
  "probes": {
    "red_lead":   {"length": 0.8, "tip_dia": 0.003},
    "black_lead": {"length": 0.8, "tip_dia": 0.003}
  },
  "physics": {
    "mass": 0.38,
    "friction_static": 0.65,
    "grab": "pointer_click + drag"
  },
  "interaction": "place probe tips on terminal → voltage reading appears on display"
}
```

Voltage readings returned at each test point are correct per the fault state. A voltage reading at the supply side of `CB_lighting` returns 230V; at load side returns 0V — confirming the CB is open.

---

## 2.5 Lighting Configuration

```
ambient:     HemisphereLight [0.88,0.86,0.82] / [0.32,0.30,0.28] | intensity 0.4
             (simulates overcast daylight leaking through skylights)
skylight:    DirectionalLight through 2 skylights [0.95,0.93,0.88]
             position [0,8,0] | castShadow true | mapSize 2048
window_leak: 3× PointLight at wall openings | color [0.9,0.88,0.78] | intensity 0.6
task_bench:  2× SpotLight above benches [0.95,0.90,0.78] | angle 0.4 | penumbra 0.3
```

When lights are repaired: all 4 FluorescentLight SpotLights activate:
```
color:    [0.98,0.97,0.90] | intensity 2.2 | angle 0.8 | penumbra 0.15
shadow:   castShadow true | mapSize 1024 (4× fittings)
flicker:  0.2 s initial flicker (random intensity 0.6–2.2) then stabilise
```

---

## 2.6 Assessment Validation Logic

**QTB entry gates:**
1. Q field ≥ 30 chars before DB door can be opened
2. T field ≥ 40 chars after DB is opened, before any CB interaction
3. B field ≥ 30 chars before QTB can be locked

**Step validation:**
```
Step 1 – Observe:    Learner navigates to each light fitting within 1.0 m → logged ✓
Step 2 – Map:        System Map has ≥ 3 nodes and ≥ 2 connections → logged ✓
Step 3 – Test DB:    DB door opened → logged ✓
Step 4 – Identify:   CB_lighting inspected (camera within 0.4 m, dwell > 2s) → logged ✓
Step 5 – QTB Lock:   All three QTB fields submitted → logged ✓
Step 6 – Rectify:    CB_lighting toggled to ON position → logged ✓
Step 7 – Verify:     All 4 lights confirmed active (learner navigates within 1.0 m 
                     of each fitting post-repair) → logged ✓
```

**Scoring rubric (auto-marked):**

| Criterion | Weight | Pass |
|-----------|--------|------|
| Correct fault identified (CB_lighting) | 30% | Binary |
| QTB Q: assumption challenged (keyword parse) | 15% | ≥ 2 challenge verbs |
| QTB T: physical observations recorded | 20% | ≥ 3 observation statements |
| QTB B: logical inference from T evidence | 20% | Reasoning connective present |
| System Map correctly represents supply chain | 15% | ≥ 3 nodes, fault node marked |
| **Total** | **100%** | **≥ 65%** |

---

<a name="part-3"></a>
# PART 3 — FND-TT-01 SIM 02: THE DEAD PANEL

---

## 3.1 Simulation Brief

**Module:** FND-TT-01 | **Unit:** 1.4 (Troubleshooting Methodologies)
**Assessment weight:** 30% of module total | **Pass threshold:** 65%
**Notional time:** 60–90 minutes

**Pedagogical purpose:** The learner must select and apply a structured troubleshooting methodology (from the four taught: Half-Split, Top-Down, Bottom-Up, Compare-and-Swap) to a rooftop solar system that produces zero voltage output. The QTB step must be completed before methodology selection is locked in. The learner must justify their methodology choice physically (first principles basis).

**Scenario:** Rooftop of a community hall. A 4-panel, 2-string solar PV system is installed on a zincalume corrugated iron roof. The string inverter shows zero input. The learner must systematically isolate the fault.

**Fault embedded:** One MC4 connector in String 1 has an internal contact failure (not visible externally — the connector appears seated). It is located at the junction between Panel 02 and Panel 03 in the string. This requires the learner to progress through testing to the third test point before isolating the fault.

---

## 3.2 Scene Geometry

### Global Scene
```json
{
  "world_origin":  [0, 0, 0],
  "roof_pitch":    22.0,
  "hdri":          "overcast_cape_town_4k.hdr",
  "wind":          {"intensity_range": [0.25,1.2], "gust_freq": 0.8, "direction": [1.0,0.0,0.8]},
  "gravity":       [0, -9.81, 0],
  "time_of_day":   "10:00",
  "cloud_coverage": 0.85,
  "rain":          false
}
```

### Roof Structure
```json
{
  "purlins": {
    "count":     12,
    "type":      "extrude_beam",
    "dimensions":[3.6, 0.1, 0.05],
    "spacing":   0.6,
    "material":  "treated_pine",
    "position_first": [0.0, 0.0, -1.2]
  },
  "corrugated_sheets": {
    "count":      24,
    "dimensions": [3.0, 0.84, 0.00047],
    "profile":    {"amplitude": 0.012, "pitch": 0.076, "count": 8},
    "material":   "zincalume_corrugated",
    "uv_repeat":  [1, 4],
    "imperfections": {
      "rust_pits":  {"count": 2450, "size": "0.0008–0.003 m", "depth": 0.00012},
      "dents":      {"count": 18, "diameter": "8–25 mm", "depth": "0.3–0.8 mm"}
    }
  }
}
```

### PV Array (4 panels, 2 strings)
```json
{
  "mounting_rails": {
    "type":      "aluminium_extrusion",
    "profile":   [[0,0],[0,0.04],[0.01,0.04],[0.01,0.03],[0.015,0.03],[0.015,0]],
    "length":    3.6,
    "count":     4,
    "positions": [[-0.525,1.05,-1.2],[-0.525,1.05,0.6],[0.525,1.05,-1.2],[0.525,1.05,0.6]],
    "material":  "aluminium_anodized"
  },
  "panels": {
    "count": 4,
    "spec":  "440W_Canadian_Solar",
    "frame": {
      "type":          "rounded_box",
      "dimensions":    [1.05, 1.82, 0.035],
      "corner_radius": 0.008,
      "segments":      [32, 64, 8],
      "material":      "aluminium_anodized"
    },
    "glass": {
      "dimensions": [1.03, 1.77, 0.0035],
      "material":   "pv_glass"
    },
    "cells": {
      "count": 72, "layout": [12, 6],
      "cell_dims": [0.0396, 0.156, 0.0002],
      "busbars": {"width": 0.0015, "tabs": 156},
      "emissive": true
    },
    "junction_box": {
      "dimensions": [0.12, 0.08, 0.025],
      "material":   "abs_polycarbonate",
      "diode_slots": 3
    },
    "physics": {
      "mass": 22.0,
      "com_offset": [0.0, 0.18, 0.0],
      "moments_inertia": [28.5, 1.2, 29.1],
      "drag_coefficient": 1.05,
      "wind_load_area": 1.91
    },
    "positions": [
      {"id":"P01","pos":[-0.525,1.32,-1.2],"string":1},
      {"id":"P02","pos":[-0.525,1.32, 0.6],"string":1},
      {"id":"P03","pos":[ 0.525,1.32,-1.2],"string":2},
      {"id":"P04","pos":[ 0.525,1.32, 0.6],"string":2}
    ]
  },
  "mid_clamps": {
    "per_panel": 4,
    "body":      [0.04, 0.03, 0.025],
    "bolt_m8":   {"shank": [0.008, 0.05], "thread_pitch": 0.00125,
                  "head_flats": 0.013, "head_height": 0.006},
    "torque_target": 12.0
  }
}
```

### DC Wiring Harness
```json
{
  "dc_cable_6mm2": {
    "outer_dia":     0.008,
    "conductor_dia": 0.003,
    "stranding":     49,
    "colour":        {"positive": "red", "negative": "black"},
    "path":          "spline_along_rail_underside"
  },
  "mc4_connectors": {
    "count":  8,
    "male": {
      "dimensions": [0.065, 0.018, 0.018],
      "material":   "abs_polycarbonate"
    },
    "female": {
      "dimensions": [0.052, 0.016, 0.016],
      "material":   "abs_polycarbonate"
    },
    "physics": {
      "mass":            0.028,
      "friction_static": 0.42,
      "crimp_force":     45.0
    },
    "acoustic": {
      "snap_freq":  1850,
      "harmonics":  [3700, 7400, 11100],
      "duration":   0.045,
      "decay":      0.12
    },
    "haptic": {
      "intensity":  0.65,
      "frequency":  1850,
      "duration":   0.08
    },
    "fault": {
      "location":    "MC4_P01_P02_positive_String1",
      "type":        "internal_contact_failure",
      "external_appearance": "normal_seated",
      "voltage_at_pair": 0,
      "detectable_by":   "multimeter_voltage_test_only"
    }
  }
}
```

### String Inverter
```json
{
  "enclosure":  [0.40, 0.60, 0.20],
  "position":   [1.2, 1.1, -0.3],
  "material":   "abs_polycarbonate",
  "hmi": {
    "dimensions": [0.18, 0.10, 0.005],
    "position":   [0.10, 0.55, 0.195],
    "display":    {"type": "emissive_lcd", "state": "ERROR_NO_DC_INPUT"}
  },
  "heatsink": {
    "fins":    {"count": 48, "dims": [0.35, 0.008, 0.045], "spacing": 0.008},
    "fan":     {"dia": 0.12, "grill": [0.30, 0.20]}
  },
  "dc_inputs": {
    "string1_pos": {"type": "MC4_female", "voltage_reading": 0},
    "string1_neg": {"type": "MC4_female"},
    "string2_pos": {"type": "MC4_female", "voltage_reading": 88.0},
    "string2_neg": {"type": "MC4_female"}
  }
}
```

---

## 3.3 Methodology Selection Interface

Before the learner can take any measurement, a full-screen methodology selection panel appears (after QTB entry is locked):

**Panel title:** *"Select your troubleshooting approach — then explain why in the space below."*

**Four options presented:**
- Half-Split — diagram showing midpoint test of a linear system
- Top-Down — diagram showing source-to-load progression
- Bottom-Up — diagram showing load-to-source progression
- Compare-and-Swap — diagram showing component substitution

**Required field:** *"In the QTB B field, explain: what physical reason makes this method the most efficient choice for this system?"*

**Scoring:** The Top-Down method is the optimal choice for this scenario (known-good source, hierarchical supply structure, string inverter accessible). Accept Top-Down or Half-Split (both are justifiable). Bottom-Up or Compare-and-Swap for first pass are both pedagogically suboptimal — accepted but logged; facilitator review flagged.

**No choice is blocked** — the learner may proceed with any method. The assessment scoring reflects the choice's efficiency relative to the optimal path.

---

## 3.4 Test Point Interaction (Multimeter)

The same multimeter model from Sim 01 is available. Test points are invisible `Mesh` objects with a sphere collider (radius 0.012 m) placed at each junction:

```
TP-01: String 1 output terminals at inverter DC input — voltage: 0V  (fault)
TP-02: Mid-rail test — between P02 and P03, String 1 — voltage: 0V  (fault upstream)
TP-03: P01 output — voltage: 0V  ← fault between P01 output and TP-02
TP-04: P01/P02 MC4 pair — connector-body probe position — voltage: 0V ← isolates fault
TP-05: String 2 output at inverter — voltage: 88.0V (healthy — 2×44V panels)
```

On probe contact with a test point, the multimeter display animates a reading over 0.8 s (analogue needle rise effect). Audio: `probe_contact_beep` 880 Hz, duration 0.12 s, decay 0.3. A `TEST_LOG` entry is auto-generated in the assessment record.

**QTB requirement:** The QTB T field must be updated (minimum one new sentence) after each test point reading before the next test can be initiated. Attempting to probe a second test point without updating T: soft block + prompt.

---

## 3.5 Fault Confirmation and Resolution

When the learner isolates the fault to MC4_P01_P02:

1. An X-ray-style cutaway view of the MC4 connector renders (cross-section camera, depth of field applied to outer surfaces). The internal contact blade is shown with a 0.15 mm air gap between contact surfaces — the physical mechanism of the failure.
2. A pop-up label: *"Internal contact failure: the contact blade has backed out of the socket during a thermal expansion cycle. The physical gap prevents current flow — Ohm's Law: R → ∞, I = 0."*
3. The learner selects: **Replace connector** (correct) or **Re-seat connector** (accepted, but penalty flag). Neither action is shown as a 3D interaction — resolution is menu-based to keep scope in the diagnostic, not the installation skill.
4. QTB B field must be completed before resolution can be confirmed.

---

## 3.6 Assessment Validation Logic

**Validation sequence:**
```
Gate 1: QTB Q + T fields submitted before methodology selected
Gate 2: Methodology selection + written justification submitted
Gate 3: Test points visited in a logical sequence (not random)
Gate 4: QTB T updated after each test point
Gate 5: Fault correctly isolated to MC4_P01_P02
Gate 6: QTB B submitted before resolution confirmed
```

**Scoring rubric:**

| Criterion | Weight | Pass |
|-----------|--------|------|
| Fault correctly identified (MC4_P01_P02) | 35% | Binary |
| Methodology justified with physical basis (QTB B parse) | 20% | Physics-grounded language present |
| Test sequence efficiency (fewest redundant tests) | 15% | ≤ 5 test points for Top-Down |
| QTB T field: observation quality per test | 20% | ≥ 2 observation statements per test |
| QTB Q field: assumptions challenged at outset | 10% | Challenge verbs present |
| **Total** | **100%** | **≥ 65%** |

---

<a name="part-4"></a>
# PART 4 — FND-IW-01 SIM 01: FIRST DAY ON SITE

---

## 4.1 Simulation Brief

**Module:** FND-IW-01 | **Unit:** 1.2 (Hazard Identification)
**Assessment weight:** 25% of module total | **Pass threshold:** 65%
**Notional time:** 45–60 minutes

**Pedagogical purpose:** The learner navigates a large commercial fit-out site and identifies 18 embedded hazards across all 8 hazard categories taught in Unit 1.2. For each hazard, they must record: the category, the physical mechanism of harm, and a risk rating using the 5×5 matrix. A QTB pre-walk entry must be completed before navigation begins.

**Scenario:** Ground floor of a large commercial building under fit-out. Partially complete MEP installation. Active construction with simulated workers in background. The learner enters as a new apprentice on their first day.

---

## 4.2 Scene Layout and Geometry

### Global Scene
```json
{
  "world_origin":    [0, 0, 0],
  "floor_plan":      "L-shaped | 28m × 16m + 12m × 8m wing",
  "floor_level":     0.0,
  "ceiling_height":  4.2,
  "hdri":            "industrial_workshop_4k.hdr",
  "gravity":         [0, -9.81, 0]
}
```

### Primary Structural Materials

**Epoxy floor (main area):**
```
dimensions:  [28.0, 0.08, 16.0]
position:    [0, -0.04, 0]
material:    epoxy_floor
segments:    [192, 1, 96]
imperfections: wheel_tracks 8 (length 2–6 m, roughness +0.12)
               tool_scrapes 22 (length 0.1–0.4 m)
```

**Exposed concrete columns (12):**
```
per_column:  CylinderGeometry [0.4, 0.4, 4.2] | segments 64 | material: bare_concrete
positions:   grid_4x3 at [3.5, 7.0, 10.5, 14.0] × [2.5, 8.0, 13.5]
imperfections: formwork_tie_holes 8 per column (dia 0.025 m)
               form_release_agent_streaks: opacity 0.2
```

**Galvanised cable tray (ceiling-mounted runs):**
```
tray_width:  0.3 m
tray_depth:  0.1 m
wall_thickness: 0.002 m
material:    galvanised_steel
runs:
  main_north_south: length 16.0 m | position y: 3.8 | x: 7.0
  main_east_west:   length 22.0 m | position y: 3.8 | z: 8.0
  branch_01:        length 6.0 m  | position y: 3.8 | x: 14.0
support_brackets:   count 48 | spacing 1.5 m | material: galvanised_steel
cable_bundles:      3 bundles per tray run | outer_dia 0.04 m each
```

**Perimeter scaffolding (east wall — active work zone):**
```
standards:   CylinderGeometry [0.048, 0.048, 5.0] | material: aisi304_stainless
             count: 12 | grid_3x4
ledgers:     CylinderGeometry [0.038, 0.038, 2.5] | count: 18
transoms:    CylinderGeometry [0.038, 0.038, 1.2] | count: 24
boards:      BoxGeometry [2.4, 0.05, 0.225] | material: treated_pine | count: 36
toe_boards:  BoxGeometry [2.4, 0.15, 0.02] | material: treated_pine
```

---

## 4.3 The 18 Embedded Hazards

Each hazard is an independently flagged, invisible trigger volume (`BoxGeometry` collider, `visible: false`). When the learner enters the volume or focuses their gaze within 1.5 m, the Hazard Register panel opens.

The hazard register panel fields (per hazard):
- **Category** (dropdown: 8 options)
- **What I can see** (free text — maps to T step)
- **Physical mechanism of harm** (free text — scored for physics content)
- **Likelihood** (1–5 dropdown)
- **Severity** (1–5 dropdown)
- **Risk score** (auto-calculated: L × S)
- **Proposed control** (free text)

| # | Hazard | Category | Mechanism | Position | Risk Score |
|---|--------|----------|-----------|----------|-----------|
| 01 | Unguarded floor opening (cable chase) | Physical | Potential energy — fall → kinetic → tissue damage | [6.0, 0.0, 4.0] | 15 (Extreme) |
| 02 | Live extension cord across walkway | Electrical | Current pathway through body if conductor breached underfoot | [4.0, 0.0, 8.0] | 12 (High) |
| 03 | Unsecured gas cylinder (LPG, horizontal) | Physical | Valve shear → pressure release → explosive projection | [2.0, 0.0, 14.0] | 16 (Extreme) |
| 04 | Unmarked wet epoxy patch | Physical | Friction coefficient drop → slip → kinetic impact | [12.0, 0.0, 3.0] | 9 (Medium) |
| 05 | Cement dust cloud from mixing | Chemical | Caustic particle inhalation → lung tissue alkali burn | [18.0, 0.0, 6.0] | 12 (High) |
| 06 | Unprotected scaffold edge at 3.8 m (no guardrail) | Physical | Free fall 3.8 m → kinetic energy → trauma | [24.0, 3.8, 10.0] | 20 (Extreme) |
| 07 | Worker using angle grinder without eye protection | Physical | High-velocity metal projectile → penetration injury | [16.0, 0.0, 12.0] | 16 (Extreme) |
| 08 | Exposed rebar protruding from slab | Physical | Puncture/impalement → penetrating trauma | [8.0, 0.0, 15.0] | 9 (Medium) |
| 09 | Chemical drum (MEK solvent) unlabelled, open | Chemical | Vapour inhalation → CNS depression; ignition risk | [22.0, 0.0, 2.0] | 12 (High) |
| 10 | Uneven floor level transition (50mm step) | Ergonomic | Repetitive ankle stress; trip → fall if carrying load | [14.0, 0.0, 7.0] | 6 (Medium) |
| 11 | No task lighting in north corridor | Physical | Inadequate luminance → trip/collision with material | [1.0, 0.0, 8.0] | 9 (Medium) |
| 12 | Worker on mobile platform without harness | Physical | Fall from 2.4 m elevated platform | [20.0, 2.4, 14.0] | 16 (Extreme) |
| 13 | Forklift operating without spotter in blind corridor | Process/System | Cognitive: driver cannot see pedestrian; kinetic crush | [10.0, 0.0, 0.5] | 15 (Extreme) |
| 14 | Repeated overhead drilling — no hearing protection | Ergonomic | Cumulative acoustic trauma > 85 dB(A) | [6.0, 0.0, 12.0] | 8 (Medium) |
| 15 | Isolation label removed from DB panel | Process/System | Re-energisation of circuit believed isolated; arc flash | [3.5, 1.8, 14.5] | 20 (Extreme) |
| 16 | Workers eating in chemical storage area | Biological/Chemical | Ingestion pathway for chemical residue | [20.0, 0.0, 6.0] | 8 (Medium) |
| 17 | Cable tray run blocking emergency exit route | Process/System | Egress obstruction during emergency; entrapment | [27.5, 0.0, 3.0] | 12 (High) |
| 18 | Compressed air hose whip potential (fitting loose) | Physical | Pressurised hose recoil → ballistic impact; pneumatic injection | [15.0, 0.0, 11.0] | 9 (Medium) |

---

## 4.4 Animated Scene Dressing

The following animated elements provide environmental realism and context:

**Forklift (Hazard 13):**
```
model:      BoxGeometry approximation for web performance | LOD3 faces: 18000
path:       Patrol path: [10,0,0.5] → [10,0,-2.0] → [8,0,-2.0] → [8,0,0.5]
speed:      1.2 m/s
animation:  mast_bob 0.02 m | tyre_rotation: angular_vel = v/r (r=0.3 m)
audio:      forklift_diesel_hum 140 Hz | beep_reverse 880 Hz
```

**Ladder worker (Hazard 12):**
```
character:  Mannequin mesh | 1.78 m height | positioned at elevated platform
animation:  arm_raise cycle 3.2 s | body_sway subtle 0.8 Hz
PPE_state:  hard_hat: YES | harness: NO | hi_vis: YES (scoring trigger)
```

**Angle grinder worker (Hazard 07):**
```
character:  Mannequin mesh | working position: crouched
animation:  grinder_vibration 0.4 mm amplitude | sparks: 240 particles
             velocity [1.5, 3.0] m/s | lifespan 0.4 s
             cooling_emissive 2200K→800K
PPE_state:  hard_hat: YES | eye_protection: NO (scoring trigger)
audio:      grinder_disc_whine 4800 Hz | spark_contact 6200 Hz impulse
```

---

## 4.5 QTB Pre-Walk Protocol

Before the navigation begins, the full screen is taken by a mandatory QTB entry:

- **Q field prompt:** *"Before you walk onto this site, what assumptions are you making about its safety? List at least three things you are going to consciously challenge as you walk around."*
- **T field prompt:** *"What will you physically look for to confirm or deny each of those assumptions? Be specific about what you can see, hear, or observe."*
- **B field prompt:** *"Based on the information you've been given in the site brief, what types of hazards are most likely to be present in a fit-out context? Why — what physical activities produce those hazard types?"*

QTB entry is required but assessed as formative (scored for engagement, not accuracy). Minimum 50 characters per field before navigation unlocks.

---

## 4.6 Assessment Validation Logic

**Scoring rubric:**

| Criterion | Weight | Pass |
|-----------|--------|------|
| Hazards correctly identified (≥ 14 of 18) | 40% | 14+ = full marks; 10–13 = partial; < 10 = 0 |
| Category correctly assigned per hazard | 15% | Per-hazard; auto-marked |
| Physical mechanism of harm stated correctly | 20% | Physics-grounded language; keyword parse |
| Risk score correctly calculated (L × S) | 15% | ± 1 from reference value accepted |
| QTB pre-walk quality (formative gate) | 10% | ≥ 50 chars per field |
| **Total** | **100%** | **≥ 65%** |

**Extreme risk override:** Any Extreme hazard (score 15+) not identified by the learner triggers a mandatory facilitator review flag regardless of overall score. There are 7 Extreme hazards in this scene — a learner who misses more than 3 Extreme hazards cannot achieve a reliable pass on safety competency.

---

<a name="part-5"></a>
# PART 5 — FND-IW-01 SIM 02: SUITING UP

---

## 5.1 Simulation Brief

**Module:** FND-IW-01 | **Unit:** 1.4 (Personal Protective Equipment)
**Assessment weight:** 20% of module total | **Pass threshold:** 65%
**Notional time:** 30–45 minutes

**Pedagogical purpose:** The learner is given a work scenario (specified task in the plant room visible through the locker room window) and must select the correct PPE from a fully stocked PPE station. One item in the station is deliberately defective. The learner must identify the defect and condemn the item.

**Scenario:** Facility locker room adjacent to a mechanical plant room. Through the window, the learner can see the assigned task: a pump motor replacement on a live condensate system — elevated platform at 2.2 m, chemical (corrosive condensate) handling, noise > 88 dB(A), rotating machinery adjacent.

---

## 5.2 Scene Geometry

### Global Scene
```json
{
  "world_origin": [0, 0, 0],
  "room_dims":    [6.0, 3.0, 5.0],
  "floor":        {"material": "epoxy_floor", "dimensions": [6.0, 0.08, 5.0]},
  "walls":        {"material": "painted_concrete_cream", "base_color": [0.86,0.84,0.78]},
  "hdri":         "industrial_workshop_4k.hdr"
}
```

### Lockers (scene dressing)
```
unit_01:  [0.5, 1.85, 0.4] × 6 units | position x: -2.5 to -0.5 | z: -2.4
material: steel_galvanised | door_handle: brass_c36000
```

### PPE Station (primary interactive zone)
```json
{
  "cabinet": {
    "dimensions": [2.4, 2.1, 0.45],
    "position":   [0.0, 1.05, -2.45],
    "material":   "steel_galvanised",
    "shelves":    5,
    "label":      "PERSONAL PROTECTIVE EQUIPMENT — PLANT ROOM ACCESS"
  }
}
```

**PPE Items on Station (all individually modelled):**

| Item | Dimensions (m) | Segments | Material | Status | Position on shelf |
|------|---------------|----------|----------|--------|-------------------|
| Hard hat (white) | [0.27,0.18,0.25] | [48,32,48] | safety_helmet_white | GOOD | shelf_1 |
| Hard hat (yellow) — defective | [0.27,0.18,0.25] | [48,32,48] | safety_helmet_yellow | CRACK in shell | shelf_1 |
| Safety glasses | [0.15,0.05,0.16] | [32,16,32] | abs_polycarbonate | GOOD | shelf_2 |
| Chemical splash goggles | [0.18,0.09,0.18] | [48,24,48] | abs_polycarbonate | GOOD | shelf_2 |
| P2 dust mask | [0.14,0.06,0.10] | [24,16,24] | neoprene_60A | GOOD | shelf_2 |
| Half-mask respirator | [0.17,0.11,0.13] | [32,24,32] | neoprene_60A | GOOD | shelf_2 |
| Nitrile gloves (Sz M) | [0.09,0.02,0.24] × 2 | [16,8,32] | nitrile_glove_blue | GOOD | shelf_3 |
| Leather work gloves | [0.10,0.02,0.25] × 2 | [16,8,32] | leather_boot material | GOOD | shelf_3 |
| Hi-vis vest (yellow) | [0.44,0.0,0.56] | flat | hi_vis_yellow | GOOD | shelf_3 |
| Hearing protection — plugs | [0.012,0.012,0.028] × 2 | [16,8,16] | neoprene_60A | GOOD | shelf_4 |
| Hearing protection — muffs | [0.24,0.22,0.08] | [48,32,24] | abs_polycarbonate | GOOD | shelf_4 |
| Safety boots (Sz 10) | [0.11,0.12,0.30] × 2 | [32,24,48] | leather_boot | GOOD | shelf_5 |
| Fall arrest harness | full_body layout | [32,16,32] | neoprene_60A straps | GOOD | shelf_5 |
| Safety lanyard (1.8m) | [coiled, dia 0.12] | spline | neoprene_60A | GOOD | shelf_5 |

**The deliberate defect — yellow hard hat:**
```
Defect type:   Hairline crack in crown shell
Visual cue:    Crack width 0.8 mm, length 38 mm, running from crown toward brim
               Visible from distance > 0.5 m at correct angle
               Slightly different surface normal at crack: depth 0.12 mm
3D modelling:  Displace vertices along crack path by -0.00012 m
               roughness increase along crack: +0.25
               AO shadow at crack: bias 0.0001, width 0.8 mm
First principles label (on condemn): "The crack creates a stress concentration.
               Under impact, the shell will fracture along this line —
               it cannot distribute the impact force across the surface.
               This hat provides no protection."
```

**Plant room window (rear wall):**
```
window:     [1.8, 1.2, 0.08] | position [0.0, 1.8, -2.4]
glass:      material pv_glass (transmission 0.75, frosted: roughness 0.08)
scene_behind: diorama visible — pump skid, elevated grating at 2.2 m,
              condensate pipework with drip stains, adjacent motor with rotating shaft
              Task label overlay: "TASK: Replace pump motor. Work at height 2.2m.
              Corrosive condensate system. Adjacent rotating machinery."
```

---

## 5.3 Interaction Model

### PPE Selection
- Learner clicks/taps an item: it highlights (emissiveIntensity 0.2, pulse 0.3 Hz)
- A properties panel opens: item name, protection category, physical protection principle (one-sentence)
- Learner clicks "Select for task" — item moves to a body-model display panel (right side of screen)
- Learner can deselect and replace

### QTB Pre-Selection Entry
Before any item can be selected, the learner completes the QTB entry for this scenario:
- **Q field:** *"What are you assuming about the PPE on this shelf? What should you check before you rely on any item?"*
- **T field:** *"From looking at the task through the window, what hazard types can you physically confirm will be present?"*
- **B field:** *"For each hazard type you've identified, what physical property of PPE must match the hazard mechanism to provide real protection?"*

### Defect Inspection
Each item has an "Inspect" interaction (secondary click / long-press). This activates a close-up camera view with DoF set so the item fills the frame (focalDistance 0.35 m, fstop 1.4). The learner must interact with the hard hat close-up view and identify the crack. On correct identification:
- "Condemn" button becomes available
- Learner must select a condemnation reason from a list
- Selecting "Structural crack in shell — risk of fracture under impact" is correct
- Correct condemnation triggers: red tag animation; first principles explanation popup; audio `tag_click` 1200 Hz

---

## 5.4 Assessment Validation Logic

**Required PPE set for this task (reference):**

| PPE Item | Required | Physical justification |
|----------|----------|----------------------|
| Hard hat | YES | Working under scaffold/crane zone; falling object risk |
| Chemical splash goggles | YES | Corrosive condensate splash to eyes |
| Half-mask respirator | YES | Chemical vapour from condensate |
| Nitrile gloves | YES | Chemical resistance to corrosive condensate |
| Hi-vis vest | YES | Active plant room with vehicle movement |
| Hearing protection (muffs) | YES | Adjacent motor noise > 88 dB(A) |
| Safety boots | YES | Standard site requirement |
| Fall arrest harness + lanyard | YES | Work at height > 2 m |
| Leather work gloves | NO | Insufficient chemical resistance |
| Safety glasses | NO | Insufficient lateral splash protection vs. goggles |
| P2 dust mask | NO | No particulate hazard; chemical vapour requires cartridge |

**Scoring rubric:**

| Criterion | Weight | Pass |
|-----------|--------|------|
| Correct PPE selected (all 8 required items) | 35% | Each item: binary; -4% per miss, -2% per wrong inclusion |
| Defective hard hat identified and condemned | 25% | Binary — must identify crack specifically |
| PPE matching to hazard mechanism (QTB B parse) | 25% | Physics-matching language present per hazard |
| QTB pre-selection quality | 15% | ≥ 40 chars per field |
| **Total** | **100%** | **≥ 65%** |

---

<a name="part-6"></a>
# PART 6 — FND-IW-01 SIM 03: SETTING OUT AND FIXING

---

## 6.1 Simulation Brief

**Module:** FND-IW-01 | **Units:** 1.6 (Tool Familiarisation) and 1.7 (Pre-Use Inspection)
**Assessment weight:** 20% of module total (Sim 03: 15% tools, 5% inspection checklist)
**Pass threshold:** 65%
**Notional time:** 60–90 minutes

**Pedagogical purpose:** The learner works through three sequential tasks on a light industrial workshop bench, each requiring correct tool selection from a 20-tool shadow board, pre-use inspection, and physically accurate use. A QTB entry is required before each tool is selected. Torque feedback is provided with failure mechanism explanation on incorrect application force.

---

## 6.2 Scene Geometry

### Global Scene
```json
{
  "world_origin": [0, 0, 0],
  "room_dims":    [10.0, 3.5, 8.0],
  "floor":        {"material": "epoxy_floor", "segments": [128,1,96]},
  "walls":        {"material": "painted_concrete_cream"},
  "ceiling":      {"material": "base_color [0.90,0.89,0.86]", "roughness": 0.72},
  "hdri":         "industrial_workshop_4k.hdr",
  "task_lighting": {
    "bench_spots": 4,
    "type":        "SpotLight",
    "color":       [0.97,0.94,0.88],
    "intensity":   3.8,
    "angle":       0.4,
    "penumbra":    0.2,
    "castShadow":  true
  }
}
```

### Workbench
```json
{
  "dimensions": [2.4, 0.9, 0.8],
  "position":   [0.0, 0.45, 0.0],
  "top": {
    "material":  "galvanised_steel",
    "thickness": 0.003,
    "dimensions":[2.4, 0.003, 0.8]
  },
  "frame":   {"material": "aisi304_stainless", "tube_section": [0.04,0.04,0.003]},
  "vice": {
    "body":        [0.22, 0.12, 0.16],
    "jaw_width":   0.12,
    "max_opening": 0.085,
    "position":    [-1.0, 0.92, 0.3],
    "physics":     "leadscrew_constraint | pitch 0.003 m per revolution"
  }
}
```

### Shadow Board (Primary Interactive Zone)
```json
{
  "board": {
    "dimensions": [2.4, 1.5, 0.025],
    "position":   [-4.8, 1.75, -3.95],
    "material":   "abs_polycarbonate | base_color [0.18,0.20,0.22]"
  },
  "tool_silhouettes": {
    "type":     "MeshBasicMaterial | color [0.08,0.08,0.08]",
    "note":     "Each tool slot has a precise silhouette cutout in the board + hook",
    "state":    "tool_absent: silhouette visible | tool_present: tool occludes silhouette"
  }
}
```

### The 20 Tools on the Shadow Board

All tools use `MeshPhysicalMaterial`. Segment counts at HD tier (within 0.5 m interaction distance). Physics: dynamic rigid bodies with mass, friction, and COM matching real tool properties.

**FASTENING TOOLS:**

```json
{
  "T01_flathead_screwdriver": {
    "handle":    [0.12, 0.024, 0.024], "segments": [24,16,24],
    "shaft":     [0.003, 0.075, 0.003], "segments": [16,1,1],
    "tip":       [0.007, 0.001, 0.007],
    "material":  "handle: abs_polycarbonate [0.6,0.12,0.10] | shaft: aisi304_stainless",
    "mass":      0.085, "com": [0,0,0.04],
    "physics":   "friction_static: 0.72"
  },
  "T02_phillips_PH2": {
    "handle":    [0.12, 0.024, 0.024], "segments": [24,16,24],
    "tip":       "cross_profile [0.006,0.004,0.002] | cam_out_force 5.2N at 5°",
    "material":  "handle: abs_polycarbonate [0.12,0.35,0.65] | shaft: aisi304_stainless",
    "mass":      0.082
  },
  "T03_pozidriv_PZ2": {
    "handle":    [0.12, 0.024, 0.024],
    "tip":       "cross + 4×45° faces | width 0.006m | depth 0.006m",
    "material":  "handle: abs_polycarbonate [0.85,0.65,0.08]",
    "mass":      0.083,
    "note":      "Visually similar to PH2 — deliberate confusion opportunity"
  },
  "T04_allen_key_5mm": {
    "body":      "L-shape: long_arm [0.10,0.005,0.005] | short_arm [0.04,0.005,0.005]",
    "material":  "aisi304_stainless",
    "mass":      0.028
  },
  "T05_combination_spanner_13mm": {
    "body":      [0.20, 0.022, 0.010], "segments": [1,16,8],
    "ring_end":  "hex_profile 13mm AF | depth 0.010m | segments 64",
    "open_end":  "jaw opening 13mm | jaw depth 0.008m",
    "material":  "aisi304_stainless",
    "mass":      0.18
  },
  "T06_adjustable_spanner_200mm": {
    "body":      [0.20, 0.035, 0.012],
    "jaw_range": [0, 0.032],
    "physics":   "jaw_adjuster: leadscrew 0.0016m pitch | pull_correct_direction: toward_fixed_jaw",
    "material":  "aisi304_stainless",
    "mass":      0.42
  }
}
```

**CUTTING TOOLS:**

```json
{
  "T07_junior_hacksaw": {
    "frame":  [0.32, 0.12, 0.006], "segments": [4,2,1],
    "blade":  [0.30, 0.013, 0.0006],
    "teeth":  {"count": 90, "pitch": 0.0033, "set_alternating": true},
    "material":"frame: aisi304_stainless | blade: high_speed_steel",
    "mass":   0.22
  },
  "T08_stanley_knife": {
    "body":    [0.16, 0.025, 0.018],
    "blade":   [0.06, 0.018, 0.0006],
    "retract": "slide_mechanism | locked: true when not in use",
    "material":"body: abs_polycarbonate [0.92,0.65,0.0] | blade: high_speed_steel",
    "mass":    0.11
  },
  "T09_wire_strippers_adjustable": {
    "handles": "two_arm | each [0.09, 0.012, 0.012]",
    "jaw":     "serrated | die_slots [6.0,4.0,2.5,1.5,1.0,0.5]mm",
    "material":"handles: abs_polycarbonate [0.85,0.12,0.08] | jaw: aisi304_stainless",
    "mass":    0.145
  }
}
```

**GRIPPING TOOLS:**

```json
{
  "T10_combination_pliers_200mm": {
    "handles": "each [0.10,0.018,0.014] | segments [1,24,16]",
    "jaws":    [0.065, 0.025, 0.018],
    "pivot":   "CylinderGeometry [0.006,0.006,0.018] | segments 32",
    "insulation": "dip-coated handle ends | thickness 0.003m | material: neoprene",
    "mass":   0.28
  },
  "T11_long_nose_pliers_150mm": {
    "handles": "each [0.09, 0.016, 0.012]",
    "jaws":    "tapered [0.012→0.004] width | length 0.06m",
    "mass":    0.19
  },
  "T12_side_cutting_pliers": {
    "handles": "each [0.09, 0.018, 0.013]",
    "blades":  "angled flush-cut | width 0.032m | depth 0.006m",
    "mass":    0.22
  }
}
```

**MEASURING TOOLS:**

```json
{
  "T13_steel_rule_300mm": {
    "body":      [0.300, 0.025, 0.001],
    "markings":  "engraved mm scale | segments_fine: every 1mm | coarse: every 10mm",
    "material":  "aisi304_stainless | base_color [0.72,0.72,0.70]",
    "mass":      0.052
  },
  "T14_tape_measure_5m": {
    "housing":   [0.08, 0.065, 0.028],
    "tape":      "flexible_strip | width 0.019m | blade_thickness 0.00013m",
    "hook":      "hook_slides: by_hook_material_thickness for internal/external compensation",
    "material":  "housing: abs_polycarbonate [0.92,0.42,0.0]",
    "mass":      0.19
  },
  "T15_spirit_level_400mm": {
    "body":      [0.400, 0.055, 0.030],
    "vials":     {
      "horizontal": {"pos": [0,0.025,0], "radius": 0.008, "length": 0.050,
                     "liquid": "mineral_oil | transmission 0.85 | tint [0.88,0.92,0.85]",
                     "bubble": "sphere_r 0.004 | material: air — less dense → always rises"},
      "vertical":   {"pos": [0.18,0.020,0]}
    },
    "material":  "body: aluminium_anodized",
    "mass":      0.28,
    "physics":   "bubble_position: y_offset = sin(level_tilt) × vial_length/2"
  },
  "T16_vernier_caliper": {
    "main_scale": [0.150, 0.018, 0.004],
    "jaw_fixed":  [0.025, 0.018, 0.012],
    "jaw_sliding":[0.025, 0.018, 0.012],
    "vernier_scale": "50div | resolution 0.02mm",
    "material":  "aisi304_stainless",
    "mass":      0.18,
    "reading_interaction": "animated slide | digital readout overlay on hover"
  }
}
```

**MARKING TOOLS:**

```json
{
  "T17_combination_square": {
    "blade":   [0.300, 0.020, 0.002],
    "stock":   [0.075, 0.050, 0.015],
    "faces":   "90° and 45° reference faces on stock",
    "material":"aisi304_stainless + abs_polycarbonate stock",
    "mass":    0.22
  },
  "T18_centre_punch": {
    "body":    "CylinderGeometry tapered [0.012→0.004] | length 0.10m | segments 32",
    "tip":     "cone angle 60° | tip_radius 0.0002m",
    "material":"tool_steel | base_color [0.35,0.35,0.38] | roughness 0.22",
    "mass":    0.08
  },
  "T19_chalk_line": {
    "housing": [0.12, 0.075, 0.048],
    "string":  "spline 10m | thickness 0.002m | chalk_coated",
    "material":"housing: abs_polycarbonate [0.18,0.22,0.82]",
    "mass":    0.32
  },
  "T20_multimeter_fluke": {
    "body":    [0.18, 0.09, 0.03],
    "display": "emissive_lcd | 4.5 digit",
    "probes":  "same spec as Sim 01",
    "material":"abs_polycarbonate [0.92,0.35,0.0]",
    "mass":    0.38
  }
}
```

---

## 6.3 The Three Tasks

### Task 1 — Measure, Mark, and Set Out
**Brief:** Mark a 450 mm centre line on a steel flat bar, with a perpendicular mark at 225 mm. Mark two drill hole positions at 112.5 mm from the centre perpendicular, on-centre.

**Required tools:** `T13_steel_rule` or `T14_tape_measure` + `T16_vernier_caliper` + `T17_combination_square` + `T18_centre_punch`

**QTB pre-selection entry required before first tool is taken from board.**

**Interaction model:**
- Tool picked up from board: left-click drag to workpiece
- Steel rule: click-drag along workpiece surface; numerical overlay shows measurement in real time
- Combination square: click to place; renders perpendicular guide line on workpiece surface
- Centre punch: click on target point; animation — arm raise + tap impact; indentation renders (sphere displacement 0.3 mm dia)
- Vernier caliper: jaw positions update in real-time via drag; reading panel updates numerically

**Tolerance:** ±1.0 mm on all marked positions for full marks; ±3.0 mm for partial.

**Failure state:** If learner uses tape measure without zeroing the hook on a reference edge: measurement shows ±0.5 mm drift. Hint fires: *"Is the hook seated against the reference edge? The hook slides by its own thickness to compensate — confirm the zero before you proceed."*

### Task 2 — Cut Conduit and Terminate Cable
**Brief:** Cut a 200 mm length of 20 mm PVC conduit, strip 50 mm of insulation from a 4 mm² cable, and crimp a bootlace ferrule to the conductor end.

**Required tools:** `T07_junior_hacksaw` + `T09_wire_strippers`

**QTB pre-selection entry required before tools are selected for this task.**

**Interaction model — hacksaw:**
- Place conduit in vice: click conduit + click vice → vice closes with leadscrew animation + audio (steel_creak 280 Hz)
- Mark cut line: requires T13 steel rule first
- Hacksaw animation: `AnimationMixer` — forward stroke 0.30 m, return 0.28 m, cycle 0.8 s
- Tooth engagement: each forward stroke removes 0.5 mm of virtual material (modelled as progressive displacement on cut face)
- Incorrect speed (lerp dt too low or too high): feedback *"Steady pace — too fast generates heat that softens the blade. Let the teeth do the work."*
- At 200 mm mark: cut completes with snap audio `conduit_snap` 2200 Hz + debris particles 8 chips (size 0.001–0.003 m)

**Interaction model — wire strippers:**
- Select cable on bench: highlighted
- Place strippers at 50 mm mark: click-drag to position
- Jaw size selection: learner must match jaw die to cable OD (4 mm² = OD 4.0 mm)
- Incorrect jaw: feedback with first principles explanation *"That jaw is too small — it will nick the conductor. Each nick creates a stress concentration that causes higher resistance and eventual fracture at that point."*
- Correct jaw + strip action: insulation peels away; conductor strands revealed with particle burst (insulation flake 0.5–2 mm)
- Ferrule crimping: ferrule tool not on shadow board (not in curriculum scope) — provided automatically with prompt

**Tolerance:** Cut length ±3.0 mm; strip length ±5.0 mm.

### Task 3 — Assemble a Bracket and Apply Torque
**Brief:** Assemble a two-piece mild steel bracket using 4× M6 hex bolts and nuts. Torque each bolt to 8 Nm using the correct tool.

**Required tools:** `T05_combination_spanner_13mm` or `T06_adjustable_spanner` + (for torque confirmation) the torque-indicating interaction layer

**QTB pre-selection entry required before tools are selected for this task.**

**Bracket geometry:**
```json
{
  "plate_A": {"dimensions": [0.20, 0.08, 0.006], "material": "galvanised_steel"},
  "plate_B": {"dimensions": [0.16, 0.08, 0.006], "material": "galvanised_steel"},
  "holes":   {"dia": 0.007, "positions_A": [[0.02,0.04,0],[0.08,0.04,0],[0.14,0.04,0],[0.19,0.04,0]]}
}
```

**M6 bolt physics:**
```json
{
  "mass":           0.018,
  "thread_pitch":   0.001,
  "shank_dia":      0.006,
  "head_hex_AF":    0.010,
  "torque_target":  8.0,
  "torque_curve": {
    "finger_tight":   1.5,
    "snug":           4.0,
    "target":         8.0,
    "yield":          12.0
  },
  "physics":        "rotational constraint | axis: bolt_axis | max_torque: 16 Nm"
}
```

**Torque feedback system:**
- As learner rotates spanner, applied torque renders as a real-time overlay bar (0–12 Nm range)
- Audio: `wrench_tighten` 1200 Hz metallic thread sound; frequency increases with torque
- Haptic: `torque_feedback torque/18.0 × 0.85` | freq `2–28 Hz torque-proportional` | waveform `sawtooth_damped`
- At 8.0 Nm: `torque_wrench_click` 4200 Hz + haptic intensity 0.85 impulse_ramp — *"Torque reached — stop turning."*
- Under-torque (< 5.5 Nm): feedback *"That bolt isn't fully engaged — under-torqued fasteners back out under vibration. The clamping force comes from the bolt's elastic stretch, not the surface friction."*
- Over-torque (> 12 Nm): failure state — bolt yields; thread strip sound `2800 Hz 0.12 s noise burst`; feedback *"Thread stripped — the material yielded beyond its elastic limit. This bolt must be replaced. Over-torquing destroys the clamping mechanism permanently."*

**Adjustable spanner misuse detection:** If learner applies torque pulling toward adjustable jaw (wrong direction):
- Spanner model shows jaw opening under load (jaw gap increases 0.5 mm via vertex animation)
- Feedback fires: *"Pull toward the fixed jaw — pulling the other way loads the adjustable jaw in the opening direction. It will slip and round the nut."*

---

## 6.4 Pre-Use Inspection Integration (Unit 1.7 — 5% assessment)

Before Task 1 begins, the learner is required to inspect two tools using the 5-step inspection protocol:

**Tool 01 presented for inspection — steel rule:** GOOD condition. All 5 inspection steps pass.

**Tool 02 presented for inspection — wire strippers:** DEFECT present. The jaw pivot rivet is slightly loose — visible as 0.3 mm lateral play when jaw is opened and closed.

```
Defect modelling:
  pivot_play:  0.3 mm lateral | rendered as jaw misalignment on open
  visual_cue:  gap at pivot face under task lighting | AO shadow at rivet seat
  category:    YELLOW (tag out for repair — functional but below spec)
  mechanism:   "A loose pivot allows the jaw to deflect under load —
                 the cutting edge won't seat squarely on the cable,
                 producing an uneven cut that may nick the conductor."
```

**Inspection scoring:** Learner must complete all 5 steps on both tools, identify the loose rivet on the wire strippers, and assign correct defect category (Yellow) for full marks.

---

## 6.5 Assessment Validation Logic

**Scoring rubric (combined Sim 03 + inspection):**

| Criterion | Weight | Pass |
|-----------|--------|------|
| Task 1 — correct tool selection + mark tolerances met | 20% | Tool logic + ±1 mm |
| Task 2 — correct jaw selection + cut/strip tolerances | 20% | Tool logic + ±3/5 mm |
| Task 3 — correct spanner direction + 8 Nm target (±0.5 Nm) | 25% | Binary torque gate |
| QTB pre-selection (×3): physics-grounded reasoning | 20% | ≥ 3 physics terms per entry |
| Pre-use inspection — defect identified + category correct | 15% | Binary for defect; 5-step log |
| **Total** | **100%** | **≥ 65%** |

---

<a name="part-7"></a>
# PART 7 — ASSESSMENT AND VALIDATION ENGINE

---

## 7.1 Validation Architecture

All assessment events are written to a server-side record in real time via HTTPS POST. The assessment record structure per simulation:

```json
{
  "learner_id":       "uuid",
  "simulation_id":    "FND-TT-01-SIM01",
  "session_start":    "ISO8601",
  "session_end":      "ISO8601",
  "qtb_entries":      [{"field":"Q","content":"...","timestamp":"...","chars":82},
                       {"field":"T","content":"...","timestamp":"...","chars":141},
                       {"field":"B","content":"...","timestamp":"...","chars":96}],
  "interaction_log":  [{"event":"db_door_opened","timestamp":"...","position":[...]},
                       {"event":"cb_lighting_toggled","result":"correct","timestamp":"..."}],
  "test_points":      [{"id":"TP-01","visited":true,"reading":"0V","timestamp":"..."}],
  "score_breakdown":  {"criterion_1":{"weight":30,"earned":30},...},
  "total_score":      88,
  "pass":             true,
  "facilitator_flags":[]
}
```

## 7.2 QTB Language Parser

The QTB parser is a lightweight in-browser NLP function applied to submitted text at field lock. It does not use an external API. It checks for:

**Q field — challenge verbs and uncertainty markers:**
```
Trigger words: assumed | not confirmed | could be | might | before testing |
               I don't know if | need to verify | taken for granted
Minimum: 2 trigger words or phrases for full Q-field score
```

**T field — observation anchors:**
```
Trigger patterns: "I can see..." | "measures..." | "reads..." | "shows..."
                  "is in [state]..." | "physically confirmed..."
                  "voltage at..." | "[component] is [ON/OFF/OPEN/CLOSED]"
Minimum: 3 observation statements for full T-field score
```

**B field — reasoning connectives:**
```
Trigger patterns: "therefore..." | "which means..." | "so the fault..."
                  "because..." | "this confirms..." | "given that..."
                  "the root cause is..." | "this indicates..."
Minimum: 2 connectives for full B-field score
```

---

## 7.3 Settling Validation (Physics)

For any step that requires a physical object to be placed in a target position before validation:

```
Linear velocity:  < 0.02 m/s
Angular velocity: < 0.05 rad/s
Acceleration:     < 0.1 m/s²
Settle window:    0.5–0.8 s before step marked complete
Position error:   tolerance per trade: overview (safety) ±0.05 m |
                                       precision (electrical) ±0.005 m
Rotation error:   overview ±0.52 rad | precision ±0.087 rad
```

## 7.4 Attention Guidance System

**Guidance layer (non-assessed):**
- Interactable objects not yet visited: `emissive pulse 0.3 Hz | intensity 0.12`
- Next required step: `wireframe_glow 20% opacity | pulse 0.5 Hz`
- Faulty component (after fault confirmed): `pulsing_red emissive | freq 0.8 Hz`

**Hint system (two levels):**
- **Level 1** fires after 180 s of inactivity on a step: *"Stop and apply T — what can you physically confirm right now?"*
- **Level 2** fires after 360 s total on a step: provides the QTB framework applied to the current scenario as a worked example using a different component

---

<a name="part-8"></a>
# PART 8 — PERFORMANCE, LOD, AND DELIVERY

---

## 8.1 LOD Configuration

```
LOD0 (mobile, < 0.5 m view): faces < 500  | texture 256 px
LOD1 (tablet, 0.5–2 m):      faces < 2000 | texture 512 px  | KTX2/Basis
LOD2 (desktop, 2–5 m):       faces < 8000 | texture 1024 px
LOD3 (primary desktop 0–2m): faces < 20000| texture 2048 px

Switch distances: LOD3→LOD2 at 2m | LOD2→LOD1 at 5m | LOD1→LOD0 at 10m
```

Primary interactive objects (within 0.5 m equivalent) use LOD3 segments per §1.1:
- Cylinder/torus: 128/96 segments
- Sphere: [64,48]
- Extrude: steps 8, bevelSegments 6

Background/distant objects use next tier down.

## 8.2 Memory Budget

```
Geometry:  128 MB total per simulation
Textures:  512 MB total per simulation
Physics:   32 MB (Rapier WASM instance)
Audio:     48 MB
Total:     < 720 MB resident
```

## 8.3 Performance Tiers

```
High-end (60 fps target):
  Objects:  ≤ 5000
  Physics:  ≤ 200 dynamic bodies
  Shadows:  all enabled | mapSize 2048

Mid-range (adaptive 45–60 fps):
  Objects:  ≤ 1500 (aggressive LOD1)
  Physics:  ≤ 80 bodies
  Shadows:  primary only | mapSize 1024
  Post-process: SMAA only; bloom disabled

Low-end (30 fps minimum):
  Objects:  LOD0/LOD1 throughout
  Physics:  kinematic only (no dynamics)
  Shadows:  disabled
  Textures: KTX2 256–512 px
```

## 8.4 Rendering Resolution Tiers

| Tier | Viewport | Render scale | Texture atlas | Use case |
|------|----------|-------------|---------------|----------|
| Minimum | 1280×720 | 1.0 | 1024 | Low-end / small window |
| HD (primary target) | 1920×1080 | 1.0–1.25 | 2048 | Standard desktop |
| HD+ | 2560×1440 | 1.25–1.5 | 2048–4096 | Fullscreen high-DPI |

**devicePixelRatio:** `Math.min(window.devicePixelRatio, 2)`. Handle resize without breaking LOD tier or quality target.

## 8.5 Asset Pipeline and Loading

```
Phase 1 (initial load):   core_geometry.glb | hdri_2k.hdr | physics_wasm
                          Show loading screen with progress bar
                          Target: < 8 s on 50 Mbps

Phase 2 (lazy, post-render): interaction_tools.glb | texture_atlas_2048.ktx2
                              Audio buffers | surface_imperfection_maps

Phase 3 (on-demand):      detailed_components (tool close-ups, defect views)
                          HD+ textures (only if devicePixelRatio > 1.5)
```

All immutable assets (hashed filenames): `Cache-Control: max-age=31536000, immutable`.

---

<a name="part-9"></a>
# PART 9 — QUALITY GATES AND SIGN-OFF CHECKLIST

---

## 9.1 Visual Realism Validation

The following must pass expert scrutiny at the distances specified:

| Check | Distance | Criterion |
|-------|----------|-----------|
| Mechanical Engineer | 0.5 m | Directional specular at grazing angles correct; F0 Fresnel; microfacet distribution; multi-scatter on metals |
| Industrial Designer | 1.0 m | Shadow contact at bolt gaps 0.1 mm; penumbra correct for fluorescent tubes; no shadow bias artefacts |
| Maintenance Technician | 0.3 m | Surface imperfections visible; tool geometry matches real counterpart; no UV seam artefacts |

**Additional visual checks:**
- Hexagonal bokeh on background geometry ✓
- Chromatic aberration matches lens model ✓
- Film grain uniform distribution ✓
- PBR values within ±10% of reference material database ✓

## 9.2 Physics Validation

```
Settling:       linear vel < 0.02 m/s | angular vel < 0.05 rad/s over 0.5 s window ✓
Valve (Sim 02): rotation reaches π/2 ± 0.05 rad | torque curve [8.2, 3.1, 11.8] Nm ✓
Tool grab:      true positive ≥ 97.2% | false positive ≤ 0.8% ✓
Torque predict: R² ≥ 0.94 ✓
Slippage force: accuracy ± 12% of reference ✓
```

## 9.3 Audio-Haptic Validation

```
MC4 snap:        1850 Hz ±50 Hz | 0.045 s ±0.005 s | haptic 0.65 ±0.05 ✓
Torque click:    4200 Hz ±100 Hz | 0.032 s | haptic 0.85 impulse ✓
Screwdriver cam: 2800 Hz ±80 Hz | 0.12 s | noise floor -24 dBFS ✓
Spatial falloff: lowpass at 440 Hz at 5 m distance ✓
```

## 9.4 Hand Tracking Validation (if MediaPipe active)

```
Fingertip RMS:  ≤ 7.8 mm world space ✓
Wrist pose:     ≤ 2.4° ✓
Palm center:    ≤ 4.2 mm ✓
Frame jitter:   RMS ≤ 1.8 mm ✓
30 s drift:     ≤ 12 mm ✓
E2E latency:    ≤ 54 ms total ✓
Power grip:     ≥ 98.4% recognition ✓
Precision pinch:≥ 97.1% recognition ✓
Tool grasp:     ≥ 96.2% recognition ✓
```

## 9.5 Assessment Engine Validation

```
QTB parser:     challenge verb detection ≥ 92% precision on test corpus ✓
Scoring:        per-criterion weights sum to 100% per simulation ✓
Gate logic:     no gate can be bypassed without meeting prerequisite ✓
Data:           all interaction events logged with ≤ 200 ms timestamp accuracy ✓
Assessment API: POST success ≥ 99.9% with retry on fail ✓
Offline:        IndexedDB backup captures all events; sync on reconnect ✓
```

## 9.6 Performance Sign-Off

```
60 fps sustained on mid-range desktop (Core i5 + GTX 1060 equivalent):
  Chrome 120+  ✓
  Firefox 121+ ✓
  Edge 120+    ✓
  Safari 17+   ✓

Frame-time variance: ± 10% of mean ✓
Tab paused when hidden (Page Visibility API) ✓
WebGL context loss handled ✓
devicePixelRatio scaling correct on retina displays ✓
```

## 9.7 Accessibility

```
WCAG 2.1 AA compliance ✓
All primary actions operable via keyboard ✓
QTB textarea focus management correct ✓
ARIA labels on all UI overlay elements ✓
Reduced-motion: pause procedural animations when prefers-reduced-motion ✓
Subtitle captions on all narrated instruction audio ✓
Minimum contrast 4.5:1 on all UI text ✓
```

## 9.8 Pre-Launch Checklist (Per Simulation)

```
[ ] World origin and gravity confirmed at [0,0,0] and [0,-9.81,0]
[ ] All geometry in metres; no unit inconsistency in physics bodies
[ ] PBR values within ±10% of material database
[ ] Surface imperfections applied to all interactive objects
[ ] HDRI assigned and IBL probes placed
[ ] All interactive colliders enabled (not just visual mesh)
[ ] Physics timestep 1/60 s fixed; Rapier WASM loaded async
[ ] QTB Clipboard widget persistent across scene transitions
[ ] All 3 QTB gate conditions implemented per simulation spec
[ ] Assessment API endpoint tested; retry logic active
[ ] All audio triggered only after user gesture
[ ] MediaPipe: getUserMedia request after user gesture; permission denial handled
[ ] LOD switching tested at 2 m, 5 m, 10 m camera distances
[ ] Memory resident < 720 MB at peak
[ ] 60 fps confirmed on minimum spec machine (GTX 1060 equivalent)
[ ] All 5 simulations independently loadable (no cross-sim dependency)
[ ] Post-processing pipeline applied; ACES tone mapping active
[ ] Shadow maps cast from all dynamic light sources affecting interactive objects
[ ] Haptic feedback fires on gamepad if connected
[ ] Keyboard navigation confirmed for all primary interactions
[ ] Captions available on all instructional audio
[ ] Loading screen with progress displayed; no white-screen hang
[ ] Offline: Cache API populated for core assets on first load
[ ] Assessment record exports cleanly to platform API schema
[ ] Facilitator flag system tested: Extreme hazard miss triggers flag
[ ] Hyper-realism validation passed at 0.3 m / 0.5 m / 1.0 m scrutiny distances
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-12 | Platform Development | Initial release — all 5 Foundation simulations |

*This document must be updated whenever curriculum assessment criteria, QTB gate logic, or tool physics specifications change. Any revision to the curriculum document (foundation_curriculum_v2.md) that affects a simulation's learning objective or assessment rubric requires a corresponding update to the relevant Part of this guide before development proceeds.*

---

*End of Document*
