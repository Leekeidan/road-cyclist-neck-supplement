# From raw signal to reported number

This document follows **one participant, one measure, one moment in time** all
the way through the analysis, naming the file that holds the value at every
stage. Nothing here requires reading code. Every number below can be opened and
checked in the files in `data/`.

The example: **the co-contraction of the left sternocleidomastoid and the left
middle trapezius (SCML–MTL) for participant 573, during the Wingate test.**

---

## The nine stages at a glance

| # | Stage | File | Value at *t* = 1 s |
|---|---|---|---|
| 1 | Raw recording | *not deposited* — Delsys export, ~2000 Hz | — |
| 2 | MVC reference | `subject_573_EMG_1sec_results.xlsx`, sheet `MVC` | SCML 0.510966 mV · MTL 0.145243 mV |
| 3 | Filtered, normalised, binned | same file, sheet `WanT` | SCML **7.081318 %MVC** · MTL **5.233028 %MVC** |
| 4 | Co-contraction | `CoContraction_Results/WanT_co_contraction.xlsx` | **0.849908** |
| 5 | All measures merged | `subject_573_CCI_IMF_EMG_Angles_merged.xlsx` | 0.849908, alongside angles and IMF |
| 6 | Stacked across participants | `data/chapter2_emg/CCI/CCI_SCML_MTL_all_CCI_for_R.csv` | 0.849908, row `subject_573 / WanT / 0.999794` |
| 7 | Participant summary | same file, averaged | mean **0.3962** over 47 populated bins |
| 8 | Cohort summary | same file, all participants | mean **0.5586**, SD 0.1931, *n* = 59 |
| 9 | Reported statistic | the Chapter 2 co-contraction analysis | repeated-measures ANOVA across the five phases |

Stages 2–7 are all inside `data/per_subject/subject_573/`.

---

## Stage by stage

### 1. The raw recording

Eight surface EMG channels sampled continuously through the protocol, exported
from the Delsys system as CSV. These files are roughly 42 MB per participant and
are **not deposited here** — they are unprocessed voltage and require the MATLAB
extraction layer to become anything interpretable.

### 2. The normalisation reference

Before the cycling protocol, each muscle performed a dedicated maximum
voluntary contraction trial. The peak filtered amplitude from that trial becomes
the denominator for everything afterwards. For participant 573:

| Muscle | MVC_max (mV) |
|---|---|
| MTL | 0.145243 |
| MTR | 0.397983 |
| **SCML** | **0.510966** |
| SCMR | 0.476242 |
| STL | 0.267155 |
| STR | 0.298790 |
| UTL | 0.351083 |
| UTR | 0.577132 |

This table is the `MVC` sheet of `subject_573_EMG_1sec_results.xlsx`.

### 3. Filtering, normalisation and binning

The raw signal is converted to millivolts, band-pass filtered (4th-order
Butterworth, 20–450 Hz), notch filtered at 50 Hz, divided by the MVC reference
and multiplied by 100, smoothed with a 0.5-second moving RMS, then averaged into
**one-second bins**. Full detail in [`METHOD_MVC.md`](METHOD_MVC.md).

The result is the `WanT` sheet — 47 rows, one per second, one column per muscle:

| `UTL_time_sec` | `SCML` | `MTL` |
|---|---|---|
| 0.999794 | 7.081318 | 5.233028 |
| 1.999588 | 12.178397 | 10.329893 |
| 2.999382 | 86.718991 | 39.482527 |
| 3.999970 | 120.149793 | 31.129884 |

Read that third row as: two seconds into the Wingate, the left SCM was firing at
87 % of its maximum voluntary contraction.

### 4. Co-contraction — the arithmetic, in full

The co-contraction index asks how *evenly* the two muscles are sharing the work:

```
CCI = 2 × min(m₁, m₂) / (m₁ + m₂)
```

At *t* = 1 s, with SCML = 7.081318 and MTL = 5.233028:

```
min(7.081318, 5.233028)  =  5.233028
2 × 5.233028             =  10.466056
7.081318 + 5.233028      =  12.314346
10.466056 / 12.314346    =  0.849908
```

Open `CoContraction_Results/WanT_co_contraction.xlsx` and the stored value in
column `CCI_SCML_MTL` at `time_sec` 0.999794 is **0.849908**. The same check at
*t* = 2 s gives `2 × 10.329893 / 22.508290 = 0.917875`, which is the stored
value exactly.

A value of 0.85 means the two muscles were close to equally active. A value near
0 would mean one was doing nearly all the work. See
[`METHOD_CCI.md`](METHOD_CCI.md).

### 5. Merging the measures

`subject_573_CCI_IMF_EMG_Angles_merged.xlsx` places co-contraction, median
frequency, muscle activation and cervical angles side by side on the same
one-second grid, so that a given second can be read across all measures at once.

### 6. Stacking across participants

`data/chapter2_emg/CCI/CCI_SCML_MTL_all_CCI_for_R.csv` is the same value in a long table
covering everyone — 18,000 rows of `subject, muscle_pair, phase, sec, value`:

```
subject_573,CCI_SCML_MTL,WanT,0.999794,0.849908
```

Identical to stage 4. Nothing is transformed here; the data is only reshaped.

### 7. Summarising the participant

Averaging participant 573's Wingate bins gives a mean co-contraction of
**0.3962**. The phase is padded to 60 bins, of which **47 hold data** — the
Wingate window brackets the 30-second effort. 

### 8. Summarising the cohort

Across all participants, SCML–MTL co-contraction during the Wingate averages
**0.5586** (SD 0.1931, *n* = 59 # one participant has no Wingate data).
Participant 573, at 0.3962, sits 47th of 59: this rider distributed the load
between the two muscles *less* evenly than most.

### 9. The reported statistic

The Chapter 2 analysis takes the twelve long tables and runs a one-way
repeated-measures ANOVA per muscle pair, testing whether co-contraction differs
across the five exercise phases, with Greenhouse–Geisser correction where
sphericity is violated and Bonferroni-corrected Wilcoxon post-hoc tests. Those
outputs are the numbers in Chapter 2.

---

## Doing this for any other measure

The shape is always the same. Every measure begins in the same filtered,
normalised, one-second binned EMG or angle series, and every one has a METHOD
document giving its formula, units, sign convention and exclusions:

| Measure | Per-participant file | Group file | Method |
|---|---|---|---|
| Muscle activation | `subject_XXX_EMG_1sec_results.xlsx` | `data/files_help/master_all_subjects_all_phases.csv` | [`METHOD_MVC.md`](METHOD_MVC.md) |
| Median frequency / fatigue | `IMF.xlsx`, `subject_XXX_IMF_detailed.xlsx` | `data/chapter2_emg/IMF/IMF_LongFormat_*.csv` | [`METHOD_IMF.md`](METHOD_IMF.md) |
| Co-contraction | `CoContraction_Results/` | `data/chapter2_emg/CCI/` | [`METHOD_CCI.md`](METHOD_CCI.md) |
| Cervical angles | `subject_XXX_binogram_1sec_results.xlsx` | `data/files_help/master_all_subjects_all_phases.csv` | — |
| Bilateral asymmetry | derived from the master table | — | [`METHOD_asymmetry.md`](METHOD_asymmetry.md) |
| Activation phenotypes | — | `data/chapter2_emg/clustering_scm/` | [`METHOD_clustering.md`](METHOD_clustering.md) |
| Method reliability (ICC) | — | not in this repository — see the method-validation study, Keidan et al. (2025) | — |

Pick a participant, open the per-participant file, and the same nine stages
apply.
