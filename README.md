# Image Stitching Pipeline

> A from-scratch panoramic image stitching pipeline built with classical computer vision — no deep learning, no OpenCV stitcher. Harris corner detection, normalized patch descriptors, nearest-neighbour matching, RANSAC robust estimation, and affine warping.

---

## Table of Contents

- [Image Stitching Pipeline](#image-stitching-pipeline)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Pipeline](#pipeline)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Results](#results)
  - [Parameters](#parameters)
  - [Project Structure](#project-structure)
  - [Notes \& Limitations](#notes--limitations)

---

## Overview

This notebook implements a complete image stitching pipeline from scratch.

Two overlapping images are loaded through a GUI file picker, processed using a sequence of classical computer vision techniques, and stitched into a single panorama.

Sensitivity analyses are included for both patch size and number of matches to evaluate and justify the selected hyperparameters.

---

## Pipeline

```text
Left image + Right image  (selected via GUI)
           │
           ▼
  Harris Corner Detection
           │
           ▼
  Patch Descriptor Extraction
  sensitivity analysis: patch_size ∈ {5, 9, 13, 17, 21} → selected: 21
           │
           ▼
  Descriptor Matching
  ├── Euclidean distance       (lower = more similar)
  └── Normalized correlation   (higher = more similar)
           │
           ▼
  Top-K Match Selection
  sensitivity analysis: top_k ∈ {50, 100, 150, 200} → selected: 100
           │
           ▼
  RANSAC Affine Estimation
  (1000 iterations, threshold = 25 px)
           │
           ▼
  Warp & Stitch → Panorama
           │
           ▼
  Accuracy Evaluation + Save Results
```

---

## Installation

```bash
pip install opencv-python scikit-image matplotlib numpy pandas
```

> **Tkinter** is required for the GUI image picker.  
> It ships with most Python installations. On Linux you may additionally need:

```bash
sudo apt install python3-tk
```

---

## Usage

1. Clone or download this repository
2. Open the notebook:

```bash
jupyter notebook images_stitching.ipynb
```

3. Run all notebook cells (`Cell → Run All`)
4. Two file-picker dialogs will open:
   - Select the **left** image first
   - Then select the **right** image
5. Results are automatically saved in the `results/` folder

---

## Results

After a successful run, the `results/` folder contains the following files (each suffixed with a timestamp):

| File | Description |
|------|-------------|
| `panorama_<ts>.png` | Final stitched panorama |
| `harris_corners_<ts>.png` | Harris corner detections overlaid on both images |
| `inlier_matches_<ts>.png` | RANSAC inlier correspondences |
| `sensitivity_patch_size_<ts>.png` | Sensitivity analysis for patch sizes `{5, 9, 13, 17, 21}` |
| `sensitivity_top_k_<ts>.png` | Sensitivity analysis for top-K values `{50, 100, 150, 200}` |

---

## Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `patch_size` | `21` | Descriptor window size around each keypoint |
| `top_k` | `100` | Number of candidate matches passed to RANSAC |
| `num_iterations` | `1000` | Number of RANSAC iterations |
| `threshold` | `25.0` | RANSAC inlier threshold (squared pixel distance) |
| `sample_size` | `3` | Minimum point pairs required to estimate an affine transform |
| `k` (Harris) | `0.04` | Harris detector sensitivity parameter |
| `min_distance` | `5` | Minimum distance between detected corners |
| `threshold_rel` | `0.01` | Relative threshold for corner peak detection |

---

## Project Structure

```text
.
├── images_stitching.ipynb   # Main notebook
├── README.md
└── results/                 # Auto-created on first run
    ├── panorama_*.png
    ├── harris_corners_*.png
    ├── inlier_matches_*.png
    ├── sensitivity_patch_size_*.png
    └── sensitivity_top_k_*.png
```

---

## Notes & Limitations

- **Affine vs Homography**  
  An affine transform (6 DOF) works well for images captured with mostly translational or small rotational shifts on approximately planar scenes.  
  For strong perspective distortions or non-planar 3D scenes, homography estimation (8 DOF) would be more appropriate.

- **No blending**  
  The current implementation performs a hard overlay of image 1 on the warped image 2.  
  Seam blending techniques such as linear blending or multiband blending would produce smoother transitions.

- **Headless environments**  
  Tkinter requires a graphical display.  
  When running on remote servers or headless systems, the GUI loading step should be replaced with direct `cv2.imread()` calls.

- **Image overlap requirement**  
  The pipeline assumes the presence of a visible overlapping region with sufficient texture and detectable corners.  
  Very smooth or repetitive textures may reduce matching quality.