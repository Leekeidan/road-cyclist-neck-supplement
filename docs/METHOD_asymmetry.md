# Bilateral asymmetry index

## 1. What it is

A measure of how unevenly a bilateral muscle pair is loaded — how much more the
left side is working, or is larger, than the right. Unlike the co-contraction
index it is **directional**: its sign tells you which side dominates.

---

## 2. THE SIGN CONVENTION — read this first

> ## **Positive = LEFT dominant.**
> **Every asymmetry measure in this dissertation is computed as
> LEFT minus RIGHT

A positive value means the left side is more active (EMG) or larger
(ultrasound) than the right. A negative value means the right side dominates.
Zero means the sides are equal.

---

## 3. Formula

Three forms are used, all sharing the same sign convention.

**(a) Normalised asymmetry index — the primary form**

```
AI = 100 × (L − R) / (L + R)
```

**(b) Signed difference**

```
D = L − R
```

**(c) Absolute (unsigned) asymmetry**

```
|D| = |L − R|
```

| Term | Definition |
|---|---|
| `L` | Left-side value — mean %MVC for that muscle in that phase, or the ultrasound dimension of the left muscle |
| `R` | Right-side value, same measure, same phase |

Form (c) discards direction and measures magnitude of imbalance only. It is
used where the question is "how asymmetric is this participant?" rather than
"which side dominates?"

## 4. Units

| Form | Units | Range |
|---|---|---|
| (a) Normalised index | **percent (%)** | −100 to +100 |
| (b) Signed difference, EMG | **%MVC** | unbounded |
| (b) Signed difference, ultrasound | **centimetres (cm)** | unbounded |
| (c) Absolute asymmetry | same as (b), always ≥ 0 | — |

Four bilateral pairs are indexed: `Asym_MT` (MTL, MTR), `Asym_SCM` (SCML,
SCMR), `Asym_ST` (STL, STR), `Asym_UT` (UTL, UTR).

## 5. Pipeline from raw file to value

**Activation asymmetry (Chapters 1, 2, 4)**

1. Start from the 1-second %MVC bins (see `METHOD_MVC.md`).
2. Average within participant × phase to a single mean %MVC per muscle.
3. Apply the formula to the left/right pair.
4. Pivot to one column per pair per phase.

Phases used for the primary index: `Second_threshold`, `WanT`, `Vo2max`; a
parallel script covers `Warmup` and `Cooldown`.

**Structural asymmetry (Chapters 3, 4)**

Left and right ultrasound dimensions are read from `us_merged_n35.xlsx` by
label — muscle, level, plane, measurement, side — and differenced. Chapter 4's
D1 analysis uses SCM at C4, longitudinal plane, AP of belly.

## 6. Exclusions and outlier handling

- Pairs with a missing side yield `NaN` and are dropped pairwise by the
  correlation and group tests.
- Where activation asymmetry is combined with %MVC data, the **400 %MVC
  artefact ceiling** applies (see `METHOD_MVC.md`).
- No outlier removal is applied to the asymmetry values themselves.
- Bonferroni correction is applied across the family of asymmetry tests — for
  the primary index, 12 comparisons (4 pairs × 3 phases), giving α = 0.00417.

## 7. Where it is computed — and where the sign differs

**Activation asymmetry (Chapters 1 and 4) is left-minus-right throughout.**
**The Chapter 3 morphology table is right-minus-left**, and labels itself as
such. Check the column header or the axis label before interpreting a sign.

| Scope | Form | Convention |
|---|---|---|
| Ch 1 — primary index (Spearman against VAS) | (a) | **L − R** |
| Ch 1 — warm-up/cool-down variant | (a) | **L − R** |
| Ch 1 — ordinal pain groups | (b) and (c) | **L − R** |
| Ch 4 — SCM structure × activation (D1) | (b) | **L − R** |
| Ch 4 — integrated analysis and figures | (b) | **L − R** |
| Ch 4 — clustering input | (c), normalised by the mean | magnitude only |
| **Ch 3 — bilateral Wilcoxon table** | **(b)** | **R − L** |
| Ch 3 — bilateral survivor figure | (b) | **L − R** |

**Note the split within the Chapter 3 analysis.** A single routine produces
both the statistics table and its accompanying figure, but computes the
difference in opposite directions: the table reports `right − left` under the
column headers `Mean Diff (R-L)` and `Median Diff (R-L)`, while the boxplot of
Bonferroni survivors computes `left − right`. Both are internally labelled, and
the Wilcoxon *p*-values are unaffected because the test is insensitive to the
order of its arguments — but the reported *difference* in the table and the
difference behind the figure carry opposite signs.

Chapter 3's narrative is written against the R − L table, which is why it
describes the posterior extensors as right-dominant.

