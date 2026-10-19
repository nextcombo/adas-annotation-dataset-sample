# Production-Grade Autonomous Driving Annotation Dataset — Sample Description

> This document is written from a direct, file-by-file inspection of the actual sample files in the `data car` folder on the Desktop. It records the data layout, sensor configuration, annotation format, and per-category details. **Important: this folder is a *sample* dataset, not the full dataset**, and is intended only for evaluating data format and quality — it must not be used directly as a training set.

> **Contact / Data Access** — If you need the full dataset or a partial subset, please contact **lonlonago@foxmail.com**. Data is billed by volume (priced per quantity).

---

## 0. Important Notice (Sample Nature)

| Item | Description |
|---|---|
| Data nature | **Sample** — for format evaluation and quality check, not the full dataset |
| Full-dataset scale | Up to **~5 million frames** of production-grade ADAS annotation data available |
| This sample contains | 1 multi-sensor fusion sequence sample (`demo.bin`) + 4 category archives (`*.7z`) |
| Compliance | Face and license-plate anonymization completed, plus sensitive geo-frame screening — compliant with relevant Chinese automotive data-security regulations |

---

## 1. Desktop File Inventory

The `data car` directory contains 5 files, ~1.73 GB total:

| File | Size | Type | Contents |
|---|---|---|---|
| `demo.bin` | 332 MB | **tar archive** (extension mislabeled as .bin) | 1 fusion-perception sequence sample (point cloud + images + calibration + label dir) |
| `BEV车道线.7z` | 444 MB | 7z archive | BEV lane category sample (point cloud + polyline annotations) |
| `CONE.7z` | 213 MB | 7z archive | Traffic-cone / construction-obstacle category sample (image + 2D boxes) |
| `TLR.7z` | 293 MB | 7z archive | Traffic-light / traffic-sign category sample (image + 2D boxes / polygons) |
| `XMS-乘用车-MOD.7z` | 447 MB | 7z archive | Passenger-vehicle multi-object-detection sample (CMS rear-view image + 2D boxes + skeletons) |

> `demo.bin` is actually a tar archive — use `tar -tf demo.bin` to list and `tar -xf demo.bin` to extract. The other four are standard 7z files, extractable with 7-Zip / py7zr.

---

## 2. Data Organization

### 2.1 `demo.bin` — Multi-Sensor Fusion Sequence Sample

Extracted directory structure (sequence `202607071329_3`):

```
demo/
├── readme.md                         # data-organization note
├── front_220_lidar_pmf.json          # sensor calibration (intrinsics / extrinsics / vehicle coords)
├── cloud/202607071329_3/             # LiDAR point clouds (.pcd), 113 frames
├── images/202607071329_3/            # camera images (.jpg), 113 frames
└── label/202607071329_3/label/       # annotation results (.json), 51 files (all empty arrays [] in this sample)
```

- `readme.md` text:
  > cloud: point cloud / images: images / label: annotation results / front_220_lidar_pmf.json: sensor parameters, including camera intrinsics and LiDAR-to-camera extrinsics.
- The sequence has matching point-cloud and image counts (113 frames each), but the `label` directory's JSONs are **all empty arrays `[]`** in this sample — i.e., `demo.bin` only demonstrates the fusion data layout (point cloud + image + calibration); it does not contain filled annotations.
- Image name example: timestamped frames under `202607071329_3`.

#### 2.1.1 Sample Figures (extracted directly from demo.bin)

Synchronized camera-image samples (front 220° wide-FOV, 5115×1284 originals scaled down; 5 frames sampled across the sequence):

![demo camera image 1](figures/demo/demo_raw_01.jpg)
*Frame 1/5 — front 220° wide-FOV camera, undistorted*

![demo camera image 2](figures/demo/demo_raw_02.jpg)
*Frame 2/5*

![demo camera image 3](figures/demo/demo_raw_03.jpg)
*Frame 3/5*

![demo camera image 4](figures/demo/demo_raw_04.jpg)
*Frame 4/5*

![demo camera image 5](figures/demo/demo_raw_05.jpg)
*Frame 5/5*

LiDAR point-cloud top-down (BEV) renders of the same sequence (~120–150k points per frame, color = height Z):

![demo point cloud BEV 1](figures/demo/demo_bev_01.png)
*Frame 1/5 — LiDAR BEV top-down (X forward / Y lateral, meters)*

![demo point cloud BEV 2](figures/demo/demo_bev_02.png)
*Frame 2/5*

![demo point cloud BEV 3](figures/demo/demo_bev_03.png)
*Frame 3/5*

![demo point cloud BEV 4](figures/demo/demo_bev_04.png)
*Frame 4/5*

![demo point cloud BEV 5](figures/demo/demo_bev_05.png)
*Frame 5/5*

3D view of frame 1 (color = height):

![demo point cloud 3D](figures/demo/demo_3d_01.png)
*LiDAR 3D view — road surface, roadside buildings and vehicle outlines visible*

### 2.2 `*.7z` — CVAT-Format Category Packages

The four 7z files share a highly consistent internal structure, each corresponding to one category and one captured sequence:

```
<CategoryName>/
└── <SequenceID>/
    ├── annotations.xml               # CVAT annotation file (core; holds all frame annotations)
    └── images/                       # image / point-cloud source data
```

- **The annotation format is CVAT (Computer Vision Annotation Tool) standard XML export**, a common industry format for autonomous-driving / vision labeling. It can be re-imported into CVAT or converted to COCO / YOLO / KITTI, etc.
- Each package contains a single `annotations.xml` (all frames of the sequence); source data is placed under `images/` in per-scene subfolders.

---

## 3. Sensor Configuration & Coordinate Systems

`front_220_lidar_pmf.json` inside `demo.bin` provides the calibration for the fusion sequence and is key to understanding the coordinate systems:

| Field | Meaning | Sample value |
|---|---|---|
| `data_path` | original data path hint | `/mid/TRX5_01/20260707/sync/front_220_pmf` |
| `intrinsic` | camera intrinsics (3×3) | fx≈1472.41, fy≈1472.09, cx=2557.5, cy=642.0 |
| `dist_coeffs` | distortion coefficients | `[0,0,0,0]` (already undistorted) |
| `extrinsic` | LiDAR→camera extrinsics (4×4 homogeneous) | translation ≈ [-0.009, -0.575, -0.911] m |
| `to_vcs` | transform to Vehicle Coordinate System (VCS) (4×4) | translation ≈ [0.012, 1.359, -2.029] m |

**Key inferences**:
- Intrinsics `cx=2557.5` (≈ half the image width) correspond to a **5115×1284** wide-FOV camera — exactly matching the CONE category image size (5115×1284). This indicates CONE/TLR use a "front 220° wide-FOV camera" (`front_220` = 220° FOV; image filenames carry `_undist` meaning undistorted).
- The dataset is a **multi-sensor fusion** architecture: LiDAR point cloud + front / surround camera images, mutually projectable via calibration (image 2D box ↔ point cloud / BEV).

### 3.1 BEV Coordinate Note (Important)

The BEV lane category is **not annotated in pixels**; instead, based on the point cloud in a Bird's-Eye View (BEV), lane markings are annotated as polyline point sequences whose coordinates are in **meters (local BEV coordinate system)**:

- Measured polyline coordinate range: `x ∈ [-72.57, 280.81]`, `y ∈ [-72.58, 281.70]`
- Origin at the ego vehicle; X axis extends forward (up to ~280 m), Y axis is lateral left/right (~±72–281 m)
- Therefore `image/@width` in CVAT has no real pixel meaning for BEV (placeholder values appear in the sample); do not interpret it as pixel size — treat it as BEV grid coordinates.

---

## 4. Annotation Format Details (CVAT XML)

### 4.1 Common Fields

Each `<image>` node carries `name` (source filename), `width`, `height`, `id`; its child nodes are the specific annotations, all carrying:

| Attribute | Meaning |
|---|---|
| `label` | class name |
| `source` | annotation source (all `manual` in sample, i.e., human-labeled) |
| `occluded` | whether occluded (0/1) |
| `z_order` | stacking order |

### 4.2 Annotation Type Reference

| Annotation type (XML tag) | Fields | Applicable category |
|---|---|---|
| `box` (2D bounding box) | `xtl, ytl, xbr, ybr` (top-left / bottom-right pixel coords) | CONE / TLR / XMS |
| `polyline` | `points` ("x,y;x,y;…" meter-based BEV coords) | BEV lane |
| `polygon` | `points` (pixel coords) | TLR (ground text, etc.) |
| `skeleton` (keypoints) | composed of multiple nodes, with `label` | XMS (pedestrian / rider keypoints) |

### 4.3 Attribute System (example: XMS passenger-vehicle MOD)

XMS defines rich **attributes** in its CVAT label definitions, reflecting the granularity of a production dataset:

| Attribute | Values |
|---|---|
| Truncation level | None(0%) / Low(0–30%) / Medium(30–50%) / High(50–70%) / VeryHigh(70–90%) |
| Occlusion level | same five levels as above |
| Visible face | front / rear (car, van, bus) |
| Truck type | light / heavy / engineering / other (truck only) |
| Oversized width | No / Yes (truck only) |
| Has person | with person / without person / uncertain (tricycle only) |
| Low-detection-confidence | No / Yes |
| Low-class-confidence | No / Yes |
| Picture attribute | No / Yes (target in a picture/image, not real scene) |
| Fake attribute | No / Yes (virtual / synthetic target) |
| Negative sample | No / Yes |

> Such attributes support hard-example mining, quality grading, and stratified evaluation — a key difference between production datasets and academic ones.

---

## 5. Per-Category Data Cards (Measured from Sample)

> "Frames / annotations" below are measured values inside the sample archives; the full dataset is far larger.

| Category | Source | Annotation type | Frames | Total ann. | Main classes |
|---|---|---|---|---|---|
| **BEV lane** | PCD point cloud | polyline | 56 | 883 | lane(682), curb(201) |
| **CONE** (cone/construction) | jpg 5115×1284 | box | 154 | 1096 | traffic cone(502), anti-collision barrel(199), isolation column(128), diversion sign(99), construction warning sign(92), etc. (12 classes) |
| **TLR** (light/sign) | jpg multi-res | box + polygon | 203 | 2648 | light box(1362), traffic-light box(919), arrow box(267), ground text(95), etc. |
| **XMS passenger MOD** | jpg 1936×1552 (CMS rear) | box + skeleton | 617 | 4742 | car(2643), truck(524), pedestrian(593), 2-wheeler(431), bus(131), tricycle(106), van(314) |

### 5.1 BEV Lane
- Bird's-eye-view annotation on LiDAR point cloud; **meter-based BEV-coordinate polylines, not pixels**.
- Only 2 classes: **lane**, **curb**.
- Suited to BEV perception, lane generation, drivable-area tasks; complements `ADAS_FreeSpace` and `ADAS_LLD` in the full set.

**Measured sample figures (point-cloud top-down renders + annotation overlays, extracted from BEV车道线.7z):**

Raw LiDAR top-down point clouds (true-RGB coloring, 540–900k points per frame):

![BEV point cloud 1](figures/bev/bev_lidar_01.png)
*Frame 1/4 — LiDAR BEV top-down (true-RGB coloring, X forward up to ~280 m)*

![BEV point cloud 2](figures/bev/bev_lidar_02.png)
*Frame 2/4*

![BEV point cloud 3](figures/bev/bev_lidar_03.png)
*Frame 3/4*

![BEV point cloud 4](figures/bev/bev_lidar_04.png)
*Frame 4/4*

Lane / curb annotation overlays (red / green / yellow / blue = different polyline objects; meter-based coordinates plotted directly in the point-cloud frame):

![BEV annotation overlay 1](figures/bev/bev_lane_01.png)
*Frame 1/4 — lane + curb polylines overlaid on the point cloud; annotations align with road structure*

![BEV annotation overlay 2](figures/bev/bev_lane_02.png)
*Frame 2/4*

![BEV annotation overlay 3](figures/bev/bev_lane_03.png)
*Frame 3/4*

![BEV annotation overlay 4](figures/bev/bev_lane_04.png)
*Frame 4/4*

> ⚠️ **Export quirk found in this sample (measured)**: in some `<polyline>` elements the `points` string **interleaves coordinates of multiple sub-curves** (e.g., one polyline mixes a lateral arm at y≈-2.5, a longitudinal arm at x≈-2.36, and corner points in rotation). Connecting them in file order produces spurious filled triangles. Consumers should split/rebuild curves by geometric continuity or confirm the export format with the provider. In the overlays above, near-straight segments are re-sorted and drawn as lines; heavily interleaved ones are shown as scatter points.

### 5.2 CONE (Cone / Construction Obstacle)
- **2D bounding boxes** on front 220° wide-FOV undistorted images (5115×1284).
- 12 fine-grained classes covering construction / cone scenes: traffic cone, traffic pole, isolation column, other pole, anti-collision barrel, triangle sign, diversion sign, construction warning sign, mixed warning sign, unknown, mask, carton.
- Sub-scene folder example: `shigongjingshipai` (construction warning sign).
- A construction-specific subset of `ADAS_2D_MOD`, critical for AEB / avoidance functions.

**Measured sample figures (raw images + 2D-box annotation overlays, extracted from CONE.7z; all 5 frames carry annotations):**

![CONE raw 1](figures/cone/cone_raw_01.jpg)
*Frame 1/5 — front 220° wide-FOV raw image: intersection work zone with cones / barriers / warning signs grouped*

![CONE annotation overlay 1](figures/cone/cone_ann_01.jpg)
*Frame 1/5 — 2D-box overlay (labels above boxes are CVAT class names)*

![CONE raw 2](figures/cone/cone_raw_02.jpg)
*Frame 2/5*

![CONE annotation overlay 2](figures/cone/cone_ann_02.jpg)
*Frame 2/5 — annotation overlay*

![CONE raw 3](figures/cone/cone_raw_03.jpg)
*Frame 3/5*

![CONE annotation overlay 3](figures/cone/cone_ann_03.jpg)
*Frame 3/5 — annotation overlay*

![CONE raw 4](figures/cone/cone_raw_04.jpg)
*Frame 4/5*

![CONE annotation overlay 4](figures/cone/cone_ann_04.jpg)
*Frame 4/5 — annotation overlay*

![CONE raw 5](figures/cone/cone_raw_05.jpg)
*Frame 5/5*

![CONE annotation overlay 5](figures/cone/cone_ann_05.jpg)
*Frame 5/5 — annotation overlay*

### 5.3 TLR (Traffic Light / Traffic Sign)
- Multi-resolution images (including 5115×1284, 4915×2478, 4726×1192, etc.), indicating data aggregated from **multiple cameras / viewpoints** (wide-FOV + possibly fisheye).
- Annotations include `box` (light box, traffic-light box, arrow box) and `polygon` (ground text), 5 classes total.
- Sub-scene example: `jinjulidamubiaojiaotongdeng` (prohibitory-target traffic light).
- Supports traffic-light state recognition, sign detection, ground-text recognition, etc.

**Measured sample figures (raw images + box/polygon overlays, extracted from TLR.7z; all 5 frames carry annotations):**

![TLR raw 1](figures/tlr/tlr_raw_01.jpg)
*Frame 1/5 — front wide-FOV raw image (one of the multi-resolution sequences)*

![TLR annotation overlay 1](figures/tlr/tlr_ann_01.jpg)
*Frame 1/5 — light-box / traffic-light / arrow 2D-box overlay*

![TLR raw 2](figures/tlr/tlr_raw_02.jpg)
*Frame 2/5*

![TLR annotation overlay 2](figures/tlr/tlr_ann_02.jpg)
*Frame 2/5 — annotation overlay*

![TLR raw 3](figures/tlr/tlr_raw_03.jpg)
*Frame 3/5*

![TLR annotation overlay 3](figures/tlr/tlr_ann_03.jpg)
*Frame 3/5 — annotation overlay (includes polygon ground-text class)*

![TLR raw 4](figures/tlr/tlr_raw_04.jpg)
*Frame 4/5*

![TLR annotation overlay 4](figures/tlr/tlr_ann_04.jpg)
*Frame 4/5 — annotation overlay*

![TLR raw 5](figures/tlr/tlr_raw_05.jpg)
*Frame 5/5*

![TLR annotation overlay 5](figures/tlr/tlr_ann_05.jpg)
*Frame 5/5 — annotation overlay*

### 5.4 XMS Passenger-Vehicle MOD (CMS Electronic Mirror Multi-Object Detection)
- Images from a **CMS (Camera Monitor System, electronic rearview mirror) rear camera** (1936×1552); folder name `CMS_mod_Rear`.
- Contains both **2D bounding boxes (box)** and **skeleton keypoints**: boxes for vehicles / pedestrians / riders detection; skeletons for pedestrian / rider pose keypoints.
- 7 main classes + complete attribute system (see 4.3).
- Corresponds to **CMS‑mod electronic-mirror object detection** (passenger-vehicle version) in the full set; pairs with the commercial-vehicle scarce category `CMS‑lld fisheye lane` under the CMS series.

**Measured sample figures (raw images + 2D-box / skeleton-keypoint overlays, extracted from XMS-乘用车-MOD.7z; all 5 frames carry annotations):**

![XMS raw 1](figures/xms/xms_raw_01.jpg)
*Frame 1/5 — CMS electronic-mirror rear view (1936×1552; the mirrored text is a normal optical property of CMS rear view)*

![XMS annotation overlay 1](figures/xms/xms_ann_01.jpg)
*Frame 1/5 — pedestrian / 2-wheeler 2D boxes + skeleton keypoints overlay (labels above boxes are CVAT class names)*

![XMS raw 2](figures/xms/xms_raw_02.jpg)
*Frame 2/5*

![XMS annotation overlay 2](figures/xms/xms_ann_02.jpg)
*Frame 2/5 — annotation overlay*

![XMS raw 3](figures/xms/xms_raw_03.jpg)
*Frame 3/5*

![XMS annotation overlay 3](figures/xms/xms_ann_03.jpg)
*Frame 3/5 — annotation overlay*

![XMS raw 4](figures/xms/xms_raw_04.jpg)
*Frame 4/5*

![XMS annotation overlay 4](figures/xms/xms_ann_04.jpg)
*Frame 4/5 — annotation overlay*

![XMS raw 5](figures/xms/xms_raw_05.jpg)
*Frame 5/5*

![XMS annotation overlay 5](figures/xms/xms_ann_05.jpg)
*Frame 5/5 — annotation overlay*

> 📷 **About all figures in this document**: every figure is extracted / rendered from the actual sample data and stored in the `figures/` folder next to this file (relative paths — ship the folder together with the document). Point-cloud render params: top-down X forward / Y lateral (meters), color = height or true RGB; annotation overlays are drawn directly from the parsed CVAT XML with no manual touching-up.

---

## 6. Full-Dataset Category List (from supplementary info)

This sample covers only **part** of the categories; the full set (~5M frames) is broader:

### 6.1 Passenger-Vehicle Main Categories (sample covers part)
| Category | Description | In this sample |
|---|---|---|
| ADAS_2D_MOD | 2D multi-object detection | Yes (CONE/TLR are specialized subsets) |
| ADAS_3D_MOD | 3D multi-object detection | No |
| AEB dangerous cases | automatic-emergency-braking hazardous scenarios | No |
| ADAS 4-class segmentation | semantic segmentation (4 classes) | No |
| ADAS_LLD | low-level lane detection | No |
| ADAS_FreeSpace | drivable free-space | No |
| BEV lane | bird's-eye-view lane | ✅ Yes (BEV车道线.7z) |

### 6.2 Commercial-Vehicle Scarce Categories (not directly in sample; XMS is the passenger-vehicle reference)
| Category | Description |
|---|---|
| CMS‑mod | electronic-mirror object detection (passenger-vehicle version in XMS sample) |
| CMS‑lld | electronic-mirror fisheye lane annotation |
| AVLS‑foot-keypoints | foot-keypoint regulatory dataset |

---

## 7. Sample vs. Full Dataset

| Dimension | This sample | Full dataset |
|---|---|---|
| Scale | ~1030 annotated frames + 113 template frames | ~50 million frames |
| Category coverage | BEV / CONE / TLR / XMS (+ demo template) | all 7+3 categories above |
| Annotation completeness | demo labels empty; 7z labels complete & usable | full annotations |
| Use | format evaluation, QC, POC | model training / evaluation |
| Anonymization | anonymized (face / plate / geo-frame) | same, compliant |

---

## 8. Usage Notes & Cautions

1. **Extraction tools**: `demo.bin` via tar; `*.7z` via 7-Zip / `py7zr` (Python).
2. **Annotation parsing**: `annotations.xml` inside 7z is CVAT format — import via CVAT or parse with `xml.etree`; can be converted to COCO / YOLO / KITTI in one step.
3. **BEV coordinates**: BEV lane uses meter-based polyline coordinates — do not treat as pixels; understand together with the point-cloud frame and BEV range.
4. **Fusion projection**: `front_220_lidar_pmf.json` in `demo.bin` provides intrinsics / extrinsics / vehicle coords, usable for joint training and validation of image 2D box ↔ point cloud / BEV.
5. **Attribute usage**: Categories like XMS carry truncation / occlusion levels, negative samples, fake / picture flags — recommended for stratified evaluation and hard-example mining.
6. **Compliance**: Data is anonymized and geo-sensitive frames screened; distribution and use must still comply with Chinese automotive data-security regulations; do not attempt to recover real geographic locations or identities.
7. **Sample limitation**: This folder is a format sample only — **not for direct training**; contact the provider for full-dataset authorization and download.

---

## 9. Appendix

### 9.1 Image / Point-Cloud Naming Patterns
- CONE/TLR: `220_<timestamp>_<seq>_undist.jpg` (`220` = 220° FOV, `undist` = undistorted)
- XMS: `CMS_mod_Rear_<date>_<cam>_out/<date>-<seq>_<sub>.jpg`
- BEV: `202603121147_1.pcd` (date_time_seq)
- Sequence IDs inside 7z: 6816 / 6229 / 6280 / 1221 (one capture sequence per category)

### 9.2 Quick Reference of Key Measured Values
- BEV polyline coordinate range: x∈[-72.57, 280.81], y∈[-72.58, 281.70] (meters)
- Camera intrinsics: fx≈1472.4, fy≈1472.1, cx=2557.5, cy=642.0 (undistorted)
- CONE image size: 5115×1284 ｜ XMS image size: 1936×1552 ｜ TLR: mixed resolutions

---
*Document compiled from a direct file inspection of the `data car` sample (2026-10-18) and the provider's supplementary information. All annotations are human-labeled (source=manual).*

---

## 10. Contact / Data Access

If you need the **full dataset** or a **partial subset**, please contact **lonlonago@foxmail.com**.

- Data is billed by volume — **priced per quantity** (e.g., by number of frames / sequences).
- This sample (the `data car` folder) is provided for format and quality evaluation only and does not include the full dataset.
