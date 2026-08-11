# CCI — co-contraction index

## 1. What it is

A measure of how evenly two muscles are active at the same instant, computed
for twelve muscle pairs across the cervical region. It answers "are these two
muscles sharing the load, or is one dominating?" — independently of how hard
either is working in absolute terms.

## 2. Formula

```
CCI = 2 · min(m₁, m₂) / (m₁ + m₂)
```

| Term | Definition |
|---|---|
| `m₁`, `m₂` | Activation of the two muscles in the pair at the same one-second bin, in %MVC (see `METHOD_MVC.md`) |

Edge cases, exactly as implemented:

- if either input is `NaN` → result is `NaN`
- if `m₁ + m₂ = 0` → result is `0`

## 3. Sign conventions and units

- **Dimensionless, bounded 0 to 1.** 
- **0** = one muscle entirely dominant (the other is silent).
- **1** = both muscles equally active — maximal co-contraction.
- **The index is symmetric**: `CCI(m₁, m₂) = CCI(m₂, m₁)`. It carries **no
  directional information** — it cannot tell you *which* muscle dominates. For
  that, see `METHOD_asymmetry.md`.
- Observed range in these data: 0.03 to 1.00.

## 4. Pipeline from raw file to value

1. **Input** — the 1-second %MVC bins from
   `subject_XXX_EMG_1sec_results.xlsx` (see `METHOD_MVC.md` for filter settings
   and binning).
2. **Phase windowing** —
   - `Warmup`, `CoolDown`, `WanT`: taken from their own recordings.
   - `Second_Threshold` and `VO2max`: extracted from the continuous ramp as
     **60-second** windows beginning at the per-participant onset times in
     `data/files_help/start_Times_Vo2max.csv`. A window is accepted if at least
     **40 seconds** of valid data are present (`MIN_ACCEPTABLE_DURATION = 40`).
3. **Pairwise computation** — the formula above, applied bin by bin to all
   twelve pairs.
4. **Merge** — per-participant results are stacked into the twelve long-format
   tables in `data/chapter2_emg/CCI/`, columns `subject, muscle_pair, phase, sec, value`.

**Muscle pairs (12).** Bilateral: `MTL–MTR`, `SCML–SCMR`, `STL–STR`,
`UTL–UTR`. Ipsilateral: `SCML–UTL`, `SCML–MTL`, `SCML–STL`, `SCMR–UTR`,
`SCMR–MTR`, `SCMR–STR`, `UTL–MTL`, `UTR–MTR`.

**Window lengths actually present in the data.** Each phase is padded to 60
one-second bins, with `NaN` in unpopulated bins. Populated bins per phase:
Warmup, Second Threshold, VO₂max and Cooldown are fully populated at 60;
**WanT carries a median of 47 populated bins (range 35–47)**, a window that
brackets the 30-second effort rather than matching it. One participant has no
WanT data, so the effective *n* at WanT is **59**.

> The Chapter 1 master tables define `WanT` differently — exactly 31 bins
> spanning 1.0–31.0 s. Analyses that place WanT co-contraction alongside WanT
> kinematics are comparing two different windows.

## 5. Exclusions and outlier handling

- **No outlier removal is applied to the statistics.** All 60 participants are
  retained.
- **1.5 × IQR filtering is display-only**, applied when drawing scatter plots
  so they read cleanly. It does not affect any reported test.
- Muscles failing the data-quality check (fewer valid seconds than the phase
  minimum) are logged to `MUSCLE_DATA_QUALITY_REPORT.xlsx` 

## 6. Where it is computed

Implemented in **Python**. The index itself is a four-line function applied bin
by bin; it appears in two places, computed identically — once when building the
Chapter 1 master table, and once in the per-participant Chapter 2 pipeline. A
third routine reuses it to detect sustained strong co-contraction, logging
episodes above 0.8 lasting at least 3 seconds.

The Chapter 2 pipeline then merges the per-participant results across
participants, reshapes them to the wide and long formats, and tests normality
with Shapiro–Wilk.

**Reported statistics:** a one-way repeated-measures ANOVA per muscle pair
(`pingouin`), testing the effect of phase, with Greenhouse–Geisser correction
where sphericity is violated and Bonferroni-corrected Wilcoxon signed-rank
post-hoc tests. An earlier **R** implementation used the Friedman test with
Wilcoxon post-hoc and linear mixed-effects models; the two agree on all 12/12
significance decisions.

