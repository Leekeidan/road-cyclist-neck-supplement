# The four clustering variables

This folder holds the Chapter 1 kinematic phenotype analysis (H9). Participants
are grouped by **four cervical kinematic features**, selected from the
univariate results of H1, H6 and H7 rather than from the full set of angle
measures.

## The four variables

| Column | What it measures | Units |
|---|---|---|
| `horiz_mean_ST` | Mean horizontal **rotation** angle during the second-threshold phase | degrees |
| `sag_slope_100W` | **Sagittal flexion per 100 W** during the VO₂max ramp | degrees / 100 W |
| `sag_drift_WU_CD` | **Sagittal drift from warm-up to cool-down** | degrees |
| `sag_peak_WU` | **Peak sagittal flexion during warm-up** | degrees |

Positive sagittal values are flexion; positive horizontal values are rotation.

Two of the four are absolute postures (`horiz_mean_ST`, `sag_peak_WU`), and an
absolute angle depends on how the sensors sat on that participant. Both are
therefore expressed **relative to each participant's own static-neutral
recording**, removing between-participant calibration offsets. The two
change-based features are differences and are unaffected by referencing.

Range of motion and within-phase variability were considered and excluded, as
was the frontal plane, which is prone to gimbal artefact.

## Method

k-means on z-scored features, Euclidean distance, evaluated over k = 2–6.
Parameters, the standardisation, the exclusions and the validity indices used
are in [`../../../docs/METHOD_clustering.md`](../../../docs/METHOD_clustering.md).

Cluster numbers are arbitrary labels from the initialisation. They carry no
ordering, and Cluster 0 here has no relationship to Cluster 0 in any other
analysis.

## Files

| File | Contents |
|---|---|
| `clustering_variables_all_subjects.csv` | The four features per participant, as computed |
| `clustering_variables_complete_cases.csv` | The same with the one missing value filled |
| `clustering_results_k2.csv`, `_k3.csv` | Features plus assigned cluster |
| `cluster_comparison_k2.csv`, `_k3.csv`, `k2_detailed_comparison.csv` | Group comparisons on the four features |
| `cluster_quality_metrics.csv` | Silhouette, Davies–Bouldin and Calinski–Harabasz for k = 2–6 |
| `descriptive_statistics.csv` | Distribution of each feature across the cohort |
| `correlation_matrix.csv` | Correlations among the four features |
| `subject_PC1_scores.csv` | First principal component score per participant |
| `artifact_removal_log.csv` | Angles beyond ±90° removed as gimbal artefact, counted per participant |
| `anthropometric/` | Clusters and PC1 tested against body measures |
| `pain_regression/` | Confirmatory permutation regression of VAS on the four features; `REGRESSION_SUMMARY.txt` documents the model |
| `slope_sign_x_cluster_results.xlsx` | Direction of the flexion slope against cluster membership |
| `PC1 and VAS correlation.prism` | GraphPad Prism file; requires a Prism licence to open |
| figures | `pca_visualization.png`, `optimal_k_analysis.png`, `correlation_matrix.png`, `variable_distributions.png` |

> One participant has no `sag_slope_100W`. The clustering uses a median-filled
> value for them; the regression drops the row instead, which is why the two
> report different *n*.
