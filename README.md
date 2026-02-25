# Comparative Analysis of Disparity-Filtered vs. Unrestricted Feature Matching in Visual Odometry

Author: Shreyas Yellenki  
Advisors: Dr. David Wettergreen, Tushaar Jain  
Carnegie Mellon University – 16-597

## Overview

This project presents a comparative analysis of two visual odometry (VO) pipelines developed and evaluated on the Argus High Slip dataset for lunar rover navigation.

The primary question investigated:

> Should feature correspondences be restricted to points with valid depth (disparity-filtered), or can a depth-agnostic 2D RANSAC approach produce comparable motion estimates while increasing robustness?

The work evaluates geometric consistency, runtime tradeoffs, translation/rotation divergence, and metric scale recovery.

---

## Motivation

The original MoonRanger VO pipeline:

Feature Detection (Frame A)  
→ Disparity Filter  
→ Feature Matching  
→ Disparity Filter  
→ 3D Clique-Based Outlier Rejection  

This guarantees valid depth for all retained features but discards potentially useful correspondences early in the pipeline.

In sparse-depth environments (e.g., textureless sand), this leads to:
- Reduced feature counts
- Brittle motion estimation
- Higher failure probability

### Proposed Alternative

Feature Detection (Frame A)  
→ Feature Matching (no depth restriction)  
→ 2D RANSAC (Fundamental Matrix)  
→ Essential Matrix + Pose Recovery  

This approach:
- Removes disparity pre-filtering
- Operates purely in image space
- Allows more correspondences
- Uses geometry instead of depth constraints for outlier rejection

---

## Key Contributions

### 1. 2D RANSAC-Based Correspondence Pipeline

- Estimates the Fundamental Matrix using RANSAC
- Converts to Essential Matrix using camera intrinsics
- Recovers pose via chirality checks (`cv2.recoverPose`)
- Produces rotation and translation direction (unit vector)

Unlike the original 3D clique solver, this approach:
- Requires no depth measurements
- Utilizes a larger feature pool
- Increases robustness in low-disparity regions

---

### 2. Computational Performance Analysis

The tradeoff:

- More candidate features → higher matching cost
- RANSAC iterations impact runtime
- 2D RANSAC vs. 3D clique solver comparison

Findings:
- Increasing `max_iters` in RANSAC had negligible gains in correspondence count
- Runtime increased with iteration count
- The proposed pipeline generally runs slower due to larger search space

Runtime was broken down into:
- Feature matching
- Outlier rejection

---

### 3. Geometric Motion Comparison

Both pipelines produce:

- Rotation matrices (R)
- Translation vectors (t)

To compare:

- Original translation (metric) normalized to unit length
- Angular deviation between translation directions measured
- Relative rotation computed:
  
  C = R_newᵀ R_orig

- Axis-angle representation used for interpretability

Important note:
Divergence between pipelines does not imply correctness of either — both are subject to noise, depth uncertainty, and algorithmic approximations.

---

### 4. Metric Scale Recovery

Since the RANSAC pipeline yields only translation direction, scale must be recovered separately.

Using depth-available correspondences, scale is solved via least squares:

Minimize:

‖ R P₁ + s t̂ − P₀ ‖²

Closed-form solution:

s = -(1/N) Σ (aᵢ · t̂)

Where:
- aᵢ = (R P₁ᵢ − P₀ᵢ)
- t̂ is unit translation
- N is number of depth-valid correspondences

This enables metric translation recovery while still leveraging depth-free matching.

---

## Results

### Observations

- Significant increase in feature correspondences
- Comparable rotational estimates
- Translation direction divergence requires further validation
- Early dataset frames (stationary rover) produce noisy angular comparisons
- Scale recovery behaves consistently when sufficient depth points exist

### Failure Modes Analyzed

New pipeline:
- RANSAC failure
- Too few points
- Scale computation error

Original pipeline:
- Small clique
- Depth-related failures

---

## Future Work

- Multi-dataset validation
- Refined feature tiling strategy (40×40 → potentially 20×20)
- Hybrid depth-aware RANSAC models
- Improved scale robustness

---

## Takeaway

Removing disparity pre-filtering increases feature utilization and robustness in sparse-depth environments. While computationally more expensive, the depth-agnostic 2D RANSAC pipeline produces motion estimates geometrically consistent with the original 3D depth-constrained method.

This work highlights the tradeoff between depth certainty and correspondence richness in visual odometry systems for planetary robotics.
