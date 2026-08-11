# Understanding the Road Cyclist's Neck — Data and Methods Supplement

This repository is the data-and-methods supplement to the doctoral dissertation
*Understanding the Road Cyclist's Neck* (Lee Keidan). It contains the data
behind the reported results, at every stage from per-participant output to the
group tables, together with documentation of how each measure was computed — so
that a reader can follow how a raw signal became a reported number. Sixty trained male
cyclists completed an instrumented laboratory protocol (warm-up, second
ventilatory threshold, VO₂max ramp, 30-second Wingate, cool-down) with
eight-channel surface EMG of the cervical musculature, inertial measurement of
cervical kinematics in three planes, and B-mode ultrasound of cervical muscle
morphology. The dissertation examines neck pain (Chapter 1), neuromuscular
activation and co-contraction (Chapter 2), muscle morphology (Chapter 3), and
the integration of the three (Chapter 4).

> **Scope.** This repository holds the participant-level derived data for
> **all 60 participants** — every reported number can be traced
> back to the per-participant values it was computed from. The underlying raw
> recordings (approximately 1.5 GB of unprocessed EMG and inertial signal) are
> not deposited here; see [Data availability](#data-availability).

---

## Start here
**If you want the definition of a particular form of measurement**, read the relevant
`docs/METHOD_*.md`. Each gives the formula with every term defined, the sign
convention and units, the processing chain, the exclusions actually applied, and
the known limitations.

**If you want to see how a reported number was produced**, read
**[`docs/PIPELINE.md`](docs/PIPELINE.md)**. It follows one participant, one
measure, one moment in time through all nine stages of the analysis — naming the
file and showing the actual value at each step, and working the arithmetic by
hand at the point where the measure is computed. It requires no programming.

**If you want the analysis code**, it is not deposited here — see
[Code availability](#code-availability).

## Repository map

| Path | Contents |
|------|----------|
| `docs/PIPELINE.md` | Raw signal to reported number, followed end to end for one participant. **The best place to start.** |
| `docs/METHOD_*.md` | One document per derived measure: %MVC, IMF, CCI, bilateral asymmetry, cluster assignment. |
| `data/per_subject/` | Participant-level output for all 60 participants — activation, angles, median frequency, co-contraction, merged tables. |
| `data/files_help/` | The master analysis tables and the lookups that feed them. |
| `data/chapter1_pain/` … `data/chapter4_integration/` | Results, one folder per chapter. |
| `requirements.txt` | The Python environment the analyses were run in, pinned. |

---

## Environment

Python **3.14.2**. Pinned dependencies are in [`requirements.txt`](requirements.txt):

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Some analyses are in R (Shapiro–Wilk normality testing, Friedman and
linear-mixed-effects models, several figures). R version was not captured at run
time and is therefore not pinned. Packages used: `tidyverse`, `rstatix`,
`ggpubr`, `dplyr`, `readr`, `readxl`, `openxlsx`, `lme4`, `lmerTest`, `emmeans`,
`broom`, `car`, `ez`, `effsize`, `factoextra`, `cluster`, `ggplot2`, `ggsignif`,
`gridExtra`, `cowplot`, `grid`, `tidyr`.

The signal-extraction layer requires **MATLAB**; it was not re-run for this
supplement.

---

## Order of analysis

The chain runs in one direction: signal extraction produces a per-participant
results tree, and every chapter analysis consumes it. The steps below name what
each stage produces, so that any table in `data/` can be located in the chain.
Script names are given to identify each step unambiguously; the scripts
themselves are not deposited here (see [Code availability](#code-availability)).

### 0. Extraction (MATLAB) — produces the per-participant results tree

Writes, per participant, a `subject_XXX_EMG_1sec_results.xlsx` (one sheet per
phase, plus an `MVC` sheet), a `subject_XXX_binogram_1sec_results.xlsx`
(cervical angles), an `IMF.xlsx`, and a `CoContraction_Results/` folder — all
of which are in `data/per_subject/`. Every later stage reads from this tree.

### 1. Chapter 1 — pain, kinematics and activation

| Order | Script | Produces |
|---|---|---|
| 1 | `Part_1_CCI_IMF_EMG_Angles_merged_time.py` | Per subject: CCI + IMF + EMG %MVC merged on 1-second bins |
| 2 | `Part_2_extract_EMG_Angles_all_phases.py` | `subject_XXX_CCI_IMF_EMG_Angles_merged.xlsx` |
| 3 | `Part_3_stack_and_merge_master.py` | `master_all_subjects_all_phases.csv` (joins `Anthropometric_data.xlsx` and `pain.xlsx`) |
| 4 | `Part_4_compute_CCI.py` | `master_all_subjects_all_phases_with_CCI.csv` — the analysis master table |
| 5 | `H1-angles/`, `H2-MVC/`, `H3-CCI/`, `H4-Asymmetry/`, `Ordinal_pain_Analysis_*.py`, `Acute_VAS_vs_*.py` | Per-hypothesis correlation and group-comparison tables |
| 6 | `kinematic_4var_staticref_clustering.py`, then `kinematic_pain_regression.py` | Two-phenotype k-means solution and the confirmatory permutation regression |

### 2. Chapter 2 — activation, fatigue and co-contraction

Each measure is a numbered pipeline; run in order within each folder.

- **MVC** — `MVC_Step1_Merge_files.py` → `MVC_Step2_create_wide_and_long.py` → `MVC_Step3_Normality_Testing_ShapiroWilk.py` → `MVC_Analysis.py`
- **IMF** — `IMF_Step1_Extract_VO2max_SecondThreshold_Raw.py` → `IMF_Step2_IMF_calculation.py` → `IMF_Step3_Format_Converter.py` → `IMF_Step4_Slope_Analysis.py` → `IMF_Step5_Fatigue_Duration_MultiThreshold.py`
- **CCI** — `CCI_Step1_Merge.py` → `CCI_Step2_Format.py` → `CCI_Step3_Normality_Testing.py` → then either `CCI_Step4_Analysis_Friedman_and_lmer.R` (original analysis) or `CCI_LMM_Analysis.py` (the repeated-measures ANOVA rerun that is reported)
- **Cluster** — `cluster_analysis_5_SCM_VARS_PYTHON.py` for the SCM activation phenotypes

### 3. Chapter 3 — ultrasound morphology

`build_merged_files.py` **must run first** — it reads the raw measurement
workbook and writes `us_measurments.xlsx`, `us_merged.xlsx`,
`us_merged_n35.xlsx` and `us_descriptives_n35.xlsx`. Then:
`morphology_pain_analysis.py` (normality, bilateral Wilcoxon, pain groups) →
the correlation scripts (`angle_comparison.py`, `EMG_MVC_comparison.py`,
`IMF_comparison.py`, `anthropometric_vs_size_correlations.py`) →
`fitness_cluster_US.py`, `results_figures.py`, `scm_cluster_comparison.py`.

### 4. Chapter 4 — integration

`build_integrated_dataset.py` → `ch4_integrated_analysis.py` →
`ch4_pca_and_flip.py` → `ch4_make_figures.py` → `fig9_D1_scm_asymmetry.py`

---

## Data

Two shared inputs sit at the root of `data/`; everything else is organised by
chapter.

| Folder | Contents |
|---|---|
| `data/per_subject/` | Participant-level output for all 60 participants — the input to every chapter |
| `data/files_help/` | Master analysis tables and the lookups that feed them |
| `data/chapter1_pain/` | Kinematics, pain and the kinematic phenotypes |
| `data/chapter2_emg/` | Activation, fatigue, co-contraction and the SCM phenotypes |
| `data/chapter3_morphology/` | Ultrasound morphology |
| `data/chapter4_integration/` | Integration of the three |

### Reading the data — two things to know

**`WanT` means a different window in the two sets of tables.** In the Chapter 1
master tables it is exactly 31 one-second bins spanning 1.0–31.0 s — the
30-second effort itself. In the Chapter 2 co-contraction tables it is a median
of 47 populated bins (range 35–47), a window that brackets the effort with
lead-in and lead-out. Comparing Wingate co-contraction against Wingate
kinematics compares two different windows.

**One participant is absent from the Wingate analyses.** Participant 597's
Wingate recording is mislabelled — the sheet and file carry another
participant's identifier — so the analysis scripts, which match on the exact
name `WanT`, never found it. That participant therefore falls out of every
Wingate co-contraction result (*n* = 59 rather than 60) and out of the
Chapter 2 activation clustering, which reports 59 complete cases. The data
itself is intact; the analyses were not re-run to include it.

**Bilateral asymmetry is signed differently in Chapter 3.** Activation
asymmetry in Chapters 1 and 4 is left-minus-right, so positive means left
dominant. The Chapter 3 morphology table is right-minus-left and labels its
columns `Mean Diff (R-L)`, while the figure from the same analysis uses
left-minus-right. Check the column header or axis label before interpreting a
sign. See [`docs/METHOD_asymmetry.md`](docs/METHOD_asymmetry.md).


### `data/per_subject/` — participant-level output, all 60 participants

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
1.5 GB across the cohort) are **excluded**. They are unprocessed signal, require
MATLAB and the extraction layer to be interpretable, and are available from the
author on request.


### `data/files_help/` — master analysis tables and lookups

The Chapter 1 analyses all read from one of two master tables.

| File | Rows | Cols | Contents |
|---|---|---|---|
| `master_all_subjects_all_phases.csv` | 15,435 | 27 | 60 participants × 5 phases, one-second bins. `subject`, `Period`, `Time (s)`, eight EMG channels as %MVC (`MTL MTR SCML SCMR STL STR UTL UTR`), three cervical angles (`angle sagital`, `angle frontal`, `angle horzintal`), pain (`vas`, `pain_bulian`), and anthropometrics/fitness (`Height_cm`, `Weight_kg`, `BMI`, `Head_Circumference_cm`, `Neck_Circumference_cm`, `Fat_Percent`, `Years_Cycling`, `Other_Sport`, `Vo2max`, `WanT`, `missing_data`) |
| `master_all_subjects_all_phases_with_CCI.csv` | 15,435 | 39 | The same table plus the twelve `CCI_{PAIR}` columns written by `Part_4_compute_CCI.py` |

Rows per phase: Warmup 3,629 · Second\_threshold 3,429 · VO2max 2,890 ·
WanT 1,829 · Cooldown 3,658.

Supporting lookups: `Anthropometric_data.xlsx` (64 rows × 15), `pain.xlsx`,
`VAS_phases.csv` (VAS at six timepoints), `RPE.csv`,
`Vo2max_results.xlsx`, `combined_data_boolian_vas.xlsx` (pain grouping
and PC1 score), `start_Times_Vo2max.csv` (per-participant onset of the VO₂max
and second-threshold windows), `segment.xlsx` (per-participant Euler segment
choice per plane), `extraction_all_phases_summary*.csv`,
`vo2max_data_coverage_issues.csv`, and `master_dataset_summary.txt`.

`READ_ME.txt` is the author's original raw-data intake protocol (Delsys export
conventions, participant renaming, column layout, MATLAB and IMF run order).
Its file paths refer to a retired machine.

> Column spellings are preserved exactly as the analysis code expects them,
> including `angle horzintal` and `Sagital`.


### `data/chapter1_pain/` — kinematics, pain and phenotypes

| Subfolder | Contents |
|---|---|
| `clustering_4var_staticref/` | The reported H9 solution — the static-referenced four-variable k-means clustering, its cluster assignments, validity indices and figures |
| `pain_analyses/` | Outputs of H1–H8: correlations between pain and angles, activation, co-contraction and asymmetry; group comparisons; acute-pain change scores |
| `angle_emg_correlation/` | H7 angle–EMG coupling |
| `angle_emg_correlation_want_thirds/` | The same, split across thirds of the Wingate |
| `angle_emg_correlation_full_vo2max/` | The same across the full VO₂max ramp |

Superseded clustering variants (the non-static-referenced four-variable and the
five-variable solutions) are not included; the reported analysis is the
static-referenced one.

### `data/chapter2_emg/` — activation, fatigue and co-contraction

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

The analyses were implemented in Python, R and MATLAB. The code is not
deposited in this repository. Every derived measure is instead documented in
`docs/`, which gives for each one the formula with all terms defined, the sign
convention and units, the full processing chain including filter settings,
window lengths and normalisation reference, the exclusions actually applied,
and the known limitations. `docs/PIPELINE.md` follows a single participant
through the entire chain, showing the value held in each file at each stage and
working the arithmetic by hand where the measure is computed.

The analysis code is available from the author on reasonable request.

---

## Data availability

The study was approved by the Ethics Committee of Tel Aviv University
(approval no. 0004184-3), under the protocol *Cervical neck strain and
injuries in cyclists — risk factors and prevention*.

The participant-level derived data supporting this
dissertation are openly available in this repository, comprising per-phase
muscle activation (%MVC), cervical kinematics, median-frequency series,
co-contraction indices and merged per-participant tables for all 60
participants, together with the group-level tables from which the reported
statistics were computed. Participants are identified by three-digit code only.

The underlying raw electromyographic and inertial recordings (approximately
1.5 GB) are not deposited publicly. They are unprocessed signal requiring the
MATLAB extraction layer to be interpretable, and they fall outside the scope of
open deposit under the consent obtained. De-identified raw recordings may be
made available to qualified researchers on reasonable request to the author,
subject to approval by the Ethics Committee of Tel Aviv University.

---

## Citation

<!-- TODO — supply before minting the DOI:
     institution and degree title, submission year, ORCID,
     supervisor listing, and the Zenodo DOI once reserved. -->

```bibtex
@phdthesis{keidan_cyclist_neck,
  author  = {Keidan, Lee},
  title   = {Understanding the Road Cyclist's Neck},
  school  = {Tel Aviv University},
  year    = {TODO — submission year},
  type    = {TODO — degree title}
}

@software{keidan_cyclist_neck_supplement,
  author  = {Keidan, Lee},
  title   = {Understanding the Road Cyclist's Neck:
             Data and Methods Supplement},
  year    = {TODO},
  doi     = {TODO — Zenodo DOI},
  url     = {TODO — repository URL}
}
```

