# The four clustering variables

This folder holds the Chapter 1 kinematic phenotype analysis — the reported H9
solution. Participants here are grouped by **four cervical kinematic features**,
chosen from the univariate results (H1, H6, H7).

## The four variables

| Column | What it measures | Units |
|---|---|---|
| `horiz_mean_ST` | Mean horizontal **rotation** angle during the second-threshold phase — how far the head is habitually turned under sustained load | degrees |
| `sag_slope_100W` | **Sagittal flexion per 100 W** during the VO₂max ramp — how much further the head drops for each increment of power | degrees / 100 W |
| `sag_drift_WU_CD` | **Sagittal drift from warm-up to cool-down** — how much the resting head position has changed by the end of the protocol | degrees |
| `sag_peak_WU` | **Peak sagittal flexion during warm-up** — the most flexed position reached before the effort begins | degrees |


## What the solution was

Sixty participants, complete data on all four features. Two clusters retained:

| | Cluster 0 (*n* = 34) | Cluster 1 (*n* = 26) | *p* |
|---|---|---|---|
| `sag_peak_WU` | **21.3°** | 7.8° | *** |
| `sag_drift_WU_CD` | **−5.8°** | **+12.2°** | *** |
| `horiz_mean_ST` | **10.2°** | −0.5° | ** |
| `sag_slope_100W` | 7.1 | 2.3 | ns |

**Cluster 0 — flexed and rotated, steep-loading.** 
**Cluster 1 — neutral start, progressive drift.** 

The k = 3 solution is kept alongside as a reference only.


## Files here

- `clustering_variables_all_subjects.csv`, `clustering_variables_complete_cases.csv` — the four features per participant
- `clustering_results_k2.csv`, `clustering_results_k3.csv` — features plus the assigned cluster
- `cluster_comparison_k2.csv`, `cluster_comparison_k3.csv`, `k2_detailed_comparison.csv` — group comparisons on the features
- `cluster_quality_metrics.csv` — silhouette, Davies–Bouldin and Calinski–Harabasz for k = 2–6
- `descriptive_statistics.csv`, `correlation_matrix.csv`, `subject_PC1_scores.csv`
- `artifact_removal_log.csv` — angles beyond ±90° removed as gimbal artefact, counted per participant
- `anthropometric/` — the clusters and PC1 tested against body measures
- `pain_regression/` — the confirmatory permutation regression of VAS on the four features
- figures: `pca_visualization.png`, `optimal_k_analysis.png`, `correlation_matrix.png`, `variable_distributions.png`
