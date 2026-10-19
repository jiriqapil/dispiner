![](../../Logos/dispiner_logo.png)
## dispiner

---

* **Codename:** ***dispiner*** (*powered by* `HALIBUT`)  
* **Version:** `v1.0.0`  
* **Author:** *Jiri Kvapil*  
* **Affiliation:** Czech Academy of Sciences, Institute of Geophysics, Czech Republic  
* **Compilation requirements:** Apptainer, Docker, Singularity, or Podman (for running containers); Python 3.10+ (for local development without containers).  
* **Dependencies:** Defined in `requirements.txt` and packaged inside the Dockerfile.  
* **Year:** 2026  
* **Status:** Stable  
* **Support:** j.kvapil@ig.cas.cz  
* **License:** EUPL v1.2  

> **Citation:**  
> Kvapil, J. (2026). *dispiner: Containerised Multi-Component Surface-Wave Harmonisation and Dispersion Measurement Framework* (Version 1.0.0) [Computer Software]. https://github.com/jiriqapil/dispiner  
> (A formal software paper is currently in preparation and will update this citation upon publication).

---

***dispiner*** is an containerised computational framework built for multi-component surface-wave dispersion and amplitude measurement extracted from ambient noise cross-correlations or earthquake two-station recordings. It primarily extracts fundamental mode Rayleigh and Love wave group and phase velocity dispersion curves across vertical, radial, and transverse components, while simultaneously outputting wavefield amplitudes and coupled group and phase quality measures.

Designed specifically for high-performance computing (HPC) environments and large-scale data production, ***dispiner*** features an integrated batching system that allows users continuously build and expand regional databases as new seismic stations and records are added. Its systematically structured outputs are formatted for automated data harvesting into regional tomographic inversion databases.

By bridging the processing gap between pre-processing toolkits (such as *MSNoise*) and surface-wave inversion codes (such as *FMST* and *Geopsy/dinver*), ***dispiner*** delivers robust, harmonized datasets optimised for crustal and lithospheric-scale radial and azimuthal anisotropic tomography, attenuation tomography, and horizontal-to-vertical (HV) inversions.

---

## Key Features

* **HALIBUT Mode (Default Production Algorithm):** A fully automated processing mode optimised for robust regional and continental production runs. It integrates two-pass anchor trend harmonization, fractional-octave Butterworth filtering, and guided Bessel-zero phase retrieval into a standalone station-pair-wise workflow without requiring external a priori velocity models or regional trend calculations.
* **Complete Multi-Component Measurement:** Extracts 18 dispersion curves per station pair, covering vertical Rayleigh (ZZ), radial Rayleigh (RR), and transverse Love (TT) waves across average, causal, and acausal correlation branches for both group and phase velocities.
* **Two-Pass Anchor Trend Harmonization:** Enforces data consistency of vertical, radial, and transverse components using linear anchor trends within the microseismic period band (8–32 s) and discrete full-spectrum tie points (4–128 s) to avoid mode-jumping and local noise spikes.
* **Scale-Invariant Fractional-Octave Filtering:** Adapts Butterworth filter banks (IEC 61260-1 Base-2 standard) with a constant relative bandwidth (Δf/f0​=const) to maintain wave-cycle consistency across passbands.
* **Instantaneous Wavefield Parameter Repositioning:** Measures envelope peaks and instantaneous frequencies from the analytic signal via Hilbert transform, repositining picks onto regular 1/6-octave period grid using Piecewise Cubic Hermite Interpolating Polynomials (PCHIP).
* **Guided Phase-Velocity Retrieval:** Determines phase velocities from Bessel function zero-crossings (J0​ for Rayleigh, J2​ for Love) guided directly by harmonized group-velocity curves to eliminate cycle ambiguity without requiring external a priori regional velocity models.
* **Automated Data Quality Curation:** Evaluates signal-to-noise ratio (SNR), Bessel zero up/down crossing consistency metrics (BZQ), negative slope thresholds to identify spectral holes (λ-cut), and removes localised 26-second monochromatic noise spikes.
* **Multi-Branch Cross-Validation:** Validates measurement consistency across wavefield components, group and phase dispersion completeness, and causal/acausal branches, classifying pairs into reliability groups: `Dataset_3C`, `Dataset_1C`, and `Dataset_0C.

---

## Batching Architecture and Computational Modes

A primary operational advantage of ***dispiner*** is its integrated batching module, configured to scale seamlessly from desktop PCs for parameter testing to massive high-performance computing (HPC) environments for large-scale data production:

* **Parameter Testing on Small Datasets (`pcmono` & `pcpara`):**
  * `pcmono`: Executes tasks sequentially on a single CPU core for interactive debugging, step-by-step verification, and rapid parameter sensitivity testing.
  * `pcpara`: Parallel local execution across workstation CPU cores using multiprocessing for intermediate pilot runs on dozens to hundreds of station pairs.
* **Large-Scale Production Mode (`hpc`):**
  * The production mode partitions extensive station-pair task queues into workload chunks governed by `NTASKS_IN_BATCH` (e.g., 250, 500, or 1000 pairs per job).
  * Automatically generates self-contained OpenPBS batch scripts (`job_000.pbs`, `job_001.pbs`, etc.) and the master submission script `batch_execution/spool_queue.sh`.
* **Standardized Database Harvesting:**
  * Output files are partitioned into systematic batch directories (`output/batch_000/`, `output/batch_001/`).
  * Tabular CSV files, concatenated NumPy arrays (`dispersions_3C.npy`), and diagnostic plots are structured for automated database harvesting into regional tomographic inversion database.

---

## Preset Modes

* **`HALIBUT` (Default Production Mode):**
  * Full two-pass anchor trend harmonization across ZZ, RR, and TT components.
  * Fractional-octave Butterworth filter bank for scale-invariant group velocity extraction.
  * Group-velocity guided Bessel zero-crossing selection for phase velocities.
  * Automated three-component multi-branch cross-validation.
* **`CONVENTIONAL_GAUS` (Benchmark Mode):**
  * Single-trace Multiple Filter Analysis without wavefield harmonization.
  * Constant Gaussian filter parameter (alpha = 50) following Dziewonski et al. (1969).
  * Independent component processing with standard multi-branch cross-validation.
* **`CUSTOM` (User-Defined Mode):**
  * Selectable filter types (Butterworth or Gaussian) and filter bandwidths.
  * Configurable activation of anchor harmonization and multi-branch cross-validation.
  * User-defined velocity search windows, SNR cutoffs, and slope validation thresholds.
---
## How to Run
For complete configuration guidelines, file formatting details (including MiniSEED 9-trace stream ordering and CSV headers), and for ready-to-use workflows across cloud virtual machines, local Docker/Apptainer setups, and HPC cluster environments, refer to **`MANUAL.md`**.

### Demorun inside GitHub Virtual Machine (Codespaces)
You can run a zero-setup verification test directly in your browser using GitHub Codespaces:
1. Launch a codespace for the repository.
2. Once the browser terminal opens, execute the containerized demorun:
   ```bash
   docker build -t dispiner:test .
   docker run --rm -it -v "$(pwd)/demo_results":/app/demo/output dispiner:test

---
## Documentation
- **[MANUAL.md](./MANUAL.md):** Comprehensive operational guidelines, container setup, batch execution details, and database harvesting workflows.
---