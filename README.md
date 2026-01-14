# Perceptual Event Extractor – Mental Model

This document defines a **Python-based perceptual analysis pipeline** that extracts meaningful, AI-detectable event layers from first-person video, aligned with a 3D trajectory.  
The system is **fully independent from Spatial Lens** and produces reusable perceptual evidence for later analysis or ML research.

---

## 1. Purpose

The system answers one question only:

> **“What perceptual stimuli were present and competing for attention at each moment and position?”**

It does **not**:
- interpret architecture
- evaluate design quality
- apply spatial intent

It produces **raw perceptual layers** that can later be correlated with behavior, navigation outcomes, or trained into models.

---

## 2. Inputs

### A. Video Frames
- Sequence of JPEG frames
- Fixed or known frame rate
- First-person camera
- Contains dynamic agents (people, vehicles, signals, signage)

### B. Trajectory File (`.glb`)
- Time-ordered camera positions
- Optional camera orientation
- Same temporal domain as video frames

### Key Assumption
frame_index ↔ time ↔ 3D position

yaml
Copy code

No reprojection or raycasting is performed.

---

## 3. Core Data Unit

Each frame produces a **Perceptual Sample**:

PerceptualSample {
time: float
position: vec3
ai_layers: {
layer_name: 0 | 1
}
}

yaml
Copy code

Each sample represents **what was perceptually present** at that moment.

---

## 4. AI Event Layers (Detectable from Video Only)

All layers are binary (0 / 1).  
No confidence scores or weights are stored.

---

### A. Social & Movement Context

ai_people_present
ai_people_dense
ai_people_crossing_path
ai_following_flow
ai_counterflow_present

yaml
Copy code

Derived from:
- person detection
- crowd density
- optical flow

---

### B. Traffic & Infrastructure Signals

ai_vehicle_present
ai_vehicle_crossing
ai_traffic_light_visible
ai_crosswalk_visible
ai_signal_competing

yaml
Copy code

Captures attention-competing urban stimuli.

---

### C. Signage & Guidance

ai_sign_visible
ai_directional_sign_visible
ai_text_sign_visible
ai_icon_sign_visible

yaml
Copy code

Detection only — OCR optional in later stages.

---

### D. Visual Competition & Noise

ai_high_motion_scene
ai_visual_clutter_high
ai_luminance_dominant_object
ai_attention_competitor_present

yaml
Copy code

Explains why intended targets may not be noticed.

---

### E. Occlusion & Blocking

ai_target_occluded
ai_partial_occlusion
ai_dynamic_occlusion

yaml
Copy code

Detects foreground obstruction without knowing the target.

---

### F. Environmental Stressors (Optional)

ai_weather_present
ai_low_visibility
ai_night_condition
ai_glare_present

yaml
Copy code

Useful for robustness and stress-scenario analysis.

---

## 5. Processing Pipeline

### Step 1 — Frame-Level Perception
For each frame:
- object detection
- motion estimation
- scene classification

frame_features = {
detected_objects,
motion_score,
density_score,
luminance_stats
}

yaml
Copy code

---

### Step 2 — Layer Activation
Features are converted into binary layers using explicit thresholds.

Example:
ai_people_present = detected_people_count > 0
ai_high_motion_scene = motion_score > threshold

yaml
Copy code

---

### Step 3 — Temporal Smoothing
To avoid flicker:

layer_active = true
IF active for ≥ N consecutive frames

yaml
Copy code

Ensures layers represent **events**, not noise.

---

### Step 4 — Sample Construction
Each time step produces:

{
position_from_glb_at_t,
ai_layers_binary
}

yaml
Copy code

---

## 6. Output Format

### Output File
- `.glb` file
- Contains **point cloud only**

Each point includes attributes:

position: vec3
time: float
ai_people_present: 0|1
ai_vehicle_present: 0|1
ai_sign_visible: 0|1
ai_traffic_light_visible: 0|1
ai_high_motion_scene: 0|1
...

yaml
Copy code

This forms a **perceptual evidence point cloud**.

---

## 7. Design Principles

- Binary layers only (0 / 1)
- No spatial projection or semantic interpretation
- Thresholds explicit and configurable
- Architecture-agnostic
- ML-ready but non-ML-dependent

---

## 8. Future Use (Out of Scope)

This dataset can later be used to:
- correlate perceptual context with navigation behavior
- identify failure-prone perceptual conditions
- train ML models predicting hesitation or misrouting
- enrich Spatial Lens analysis

---

## 9. System Identity

Internal naming suggestions:
- **Perceptual Context Extractor (PCE)**
- **Situated Perception Layer Generator**

This system produces **evidence**, not judgments.

---

## 10. Visual Evidence Baking (Debug & Validation Layer)

To support **trust, validation, and communication**, the system can optionally bake visual evidence directly into the input frames.

This layer is **non-analytical** and exists purely to:
- verify AI detections
- debug thresholds
- communicate findings to non-technical stakeholders

---

## Purpose

> “Why did the system say this event was present?”

Instead of explaining models or thresholds, the system provides **visual proof**.

This is critical for:
- architectural studios
- clients
- reviewers
- future dataset curation

---

## Output

For each processed frame, an **annotated JPEG** is optionally exported.

### File Naming
frame_01234_annotated.jpg

yaml
Copy code

---

## Bounding Box Baking

For every object that activates an AI layer, the system draws:

- Bounding box
- Minimal label (layer name)
- Optional color per category

### Example Layers with Bounding Boxes

ai_people_present → bounding box on detected people
ai_vehicle_present → bounding box on vehicles
ai_sign_visible → bounding box on signage
ai_traffic_light_visible → bounding box on traffic lights
ai_dynamic_occlusion → bounding box on occluding foreground objects

yaml
Copy code

---

## Annotation Rules

- Only **objects that triggered a layer = 1** are drawn
- No confidence scores shown (binary system integrity)
- No architectural interpretation overlays
- Multiple boxes allowed per frame

---

## Visual Encoding (Suggested)

| Category            | Color (Example) |
|---------------------|-----------------|
| People              | Blue            |
| Vehicles            | Orange          |
| Signage             | Green           |
| Signals / Lights    | Red             |
| Occlusions          | Purple          |

(Color mapping configurable, not semantic.)

---

## Baked Evidence Directory Structure

output/
├── perceptual_points.glb
├── baked_frames/
│ ├── frame_00001_annotated.jpg
│ ├── frame_00002_annotated.jpg
│ └── ...

yaml
Copy code

---

## Relationship to Perceptual Samples

Each baked frame corresponds directly to one `PerceptualSample`:

frame_index
↕
time
↕
3D position
↕
ai_layers

yaml
Copy code

This allows:
- visual inspection of false positives / negatives
- manual dataset cleaning
- later supervised labeling if ML is introduced

---

## Design Principle

> **Annotated frames are evidence, not output.**

They:
- do not affect analytics
- do not alter GLB data
- can be disabled entirely in production runs

---

## Future Extensions (Optional)

- Export short GIFs per layer activation
- Heatmap overlays (still binary-driven)
- Reviewer tagging (“false positive”, “missed sign”)

---

This makes the system **auditable**, **explainable**, and **architect-friendly** without compromisin
