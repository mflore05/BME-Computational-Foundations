# BME Computing & Data Analysis Foundations

A structured repository documenting self-directed computational training, numerical array modeling, and data pipelines oriented towards biomedical engineering and translational research workflows.

## Current Progress
* **Core Python:** Algorithm design, list/dictionary comprehensions, genomic sequencing parsing (GC-content, classification, nucleotide frequency mapping, among others), sensor telemetry indexing.
* **Numerical Computing (NumPy):** Multi-dimensional array operations, vectorized mathematical calculations, broadcasting rules, array reshaping, joining, and aggregated statistical reductions.
* **Tabular Data Processing (Pandas - Active):** Series and DataFrame construction, label-based indexing, conditional masking/filtering, missing value imputation ('dropna', 'fillna'), and vectorized data transformations ('apply', 'map').
* **Tooling:** Jupyter Notebooks, Git/GitHub, macOS Terminal environment.

---

## Repository Structure

**Self-directed Python, NumPy, and Pandas practice, organized by topic.**
* `core_syntax_and_indexing/`: Applied exercises covering string manipulation on FASTA-style biological sequences and multi-slice sensor telemetry arrays.
* `02_numpy_computational_arrays/`: Vectorized array operations and multi-variable numerical processing notebooks.
* *(In Progress)* `03_pandas_biochemical_curation/`: Data ingestion and cleaning pipelines tailored to multi-well plate reader outputs and chromatography fractions.

---

## Research Application 
This repository connects experimental biochemistry with quantitative data analysis and builds the computational foundation for PhD research in Biomedical Engineering. Current applications include:
1. FPLC Peak Integration

Reported protein yield is the integrated area under an elution peak, and the area depends on where you draw the baseline. This pipeline estimates the baseline and characterizes the error in that estimate before integrating anything.

   ## Pipeline
   | Stage | Status |
   |---|---|
   | 1. Load and inspect the trace | done |
   | 2. Baseline estimation and subtraction | done |
   | 3. Peak detection | not started |
   | 4. Peak boundary assignment | not started |
   | 5. Trapezoidal integration  | not started |
   | 6. Validation against known areas | not started |

   ## Data

   * ´run_001.csv´ - 1200 points over 60 minutes: six Gaussian elution peaks on a drifting baseline with detector noise.
   * It includes an injection artifact, a partially resolved peak pair, and a peak close to the noise floor.
   * The data is simulated. The generator was written separately and is not included here, so the analysis cannot see the true peak areas.
   * **Recovered values can be scored against time afterward.**

   ## Method

   * The baseline is estimated by splitting the run into equal windows, taking a low percentile of each as an anchor, and interpolating in a linear fashion between those anchors and subtracted.
   * **Assumption:** The elution peaks only add absorbance, so the lowest values in a window are the least peak-contaminated. This holds for well-behaved UV traces, but fails whenever the signal dips below the baseline, which can occur through a refractive index mismatch at gradient transitions, air bubbles in the column, or changes in buffer absorbance. Those regions would need separate handling.

   ## Result

   * The first implementation used the window **minimum** and left a systematic  **+0.011630 AU** residual over a peak-free interval (50-56 min).
   * The offset is a property of the estimator rather than an implementation error: the minimum of n samples sits below the distribution mean by an amount that grows with n. Replacing the anchor with the **10th percentile** reduced the residual to **+0.004768**, a 59% reduction, measured by running both anchors over identical windows against the same evaluation interval.
   * The residual is positive, so the estimated baseline sits *below* the true one.
   * Propagated through integration, that inflates each peak area by approximately (offset x peak width), negligible for a tall peak, substantial for a short one.

   ## Known Limitations

   * The baseline is edge-clamped in the first and last two minutes.
   * The method assumes peaks only add absorbance.
   * The evaluation interval lies inside the windows used to compute the anchors, so the residual is an in-sample estimate. A held-out quiet region would be a stronger test.
   * Peaks are simulated as Gaussians, while real chromatographic peaks have tails.
    
   ## Running it

   * Open 'FPLC_peak_integration.ipynb' and run all cells. Requires NumPy and matplotlib.
  
   ### Future Work
2.  Non-linear parameter estimation for allosteric enzyme kinetics (Michaelis-Menten / Hill formulations).
   * Motivated by the same problem as the FPLC pipeline: A reported Vmax or Km is the output of a fit, and the reliability of that fit is rarely reported next to it.
