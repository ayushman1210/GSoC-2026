# Google Summer of Code 2026: Final Work Product Report

# PEcAn: Benchmarking and Validation Framework

> [!IMPORTANT]
> **Project Link for GSoC Dashboard Submission**: This directory (`gsoc_2026/`) serves as the self-contained official final work report for the **PEcAn Benchmarking and Validation Framework** project undertaken during **Google Summer of Code (GSoC) 2026**.

---

## Project Metadata

- **Student Name**: Ayushman ([@ayushman1210](https://github.com/ayushman1210))
- **Mentors**: David LeBauer ([@dlebauer](https://github.com/dlebauer)), Akash ([@akash-g5](https://github.com/akash-g5))
- **Organization**: [PEcAn Project](https://github.com/PecanProject/pecan)
- **Repository**: [PecanProject/pecan](https://github.com/PecanProject/pecan)
- **Project Goal**: Build a General-Purpose, Dataframe-First Validation Toolkit for PEcAn, decoupling validation metrics, time/space alignment, and automated reporting from database schemas (BETYdb).

---

## Quick Navigation

- 📊 **[GSoC 2026 Final Project Quarto Report (HTML)](./GSoC_2026_Final_Report.html)** ([.qmd Source](./GSoC_2026_Final_Report.qmd))
- 📈 **[Salinas SOC Ensemble Benchmark Report (HTML)](./Salinas_SOC_Validation_Report.html)**
- 🌲 **[AmeriFlux Tower Benchmark Report (HTML)](./AmeriFlux_Validation_Report.html)**
- 🎓 **[Validation Framework Tutorial Vignette (HTML)](./validation_framework_tutorial.html)**
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

```
                             [ User Data Sources ]
                         (CSVs, NetCDFs, RDS, Tabular)
                                       │
                                       ▼
                   ┌──────────────────────────────────────┐
                   │           Data Intake API            │
                   │  (`load_data`, `load_csv`, etc.)     │
                   └──────────────────────────────────────┘
                                       │
                                       ▼
                   ┌──────────────────────────────────────┐
                   │       Fast Alignment Engine          │
                   │ (`align_by_time` / `findInterval`)   │
                   └──────────────────────────────────────┘
                                       │
                                       ▼
                   ┌──────────────────────────────────────┐
                   │       Extensible Metric Suite        │
                   │ (9 Metrics + PMU + Coverage + CRPS)  │
                   └──────────────────────────────────────┘
                                       │
                                       ▼
                   ┌──────────────────────────────────────┐
                   │    Automated Scorecard Reporting     │
                   │ (`Validation_report.qmd` + ggplot2)  │
                   └──────────────────────────────────────┘
```

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

## Real-World Benchmarks & End-to-End MVPs

### 1. Salinas Organic Cropping Systems (SOC) Multi-Site Ensemble Benchmark
- **Location**: [`examples/benchmarks/salinas_soc_ensemble/`](./examples/benchmarks/salinas_soc_ensemble/)
- Evaluated a 50-member model prediction ensemble across 8 agricultural management systems against observed Soil Organic Carbon (SOC) stock measurements (`white_salinas_2020`).

### 2. AmeriFlux Tower Flux Validation Workflow
- **Location**: [`examples/benchmarks/ameriflux/`](./examples/benchmarks/ameriflux/)
- Benchmarked ecosystem carbon and water flux outputs against high-frequency eddy-covariance tower observations.

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

## Quickstart Code Example

```r
library(PEcAn.benchmark)

# 1. Load observation data with custom variable mapping
obs_df <- load_data(
  file_path = "examples/benchmarks/salinas_soc_ensemble/small_salinas_obs.csv",
  format    = "csv",
  var_map   = list(time = "date", AbvGrndWood = "soc_stock")
)

# 2. Load model prediction ensemble
model_df <- load_data(
  file_path = "examples/benchmarks/salinas_soc_ensemble/small_salinas_ensemble.csv",
  format    = "csv"
)

# 3. Align timesteps with tolerance matching
aligned <- align_by_time(
  model_data = model_df,
  obs_data   = obs_df,
  time_col   = "posix",
  var_col    = "AbvGrndWood",
  tolerance  = 86400 # 1 day tolerance
)

# 4. Calculate metrics
rmse <- metric_RMSE(aligned$model_mean, aligned$obs)
r2   <- metric_R2(aligned$model_mean, aligned$obs)
cov  <- metric_Coverage(aligned$obs, aligned$model_q05, aligned$model_q95)

# 5. Generate automated Quarto scorecard report
generate_validation_report(
  aligned_data = aligned,
  output_file  = "Salinas_SOC_Validation_Report.html",
  title        = "Salinas SOC Ensemble Scorecard"
)
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
