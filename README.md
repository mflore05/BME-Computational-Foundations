# BME Computational Foundations

## Current Project

A baseline-correction and peak-integration pipeline for FPLC UV chromatograms, developed against a blind reference.

Reported protein yield is the integrated area under an elution peak, and that area depends on where you draw the baseline. This pipeline estimates the baseline and characterizes the error in that estimate before integrating anything.

### Pipeline
   | Stage | Status |
   |---|---|
   | 1. Load and inspect the trace | done |
   | 2. Baseline estimation and subtraction | done |
   | 3. Peak detection | not started |
   | 4. Peak boundary assignment | not started |
   | 5. Trapezoidal integration  | not started |
   | 6. Validation against known areas | not started |

### Data 

* `run_001.csv` - 1200 points over 60 minutes: six Gaussian elution peaks on a drifting baseline with detector noise.
* It includes an injection artifact, a partially resolved peak pair, and a peak close to the noise floor.
* The trace is simulated. It was generated in a separate session with an AI assistant that retained the true peak parameters, so this analysis is developed blind: recovered areas can be scored against a withheld answer rather than another method's output.

### Method

The baseline is estimated by splitting the run into equal windows, taking a low percentile from each as an anchor, interpolating linearly between those anchors, and subtracting the result.

**Assumption:** The elution peaks only add absorbance, so the lowest values in a window are the least peak-contaminated. This holds for well-behaved UV traces, but fails whenever the signal dips below the baseline, which can occur through a refractive index mismatch at gradient transitions, air bubbles in the column, or changes in buffer absorbance. Those regions would need separate handling.

### Result

The first implementation used the window **minimum** and left a systematic offset of **+0.011630 AU** over a peak-free interval (50-56 min).

The offset is a property of the estimator rather than an implementation error: the minimum of n samples sits below the distribution mean by an amount that grows with n. Replacing the anchor with the **10th percentile** reduced the residual to **+0.004768 AU**, a 59% reduction, measured by running both anchors over identical windows against the same evaluation interval.

The residual is positive, so the estimated baseline sits **below** the true one. Propagated through integration, that inflates each peak area by approximately (offset x peak width) - negligible for a tall peak, substantial for a short one.

### Known Limitations

* The baseline is edge-clamped in the first and last two minutes.
* The method assumes peaks only add absorbance.
* The evaluation interval lies inside the windows used to compute the anchors, so the residual is an in-sample estimate. A held-out quiet region would be a stronger test.
* Peaks are simulated as Gaussians, while real chromatographic peaks have tails.

### Running it

Open `04_FPLC_peak_integration/FPLC_peak_integration.ipynb` and run all cells. Requires NumPy and matplotlib.

------

## Next

Non-linear parameter estimation for allosteric enzyme kinetics (Michaelis-Menten / Hill formulations).
Motivated by the same problem as the FPLC pipeline: A reported Vmax or Km is the output of a fit, and the reliability of that fit is rarely reported next to it.

-----

## Background Work

* **Core Python: `core_syntax_and_indexing/`** Algorithm design, list/dictionary comprehensions, genomic sequencing parsing (GC-content, classification, nucleotide frequency mapping, among others), sensor telemetry indexing.
* **Numerical Computing (NumPy): `02_numpy_computational_arrays/`** Multi-dimensional array operations, vectorized mathematical calculations, broadcasting rules, array reshaping, joining, and aggregated statistical reductions.
* **Tabular Data Processing (Pandas): *(In Progress)* `03_pandas_biochemical_curation/`** Series and DataFrame construction, label-based indexing, conditional masking/filtering, missing value imputation (`dropna`, `fillna`), and vectorized data transformations (`apply`, `map`).

**Tooling:** Jupyter Notebooks, Git/GitHub, macOS Terminal environment.


