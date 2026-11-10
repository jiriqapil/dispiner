# Light mode logo
![Manual Logo](./figures/logos/Logo_whiteboard_100dpi.png#gh-light-mode-only)
# Dark mode logo
![Manual Logo](./figures/logos/Logo_blackboard_100dpi.png#gh-dark-mode-only)

# Manual

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

This manual provides complete documentation for configuring, deploying, and executing ***dispiner*** across local workstations and high-performance computing (HPC) clusters.

---
## Contents

* **1. Overview**
* **2. Theoretical Background**
  * 2.1 Methodological Evolution
  * 2.2 Software Packages & Workflow Limitations
  * 2.3 ***dispiner*** Processing Stages and Algorithms
* **3. Computational Architecture & Execution Environments**
  * 3.1 Containerisation and FAIR Replicability
  * 3.2 Computational Environments
  * 3.3 Developer Mode & Dependedencies
* **4. Directory Structure and Data Preparation**
  * 4.1 Input Directory Layout
  * 4.2 Waveform Files (`MSEED/`)
  * 4.3 Station Coordinates (`META/stations.csv`)
  * 4.4 Task Queue (`task_list.csv`)
* **5. Configuration Reference (`config.env`)**
  * 5.1 Project Path Configuration
  * 5.2 Computational Execution and Batching Switches
  * 5.3 Preset Modes
* **6. Execution (How to Run)**
  * 6.1 Execution Procedure 
  * 6.2 Batching Workflows 
  * 6.3 Execution Examples
* **7. Output Data Organization**
  * 7.1 Systematic Batch Partitioning
  * 7.2 CSV Output Files
  * 7.3 NPY Binary Outputs
  * 7.4 PNG Pairwise Execution QC Figures
  * 7.5 LOG Output
* **References 
---
## 1. Overview

***dispiner*** is a containerized computational framework designed for large-scale extraction of multi-component surface-wave dispersion and amplitude measurements from ambient noise cross-correlations or earthquake two-station recordings.

By bridging the gap between ambient noise pre-processing toolkits (such as *MSNoise*) and surface-wave inversion codes (such as *FMST* and *Geopsy/dinver*), ***dispiner*** delivers robust, harmonised datasets optimised for crustal and lithospheric-scale radial and azimuthal anisotropic tomography, attenuation tomography, and horizontal-to-vertical (HV) inversions.

Built around a flexible batching architecture, ***dispiner*** scales seamlessly from single-core parameter testing to large-scale HPC cluster execution, allowing regional datasets to be continuously expanded as new seismic stations become available. All output products are standardised for automated harvesting into downstream inversion workflows.

---

## 2. Theoretical Background

### 2.1 Methodological Evolution

Surface-wave dispersion measurement has evolved through successive methodological advances, establishing the theoretical foundations for automated wavefield analysis and anisotropic tomography:

 for multi-component wavefield harmonisation and scale-invariant fractional-octave filtering (the two core methodological improvements realised in ***dispiner***):

* **Dziewonski et al. (1969):** Introduced Multiple Filter Analysis (MFT) using narrow-band Gaussian filters with fixed resolution parameter (𝛼 = 50), establishing group-velocity dispersion extraction via analytic signal and Hilbert transform envelope calculation.

* **Herrmann (1973):** Formalized the analytical behavior and mathematical parameter equivalence of narrow-band Gaussian and rectangular filtering (such as 𝛼 = 16 𝜋), demonstrating how filter parameterisations govern time-frequency resolution trade-offs and envelope distortion relative to group-velocity curve slopes.

* **Levshin et al. (1972) & Bensen et al. (2007):** Formalized automated Frequency-Time Analysis (FTAN) with phase-matched filtering to extract group velocities and instantaneous frequencies from ambient noise cross-correlations, documenting spectral gaps and mode contamination on FTAN diagrams. Although PreMatch filtering is useful for cleaning complex records or addressing multipathing, its anti-dispersion and redispersing steps require an a priori velocity model or initial dispersion curve to collapse the signal limiting its functionality for dynamic database building, evolution, and expansion.

* **Boschi et al. (2013):** Developed frequency-domain phase-velocity retrieval based on zero crossings of the zeroth-order Bessel function (J0) from the real part of vertical Rayleigh cross-correlation spectra, constrained by the 2pi phase.

### 2.2 Software Packages & Workflow Limitations

The application of surface-wave dispersion theory into practice has driven the development of various software packages and processing pipelines, each exhibiting specific functional limitations that motivated the design of ***dispiner*** with specific functionalities crucial for consistent surface-wave anisotropic tomography and handling large datasets:

* **Herrmann (2013):** Developed the *Computer Programs in Seismology* [*CPS*] package to provide standard computational frameworks for surface-wave analysis, though classical MFT still required manual curation to handle multi-mode interference and frequency-dependent attenuation.

* **Kästle et al. (2016) [*amb_noise_tools*], Sadeghisorkhani et al. (2018) [*GSpecDisp*], & Magrini et al. (2022) [*SeisLib*]:** Automated and expanded these Bessel zero-crossing algorithms into practical software packages. Sadeghisorkhani et al. introduced anchor points to stabilize dispersion curve picking in frequency bands with low signal-to-noise ratios. While these packages advanced automation, they still rely on resolving 2pi cycle skipping using external velocity models, dispersion trends, regional anchor points, or manual curation. Because their phase selection procedures depend on *a priori* information, they remain constrained in their ability to achieve unbiased, large-scale database expansion and dynamic evolution.

* **Mordret et al. (2014):** Represented an early effort in ambient noise full-tensor processing for high-resolution surface wave tomography. However, applying this in the  short-period band proved difficult due to intense scattering by shallow heterogeneities, multipathing, and high-amplitude surface-wave overtones that frequently caused automated dispersion pickers to fail. As a result, they had to use a graphical interface for manual dispersion curve curation. While this manual approach successfully built high-resolution anisotropic models, it remains impractical for large ambient-noise datasets requiring automated processing.

* **Schippkus et al. (2018) & Qorbani et al. (2020):** Computed full-tensor cross-correlations across vertical, radial, and transverse components, yet stopped short of a fully harmonised workflow. Specifically, Schippkus et al. noted that analyzing Love waves required an adapted dispersion curve measurement and selection process (proposing that Rayleigh and Love waves should be inverted jointly) but ultimately restricted their practical implementation primarily to Rayleigh-wave components. Similarly, Qorbani et al. processed Rayleigh and Love paths independently to build separate shear velocity models, ultimately encountering noticeable discrepancies and inconsistencies between the resulting models.

* **Witek et al. (2021) & Magrini et al. (2023):** Witek et al. demonstrated the necessity of joint Rayleigh-Love inversion and short-period ambient noise integration, but relied on a fractured pipeline and exhibited residual baseline inconsistencies. In contrast, newer frameworks like Magrini et al. significantly advanced workflow automation and hierarchical Bayesian joint inversion, though they still depend on a priori reference dispersion curves to resolve cycle ambiguities during phase velocity extraction.
 
* **Lu et al. (2018):** Automated group-velocity FTAN for continental-scale ambient noise tomography, introducing distance-dependent Gaussian filter bandwidths α(Δ) to scale time-frequency resolution with path length.

* **Paul et al. (2024):** Pointed out that ambient noise and seismic anisotropy modeling across the crust and mantle relies on massive datasets from large seismic networks like AlpArray. Because of this computationally demanding processing task, data processing tools can no longer rely on simple, isolated scripts. Instead, they need high-performance computing (HPC) setups featuring batch processing, dynamic database building, and standardised outputs preconfigured for automated database harvesting and joint inversions.

* **Kvapil et al. (2021):** Used Rayleigh-wave-only data to infer crustal radial anisotropy from lower-crust velocity decrease. They pointed out that accurately resolving this feature requires the joint inversion of Rayleigh and Love waves, which depends on highly consistent multi-component dispersion curve measurements. Because existing software packages lacked an automated way to handle large, dynamically expandable multicomponent datasets in a consistent manner, an urgent methodological need emerged for a robust, consistency-driven framework capable of multi-component dispersion measurement for anisotropic surface-wave tomography.

* ***dispiner*** **(HALIBUT mode):** A specialised framework built for radial and azimuthal anisotropic inversions, bridging the gap between preprocessing pipelines like *MSNoise* and surface wave tomography codes like *FMST* and *Geopsy/dinver*. The framework utilises two core algorithms: HALI, which harmonizes linear anchor trends for Z-R-T wavefield consistency, and -BUT, which applies scale-invariant fractional-octave Butterworth filtering. By combining these methods with group-velocity guided Bessel-zero phase selection, it performs entirely data-driven measurements that eliminate the need for an a priori reference model. It features an automated multi-branch cross-validation algorithm designed for dynamic database building across large permanent and temporary seismic network deployments. Its architecture is optimised for high-performance computing (HPC) environments to handle massive ambient noise datasets and is fully containerised to ensure FAIR-compliant replicability across any platform.

### 2.3 dispiner Processing Stages and Algorithms

The ***dispiner*** workflow executes in six automated processing stages:

* **Step 1: Input Stream Formatting & Preprocessing:**

  * Reads 9-component MiniSEED cross-correlation streams (ZZ, RR, TT across average, causal, and acausal branches).

  * Requires prior instrument response removal, optional trace processing (demeaning, detrending, and spectral whitening), full tensor cross-correlation, component rotation to ZZ(vertical), RR(radial), and TT(transverse), and cross-correlation branch separation.

* **Step 2: The "HALI" Algorithm (Two-Pass Z-R-T Anchor Harmonization):**

  * Evaluates cross-correlations through wide 1/2-octave Butterworth filters across microseismic (8–32 s) and discrete full-spectrum (4–128 s) bands.

  * Tests Rayleigh consistency (Z ≈ R); defines reference ZR_ref = mean(Z, R) or selects the higher-quality component when Z ≠ R.

  * Tests Rayleigh consistency (Z ≈ R); defines reference ZR_ref = mean(Z, R) or selects the higher-quality component when Z ≠ R.

  * Two-pass structure: Pass 1 establishes global velocity bounds; Pass 2 defines local dynamic tracking corridors.

* **Step 3: The "BUT" Algorithm (Scale-Invariant Butterworth Group Velocity Tracking):**

  * Adapts Butterworth filter banks conforming to the IEC 61260-1 Base-2 standard, positioning center frequencies on periods with a constant relative bandwidth (df/f 
0
​
 =const).

  * Directional group-velocity tracking initiated at the dominant energy band (T 
init
​
 =16 s) and progressing bidirectionally across the frequency spectrum.
 

  * Extracts envelope maxima and instantaneous frequencies ω 
inst
​
 =dϕ/dt via Hilbert transformation.

* **Step 4: Monotonic PCHIP Grid Repositioning:**

  * Repositions irregular instantaneous period-velocity pairs onto the standardized 38-period logarithmic grid (2.2–161.3 s in 1/6-octave steps).

  * Piecewise Cubic Hermite Interpolating Polynomials (PCHIP) preserve local shape and prevent spurious overshoot near steep velocity gradients.

* **Step 5: Group-Guided Bessel Zero Phase Velocity Retrieval:**

  * Determines zero-crossing candidates from the real cross-correlation spectrum using J₀ (Rayleigh) and J₂ (Love) Bessel functions.

  * Uses the harmonized group-velocity curve as a reference corridor to select the physical phase branch, resolving 2π cycle skipping without requiring external regional velocity models.

  * Automatically filters out the 26-second monochromatic Gulf of Guinea noise resonance.

* **Step 6: Quality Measures & Multi-Branch Cross-Validation:**

  * Applies selection criteria to filter unreliable measurements:
  
    * *Interstation Distance / Wavelength (λ-cut):* Rejects picks where inter-station distance Δ<2λ (or 3λ), where wavelength λ=v⋅T.

    * *Signal-to-Noise Ratio (SNR):* Group and phase velocity picks require signal-to-noise ratio (SNR) limits.

    * *Negative Slope Cutoff:* Fundamental-mode surface-wave group velocity curves exhibit physical bounds on negative velocity gradients. Unrealistic steep negative slopes caused by spectral holes or higher-mode interference are rejected.

    * *26-Second Noise Spike Removal:* Automatically mitigates monochromatic energy spikes from the Gulf of Guinea microseism at 26 s period.

  * Validates measurement consistency across wavefield components (Z, R, T) and wavefield branches (causal, acausal, average), categorizing outputs into Dataset_3C, Dataset_1C, and Dataset_0C tiers.
  
---

## 3. Computational Architecture & Execution Environments

### 3.1 Containerization and FAIR Replicability

Scientific surface-wave dispersion workflows often suffer from software dependency drift (e.g., conflicting versions of numerical solvers, digital filters, or signal processing packages). ***dispiner*** eliminates environment bias through fully self-contained container images:

* **Docker:** Designed for local workstations, cloud instances, and continuous integration testing.

* **Apptainer / Singularity:** Tailored for multi-user university and national laboratory HPC clusters requiring unprivileged, rootless execution.

* **FAIR Replicability:** Encapsulates identical versions of Python, ObsPy, NumPy, SciPy, and Pandas, ensuring that identical input cross-correlations yield identical dispersion picks regardless of host operating system.

* **Database Evolution:** Enables users to ingest new cross-correlation records into existing tomographic archives over time using the exact same algorithmic binary, preventing artificial bias in long term regional studies.


### 3.2 Computational Environments

***dispiner*** provides three operational execution pathways controlled by the `SUBMIT_MODE` parameter:

1. **Single-Core Workstation:** Executes station-pair tasks sequentially on a single CPU core. Recommended for initial parameter testing, configuration debugging, and examining individual dispersion curve picks on small test sets (e.g., 1–10 station pairs).


2. **Multi-Core Parallel Workstation:** Utilizes Python multiprocessing to distribute station pairs across local workstation CPU cores. Recommended for intermediate pilot runs on dozens to hundreds of station pairs.


3. **High-Performance Computing Cluster (HPC):** Designed for massive datasets containing almost unlimited number of of station pairs. The orchestrator automatically partitions the task queue into discrete batch chunks, writes self-contained OpenPBS batch scripts, and prepares the master dispatch script.

### 3.3 Developer Mode & Dependencies

For developers or environments where containers cannot be executed, dispiner can run locally in Python 3.9+ using the provided dependency manifest (`requirements.txt`):

```bash
# Environment setup via conda / mamba and pip
conda create -n dispiner-env python=3.10 -y
conda activate dispiner-env
pip install -r requirements.txt

```

Required third-party dependencies specified in `requirements.txt`:

* `numpy`: Vectorized numerical and matrix operations.
* `scipy`: Digital filtering (Butterworth), Hilbert transformation, and PCHIP interpolation.
* `matplotlib`: Generating diagnostic and publication summary figures.
* `pyproj`: Cartographic projections and coordinate transformations.
* `netCDF4`: Reading and writing ETOPO and grid-based datasets.
* `basemap` & `basemap-data-hires`: Geographic plotting and map generation.
* `obspy`: Reading and handling MiniSEED waveforms and seismic metadata.
* `Pillow`: Image processing utilities.
* `python-dotenv`: Environment variable and configuration management from `config.env`.

---
## 4. Directory Structure and Data Preparation

The ***dispiner*** project directory layout can be fully customised in `config.env`, though the recommended structure is preconfigured in defaults for the demo run.

### 4.1 Input Directory Layout

```text
input/
├── MSEED/
│   ├── NET1.STA1_NET2.STA2.mseed
│   └── ...
├── META/
│   ├── stations.csv
│   ├── ETOPO/                      (Optional: netCDF topography raster)
│   └── QGIS/                       (Optional: tectonic fault and basin CSVs)
└── task_list.csv

```

### 4.2 Waveform Files (`MSEED/`)

Each station pair requires a single MiniSEED file named:
`{NET1}.{STA1}_{NET2}.{STA2}.mseed`

* **MiniSeed stream formatting:**
  * Sampling Rate: 10.0 Hz.
  * Time Lag: 720 s maximum lag time (7201 samples).
  * MiniSEED file must contain 9 traces arranged in the following trace index order:

| Trace Index | Component | Branch | Description |
| --- | --- | --- | --- |
| `0` | `ZZ` | `AV` | Vertical Rayleigh wave, symmetric average
| `1` | `ZZ` | `AC` | Vertical Rayleigh wave, acausal branch (t<0, time-reversed)
| `2` | `ZZ` | `CA` | Vertical Rayleigh wave, causal branch (t>0)
| `3` | `RR` | `AV` | Radial Rayleigh wave, symmetric average
| `4` | `RR` | `AC` | Radial Rayleigh wave, acausal branch (t<0, time-reversed)
| `5` | `RR` | `CA` | Radial Rayleigh wave, causal branch (t>0)
| `6` | `TT` | `AV` | Transverse Love wave, symmetric average
| `7` | `TT` | `AC` | Transverse Love wave, acausal branch (t<0, time-reversed)
| `8` | `TT` | `CA` | Transverse Love wave, causal branch (t>0)

* **Pre-processing Requirements:**

  * Continuous recordings must be corrected for instrument response, with optional demeaning, detrending, and spectral whitening.

  * Preprocessing requires full tensor cross-correlation followed by component rotation to ZZ (for vertical Rayleigh wave extraction), RR (for horizontal radial Rayleigh wave extraction), and TT (for transverse Love wave extraction).

  * Traces are separated into causal (t > 0), acausal (t < 0, time-reversed), and symmetric average branches.


### 4.3 Station Coordinates (`META/stations.csv`)

A comma-separated file providing geographical coordinates for all stations in the task list.

Mandatory column headers:
`StationID,Latitude,Longitude,Elevation`

Example `stations.csv`:

```text
StationID,Latitude,Longitude,Elevation
GR.UBR,47.6800,10.1070,903.0
PL.OJC,50.2200,19.7980,391.0
CZ.KHC,49.1308,13.5781,745.0

```

### 4.4 Task Queue (`task_list.csv`)

Defines the station pairs to be processed in the current run.

* Columns:
`id,pair,stack,use`
  * `pair`: Identifier matching the MiniSEED filename (`{NET1}.{STA1}_{NET2}.{STA2}`).

  * `stack`: Number of days stacked in the cross-correlation.

  * `use`: Execution switch. Only rows where `use` is `Y` or `Yes` are processed.


Example `task_list.csv`:

```text
id,pair,stack,use
1,GR.UBR_PL.OJC,74,Y
2,GR.UBR_CZ.KHC,120,Y
3,PL.OJC_CZ.KHC,95,N

```

---
## 5. Configuration Reference (`config.env`)

Runtime parameters are managed in `config.env` or passed directly to `setup_params()` in Python.

### 5.1 Project Path Configuration

| Variable | Default Value | Description |
| --- | --- | --- |
| `INPUT_DIR` | `./demo/input` | Root directory containing input subdirectories
| `OUTPUT_DIR` | `./demo/output` | Destination directory for output batches
| `MSEED_PATH` | `./demo/input/MSEED` | Directory containing MiniSEED waveform files
| `STATIONS_PATH` | `./demo/input/META/stations.csv` | Path to station coordinate CSV file
| `TASK_LIST` | `./demo/input/task_list_demo.csv` | Path to task list CSV file
| `ETOPO_NC` | `None` | Path to optional ETOPO netCDF topography raster
| `QGIS_DIR` | `None` | Path to optional directory with GIS structural vectors

### 5.2 Computational Execution and Batching Switches

| Variable | Default Value | Options / Format | Description |
| --- | --- | --- | --- |
| `SUBMIT_MODE` | `pcmono` | `pcmono`, `pcpara`, `hpc`| Target execution environment
| `NTASKS_IN_BATCH` | `1000` | Integer | Number of station pairs per batch execution script
| `ANCHOR_HARMONISATION` | `True` | `True`, `False` | Activates two-pass Z-R-T anchor harmonisation
| `CROSSVALIDATION` | `True` | `True`, `False` | Activates multicomponent (Z-R-T) multi-branch (CA/AC/AV) cross-validation
| `VMIN_GRID` | `1.0` | Float (km/s) | Lower velocity limit for FTAN grid computation
| `VMAX_GRID` | `8.0` | Float (km/s) | Upper velocity limit for FTAN grid computation
| `VMIN_PICK` | `1.5` | Float (km/s) | Minimum physically acceptable velocity for picks
| `VMAX_PICK` | `5.0` | Float (km/s) | Maximum physically acceptable velocity for picks
| `MAP_STYLE` | `Public` | `Public`, `Private` | Map background style for summary figures
| `IMAGE_QUALITY` | `Publish` | `Publish`, `Preview` | Figure resolution (`Publish`=300 DPI, `Preview`=96 DPI)
| `SHOW_FIGURE` | `False` | `True`, `False` | Display summary figures interactively during execution

### 5.3 Preset Modes

* **`HALIBUT` (Production Default):**

  * Full two-pass anchor trend harmonization across Z, R, and T.

  * Fractional-octave Butterworth filter bank for scale-invariant group velocity extraction.

  * Group-velocity guided Bessel zero selection for phase velocity.

  * Three-component multi-branch cross-validation enabled.

* **`CONVENTIONAL_GAUS` (Benchmark Mode):**

  * Independent component processing with andchor trend harmonisation disabled.
 
  * Gaussian filter MFT with constant alpha = 50.
 
  * Three-component multi-branch cross-validation enabled.


* **`CUSTOM` (User Defined Mode):**

  * Selectable filter types (Butterworth or Gaussian) and filter bandwidths.

  * Configurable activation of anchor harmonization and multi-branch cross-validation.

  * User-defined velocity search windows, SNR cutoffs, and slope validation thresholds.

  * Toggleable three-component multi-branch cross-validation enabled.

---

## 6. Execution (How to Run)

### 6.1 Execution Procedure

Execution is initiated by running `python3 run.py`, which reads parameters from `config.env` and partitions the task queue into isolated batch workspaces inside the `./batch_execution/` directory.

To execute the workflow, run:

```bash
python3 run.py

```

### 6.2 Batching Workflows

Depending on the selected `SUBMIT_MODE`, the orchestrator dispatches execution to the corresponding launcher script located in the `./launchers/` directory (`pcmono.sh`, `pcpara.sh`, or `hpc.sh`). Each launcher manages the execution environment and invokes the core `dispiner` processing module located in `./dispiner/__main__.py`) according to selected mode.

Operational submit modes include:

* **`SUBMIT_MODE=pcmono`**: Dispatches execution via `./launchers/pcmono.sh`, running station-pair tasks sequentially in the foreground on the local workstation.


* **`SUBMIT_MODE=pcpara`**: Dispatches execution via `./launchers/pcpara.sh`, running station-pair tasks concurrently across parallel local CPU cores.


* **`SUBMIT_MODE=hpc`**: Dispatches execution via `./launchers/hpc.sh`, generating isolated OpenPBS job scripts for containerized cluster scheduling.

### 6.3 Execution Examples

For comprehensive, step-by-step instructions and ready-to-use commands across GitHub Codespaces, local Docker/Apptainer containers, and HPC cluster environments, refer to **[execution_examples.txt](./execution_examples.txt)**.

----
## 7. Output Data Organization

### 7.1 Systematic Batch Partitioning

***dispiner*** organises results into discrete batch subdirectories:

```text
output/
└── batch_000/
    ├── CSV/
    │   ├── Dataset_3C/             (3-component validated pairs)
    │   │   ├── group_velocity/
    │   │   ├── phase_velocity/
    │   │   ├── amplitude/
    │   │   ├── attribute/
    │   │   ├── networks.csv
    │   │   ├── stations.csv
    │   │   └── pairs.csv
    │   └── Dataset_1C/             (1-component acceptable pairs)
    ├── NPY/
    │   ├── dispersions_3C.npy
    │   ├── dispersions_1C.npy
    │   └── dispersions_0C.npy
    ├── PNG/
    │   └── NET1.STA1_NET2.STA2_Publ.png
    └── LOG/
        ├── jobconfig_timestamp.log
        └── execution_timestamp.log

```

This layout enables automated routines to harvest completed batches to update existing databases. 
 

### 7.2 CSV Output Files

All CSV output tables sample dispersion values at **38 discrete periods** in 1/6-octave logarithmic steps from 2.2 s to 161.3 s:

`2.2, 2.5, 2.8, 3.2, 3.6, 4.0, 4.5, 5.0, 5.7, 6.3, 7.1, 8.0, 9.0, 10.1, 11.3, 12.7, 14.3, 16.0, 18.0, 20.2, 22.6, 25.4, 28.5, 32.0, 35.9, 40.3, 45.3, 50.8, 57.0, 64.0, 71.8, 80.6, 90.5, 101.6, 114.0, 128.0, 143.7, 161.3 s`

Each CSV file contains embedded header comment blocks designed for smooth SQL database harvesting:

* **Velocity Dispersion Files (`group_velocity/` and `phase_velocity/`)**

Group and phase velocity measurement tables include header comment blocks defining attributes, components, branches, units, and period sampling:

```text
# Attribute: GRP (group velocity) / PHA (phase velocity)
# Wave type: RZZ (Rayleigh vertical) (also RRR, LTT)
# CCF branch: AV (average) (also CA, AC)
# Pair: station1_station2 identifier
# Val<i>: group velocity pick at period Period[i]
# Units: km/s
# Missing value: nan
# Period sampling in 1/6-octave steps; Effective period range 2.2 s-161 s

```

* **Amplitude (`amplitude/`)**

Instantaneous envelope amplitude measurement files contain the following header metadata:

```text
# Attribute: AMP (amplitude)
# Wave type: RZZ (Rayleigh vertical) (also RRR, LTT)
# CCF branch: AV (average) (also CA, AC)
# Pair: station1_station2 identifier
# Val<i>: instantaneous envelope amplitude (a.u.) at period Period[i]
# Units: dimensionless
# Missing value: nan
# Period sampling in 1/6-octave steps; Effective period range 2.2 s-161 s

```

* **Attribute files (`attribute/`)*

Diagnostic quality and signal-to-noise ratio files (`SNR_*.csv` and `BZQ_*.csv`) embed explicit field definitions:

```text
# Attribute: SNR (signal-to-noise ratio) / BZQ (Bessel zero quality metric)
# Wave type: RZZ (Rayleigh vertical) (also RRR, LTT)
# CCF branch: AV (average) (also CA, AC)
# Pair: station1_station2 identifier
# Val<i>: signal-to-noise ratio at period Period[i] / time difference between upward and downward Bessel zero-crossing velocity picks, normalized by Period[i]
# Units: dB / % (percent of period)
# Missing value: nan
# Period sampling in 1/6-octave steps; Effective period range 2.2 s-161 s

```

* **Metadata (`networks.csv`, `stations.csv`, `pairs.csv`)**

Batch metadata files provide structural and geographical definitions for network inventories, stations, and inter-station paths:

```text
# networks.csv
# FDSNCode: FDSN network code or XX for unregistered temporary networks/experiments

# stations.csv
# StationID: network code and station code (network.station)
# Latitude: station latitude in decimal degrees (°N)
# Longitude: station longitude in decimal degrees (°E)
# Elevation: station elevation in meters above sea level (m)

# pairs.csv
# Pair: station1_station2 identifier
# InterstationDistance: great-circle distance between stations in km
# Azimuth: forward azimuth from station1 to station2 in degrees
# BackAzimuth: reverse azimuth from station2 to station1 in degrees
# MidLat: midpoint latitude in decimal degrees (°N)
# MidLon: midpoint longitude in decimal degrees (°E)
# DataCompletenessTag: dispersion data completeness based on Z and RT correlation coverage (ZxRTy) where 2 = full AV with both CA and AC branches, 1 = AV with incomplete CA/AC coverage, and 0 = missing average (AV)
# NumberOfDays: number of days stacked for ambient-noise cross-correlation (EGF)

```

### 7.3 NPY Binary Outputs

Developer Data Structures (`NPY/`) consist of  aggregated NumPy binary files containing structured dictionaries with pick arrays, quality masks, and metadata. This format provides access to raw data structures for custom pipeline development, enabling users to integrate ***dispiner*** outputs into external post-processing orkflows, custom inversion routines, or specialized visualization tools.

Load in Python:

```python
import numpy as np

data_3c = np.load('output/batch_000/NPY/dispersions_3C.npy', allow_pickle=True).item()
periods = data_3c['periods']
pair_names = data_3c['pairs']
grp_zz_av = data_3c['GRP_RZZ_AV']  # 2D array: (n_pairs, 38)

```

### 7.4 PNG Pairwise Execution QC Figures

For each processed station pair, a comprehensive multi-panel quality control (QC) figure is generated to monitor and verify the entire measurement workflow. These pairwise figures are structured into a standardized layout:

* **Map Panel (Top Left):** Displays the inter-station path plotted over regional topography, including path orientation, great-circle distance, and stack length.
* **Final Cross-Validated Dispersion Curves (Top Right):** Summarizes the multi-component (Z, R, T) cross-validated group and phase velocity curves alongside wavelength cutoff boundaries (1 lambda, 2 lambda, 3 lambda).
* **Cross-Correlation Traces Panels:** Arranged in three component columns (ZZ, RR, TT), displaying the waveform traces across symmetric average (AV), causal (CA), and acausal (AC) branches.
* **FTAN Dispersion Measurement Panels:** Three-column frequency-time analysis diagrams showing initial anchor trends, tracking corridors, PCHIP-repositioned group velocity picks, and Bessel zero-crossing phase picks (J0 / J2).
* **Amplitude & Apparent Attenuation Panels:** Three-column plots showing instantaneous envelope amplitude measurements converted to apparent attenuation profiles across wavefield branches.
* **Real Part of Cross-Correlation Spectra Panels:** Displays the real component of the cross-correlation spectra across all components and branches.
* **Measurement Reliability Panels:** Period-dependent quality metrics displaying signal-to-noise ratios (SNR in dB) and Bessel Zero Quality metric (BZQ in %).

Image resolution and map background styles are fully configurable through the IMAGE_QUALITY and MAP_STYLE parameters in config.env.

### 7.5 LOG Output

Execution logs storing configuration parameters, runtime environment details, and per-task status reports.

----
## References

Bensen, G. D., M. H. Ritzwoller, M. P. Barmin, et al. 2007. "Processing
Seismic Ambient Noise Data to Obtain Reliable Broad-Band Surface Wave
Dispersion Measurements." *Geophys. J. Int.* 169 (3): 1239--60.
<https://doi.org/10.1111/j.1365-246X.2007.03374.x>.

Beyreuther, Moritz, Robert Barsch, Lion Krischer, Tobias Megies, Yannik
Behr, and Joachim Wassermann. 2010. "ObsPy: A Python Toolbox for
Seismology." *Seismol. Res. Lett.* 81 (3): 530--33.
<https://doi.org/10.1785/gssrl.81.3.530>.

Boschi, Lapo, Cornelis Weemstra, Julie Verbeke, Göran Ekström, Andrea
Zunino, and Domenico Giardini. 2013. "On Measuring Surface Wave Phase
Velocity from Station--Station Cross-Correlation of Ambient Signal."
*Geophys. J. Int.* 192 (1): 346--58.
<https://doi.org/10.1093/gji/ggs023>.

Dziewoński, Adam M., S. Bloch, and M. Landisman. 1969. "A Technique for
the Analysis of Transient Seismic Signals." *Bull. Seismol. Soc. Am.* 59
(1): 427--44.

Herrmann, Robert B. 1973. "Some Aspects of Band-Pass Filtering of
Surface Waves." *Bull. Seismol. Soc. Am.* 63 (2): 663--71.
<https://doi.org/10.1785/BSSA0630020663>.

Herrmann, Robert B. 2013. "Computer Programs in Seismology: An Evolving
Tool for Instruction and Research." *Seismol. Res. Lett.* 84 (6):
1081--88. <https://doi.org/10.1785/0220110096>.

Kästle, Emanuel D., Riaz Soomro, Cornelis Weemstra, Lapo Boschi, and
Thomas Meier. 2016. "Two-Receiver Measurements of Phase Velocity:
Cross-Validation of Ambient-Noise and Earthquake-Based Observations."
*Geophys. J. Int.* 207 (3): 1493--512.
<https://doi.org/10.1093/gji/ggw341>.

Kvapil, J., J. Plomerová, H. Kampfová Exnerová, V. Babuška, G. Hetényi,
and AlpArray Working Group. 2021. "Transversely Isotropic Lower Crust of
Variscan Central Europe Imaged by Ambient Noise Tomography of the
Bohemian Massif." *Solid Earth* 12 (5): 1051--74.
<https://doi.org/10.5194/se-12-1051-2021>.

Lecocq, Thomas, Corentin Caudron, and Florent Brenguier. 2014. "MSNoise,
a Python Package for Monitoring Seismic Velocity Changes Using Ambient
Seismic Noise." *Seismol. Res. Lett.* 85 (3): 715--26.
<https://doi.org/10.1785/0220130073>.

Levshin, Anatoli, V. F. Pisarenko, and G. A. Pogrebinsky. 1972. "On a
Frequency-Time Analysis of Oscillations." *Ann. Geophys.* 28 (2):
211--18.

Lu, Yang, Laurent Stehly, Anne Paul, and AlpArray Working Group. 2018.
"High-Resolution Surface Wave Tomography of the European Crust and
Uppermost Mantle from Ambient Seismic Noise." *Geophys. J. Int.* 214
(2): 1136--50. <https://doi.org/10.1093/gji/ggy188>.

Magrini, Fabrizio, Emanuel Kästle, Simone Pilia, Nicholas Rawlinson, and
Luca De Siena. 2023. "A New Shear-Velocity Model of Continental
Australia Based on Multi-Scale Surface-Wave Tomography." *J. Geophys.
Res. Solid Earth* 128 (7): e2023JB026688.
<https://doi.org/10.1029/2023JB026688>.

Magrini, Fabrizio, Sebastian Lauro, Emanuel Kästle, and Lapo Boschi.
2022. "Surface-Wave Tomography Using SeisLib: A Python Package for
Multiscale Seismic Imaging." *Geophys. J. Int.* 231 (2): 1011--30.
<https://doi.org/10.1093/gji/ggac236>.

Mordret, Aurélien, Diane Rivet, Matthieu Landès, and Nikolaï M. Shapiro.
2015. "Three-Dimensional Shear Velocity Anisotropic Model of Piton de La
Fournaise Volcano (La Réunion Island) from Ambient Seismic Noise." *J.
Geophys. Res. Solid Earth* 120 (1): 406--27.
<https://doi.org/10.1002/2014JB011654>.

Paul, Anne, Helle A. Pedersen, Thomas Bodin, et al. 2024.
"Methodological Advances in Seismic Noise Imaging of the Alpine Area."
*Comptes Rendus. Géoscience* 356 (S4): 5--39.
<https://doi.org/10.5802/crgeos.261>.

Qorbani, E., D. Zigone, M. R. Handy, G. Bokelmann, and AlpArray-EASI
Working Group. 2020. "Crustal Structures Beneath the Eastern and
Southern Alps from Ambient Noise Tomography." *Solid Earth* 11 (5):
1947--68. <https://doi.org/10.5194/se-11-1947-2020>.

Rawlinson, Nicholas, and Malcolm Sambridge. 2005. "The Fast Marching
Method: An Effective Tool for Tomographic Imaging and Tracking Multiple
Phases in Complex Layered Media." *Exploration Geophysics* 36 (4):
341--50. <https://doi.org/10.1071/EG05341>.

Sadeghisorkhani, Hamzeh, Ólafur Gudmundsson, and Ari Tryggvason. 2018.
"GSpecDisp: A Matlab GUI Package for Phase-Velocity Dispersion
Measurements from Ambient-Noise Correlations." *Comput. Geosci.* 110:
41--53. <https://doi.org/10.1016/j.cageo.2017.09.006>.

Scarponi, M., J. Kvapil, J. Plomerová, S. Solarino, and G. Hetényi.
2024. "New Constraints on the Shear Wave Velocity Structure of the Ivrea
Geophysical Body from Seismic Ambient Noise Tomography (Ivrea-Verbano
Zone, Alps)." *Geophys. J. Int.* 236 (2): 1089--105.
<https://doi.org/10.1093/gji/ggad470>.

Schippkus, S., D. Zigone, G. Bokelmann, and AlpArray Working Group.
2018. "Ambient-Noise Tomography of the Wider Vienna Basin Region."
*Geophys. J. Int.* 215 (1): 102--17.
<https://doi.org/10.1093/gji/ggy259>.

Wathelet, Marc, Jean‐Luc Chatelain, Cécile Cornou, et al. 2020. "Geopsy:
A User‐friendly Open‐source Tool Set for Ambient Vibration Processing."
*Seismol. Res. Lett.* 91 (3): 1878--89.
<https://doi.org/10.1785/0220190360>.

Witek, M., S.-J. Chang, D. Y. Lim, S. Ning, and J. Ning. 2021. "Radial
Anisotropy in East Asia from Multimode Surface Wave Tomography." *J.
Geophys. Res. Solid Earth* 126 (7): e2020JB021201.
<https://doi.org/10.1029/2020JB021201>.

----
