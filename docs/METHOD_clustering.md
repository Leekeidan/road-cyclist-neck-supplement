# Cluster assignment — k-means and PCA

## 1. What it is

An unsupervised grouping of participants into a small number of behavioural
phenotypes, based on several measured variables at once rather than one at a
time. Principal component analysis is used alongside it to describe how much of
the variation those variables share, and to place participants on a
low-dimensional map for plotting.

## 2. Formula

**k-means** partitions *n* participants into *k* clusters by minimising the
within-cluster sum of squared Euclidean distances:

```
argmin  Σ   Σ   ‖ xᵢ − μⱼ ‖²
  S    j=1..k  i ∈ Sⱼ
```

| Term | Definition |
|---|---|
| `xᵢ` | Participant *i* as a vector of z-scored features |
| `μⱼ` | Centroid (mean vector) of cluster *j* |
| `Sⱼ` | The set of participants assigned to cluster *j* |

**Standardisation**, applied before clustering so that features with larger
natural ranges do not dominate the distance:

```
z = (x − mean(x)) / SD(x)
```

**PCA** re-expresses the standardised features as orthogonal components ordered
by variance explained; PC1 is the single axis capturing the most variation.

## 3. Sign conventions and units

- **Cluster labels are nominal and arbitrary.** "Cluster 0" and "Cluster 1"
  carry no ordering and no inherent meaning — the numbering is an artefact of
  initialisation. Clusters must always be described by their feature profile,
  never by their index.
- **z-scored features are dimensionless**, in standard deviations.
- **PC scores are dimensionless.** Their sign is arbitrary: a component and its
  negation are the same component, so the direction of PC1 must be read from
  its loadings, not assumed.
- Variance explained is reported as a percentage.

## 4. Pipeline from raw file to value

### Chapter 1 — kinematic phenotypes (4 features)

1. **Features**, selected from the univariate results (H1, H6, H7):
   mean horizontal rotation angle at second threshold; sagittal flexion slope
   per 100 W during the VO₂max ramp; sagittal drift from warm-up to cool-down;
   peak sagittal flexion during warm-up.
2. **Static referencing** — absolute angles are expressed relative to each
   participant's own static-neutral posture (the mean of the static recording),
   removing between-participant calibration offsets. The two change-based
   features (drift, slope) are mathematically unaffected by this.
3. **Standardisation** — `sklearn.preprocessing.StandardScaler` (population SD,
   `ddof = 0`).
4. **Clustering** — `sklearn.cluster.KMeans`, Euclidean distance,
   **`n_init = 100`**, **`max_iter = 100`**, **`random_state = 42`**.
5. **Choosing k** — evaluated over **k = 2–6** using the silhouette
   coefficient, the Davies–Bouldin index and the Calinski–Harabasz index.
6. **Characterisation** — clusters compared on anthropometric, fitness and pain
   variables by Mann–Whitney U; PC1 correlated against the same variables by
   Spearman's rho.

### Chapter 2 — SCM activation phenotypes (5 variables)

Focused on the sternocleidomastoid (SCML, SCMR), which carried the strongest
fatigue signal. **Baseline phase = Warmup; fatigue phase = WanT; the
co-contraction pair used is SCML–MTL** (ipsilateral flexor–extensor).
Standardisation z-score, distance Euclidean, seed 42.

**60 participants collected, 59 complete cases.** Solutions were compared at
k = 2 (silhouette **0.443**) and k = 3 (silhouette **0.426**); **k = 2 was
retained**. The k = 3 solution isolates a singleton outlier and is kept only
as a reference figure.

1. Participants with **at most one missing variable** are retained; remaining
   gaps are **median-imputed** over the kept rows.
2. Standardise, then k-means as above.

### Chapter 4 — integrated clustering

`build_integrated_dataset.py` assembles kinematic, activation and morphology
features into one table; `ch4_pca_and_flip.py` runs the PCA;
`cluster_SCM_5vars_plus_muscleSize.py` adds signed SCM structural asymmetry
(see `METHOD_asymmetry.md`) to the Chapter 2 feature set.

## 5. Exclusions and outlier handling

- **Chapter 1**: angle values beyond **±90°** are removed before analysis as
  physiologically implausible (gimbal wrap); the number removed is logged.
- **Chapter 2**: participants missing more than one clustering variable are
  dropped; single gaps are median-imputed.
- No outlier removal is applied to the cluster solution itself.

## 6. Where it is computed

All clustering is implemented in **Python** with `scikit-learn`
(`StandardScaler`, `KMeans`, `PCA`), except the Chapter 2 stability checks,
which are in **R**.

| Scope | Implementation |
|---|---|
| Ch 1 — kinematic phenotypes (reported) | Static-referenced four-variable k-means, plus a confirmatory permutation regression in `statsmodels` |
| Ch 2 — SCM phenotypes (reported) | Five-variable k-means, ported from an earlier R implementation and validated against it |
| Ch 2 — stability checks | R, resampling the k = 2 solution |
| Ch 3 — fitness clusters | k-means on the fitness variables, with PCA for display |
| Ch 4 — integrated | The Chapter 2 feature set extended with signed SCM structural asymmetry, plus the integrated PCA |

