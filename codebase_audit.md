# Audit of Existing Code: PEcAn Benchmarking & SDA Packages

## Executive Summary

As part of Phase 1 of the GSoC 2026 Validation Toolkit project, a comprehensive audit was conducted across the legacy `PEcAn.benchmark` module and the `PEcAn.assim.sequential` (State Data Assimilation, SDA) package. The primary goal was to identify architectural bottlenecks, design anti-patterns, and structural limitations in the existing codebase to inform the design of a modern, decoupled, dataframe-first validation framework.

---

## Audit of the Legacy `PEcAn.benchmark` Package

The legacy benchmarking module was designed around tight integration with **BETYdb** (PEcAn's central PostgreSQL relational database). While this facilitated database-driven workflows, it introduced several severe architectural limitations:

### 1. Hard Database Coupling
Functions like `calc_benchmark()`, `align_data()`, and the original metric calculation routines implicitly assumed inputs were retrieved directly from BETYdb using specific database IDs (`benchmark_id`, `input_id`, `site_id`) and relational schemas. 
- **Consequence**: Users could not benchmark model outputs against offline CSVs, NetCDF files, or external observational datasets without first inserting records into BETYdb.

### 2. Monolithic Alignment & Performance Bottlenecks
The time-alignment logic in legacy `match_timestep.R` and `align_data.R` relied on nested R loops to match observation timestamps with model predictions:
- **Consequence**: Execution was extremely slow on multi-year time series. Furthermore, the absence of explicit matching tolerances caused valid data points to be silently dropped during subtle timestep mismatches (e.g., half-hourly vs. hourly timestamps, or floating-point timestamp drift).

### 3. Hardcoded Metrics & Lack of Extensibility
Statistical validation metrics were hardcoded directly within execution loops rather than abstracted into pure, reusable functions:
- **Consequence**: External scripts or packages (such as SDA workflows) could not reuse metric calculations without triggering full database lookups. Adding new metrics required modifying core execution functions.

---

## Audit of the `PEcAn.assim.sequential` (SDA) Package

The SDA package contained numerous one-off "Magic" validation and data-processing scripts used across various research campaigns (e.g., PalEON, Willow Creek, EFI forecasting). 

### Audited Scripts:
- `modules/assim.sequential/R/SDA_OBS_Assembler.R`
- `modules/assim.sequential/inst/paleon_sda.R`
- `modules/assim.sequential/inst/SDA_runner.R`
- `modules/assim.sequential/R/Helper.functions.R`
- `modules/assim.sequential/inst/WillowCreek/gapfill_WCr.R`

### Key Audit Findings:

#### 1. Redundant Data Intake Logic
Scripts like `SDA_OBS_Assembler.R` and `gapfill_WCr.R` contained highly specific, duplicated code blocks for reading NetCDF and CSV files, handling missing values, gap-filling, and renaming columns into PEcAn standard variable names.

#### 2. Boilerplate Overload
The alignment logic in `Helper.functions.R` and `SDA_runner.R` shared the exact same mathematical intent as `PEcAn.benchmark` alignment functions, but was implemented completely independently using ad-hoc slicing.

#### 3. In-Memory Dataframe Pipelines Needed
SDA workflows naturally generated in-memory `data.frame` or `data.table` structures during sequential filtering steps. SDA needed a lightweight way to pass these structures directly into a validation pipeline without database round-trips.

---

## Solutions Extracted & Applied in PRs

The audit established that the path forward was a **Dataframe-First Architecture**. Recent PRs successfully transformed these audit findings into production capabilities:

```
[ Arbitrary Tabular/NetCDF Data ]
               │
               ▼
   Data Intake API (`load_data`, `data_intake`)
               │
               ▼
   Alignment Pipeline (`align_by_time` / tolerance binary search)
               │
               ▼
   Metric Registry (`metric_Bias`, `metric_RMSE`, `metric_R2`, `metric_PMU`, `metric_Coverage`, `metric_CRPS`)
               │
               ▼
   Visualization & Reporting (`metric_timeseries_plot`, `Validation_report.qmd`)
```

### Architectural Improvements by PR:

1. **Functional Metric Isolation ([PR #4041](https://github.com/PecanProject/pecan/pull/4041))**:
   - Decoupled core statistical metrics into standalone, pure-R functions (`metric_*.R`) that accept standard numeric vectors or dataframes. Sanitized test environments and removed unreliable fallback logic.

2. **Data Intake & Registry Refactor ([PR #4017](https://github.com/PecanProject/pecan/pull/4017))**:
   - Introduced `pecan_metric_registry` for dynamic metric registration, `align_by_time` using fast `findInterval` binary search logic with configurable time tolerances, and dynamic metadata passthrough.

3. **Numeric Metrics & Uncertainty Calibration Coverage ([PR #4032](https://github.com/PecanProject/pecan/pull/4032) / [Issue #4027](https://github.com/PecanProject/pecan/issues/4027))**:
   - Completed unit test coverage for all 9 legacy numeric metrics and expanded the pipeline to support Predictive Model Uncertainty (PMU) and Prediction Interval Coverage metrics by cleanly passing `obs_se`, `obs_n`, and `model_q` quantiles.

4. **Automated Reporting & Visualizations ([PR #4040](https://github.com/PecanProject/pecan/pull/4040) / [Issue #4037](https://github.com/PecanProject/pecan/issues/4037))**:
   - Modernized plotting functions (`metric_timeseries_plot.R`, `metric_residual_plot.R`, `metric_scatter_plot.R`) to accept generic dataframes. Introduced parameterized Quarto scorecard template `Validation_report.qmd`.

5. **Biogeochemistry MVP Integration ([PR #4062](https://github.com/PecanProject/pecan/pull/4062) / [Issue #4059](https://github.com/PecanProject/pecan/issues/4059))**:
   - Built Salinas SOC 50-member ensemble multi-site benchmark evaluating Soil Organic Carbon predictions against empirical field data. Added AmeriFlux tower workflow and `efi_long_to_array()` parser for EFI long format predictions.

6. **Final Documentation & Tutorial Vignette ([PR #4087](https://github.com/PecanProject/pecan/pull/4087) / [PR #4076](https://github.com/PecanProject/pecan/pull/4076))**:
   - Authored `validation_framework_tutorial.Rmd` vignette, expanded `testthat` suite to 16 test files, and fully documented exported functions with Roxygen2 blocks.

---

## Conclusion

By stripping out BETYdb dependencies from core benchmarking routines and unifying data intake with flexible alignment and Quarto reporting, the new Validation Toolkit serves as a universal, high-performance validation framework for any ecosystem modeling workflow in PEcAn.
