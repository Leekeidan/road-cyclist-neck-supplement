# H8 — SCM activation clusters and pain

Tests whether the SCM activation groups identified by cluster analysis in
Chapter 2 relate to baseline (at-home) neck pain. A cross-chapter test: cluster
membership from Chapter 2 against the pain measure from Chapter 1.

## What is in this folder

`scm_cluster_and_pain.xlsx` — the joined table this hypothesis is built on. One
row per participant: `cluster` (the Chapter 2 SCM group), `pain_boolian`, `vas`,
and a `PC1 score` column that is empty. 65 rows; 60 have a cluster, 64 have a
VAS.

## Provenance caution

**The analysis output is not deposited.** No file under `../../chapter2_emg/`
records a cluster-against-pain test; this folder holds the input, not the
result.

**The deposited files do not reproduce the statistic reported in the chapter.**
Recomputing the correlation from the table above, and from the current Chapter 2
clustering solution joined to `pain.xlsx`, gives four different values, none of
which matches the reported one — though all four support the same conclusion.
This file's `cluster` column agrees with the current Chapter 2 solution for 55
of 59 participants, so it is not the same partition.

> **[UNCLEAR — needs author confirmation]** Which cluster solution and which
> pain variable produced the reported figures.

## Related

- Chapter 2 clustering solution — `../../chapter2_emg/clustering_scm/`
- Chapter 1 kinematic phenotypes, a different clustering on different
  features — `../H9_clustering/`
- Method — [`../../../docs/METHOD_clustering.md`](../../../docs/METHOD_clustering.md)
