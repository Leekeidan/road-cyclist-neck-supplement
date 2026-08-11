# %MVC — EMG amplitude normalisation

## 1. What it is

Surface EMG amplitude expressed as a percentage of each muscle's maximum
voluntary contraction, so that channels and participants can be compared on a
common scale. Every downstream activation measure in this dissertation — the
co-contraction index, the asymmetry indices, the SCM activation phenotypes —
is built on %MVC values, not on raw millivolts.

## 2. Formula

```
%MVC(t) = 100 × x_filt(t) / MVC_max
```

| Term | Definition |
|---|---|
| `x_filt(t)` | Band-pass and notch filtered EMG at time *t*, in mV |
| `MVC_max` | The maximum amplitude recorded for that muscle during its dedicated MVC trial, in mV, read from the `MVC` sheet as `MVC_max [mV]` |

The reported per-bin value is a moving RMS of this normalised signal,
subsequently averaged into fixed bins:

```
RMS(t) = sqrt( movmean( %MVC(t)² , w ) ),   w = round(0.5 × fs) samples
```

## 3. Sign conventions and units

- **Units: percent (%).** A value of 100 means activation equal to the MVC
  reference for that muscle.
- **Always non-negative.** The moving RMS is a magnitude; there is no sign.
- Values above 100 % are physiologically possible (dynamic contraction
  exceeding the isometric reference) and do occur in these data.
- Eight channels, named `{muscle}{side}`: `UTL UTR` (upper trapezius),
  `MTL MTR` (middle trapezius), `STL STR` (splenius capitis /
  spinotransversalis), `SCML SCMR` (sternocleidomastoid). `L` = left,
  `R` = right.

## 4. Pipeline from raw file to value

1. **Input** — Delsys export, one CSV per phase per participant, with a paired
   time column per EMG channel.
2. **Sampling rate** — derived from the time vector, not assumed
   (`extract_sampling_frequency.m`).
3. **Unit conversion** — raw volts multiplied by 1000 to give mV.
4. **Zero removal** — samples where the raw signal is exactly zero are dropped
   along with their timestamps (dropout removal).
5. **Band-pass** — 4th-order Butterworth, **20–450 Hz**, applied with
   `filtfilt` (zero phase, so effectively 8th order).
6. **Notch** — **50 Hz** mains, `iircomb` with Q = 50, applied with `filtfilt`.
7. **Normalisation** — multiply by 100 and divide by the MVC reference.
8. **Moving RMS** — 0.5 s window (`round(0.5 × fs)` samples).
9. **Binning** — the RMS trace is averaged into **1-second** bins (and, in
   parallel, 10-second bins) via `moving_average_values.m`. The 1-second series
   is what every downstream analysis consumes, written to
   `subject_XXX_EMG_1sec_results.xlsx`, one sheet per phase plus an `MVC` sheet.

**Normalisation reference:** the MVC trials are separate recordings, one per
muscle (`999_MVC_UT.csv`, `..._SCML.csv`, and so on), processed by
`extract_mvc_value.m`.

> In `normalize_emg_to_mvc.m` the divisor is written `mvc_values(1)` — the first
> entry of the MVC table — inside the per-muscle loop, rather than
> `mvc_values(imuscle)`. Documented here as the code reads.

## 5. Exclusions and outlier handling

- Exact-zero samples are removed before filtering (step 4 above).
- A **400 %MVC ceiling** is applied in downstream analyses uniformly— values above 400 %
  are treated as artefact and dropped — for example when building the Chapter 4
  SCM activation asymmetry.
- No participant-level exclusion is applied at this stage; missing channels
  propagate as `NaN`.

## 6. Where it is computed

Signal extraction is implemented in **MATLAB**. Filtering uses a 4th-order
Butterworth band-pass at 20–450 Hz followed by a 50 Hz notch, both applied with
`filtfilt` for zero phase distortion. The sampling rate is recovered from the
time vector rather than assumed. Normalisation, the 0.5-second moving RMS and
the binning into 1-second and 10-second series are performed in a single pass,
writing `subject_XXX_EMG_1sec_results.xlsx`. The MVC reference is extracted
from the dedicated per-muscle MVC trials.

Group-level statistics are implemented in **Python** as a four-step sequence:
merge the per-participant files, reshape to wide and long form, test normality
with Shapiro–Wilk, then run the phase and muscle comparisons.

