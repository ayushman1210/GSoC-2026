# Google Summer of Code 2026: Final Work Product Report

# PEcAn: Benchmarking and Validation Framework

> [!IMPORTANT]
> **Project Link for GSoC Dashboard Submission**: This repository serves as the official final work report for the **PEcAn Benchmarking and Validation Framework** project undertaken during **Google Summer of Code (GSoC) 2026**.

---

## 🌟 Key Findings & Main Accomplishments

Before delving into metadata and pull requests, here are the primary scientific and technical results achieved by the new validation framework:

### 1. Framework Validation: Statistical Error Model Parameter Recovery
To rigorously test the framework's diagnostic accuracy, synthetic observations were generated from model outputs using an explicit statistical error model with known bias ($\beta = 0.30$) and noise ($\tau = 0.50$). The decoupled benchmarking engine successfully recovered the theoretical closed-form expectations:

| Metric | Closed-Form Theoretical Expectation | Empirical Calculated Metric | Scientific Significance |
| :--- | :---: | :---: | :--- |
| **BIAS** | `-1.89` | **`-1.79`** | Recovers directional error ($\beta_v = 0.30$) |
| **RMSE** | `3.68` | **`3.61`** | Quantifies total L2 error magnitude |
| **MAE** | `2.96` | **`2.86`** | Confirms robust L1 distance bounds |
| **R²** | `0.80` | **`0.79`** | Recovers variance fraction ($\frac{1}{1 + \tau_v^2}$) |

### 2. UQ Calibration Finding: Diagnosing Ensemble Under-Dispersion
Evaluating Prediction Interval Coverage (`metric_Coverage`) on a **90% target interval** returned an empirical coverage rate of **`0.44`**. This result provides immediate empirical evidence of ensemble variance under-dispersion, illustrating how the new UQ metric suite pinpoints model calibration deficiencies that standard point metrics (like RMSE) miss.

### 3. Before vs. After Architectural Transformation

```
  BEFORE (Legacy Architecture)                      AFTER (GSoC 2026 Validation Framework)
 ┌─────────────────────────────┐                  ┌─────────────────────────────────────────┐
 │   Relational DB (BETYdb)    │                  │ Arbitrary User Data (CSV, RDS, NetCDF)  │
 └──────────────┬──────────────┘                  └────────────────────┬────────────────────┘
                │ Tight Coupling                                       │ Dataframe-First
                ▼                                                      ▼
 ┌─────────────────────────────┐                  ┌─────────────────────────────────────────┐
 │ Monolithic Database Scripts │                  │  Pure R Intake API (`load_data.R`)      │
 └──────────────┬──────────────┘                  └────────────────────┬────────────────────┘
                │ Slow Loops                                           │ `findInterval` Binary Search
                ▼                                                      ▼
 ┌─────────────────────────────┐                  ┌─────────────────────────────────────────┐
 │ Hardcoded Metric Evaluation │                  │  Isolated Metric Suite + UQ (9+ metrics)│
 └──────────────┬──────────────┘                  └────────────────────┬────────────────────┘
                │ Fixed Schemas                                        │ Parameterized Scorecard
                ▼                                                      ▼
 ┌─────────────────────────────┐                  ┌─────────────────────────────────────────┐
 │ Manual Database Benchmarks  │                  │  Automated Quarto Scorecard Reports     │
 └─────────────────────────────┘                  └─────────────────────────────────────────┘
```

---

## Project Metadata

- **Student Name**: Ayushman ([@ayushman1210](https://github.com/ayushman1210))
- **Mentors**: David LeBauer ([@dlebauer](https://github.com/dlebauer)), Akash ([@divine7022](https://github.com/divine7022))
- **Organization**: [PEcAn Project](https://github.com/PecanProject/pecan)
- **Repository**: [PecanProject/pecan](https://github.com/PecanProject/pecan)
- **Project Goal**: Build a General-Purpose, Dataframe-First Validation Toolkit for PEcAn, decoupling validation metrics, time/space alignment, and automated reporting from database schemas (BETYdb).

---

## Quick Navigation (Rendered Reports)


- 📊 **[GSoC 2026 Final Project Quarto Report (HTML)](https://ayushman1210.github.io/GSoC-2026/GSoC_2026_Final_Report.html)** ([Repository File](./GSoC_2026_Final_Report.html) | [.qmd Source](./GSoC_2026_Final_Report.qmd))
- 📈 **[Salinas SOC Ensemble Benchmark Report (HTML)](https://ayushman1210.github.io/GSoC-2026/Salinas_SOC_Validation_Report.html)** ([Repository File](./Salinas_SOC_Validation_Report.html))
- 🌲 **[AmeriFlux Tower Benchmark Report (HTML)](https://ayushman1210.github.io/GSoC-2026/AmeriFlux_Validation_Report.html)** ([Repository File](./AmeriFlux_Validation_Report.html))
- 🎓 **[Validation Framework Tutorial Vignette (HTML)](https://ayushman1210.github.io/GSoC-2026/validation_framework_tutorial.html)** ([Repository File](./validation_framework_tutorial.html))
- 📄 [GSoC 2026 Workplan & Timeline](./workplan.md)
- 🔍 [Codebase Audit & Architecture Report](./codebase_audit.md)
- 🔀 [Merged PR & Issue Inventory](./pr_inventory.md)
- 📚 [Tutorial Vignette & Quickstart Guide](./tutorial_guide.md)
- 📖 [Tutorial Vignette Source (`validation_framework_tutorial.Rmd`)](./vignettes/validation_framework_tutorial.Rmd)

---

## Executive Summary & Abstract

Historically, PEcAn's legacy `PEcAn.benchmark` module was tightly coupled to BETYdb (PEcAn's central relational PostgreSQL database). This architecture required all model outputs and observational datasets to be registered in the database before any statistical alignment or benchmarking could take place. In addition, state data assimilation (SDA) and field campaign workflows maintained redundant, custom scripts for data parsing, alignment, and scoring.

During **GSoC 2026**, the project pivoted away from refactoring rigid legacy database code to build a **General-Purpose Validation Toolkit**. The new toolkit operates on a **Dataframe-First Architecture**, allowing users to flexibly ingest arbitrary tabular (CSV, TSV, RDS) or gridded (NetCDF) datasets, map variable names natively, align observations and predictions using high-performance binary search routines, calculate statistical and uncertainty metrics, and auto-generate publication-ready HTML/PDF benchmark scorecards using Quarto.

---

## Summary of Merged Pull Requests & Contributions

All project goals across Phases 1 through 5 were successfully completed, tested, documented, and merged into the main `PEcAnProject/pecan` repository:

| Phase | PR / Issue Link | Scope & Summary | Status |
| :--- | :--- | :--- | :---: |
| **Phase 1 & 2** | [PR #4017](https://github.com/PecanProject/pecan/pull/4017) | **Data Intake API & Baseline Refactor**: Abstracted data loading out of BETYdb into pure R loaders (`load_data.R`). Implemented `align_by_time` with fast `findInterval` binary search tolerances and added `pecan_metric_registry`. | **Merged** |
| **Phase 2** | [PR #4041](https://github.com/PecanProject/pecan/pull/4041) | **Standalone Metric Isolation**: Decoupled core statistical metrics (`RMSE`, `MAE`, `R2`, `Correlation`, `Bias`) into pure functions. Sanitized test environments and removed unreliable fallbacks. | **Merged** |
| **Phase 2** | [PR #4032](https://github.com/PecanProject/pecan/pull/4032)<br>[Issue #4027](https://github.com/PecanProject/pecan/issues/4027) | **Numeric Metrics & UQ Calibration Coverage**: Achieved 100% test coverage for all 9 legacy numeric metrics. Added Predictive Model Uncertainty (`metric_PMU`) and Prediction Interval Coverage (`metric_Coverage`). Documented `metric_R2`. | **Merged** |
| **Phase 3** | [PR #4040](https://github.com/PecanProject/pecan/pull/4040)<br>[Issue #4037](https://github.com/PecanProject/pecan/issues/4037) | **Automated Reporting & Visualizations**: Modernized `ggplot2` plotting suite. Built parameterized Quarto scorecard template (`Validation_report.qmd`) and `generate_validation_report()` API. | **Merged** |
| **Phase 4** | [PR #4062](https://github.com/PecanProject/pecan/pull/4062)<br>[Issue #4059](https://github.com/PecanProject/pecan/issues/4059) | **MVP Integration & Real-World Benchmarks**: Constructed Salinas SOC multi-site 50-member ensemble benchmark and AmeriFlux tower validation workflow. Developed `efi_long_to_array()` parser for EFI long format forecasts. | **Merged** |
| **Phase 5** | [PR #4087](https://github.com/PecanProject/pecan/pull/4087)<br>[PR #4076](https://github.com/PecanProject/pecan/pull/4076) | **Final Handoff, Documentation & Vignette**: Authored comprehensive tutorial vignette (`validation_framework_tutorial.Rmd`), expanded `testthat` suite to 16 test files, and completed Roxygen2 blocks across all exported functions. | **Merged** |

---

## Technical Architecture & Key Innovations

### 1. Data Intake Abstraction ([`R/load_data.R`](./R/load_data.R), [`R/data_intake.R`](./R/data_intake.R))
- Flexibly loads tabular datasets (`load_csv`, `load_tab`, `load_rds`) and gridded files (`load_netcdf`).
- Translates column names dynamically into standard PEcAn variable names using simple YAML/list configurations.

### 2. High-Performance Timestamp Alignment ([`R/align_data.R`](./R/align_data.R))
- Replaced nested, slow loops with fast binary search matching (`findInterval`).
- Accepts user-configurable temporal tolerances (e.g. 1 hour, 1 day) to handle sampling rate mismatches without dropping data.

### 3. Complete Statistical & Uncertainty Quantification (UQ) Metric Suite ([`R/`](./R/))
- **Numeric Statistical Metrics**: `Bias`, `RMSE`, `MSE`, `MAE`, `R2`, `Correlation` (`metric_cor`), `AME`, `RAE`, `Frechet`, `PPMC`.
- **Probabilistic & UQ Calibration Metrics**:
  - `metric_Coverage`: Quantifies Prediction Interval Coverage (e.g. 90% PI empirical coverage check).
  - `metric_PMU`: Calculates Predictive Model Uncertainty ratio.
  - `metric_CRPS`: Calculates Continuous Ranked Probability Score for ensemble outputs.
- **Metric Registry**: Integrated `pecan_metric_registry` to dynamically register user-defined validation metrics.

### 4. Multidimensional Array Parsing ([`R/efi_long_to_array.R`](./R/efi_long_to_array.R))
- Extracts standard Ecological Forecasting Initiative (EFI) long-format forecasts into multidimensional matrices `[time, site, ensemble]` for fast array-based validation metrics.

### 5. Automated Scorecards & Modern Visualizations
- **[`reports/Validation_report.qmd`](./reports/Validation_report.qmd)**: Parameterized Quarto scorecard template compiling tables, metrics, and plots into an interactive HTML report.
- **`generate_validation_report()`**: Single-function R wrapper API to render scorecards programmatically.
- **Visualizations**: `metric_timeseries_plot()`, `metric_scatter_plot()`, `metric_residual_plot()`, `metric_lmDiag_plot()`.

---

## Real-World Benchmarks & Workflow Integration

### 1. Salinas Organic Cropping Systems (SOC) Multi-Site Ensemble Benchmark
- **Location**: [`examples/benchmarks/salinas_soc_ensemble/`](./examples/benchmarks/salinas_soc_ensemble/)
- Evaluated a 50-member model prediction ensemble across 8 agricultural management systems against observed Soil Organic Carbon (SOC) stock measurements (`white_salinas_2020`).

### 2. AmeriFlux Tower Flux Pipeline & Ingestion Prototype
- **Location**: [`examples/benchmarks/ameriflux/`](./examples/benchmarks/ameriflux/)
- Constructed an automated observation intake pipeline fetching eddy-covariance flux observations from `ccmmf/cal-val-data`. Operates as a reusable tower validation stub ready to ingest carbon/water flux model ensemble runs as upstream predictions land.

---

## Unit Testing & Quality Assurance

The validation toolkit features a comprehensive `testthat` suite consisting of **16 test files** in [`tests/`](./tests/) covering 100% of lifted loader, alignment, metric, visualization, and vignette execution routines:

```
tests/
├── test-align_pft.R
├── test-data_intake.R
├── test-efi_long_to_array.R
├── test-ensemble_integration.R
├── test-load_csv.R
├── test-load_data.R
├── test-load_netcdf.R
├── test-load_rds.R
├── test-load_tab.R
├── test-metric_Bias.R
├── test-metric_CRPS.R
├── test-metrics.R
├── test-numeric_metrics.R
├── test-run_benchmark.R
├── test-vignette_execution.R
└── test-visualization.R
```

To run the full unit test suite:
```r
testthat::test_dir("tests")
```

---

## Tested Quickstart Code Example

```r
library(PEcAn.benchmark)

# 1. Locate built-in simulation fixtures in package extdata
extdata_dir <- system.file("extdata", package = "PEcAn.benchmark")
if (extdata_dir == "" || !file.exists(file.path(extdata_dir, "simulated_model_ensemble.csv"))) {
  extdata_dir <- "modules/benchmark/inst/extdata"
}

model_csv <- file.path(extdata_dir, "simulated_model_ensemble.csv")
obs_csv   <- file.path(extdata_dir, "simulated_observations.csv")

raw_model <- read.csv(model_csv)
raw_obs   <- read.csv(obs_csv)

# 2. Reshape ensemble member columns into matrix
ens_cols <- grep("^model_[0-9]+$", colnames(raw_model), value = TRUE)
ens_mat  <- as.matrix(raw_model[, ens_cols])

model_summary <- data.frame(
  time      = as.POSIXct(raw_model$time, tz = "UTC"),
  value     = raw_model$ensemble_mean,
  model_q05 = raw_model$model_q2.5,
  model_q95 = raw_model$model_q97.5,
  site      = "SiteA"
)

obs_summary <- data.frame(
  time    = as.POSIXct(raw_obs$time, tz = "UTC"),
  value   = raw_obs$obvs,
  obvs_sd = raw_obs$obvs_sd,
  site    = "SiteA"
)

# 3. Align timesteps using fast binary search matching
aligned_df <- align_by_time(model_summary, obs_summary, tolerance_secs = 1800)
aligned_indices <- match(aligned_df$time, model_summary$time)
attr(aligned_df, "ensemble_matrix") <- ens_mat[aligned_indices, , drop = FALSE]

# 4. Compute comprehensive point & UQ metrics
metrics_results <- compute_metrics(
  aligned_df,
  metrics = c("BIAS", "RMSE", "MAE", "R2", "COVERAGE", "CRPS")
)

print(metrics_results)

# 5. Render diagnostic timeseries plot
p_ts <- metric_timeseries_plot(aligned_df, var = "NEE (umol m-2 s-1)")
```

---

## Message From Ayushman

> Thank you, PEcAn Project, for giving me this wonderful opportunity! 🙏
>
> A special thanks to my mentors, **David LeBauer** and **Akash**, for constantly supporting and motivating me throughout my journey. Your guidance, encouragement, and willingness to always be there whenever I faced challenges have meant a lot to me.
>
> I’m truly grateful for your support and for everything I’ve learned along the way. ❤️
>
> Thank you, PEcAn, once again, for this amazing opportunity!

