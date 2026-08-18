# Chapter 4 — integration

Brings the three preceding chapters into one analysis: cervical kinematics
(Chapter 1), SCM activation (Chapter 2) and muscle morphology (Chapter 3), to
ask whether they describe the same cyclists or independent ones.


## Start with the logs

| File | Contents |
|---|---|
| `STEP1_BUILD_LOG.txt` | How the integrated table was assembled: what each source contributed, coverage across domains, the edge cases, and the cross-tabulation of the Chapter 1 kinematic phenotypes against the Chapter 2 activation phenotypes |
| `CH4_ANALYSIS_LOG.txt` | The analysis itself, parts A–D plus the MANOVA, with every model, its *N*, and its output |

## The integrated table

| File | Contents |
|---|---|
| `integrated_per_subject.csv` | One row per participant, columns drawn from all three chapters plus anthropometrics — the table every analysis below reads |
| `coverage_summary.csv` | Which participants have which domains |
| `ch4_clustered_with_dvs.csv` | The same table with the integrated cluster assignment attached |
| `kin_x_scm_crosstab.csv` | Chapter 1 phenotype against Chapter 2 phenotype |

**Coverage.** 62 distinct participants appear across the four sources, but only
**55** have all three domains, and the integrated clustering runs complete-case
on **51**. The asymmetric coverage is set out in `STEP1_BUILD_LOG.txt`: 542 and
576 have ultrasound but no cycling data; 613, 619 and 620 have cycling data but
no scan; 597 is absent from the Chapter 2 SCM clustering (see
`../chapter1_pain/H8_scm_activation/` for why).

## The six integrated features

Two per chapter, chosen to represent each domain:

| Feature | Source |
|---|---|
| `horiz_mean_ST` | Chapter 1 — mean horizontal rotation at second threshold |
| `sag_peak_WU` | Chapter 1 — peak sagittal flexion during warm-up |
| `SCM_Peak_MVC_WanT` | Chapter 2 — peak SCM activation during the Wingate |
| `SCM_Bilateral_Asymmetry` | Chapter 2 — left–right SCM activation difference |
| `semisp_C4_LongAP` | Chapter 3 — semispinalis C4, longitudinal AP |
| `semisp_POP_LongAP_R` | Chapter 3 — semispinalis POP, longitudinal AP, right |

## The analyses, and which figure belongs to each

**A integrated clustering.** k-means on the six z-scored features, compared
against Ward linkage, with bootstrap stability over 500 replicates and the
solution compared against the Chapter 1 and Chapter 2 clusterings by adjusted
Rand index.
→ `fig1_feature_correlation.png`, `fig2_pca_biplot_drivers.png`,
`fig3_cluster_stability_bootstrap.png`, `fig7_pca_scree_loadings.png`,
`ch4_ward_dendrogram.png`

**B pain regression, and the flip.** VAS regressed first on the Chapter 1
kinematic features alone, then on all six, to test whether adding activation and
morphology improves prediction. The "flip" reverses the question, regressing the
neck-loading principal component on subject-level variables — age, BMI, head and
neck circumference, VO₂max and Wingate power.
→ `fig4_pain_regression.png`, `fig5_flip_regression_null.png`,
`fig8_flip_per_feature.png`, `flip_per_feature_results.csv`

**C logistic supplementary.** Cluster membership predicted from subject-level
variables. Marked in the log as underpowered at roughly 11 events across 50
participants, and reported as supplementary only.

**D asymmetry.** Signed SCM activation asymmetry against signed structural
asymmetry (D1), and two pre-specified alternatives testing rotation and baseline
flexion against structure (D2).
→ `fig6_asymmetry_signal_vs_null.png`, `fig9_D1_scm_asymmetry.png`

**MANOVA.** A multivariate test of cluster against the two asymmetry measures,
included because the committee asked for it. The log records a caveat: activation
asymmetry is partly non-independent of the cluster it is being tested against.

## Related

- Sign conventions for the asymmetry measures —
  [`../../docs/METHOD_asymmetry.md`](../../docs/METHOD_asymmetry.md)
- The clustering method — [`../../docs/METHOD_clustering.md`](../../docs/METHOD_clustering.md)
- The two source clusterings — `../chapter1_pain/H9_clustering/` and
  `../chapter2_emg/clustering_scm/`
