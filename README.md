# BME Computational Foundations

## Current Project

A baseline-correction and peak-integration pipeline for FPLC UV chromatograms, scored against known peak areas.

Reported protein yield is the integrated area under an elution peak, and that area depends on where you draw the baseline. This pipeline estimates the baseline and characterizes the error in that estimate before integrating anything.

### Pipeline
   | Stage | Status |
   |---|---|
   | 1. Load and inspect the trace | Done |
   | 2. Baseline estimation and subtraction | Done |
   | 3. Peak detection | Done |
   | 4. Peak boundary assignment | Done |
   | 5. Trapezoidal integration  | In Progress |
   | 6. Validation on held-out trace | Not Started |

### Data 

> * `run_001.csv` - 1200 points over 60 minutes: six Gaussian elution peaks on a drifting baseline with detector noise.
> * It includes an injection artifact, a partially resolved peak pair, and a peak close to the noise floor.
> * The trace is simulated. Generated with Claude from known peak parameters. Claude also reviewed stages 1-2. Stages 1-2 were developed blind; the generator parameters were seen on Nov. 15th during Stages 3-4, so those stages are not. Final scoring will use a held-out trace (`run_002`).

### Method

The baseline is estimated by splitting the run into equal windows, taking a low percentile from each as an anchor, interpolating linearly between those anchors, and subtracting the result.

*Assumption:* The elution peaks only add absorbance, so the lowest values in a window are the least peak-contaminated. This holds for well-behaved UV traces, but fails whenever the signal dips below baseline, which can occur due to a refractive index mismatch at gradient transitions, air bubbles in the UV flow cell, or changes in buffer absorbance. Those regions would need separate handling.

#### Peak Detection and Boundary Assignment (Stages 3 & 4)
> * **Peak Detection (Stage 3)** - Points above floor + kσ are flagged, where σ = 0.003602 AU is estimated by the MAD of successive differences on a quiet stretch (50-56 min). Runs are found with `np.diff` on an edge-padded mask, and a width rule (m >= 3 consecutive points) rejects single-point detector noise. In a sweep over k, surviving runs plateau at 6 for k = 2.5-3.0; k = 3 sits on the plateau. On the quiet stretch, 2 points exceeded 3σ against 0.2 expected for Gaussian noise (n ≈ 120), so the tail may be heavier than Gaussian.
> * **Artifact Rejection (Stage 4)** - Linear interpolation edge-clamps the baseline outside the first anchor, producing a run at the start of the trace. A mask of `t > 5` removes it while keeping the first peak, whose leading edge begins near 5.5 min. An earlier cutoff of `t > 10` removed that peak.
> * **Co-elution Resolution (Stage 4)** - A big co-eluting double peak, spanning 21.5 - 22.5 min, was split by slicing the local signal and locating the valley index using `np.argmin`. Converting the relative local coordinate to an absolute global array index prevents bounds inversion and silent failure during downstream integration.

### Result

The first implementation used the window **minimum** and left a systematic offset of **+0.011630 AU** over a peak-free interval (50-56 min).

The offset is a property of the estimator rather than an implementation error: the minimum of n samples sits below the distribution mean by an amount that grows with n. Replacing the anchor with the **10th percentile** reduced the residual to **+0.004768 AU**, a 59% reduction, measured by running both anchors over identical windows against the same evaluation interval. The 10th percentile is a trade-off rather than a correct answer: it leaves 8.9% of the corrected trace below zero, against 0.4% for the minimum and 59.4% for the median.

The residual is positive, so the estimated baseline sits **below** the true one. Propagated through integration, that inflates each peak area by approximately (offset x peak width) - negligible for a tall peak, substantial for a short one.

**Why MAD of successive differences?** Differencing removes slow baseline drift: in peak windows, the std of raw values inflates roughly 100X, the differenced estimators 3-10X. MAD resists outliers: a simulated air bubble (one point, +0.5 and +5 AU) in the 52-56 min window moved the std of differences from 0.003495 to 0.0567 and 0.5629 AU, matching the predicted A/√79 within 1%, while MAD moved from 0.003342 to 0.003346. Where a peak fills more than half a window, MAD loses that advantage, which is why σ is measured on a quiet stretch rather than per window.

Applying the `k = 3` threshold, the physical width rule (`m >= 3`), and the > 5 min mask removed the early edge-clamping run. Solving the single co-eluting run at its local minimum yielded **6** peak boundaries on the development trace. Whether these rules generalize is tested on the held-out trace.

### Known Limitations

> * The baseline is edge-clamped in the first and last two minutes.
> * The method assumes peaks only add absorbance.
> * The evaluation interval lies inside the windows used to compute the anchors, so the residual is an in-sample estimate. A held-out quiet region would be a stronger test.
> * Peaks are simulated as Gaussians, while real chromatographic peaks have tails.
> * Stage 4 uses rules set by hand on this trace: the `t > 5` cutoff and the 21.5 - 22.5 min co-elution window. An edge-touching rule and a prominence-based split would replace them; the held-out trace tests whether they generalize.

### Running it

Open `04_FPLC_peak_integration/FPLC_peak_integration.ipynb` and run all cells. Requires NumPy, SciPy, and matplotlib.

------

## Next

Non-linear parameter estimation for allosteric enzyme kinetics (Michaelis-Menten / Hill formulations).
Motivated by the same problem as the FPLC pipeline: A reported Vmax or Km is the output of a fit, and the reliability of that fit is rarely reported next to it.

-----

## Background Work

> * **Core Python: `core_syntax_and_indexing/`** Algorithm design, list/dictionary comprehensions, genomic sequencing parsing (GC-content, classification, nucleotide frequency mapping, among others), sensor telemetry indexing.
> * **Numerical Computing (NumPy): `02_numpy_computational_arrays/`** Multi-dimensional array operations, vectorized mathematical calculations, broadcasting rules, array reshaping, joining, and aggregated statistical reductions.
> * **Tabular Data Processing (Pandas): *(In Progress)* `03_pandas_biochemical_curation/`** Series and DataFrame construction, label-based indexing, conditional masking/filtering, missing value imputation (`dropna`, `fillna`), and vectorized data transformations (`apply`, `map`).

**Tooling:** Jupyter Notebooks, Git/GitHub, macOS Terminal environment.


