# Understanding the Road Cyclist's Neck: Data and Methods Supplement

[![DOI](https://zenodo.org/badge/1330748279.svg)](https://doi.org/10.5281/zenodo.21993258)

This repository is the data and methods supplement to the doctoral dissertation
called *Understanding the Road Cyclist's Neck* (by ph. candidate Lee Keidan). It contains the data
behind the reported results, at every stage from individual participant output per phase to the
group tables, together with documentation of how each measure was computed. This enables
the reader to follow the entire pipeline from raw signal to a reported number. 

Sixty trained male cyclists completed a cycling laboratory protocol including warm-up, second
ventilatory threshold, VO₂max ramp, 30-second Wingate, cooldown. Cyclists were connected to an
eight-channel surface EMG of the cervical musculature, inertial measurement of
cervical kinematics in three planes, and B-mode ultrasound of cervical muscle
morphology was performed prior to cycling. The dissertation examines neck pain (Chapter 1), neuromuscular
activation and co-contraction (Chapter 2), muscle morphology (Chapter 3), and
the integration of the three (Chapter 4).

> **Scope.** This repository holds the participant-level derived data for
> **all 60 participants**; every reported number can be traced
> back to the per-participant values it was computed from. The underlying raw
> recordings (approximately 1.5 GB of unprocessed EMG and inertial signal) are
> not deposited here; see [Data availability](#data-availability).

---
**Where to find**

**If you want the definition of a particular form of measurement, read
[`docs/METHOD_*.md`](docs/METHOD_*.md)**. Each gives the formula with every term defined, the sign
convention and units, the processing chain, the exclusions actually applied.

**If you want to see how a reported number was produced**, read
**[`docs/PIPELINE.md`](docs/PIPELINE.md)**. It follows one participant, one
measure, one moment in time through all nine stages of the analysis — naming the
file and showing the actual value at each step, and working the arithmetic by
hand at the point where the measure is computed. It requires no programming.

**If you want the analysis code, it is not here; see
[Code availability](#code-availability).**

**If you want the subject raw data, it is also not here; see
[Code availability](#code-availability).**

---

## Repository map

| Path | Contents |
|------|----------|
| `docs/PIPELINE.md` | Raw signal to reported number, followed end-to-end for one participant. **This is the best place to start.** |
| `docs/METHOD_*.md` | One document per derived measure: %MVC, IMF, CCI, bilateral asymmetry, cluster assignment. |
| `data/per_subject/` | Per-participant-level output for all 60 participants including activation, angles, median frequency, co-contraction, merged tables. |
| `data/files_help/` | The master analysis tables and the lookups that feed them. |
| `data/chapter1_pain/` … `data/chapter4_integration/` | Results, one folder per chapter. |

---

## Environment

Python **3.14.2**.

Some analyses are in R (Shapiro–Wilk normality testing, Friedman and
linear-mixed-effects models, several figures). Python Packages used: `tidyverse`, `rstatix`,
`ggpubr`, `dplyr`, `readr`, `readxl`, `openxlsx`, `lme4`, `lmerTest`, `emmeans`,
`broom`, `car`, `ez`, `effsize`, `factoextra`, `cluster`, `ggplot2`, `ggsignif`,
`gridExtra`, `cowplot`, `grid`, `tidyr`.

The signal-extraction layer requires **MATLAB**.

---

## Order of analysis

The chain runs signal extraction, which produces a per-participant
results tree, and every chapter analysis consumes it. The steps below name what
each stage produces, so that any table in `data/` can be located in the chain.
Script names identify each step unambiguously.

### 0. Extraction (MATLAB): produces the per-participant results tree

Writes, per participant, a `subject_XXX_EMG_1sec_results.xlsx` (one sheet per
phase, plus an `MVC` sheet), a `subject_XXX_binogram_1sec_results.xlsx`
(cervical angles), an `IMF.xlsx`, and a `CoContraction_Results/` folder — all
of which are in `data/per_subject/`. Every later stage reads from this tree.

### 1. Chapter 1: pain, kinematics and activation

| Order | Script | Produces |
|---|---|---|
| 1 | `Part_1_CCI_IMF_EMG_Angles_merged_time.py` | Per subject: CCI + IMF + EMG %MVC merged on 1-second bins |
| 2 | `Part_2_extract_EMG_Angles_all_phases.py` | `subject_XXX_CCI_IMF_EMG_Angles_merged.xlsx` |
| 3 | `Part_3_stack_and_merge_master.py` | Stacks all participants and joins `Anthropometric_data.xlsx` and `pain.xlsx` |
| 4 | `Part_4_compute_CCI.py` | Adds the twelve co-contraction columns, giving `master_all_subjects_all_phases_with_CCI.csv` — the analysis master table |
| 5 | `H1-angles/`, `H2-MVC/`, `H3-CCI/`, `H4-Asymmetry/`, `Ordinal_pain_Analysis_*.py`, `Acute_VAS_vs_*.py` | Per-hypothesis correlation and group-comparison tables |
| 6 | `kinematic_4var_staticref_clustering.py`, then `kinematic_pain_regression.py` | Two-phenotype k-means solution and the confirmatory permutation regression |

### 2. Chapter 2: activation, fatigue and co-contraction

Each measure is a numbered pipeline; run in order within each folder.

- **MVC**: `MVC_Step1_Merge_files.py` → `MVC_Step2_create_wide_and_long.py` → `MVC_Step3_Normality_Testing_ShapiroWilk.py` → `MVC_Analysis.py`
- **IMF**: `IMF_Step1_Extract_VO2max_SecondThreshold_Raw.py` → `IMF_Step2_IMF_calculation.py` → `IMF_Step3_Format_Converter.py` → `IMF_Step4_Slope_Analysis.py` → `IMF_Step5_Fatigue_Duration_MultiThreshold.py`
- **CCI**: `CCI_Step1_Merge.py` → `CCI_Step2_Format.py` → `CCI_Step3_Normality_Testing.py` → then either `CCI_Step4_Analysis_Friedman_and_lmer.R` (original analysis) or `CCI_LMM_Analysis.py` (the repeated-measures ANOVA rerun that is reported)
- **Cluster**: `cluster_analysis_5_SCM_VARS_PYTHON.py` for the SCM activation phenotypes

### 3. Chapter 3: Ultrasound morphology

`build_merged_files.py` **must run first** — it reads the raw measurement
workbook and writes `us_measurments.xlsx`, `us_merged.xlsx`,
`us_merged_n35.xlsx` and `us_descriptives_n35.xlsx`. Then:
`morphology_pain_analysis.py` (normality, bilateral Wilcoxon, pain groups) →
the correlation scripts (`angle_comparison.py`, `EMG_MVC_comparison.py`,
`IMF_comparison.py`, `anthropometric_vs_size_correlations.py`) →
`fitness_cluster_US.py`, `results_figures.py`, `scm_cluster_comparison.py`.

### 4. Chapter 4: Integration

`build_integrated_dataset.py` → `ch4_integrated_analysis.py` →
`ch4_pca_and_flip.py` → `ch4_make_figures.py` → `fig9_D1_scm_asymmetry.py`

---

## Data

Two shared inputs sit at the root of `data/`; everything else is organized by
chapter.

| Folder | Contents |
|---|---|
| `data/per_subject/` | Participant-level output for all 60 participants — the input to every chapter |
| `data/files_help/` | Master analysis tables and the lookups that feed them |
| `data/chapter1_pain/` | Kinematics, pain and the kinematic phenotypes |
| `data/chapter2_emg/` | Activation, fatigue, co-contraction and the SCM phenotypes |
| `data/chapter3_morphology/` | Ultrasound morphology |
| `data/chapter4_integration/` | Integration of the three |

### Reading the data:

**`WanT` covers a different window in the two sets of tables, by design.** The
Chapter 1 master tables isolate the 30-second effort itself, exactly 31
one-second bins spanning 1–31 seconds. The Chapter 2 co-contraction tables
deliberately capture the tail of the EMG response beyond the effort running more than the 31 sec WanT trial.
A Wingate number should then be in light of which window it comes from.

**One participant is absent from the Wingate analyses.** Participant 597's
Wingate recording is problematic; that participant therefore falls out of every
Wingate co-contraction result (*n* = 59 rather than 60) and out of the
Chapter 2 activation clustering, which reports 59 complete cases.

**Bilateral asymmetry is signed differently in Chapter 3.** Activation
asymmetry in Chapters 1 and 4 is left-minus-right, so positive means left
dominant. The Chapter 3 morphology table is right-minus-left and labels its
columns `Mean Diff (R-L)`, while the figure from the same analysis uses
left-minus-right. See [`docs/METHOD_asymmetry.md`](docs/METHOD_asymmetry.md).


### `data/per_subject/`: participant-level output, all 60 participants
One folder per participant (`subject_541` … `subject_632`), 91 MB across 1,398
files. This is the layer between the raw recordings and the group tables: any
number reported in the dissertation can be followed back to the participant it
came from. Each folder contains, in pipeline order:

| File | Stage |
|---|---|
| `Vo2_Second_Threshold_Raw_subject_XXX.csv`, `Vo2andSecondThreshold_subject_XXX.csv` | Extracted ramp windows |
| `subject_XXX_EMG_1sec_results.xlsx` | %MVC, one sheet per phase plus `MVC` — the input to almost everything |
| `subject_XXX_EMG_10sec_results.xlsx` | The same at 10-second resolution |
| `subject_XXX_binogram_1sec_results.xlsx`, `..._10sec_...`, `..._bins_results.xlsx` | Cervical angles per phase |
| `IMF.xlsx`, `IMF_summary.xlsx`, `subject_XXX_IMF_detailed.xlsx`, `Vo2_Second_Threshold_Step1_IMF.csv`, `..._Hz_subject_XXX.csv` | Median-frequency series and slopes |
| `CoContraction_Results/` | Per-phase co-contraction (`{Phase}_co_contraction.xlsx`) plus diagnostic plots |
| `subject_XXX_CCI_IMF_EMG_Angles_merged.xlsx` | All measures merged on one-second bins — the per-subject table that `Part_3` stacks into the master |
| `subject_XXX_all_phases_extracted.csv`, `..._extracted_combined.csv` | Flattened extracts |

Two large raw files per participant (`subject_XXX_raw_angles_results.xlsx` and
`Vo2_Second_Threshold_Raw.csv`, together roughly 42 MB per participant and
1.5 GB across the cohort) are **excluded**. They are unprocessed signals, require
MATLAB and the extraction layer to be interpretable, and are available from the
author on request.


### `data/files_help/`: the master table and its lookups

Eight files: one master analysis table, and seven per-participant lookups that
feed it or the chapter analyses.

**The master table.** Every Chapter 1 analysis reads from this.

| File | Shape | Contents |
|---|---|---|
| `master_all_subjects_all_phases_with_CCI.csv` | 15,435 × 39 | 60 participants × 5 phases on one-second bins. `subject`, `Period`, `Time (s)`; eight EMG channels as %MVC (`MTL MTR SCML SCMR STL STR UTL UTR`); three cervical angles (`angle sagital`, `angle frontal`, `angle horzintal`); pain (`vas`, `pain_bulian`); anthropometrics and fitness (`Height_cm`, `Weight_kg`, `BMI`, `Head_Circumference_cm`, `Neck_Circumference_cm`, `Fat_Percent`, `Years_Cycling`, `Other_Sport`, `Vo2max`, `WanT`, `missing_data`); and the twelve `CCI_{PAIR}` co-contraction columns |

Rows per phase: Warmup 3,629 · Second\_threshold 3,429 · VO2max 2,890 ·
WanT 1,829 · Cooldown 3,658. Coverage is complete: every one of the 60
participants has an extracted file, pain data and anthropometrics.

**The lookups**, each keyed by participant.

| File | Shape | What it holds, and what uses it |
|---|---|---|
| `Anthropometric_data.xlsx` | 64 × 15 | Height, weight, BMI, head and neck circumference, body fat, years cycling, VO₂max, Wingate and VAS. Joined into the master table; also feeds the Chapter 4 integration |
| `pain.xlsx` | 63 × 6 | The pain source: `pain_bulian` and `vas` per participant, joined into the master table and read across the Chapter 1 and Chapter 3 pain comparisons. Also carries the Euler segment choice per plane, duplicating `segment.xlsx` — the two agree for all 60 shared participants |
| `VAS_phases.csv` | 66 × 7 | VAS at six timepoints through the protocol — the source of the acute-pain change scores |
| `Vo2max_results.xlsx` | 64 × 7 | VO₂max, Wingate peak power and training-volume variables; drives the fitness clustering |
| `start_Times_Vo2max.csv` | 60 × 3 | Per-participant onset of the VO₂max and second-threshold windows within the continuous ramp. Without it those two phases cannot be extracted at all |
| `segment.xlsx` | 60 × 9 | The Euler decomposition sequence chosen per participant per plane, plus coverage flags (`outliers`, `other`, `US`, `basic info`, `merged`) — the record of which rotation convention each participant's angles were computed with |
| `RPE.csv` | 61 × 6 | Rating of perceived exertion per phase. Reported in the thesis; no script in the deposited pipeline reads it |

> Column spellings are preserved exactly as the analysis code expects them (even if they are misspelled),
> Column spellings are preserved exactly as the analysis code expects them (even if they are misspelled),
> including `angle horzintal` and `Sagital`.


### `data/chapter1_pain/`: kinematics, pain and phenotypes

| Subfolder | Contents |
|---|---|
| `clustering_4var_staticref/` | The reported H9 solution — the static-referenced four-variable k-means clustering, its cluster assignments, validity indices and figures |
| `pain_analyses/` | Outputs of H1–H8: correlations between pain and angles, activation, co-contraction, and asymmetry; group comparisons; acute-pain change scores |
| `angle_emg_correlation/` | H7 angle–EMG coupling |
| `angle_emg_correlation_want_thirds/` | The same, split across thirds of the Wingate |
| `angle_emg_correlation_full_vo2max/` | The same across the full VO₂max ramp |

Superseded clustering variants (the non-static-referenced four-variable and the
five-variable solutions) are not included; the reported analysis is the
static-referenced one.

### `data/chapter2_emg/`: activation, fatigue and co-contraction

| Subfolder | Contents |
|---|---|
| `CCI/` | The twelve long-format co-contraction tables that the reported statistics consume, plus quality reports |
| `hypotheses/` | Results of H1–H6, one folder per hypothesis: 147 figures, 90 tables, 10 summary notes |
| `clustering_scm/` | The five-variable SCM activation phenotype solution — feature tables, assignments at k = 2 and k = 3, validity metrics, standardisation parameters, PC1 scores and figures |
| `MVC/`, `IMF/` | Step-pipeline outputs for activation amplitude and median frequency |

Each `CCI_{PAIR}_all_CCI_for_R.csv` holds 18,000 rows — 60 participants × 5
phases × 60 one-second bins, padded with `NaN` in unpopulated bins — with
columns `subject`, `muscle_pair`, `phase`, `sec`, `value` (CCI, 0–1).

### `data/chapter3_morphology/` — ultrasound morphology

The merged measurement workbooks (`us_merged.xlsx`, `us_merged_n35.xlsx`,
`us_descriptives_n35.xlsx`), the bilateral, pain-group, fitness-cluster and
correlation result tables, and the chapter figures.

### `data/chapter4_integration/` — integration

Outputs of the integrated analysis: the assembled multi-domain dataset, the
principal-component analysis, the integrated clustering, and the nine figures,
including the D1 SCM structure-by-activation asymmetry result.



---

## Code availability

The analyses were implemented in Python, R, and MATLAB. The code is not
deposited in this repository. Every derived measure is instead documented in
`docs/`, which gives for each one the formula with all terms defined, the sign
convention and units, the full processing chain including filter settings,
window lengths and normalization reference, the exclusions actually applied,
and the known limitations. `docs/PIPELINE.md` follows a single participant
through the entire chain, showing the value held in each file at each stage and
working the arithmetic by hand where the measure is computed.

The analysis code is available from the author on reasonable request.

---

## Data availability

The study was approved by the Ethics Committee of Tel Aviv University
(approval no. 0004184-3), under the protocol **Cervical neck strain and
injuries in cyclists - risk factors and prevention**.

The participant-level derived data supporting this
dissertation are openly available in this repository, comprising per-phase
muscle activation (%MVC), cervical kinematics, median-frequency series,
co-contraction indices and merged per-participant tables for all 60
participants, together with the group-level tables from which the reported
statistics were computed. Participants are identified by three-digit code only.

The underlying raw electromyographic and inertial recordings (approximately
1.5 GB) are not deposited publicly. They are unprocessed signals requiring the
MATLAB extraction layer to be interpretable, and they fall outside the scope of
open deposit. Raw recordings may be
made available to qualified researchers on reasonable request,
subject to approval by the Ethics Committee of Tel Aviv University.

---

## Citation

Cite the concept DOI, which always resolves to the current version:

> Keidan, L. (2026). *Understanding the Road Cyclist's Neck: Data and Methods
> Supplement* [Data set]. Zenodo. https://doi.org/10.5281/zenodo.21993258

```bibtex
@dataset{keidan_2026_cyclist_neck_supplement,
  author    = {Keidan, Lee},
  title     = {Understanding the Road Cyclist's Neck:
               Data and Methods Supplement},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.21993258},
  url       = {https://doi.org/10.5281/zenodo.21993258}
}
```

Version 1.0.0 specifically is `10.5281/zenodo.21993259`.
