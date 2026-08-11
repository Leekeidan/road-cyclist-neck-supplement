# IMF — instantaneous median frequency, and IMF slope

## 1. What it is

The median frequency of the EMG power spectrum, computed in short sliding
windows so that it can be tracked continuously through an exercise phase. A
downward drift in median frequency over time is the standard surface-EMG index
of localised muscular fatigue; the **IMF slope** quantifies that drift as a
single number per muscle per phase.

## 2. Formula

For each window, the median frequency `f_med` is the frequency that bisects the
total power of the Welch periodogram:

```
Σ  P(f)  =  ½ · Σ  P(f)
 f ≤ f_med          all f
```

Implemented as the first frequency bin at which the cumulative power reaches
half the total.

| Term | Definition |
|---|---|
| `P(f)` | Power spectral density estimated by Welch's method (Hann window) |
| `f_med` | Median frequency for that window, in Hz |

The slope is an ordinary least-squares regression of `f_med` on time:

```
f_med(t) = β₀ + β₁ · t
```

`β₁` is the reported **IMF slope**.

## 3. Sign conventions and units

- **IMF units: hertz (Hz).**
- **IMF slope units: Hz per second.**
- **A negative slope means the median frequency is falling — the fatigue
  direction.** The code encodes this explicitly: a slope is flagged as a
  fatigue finding only when `slope < 0 and p_value < 0.05`.
- Normalised IMF, where used, is expressed as a percentage of the
  participant's own baseline for that muscle.

## 4. Pipeline from raw file to value

1. **Input** — the same raw Delsys per-phase EMG exports used for both %MVC and IMF.
2. **Preprocessing** — identical to the amplitude pipeline: 4th-order
   Butterworth band-pass **20–450 Hz** and a **50 Hz** notch, both zero-phase
   (`filtfilt`).
3. **Sampling rate** — recovered from the time vector as
   `fs = 1 / mean(diff(time))`.
4. **Sliding window** — **0.512 s** (512 ms), chosen for dynamic contractions,
   with **50 % overlap** (`step = window × 0.5`).
5. **Spectral estimate** — `scipy.signal.welch`, Hann window, per segment.
6. **Median frequency** — cumulative sum of the PSD; the first bin reaching
   50 % of total power is taken as `f_med`. Windows with zero, non-finite or
   non-positive total power are skipped.
7. **Gap handling** — discontinuities in the time vector larger than 1.5 s are
   treated as recording breaks and the series is split rather than interpolated
   across them.
8. **Slope** — `scipy.stats.linregress` over the window time-stamps and IMF
   values for that muscle and phase.

**Phases processed:** `Static`, `Warmup`, `Second_threshold_extracted`,
`Vo2max_extracted`, `WanT`, `CoolDown`. The second-threshold and VO₂max windows
are extracted from the continuous ramp using per-participant onset times in
`data/files_help/start_Times_Vo2max.csv`.

**Fatigue thresholds used in the episode analysis:** a per-sample flag at
`IMF < 50 %` of baseline (`THRESHOLD_FLAG_FRACTION = 0.50`), and episode
detection at **20 %, 30 % and 50 %** drops
(`THRESHOLD_DROP_EPISODES = [0.20, 0.30, 0.50]`).

## 5. Exclusions and outlier handling

- Windows with degenerate spectra (empty, non-finite, or total power ≤ 0) are
  skipped at computation time.
- Segments shorter than one analysis window produce no IMF value.
- **1.5 × IQR outlier removal per muscle × phase** is applied before the slope
  ANOVA, in `IMF_Step4_Slope_Analysis.py`. Removed points are logged; a cleaned
  long-format file (`IMF_LongFormat_CLEANED_IQR.csv`) and a flagged file
  (`IMF_LongFormat_with_outlier_flags.csv`) are both retained in
  `data/chapter2_emg/IMF/`.

## 6. Where it is computed

Implemented in **Python**, using `scipy.signal.welch` for the spectral estimate
and `scipy.stats.linregress` for the slope, as a five-step sequence:

1. Extract the second-threshold and VO₂max windows from the continuous ramp
   using the per-participant onset times.
2. Preprocess (band-pass and notch), compute the median frequency over sliding
   windows, and fit the slope per muscle and phase.
3. Convert to the long and wide formats used by the statistics.
4. Slope statistics — repeated-measures ANOVA with `pingouin`, Bonferroni
   post-hoc, and 1.5 × IQR outlier removal per muscle × phase.
5. Fatigue-duration analysis at the 20 %, 30 % and 50 % thresholds.

