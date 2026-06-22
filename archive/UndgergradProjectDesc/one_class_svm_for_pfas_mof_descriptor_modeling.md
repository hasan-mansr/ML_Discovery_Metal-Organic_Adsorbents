# One-Class SVM for PFAS–MOF Descriptor Modeling

## Problem framing
- **Goal:** Learn the *region* of descriptor space occupied by known PFAS-adsorbing MOFs (positive-only data).
- **Why OC-SVM:** No reliable negatives or scalar target; need a soft boundary around positives.

---

## Descriptor set
**Core (chemist-prioritized):**
- PLD (Å)
- LCD (Å)
- Geometric Surface Area (m²/g)
- Void fraction
- Pore volume (cm³/g)

**Candidate / exploratory:**
- Density (g/cm³)

**Dimensionality:** 5–6 features is appropriate for OC-SVM with ~36 samples.

---

## Modeling choices
- **Single multivariate OC-SVM** is standard; >2 features is fine (2D examples are for visualization only).

### What the OC-SVM is optimizing (intuition)
- The one-class SVM learns a *compact region* of descriptor space that contains most of the positive data.
- Unlike binary SVMs, there is no opposing class pushing the boundary from the outside.
- The solution is therefore determined by a trade-off between:
  - Minimizing the volume (or norm) of the enclosing region, and
  - Allowing a controlled fraction of points to lie outside the boundary.
- This trade-off is governed primarily by the parameter **ν**, which prevents the trivial solution of an infinitely large enclosing region.
- **Single multivariate OC-SVM** is standard; >2 features is fine (2D examples are for visualization only).

### Kernel choice (justification)
**Radial Basis Function (RBF)** is preferred for one-class problems in materials descriptor spaces.

**Geometric rationale:**
- The OC-SVM objective is to *enclose* the region occupied by positive examples.
- **RBF kernels induce closed, flexible boundaries** in feature space that can wrap around clusters of points.
- This matches the scientific goal: learning a *compact support* of PFAS-compatible MOFs.

**Why not linear kernels?**
- Linear kernels learn a single half-space (a hyperplane).
- This effectively *bisects* feature space rather than enclosing a region.
- Suitable only if the data occupy a convex, monotonic region extending to infinity — unlikely for MOF descriptors.

**Why not polynomial kernels?**
- Polynomial kernels create global decision surfaces with long-range interactions.
- They tend to extrapolate aggressively and can create unphysical acceptance regions far from the data.
- Boundary shape is harder to control and interpret with small sample sizes.

**Why RBF works well here:**
- Local influence: each training point affects the boundary only within a characteristic length scale (controlled by γ).
- Naturally supports multiple disjoint or curved regions if needed.
- Well-suited to heterogeneous, nonlinear structure–property relationships common in porous materials.

**Kernel:** RBF (default for nonlinear boundaries in materials space).
- **ν (nu):** Controls allowed outliers and acts as a regularization parameter that prevents the enclosing region from expanding indefinitely.
- **γ (gamma):** Set via median pairwise distance heuristic or limited grid search.
- **ν (nu):** Controls allowed outliers (e.g., 0.1–0.2 as a starting range).
- **γ (gamma):** Set via median pairwise distance heuristic or limited grid search.

---

## Critical preprocessing
- **Scaling is mandatory.** Use z-score or robust scaling.
- Rationale: OC BF kernel relies on Euclidean distances; unscaled features (e.g., GSA) will dominate.
- Correlated descriptors (e.g., PLD, LCD, pore volume) are acceptable: the OC-SVM operates on the induced distance geometry rather than assuming feature independence.
- PCA is used strictly for visualization and interpretation, not as a required preprocessing step for training.
- **Scaling is mandatory.** Use z-score or robust scaling.
- Rationale: OC BF kernel relies on Euclidean distances; unscaled features (e.g., GSA) will dominate.

---

## When to train multiple models
Not required for dimensionality, but useful for **scientific interrogation**:
- Geometry-only vs porosity-only vs all-features models
- All-features vs all + density

Purpose: test stability and hypothesis sensitivity, not ensembling.

---

## Assessing descriptor influence and sensitivity
There is no native coefficient-based importance. Use **sensitivity analyses**:
There is no native coefficient-based importance. Use **sensitivity analyses**:

### 1) Leave-One-Feature-Out (LOFO)
- Retrain after removing each feature.
- Compare:
  - Fraction of training points flagged as outliers
  - Changes in anomaly score distribution
  - Boundary stability

### 2) Perturbation sensitivity
- Perturb one feature (±5–10%) and observe score changes.
- Interpretable for domain experts.

### 3) Linear OC-SVM (diagnostic only)
- Fit a linear OC-SVM to inspect weight magnitudes/signs.
- Use for sanity checks; do not over-interpret physically.

### 4) Explainability tools (optional)
- SHAP on the decision function/anomaly score (qualitative only; small-n caution).

---

## What not to do
- Do not report p-values or regression-style coefficients.
- Do not assume monotonic effects.
- Do not over-tune to force perceived importance.

---

## Visualizing the learned region
- Any 2D decision boundary plot represents a **conditional slice** of the learned high-dimensional region.
- PCA-based plots show the intersection of the decision function with the PCA subspace.
- Original-feature plots vary two selected descriptors while fixing all remaining features at their mean values.
- These visualizations are illustrative and should not be interpreted as complete projections of the full decision boundary.

---

## Recommended workflow
1. Train RBF OC-SVM on core features (scaled).
2. Validate stability (ν, γ sensitivity).
3. Add density; compare boundary and scores.
4. Perform LOFO and perturbation analyses.
5. Report *boundary stability and sensitivity*, not coefficients.

---

## Limitations
- Small sample size may lead to boundary uncertainty.
- Results depend on feature scaling and descriptor selection.
- 2D visualizations cannot capture full high-dimensional structure.
- Feature sensitivity reflects local behavior, not global causality.

---

## Outputs to report
- Anomaly scores (continuous)
- In/out decision at chosen threshold
- Sensitivity results supporting descriptor relevance

