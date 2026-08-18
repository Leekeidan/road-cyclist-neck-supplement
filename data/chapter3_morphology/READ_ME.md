# Chapter 3 — cervical muscle morphology

Results are filed by analysis type, following the chapter's methods. Ultrasound
measurements of five muscles — trapezius, spinotransversalis, semispinalis,
multifidus and sternocleidomastoid — at up to three cervical levels (POP, C4,
C7), in two planes, on both sides.

| Folder | Analysis |
|---|---|
| `measurements/` | The measurement tables everything else is built from |
| `descriptives_and_normality/` | Mean, SD, minimum, maximum and range for each measurement with *n* ≥ 35, and the Shapiro–Wilk normality tests that determined non-parametric testing throughout |
| `bilateral_symmetry/` | Right-versus-left differences by Wilcoxon signed-rank, for all paired measurements |
| `group_comparisons/` | Muscle dimensions compared between groups by Mann–Whitney U |
| `fitness_clustering/` | The k-means fitness clustering, and muscle dimensions compared between the resulting clusters |
| `structure_function/` | Spearman correlations between muscle dimensions and the functional measures |

## The cohort — 59 participants

Ultrasound data exists for **59 cyclists**. Scans that could not be measured
were excluded, most often because of a file-extraction problem before
measurement: the scan had been saved as **AVI video rather than DICOM**, which
does not carry the calibration needed to take dimensions from the image.
Participants affected were excluded from the morphology analyses.

This leaves Chapter 3 with a slightly different cohort from Chapters 1 and 2,
which analyse the 60 cyclists who completed the instrumented cycling protocol:

| | Participants |
|---|---|
| EMG and inertial data (Chapters 1–2) | 60 |
| Ultrasound (Chapter 3) | 59 |

Three participants (613, 619, 620) have cycling data but no usable scan; two
(542, 576) have a scan but no cycling data — 542 was never processed through the
EMG pipeline, and 576 completed the questionnaire and scan but not the cycling
test.

Any analysis joining muscle dimensions to a functional measure — everything in
`structure_function/`, and the Chapter 4 integration — is therefore limited to
those **57** before any per-measurement missingness is taken into account. That
is why the correlation tables report *n* of 54, 55 or 49 rather than 57: the
`n ≥ 35` filter then reduces 112 measurements to the 76 with adequate coverage.

## measurements/

| File | Contents |
|---|---|
| `us_measurments.xlsx` | 60 sheets — one per participant (59) plus the blank measurement template `Sheet11` |
| `us_merged.xlsx` | All 112 measurements × 59 participants in one table |
| `us_merged_n35.xlsx` | The 76 measurements available for at least 35 participants — the table almost every analysis uses |

The source workbook carries 65 sheets: the template, the 59 cyclists, and five
non-cyclist participants (`P003`, `P004`, `P006`, `P007`, `P008`) who are
excluded at the merge step, since every analysis here is cyclist-only.

The `n ≥ 35` filter is what separates the two merged files; nothing else differs.

## group_comparisons/

Five groupings, all Mann–Whitney U:

| File | Groups compared |
|---|---|
| `pain_group_mannwhitney_results.xlsx` | High versus low habitual VAS, split at the mean (2.9) |
| `pain_during_testing_mannwhitney_results.xlsx` | Pain versus no pain during testing (any VAS > 0) |
| `Vo2max_US.xlsx` | High versus low VO₂max (mean 49.0 mL·kg⁻¹·min⁻¹) and high versus low Wingate peak power (mean 741.6 W), one sheet per muscle and level |
| `scm_cluster_mannwhitney.xlsx` | The two SCM EMG activation clusters from Chapter 2 |
| `scm_pc1_correlations.xlsx` | Muscle dimensions against the SCM cluster PC1 score — the continuous form of the same comparison |

Bonferroni correction is applied per muscle-level family for the fitness,
VO₂max, Wingate and SCM-cluster comparisons; for the two pain comparisons it is
applied across all 76 measurements.

## fitness_clustering/

k-means (k = 2) on six standardised fitness variables — VO₂max, Wingate peak
power, years of cycling, training days per week, ride duration and ride
distance — with cluster quality assessed over k = 2 to 5 and PCA used to
visualise the separation. `fitness_cluster_US.xlsx` holds both the cluster
assignments and the muscle-dimension comparison between clusters, one sheet per
muscle and level, plus a `Surviving Bonferroni` sheet.

## structure_function/

Spearman correlations between muscle dimensions and four functional domains,
one subfolder each:

| Subfolder | Correlated against |
|---|---|
| `emg/` | Muscle activation (%MVC) |
| `imf/` | Instantaneous median frequency |
| `cervical_angle/` | Sagittal cervical angle |
| `anthropometrics/` | Body measures — height, weight, BMI, head and neck circumference, body fat |

For the EMG, IMF and cervical-angle analyses, phase-level statistics (mean,
maximum, minimum, range and SD) were computed for each exercise phase.

`anthropometrics/` holds one scatter plot per correlation surviving Bonferroni
correction, so the plots present there correspond one-to-one with the rows of
`significant_bonferroni.xlsx`.

## Related

- How the ultrasound measures relate to the activation measures —
  [`../../docs/METHOD_MVC.md`](../../docs/METHOD_MVC.md) and
  [`METHOD_IMF.md`](../../docs/METHOD_IMF.md)
- Bilateral asymmetry sign conventions —
  [`METHOD_asymmetry.md`](../../docs/METHOD_asymmetry.md)
- The Chapter 2 SCM clusters used in `group_comparisons/` —
  `../chapter2_emg/clustering_scm/`
