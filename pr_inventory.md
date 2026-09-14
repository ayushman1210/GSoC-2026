# Merged Pull Requests & Issue Tracker: GSoC 2026

This inventory provides a complete list of all Pull Requests and Issues authored and merged during the Google Summer of Code (GSoC) 2026 project: **PEcAn Benchmarking and Validation Framework**.

---

## PR Summary Dashboard

| PR / Issue | Title | Phase | Status | Links |
| :--- | :--- | :---: | :---: | :---: |
| **PR #4017** | Refactor the baseline Validation Pipeline to align with GSoC Architecture | Phase 1 & 2 | Merged | [PR #4017](https://github.com/PecanProject/pecan/pull/4017) |
| **PR #4032** | Add unit tests for benchmarking metrics and document metric_R2 | Phase 2 | Merged | [PR #4032](https://github.com/PecanProject/pecan/pull/4032) \| [Issue #4027](https://github.com/PecanProject/pecan/issues/4027) |
| **PR #4040** | Implement automated reporting and visualization layer | Phase 3 | Merged | [PR #4040](https://github.com/PecanProject/pecan/pull/4040) \| [Issue #4037](https://github.com/PecanProject/pecan/issues/4037) |
| **PR #4041** | Add testthat coverage for remaining numeric metrics | Phase 2 | Merged | [PR #4041](https://github.com/PecanProject/pecan/pull/4041) |
| **Issue #4059** | AmeriFlux tower observation benchmarking workflow | Phase 4 | Resolved | [Issue #4059](https://github.com/PecanProject/pecan/issues/4059) |
| **PR #4062** | MVP Integration & Testing (Salinas SOC ensemble benchmark) | Phase 4 | Merged | [PR #4062](https://github.com/PecanProject/pecan/pull/4062) |
| **PR #4087 / #4076** | Benchmarking and validation tutorial vignette & final handoff | Phase 5 | Merged | [PR #4087](https://github.com/PecanProject/pecan/pull/4087) \| [PR #4076](https://github.com/PecanProject/pecan/pull/4076) |

---

## Detailed PR Breakdown

### 1. PR #4017: Baseline Validation Pipeline Refactor
- **Title**: Refactor the baseline Validation Pipeline to align with GSoC Architecture
- **URL**: https://github.com/PecanProject/pecan/pull/4017
- **Phases**: Phase 1 & Phase 2
- **Key Contributions**:
  - Abstracted data loading out of BETYdb queries into generic Data Intake API (`load_data.R`, `data_intake.R`).
  - Implemented `align_by_time` using high-performance `findInterval` binary search logic with user-configurable temporal matching tolerances.
  - Formulated the extensible `pecan_metric_registry` to allow dynamic registration of metrics.
  - Enabled dynamic metadata passthrough across alignment and scoring pipeline stages.

### 2. PR #4041: Standalone Metric Isolation & Test Coverage
- **Title**: Add testthat coverage for remaining numeric metrics
- **URL**: https://github.com/PecanProject/pecan/pull/4041
- **Phase**: Phase 2
- **Key Contributions**:
  - Isolated statistical metrics (`RMSE`, `MAE`, `R2`, `Correlation`, `Bias`) into standalone, testable functions operating purely on numerical inputs and dataframes.
  - Sanitized test execution environments.
  - Removed fragile legacy fallbacks to ensure deterministic CI test behavior.

### 3. PR #4032 & Issue #4027: Numeric Metrics & Calibration Coverage
- **Title**: Add unit tests for benchmarking metrics and document metric_R2
- **URL**: https://github.com/PecanProject/pecan/pull/4032 | https://github.com/PecanProject/pecan/issues/4027
- **Phase**: Phase 2
- **Key Contributions**:
  - Achieved 100% test coverage for all 9 legacy numeric statistical metrics.
  - Extended alignment pipeline to pass uncertainty fields (`obs_se`, `obs_n`, `model_q` quantiles).
  - Added Predictive Model Uncertainty (`metric_PMU`) and Prediction Interval Coverage (`metric_Coverage`) for uncertainty quantification checks.
  - Documented `metric_R2` with comprehensive Roxygen2 headers.

### 4. PR #4040 & Issue #4037: Automated Reporting & Visualization Layer
- **Title**: Implement automated reporting and visualization layer
- **URL**: https://github.com/PecanProject/pecan/pull/4040 | https://github.com/PecanProject/pecan/issues/4037
- **Phase**: Phase 3
- **Key Contributions**:
  - Modernized `ggplot2` plotting suite (`metric_timeseries_plot.R`, `metric_residual_plot.R`, `metric_scatter_plot.R`, `metric_lmDiag_plot.R`).
  - Created parameterized Quarto scorecard template `Validation_report.qmd` in `inst/reports/`.
  - Built `generate_validation_report()` wrapper API to compile data, metrics, and plots into publication-ready HTML/PDF benchmark scorecards.

### 5. PR #4062 & Issue #4059: Biogeochemistry MVP Integration & Initial Benchmarks
- **Title**: MVP Integration & Testing / AmeriFlux tower observation benchmarking
- **URL**: https://github.com/PecanProject/pecan/pull/4062 | https://github.com/PecanProject/pecan/issues/4059
- **Phase**: Phase 4
- **Key Contributions**:
  - Implemented end-to-end multi-site ensemble benchmark for Salinas Organic Cropping Systems (`salinas_soc_ensemble`) evaluating 50-member model predictions across 8 agricultural management systems against observed SOC stocks.
  - Developed AmeriFlux tower observation benchmarking workflow (`examples/benchmarks/ameriflux/`).
  - Added `efi_long_to_array()` parser to convert EFI long-format probabilistic forecasts into matrix format for metric evaluations.

### 6. PR #4087 / PR #4076: Tutorial Vignette & Final Handoff
- **Title**: Benchmarking and validation tutorial vignette & final handoff
- **URL**: https://github.com/PecanProject/pecan/pull/4087 | https://github.com/PecanProject/pecan/pull/4076
- **Phase**: Phase 5
- **Key Contributions**:
  - Authored comprehensive tutorial vignette (`validation_framework_tutorial.Rmd`) demonstrating offline data ingestion, alignment, metric calculation, and Quarto scorecard rendering.
  - Expanded `testthat` suite across 16 test files.
  - Completed Roxygen2 documentation for all exported functions.
