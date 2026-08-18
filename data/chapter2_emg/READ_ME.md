# Chapter 2 — neuromuscular activation, fatigue and co-contraction

Results are filed by hypothesis, matching the numbering used in the chapter.
Three hypotheses, one per measure.

| Folder | Hypothesis | Measure |
|---|---|---|
| `H1_median_frequency/` | Does instantaneous median frequency change across exercise phases — over time and as an overall rate of decline — and do those changes differ between groups defined by EMG-based activation clustering, subjective RPE, or fitness? | IMF (Hz) |
| `H2_activation_amplitude/` | Do activation patterns differ across phases and within phases over time, and do they differ between groups defined by activation clustering, RPE, or exercise group? | %MVC |
| `H3_cocontraction/` | Does co-contraction differ across phases and over time, and do those differences vary by aerobic fitness or subjective exertion? | CCI |

## Inside each

Each hypothesis folder holds the measure's own per-phase output alongside the
group comparisons run on it.

**`H1_median_frequency/`**

| Subfolder | Contents |
|---|---|
| `imf_series_and_slopes/` | The median-frequency series and slopes per participant, phase and muscle, plus the phase-level ANOVA outputs |
| `slope_by_rpe_group/` | Slope compared across RPE groups |
| `by_phase_and_cluster/` | Phase × activation-cluster comparisons |
| `over_time_vo2max/`, `over_time_wingate/` | Change within those phases over time, with Kendall's W computed separately for the high- and low-fitness groups |

**`H2_activation_amplitude/`**

| Subfolder | Contents |
|---|---|
| `mvc_by_phase/` | %MVC per participant, phase and muscle, with the phase-level statistics |
| `by_rpe_group/` | Activation compared across RPE groups |
| `by_scm_cluster/` | Activation compared across the SCM activation clusters |
| `vo2max_and_wingate_two_way/` | Two-way models over the VO₂max and Wingate phases |
| `vo2max_and_wingate_correlations/` | Correlations over the same two phases |

**`H3_cocontraction/`**

| Subfolder | Contents |
|---|---|
| `cci_by_phase/` | The twelve muscle-pair co-contraction tables and the phase-level statistics |
| `vo2max_and_wingate/` | Co-contraction across those two phases |
| `by_scm_cluster/`, `by_rpe_group/`, `by_warmup_difficulty/` | The three group comparisons |

## Two folders outside the numbering

**`clustering_scm/`** — the SCM activation clustering itself. It produces the grouping variable that all three hypotheses test against. Holds the feature tables, the cluster assignments at k = 2 and k = 3,
the validity indices, the standardisation parameters, and the anthropometric and
variability follow-ups.
Two subfolders extend it: `Variability_Analysis/` reports within-phase
variability per participant and by group, and `Residual_Fatigue_Analysis/`
compares the clusters on their warm-up-to-cool-down change.

**`warmup_vs_cooldown/`** — every dependent variable here is a change score,
cool-down minus warm-up (`Δ%MVC` and `ΔCCI`), asking whether the neck ends the
session in a different state from where it started and whether that differs by
group. Because it tests **both** %MVC and CCI, against every grouping, it spans
H2 and H3 rather than belonging to either.

## Related

- How each measure is computed — the formula, units, filter settings, windows
  and exclusions — see [`../../docs/METHOD_IMF.md`](../../docs/METHOD_IMF.md),
  [`METHOD_MVC.md`](../../docs/METHOD_MVC.md) and
  [`METHOD_CCI.md`](../../docs/METHOD_CCI.md).
- The clustering method — [`METHOD_clustering.md`](../../docs/METHOD_clustering.md).
- One participant followed from raw signal to a reported number —
  [`../../docs/PIPELINE.md`](../../docs/PIPELINE.md).
- The SCM clusters tested against pain is a Chapter 1 hypothesis —
  `../chapter1_pain/H8_scm_activation/`.
