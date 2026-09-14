# GSoC 2026 Workplan: Benchmarking and Validation Framework

## Project Overview & Technical Approach

The primary focus of this Google Summer of Code (GSoC) 2026 project is building a **General-Purpose Validation Toolkit**, pivoting away from refactoring the rigid, BETYdb-coupled legacy `PEcAn.benchmark` module. The toolkit is designed as a flexible, reusable library of validation components utilizing a file-based data registry to rapidly ingest observation data, align time and space, and calculate statistical validation metrics against various model predictions.

---

## Detailed 175-Hour Timeline & Deliverables

| Timeframe | Phase / Tasks | Est. Hours | Status |
| :--- | :--- | :---: | :---: |
| **May 1 – May 25** | **Community Bonding & Planning**<br>• Pivot proposal to concrete workplan.<br>• Review MAGiC calibration/validation scripts.<br>• Finalize dataset selection priority (AmeriFlux vs. Meta-datasets).<br>• Draft mockups of Quarto reporting templates. | 15 hrs | Completed |
| **May 26 – June 9** | **Phase 1: Architecture & Design Pattern Extraction**<br>• Audit legacy code vs. one-off SDA/Magic scripts.<br>• Design the abstract Data Intake API.<br>• Establish generalized design patterns. | 25 hrs | Completed |
| **June 10 – June 24** | **Phase 2: Core Toolkit Implementation**<br>• Write pure-R data loading functions & YAML mapping logic.<br>• Wrap `align_data.R` and metric modules.<br>• Ensure complete BETYdb decoupling.<br>• Add `testthat` coverage for all lifted functions. | 35 hrs | Completed |
| **June 25 – July 8** | **Phase 3: Automated Reporting & Visualization**<br>• Develop `Validation_report.qmd` Quarto scorecard template.<br>• Implement `ggplot2` visualization functions.<br>• Create automated report generation API (`generate_validation_report()`). | 30 hrs | Completed |
| **July** | **Midterm Evaluation**<br>• Midterm code review and feedback integration. | 5 hrs | Passed |
| **July 9 – August 10** | **Phase 4: Biogeochemistry MVP Integration & Testing**<br>• Run end-to-end pipeline on AmeriFlux and Salinas SOC benchmarks.<br>• Refine mapping and data intake based on real-world dataset friction.<br>• Implement `efi_long_to_array()` for EFI long format matrices.<br>• Write end-to-end integration test suite. | 45 hrs | Completed |
| **August 10 – August 25** | **Phase 5: Finalizing, Documentation & Submission**<br>• Complete `roxygen2` documentation for all exported functions.<br>• Author comprehensive tutorial vignette (`validation_framework_tutorial.Rmd`).<br>• Expand `testthat` suite to 16 test files.<br>• Prepare final PRs and GSoC submission dashboard package. | 20 hrs | Completed |

---

## Detailed Phase Breakdown

### Phase 1: Architecture & Design Pattern Extraction
- **Audited Scripts**:
  - `modules/assim.sequential/R/SDA_OBS_Assembler.R`
  - `modules/assim.sequential/inst/paleon_sda.R`
  - `modules/assim.sequential/inst/SDA_runner.R`
  - `modules/assim.sequential/R/Helper.functions.R`
  - `modules/assim.sequential/inst/WillowCreek/gapfill_WCr.R`
- **Extracted Patterns**: Identified boilerplate data ingestion, temporal downsampling/upsampling, and spatial aggregation steps across legacy code.
- **Deliverable**: Abstract Data Intake API design and standardized pipeline architecture decoupled from BETYdb database schema assumptions.

### Phase 2: Core Toolkit Implementation
- **Data Intake & Parsing Abstraction**: Implemented pure R functions (`load_csv`, `load_netcdf`, `load_rds`, `load_tab`, `load_data`) that flexibly read arbitrary tabular/gridded data and map column names natively into PEcAn standard variables via configuration lists.
- **Alignment & Metric Isolation**: Extracted metrics into standalone, pure-R functions (`metric_Bias`, `metric_RMSE`, `metric_R2`, `metric_cor`, `metric_Coverage`, `metric_PMU`, `metric_CRPS`, etc.). Introduced `align_by_time` with configurable tolerances using fast binary search (`findInterval`).
- **Test Coverage Lift**: Created unit tests covering 100% of lifted loader, alignment, and metric functions (which previously had zero test coverage).
- **Deliverable**: Decoupled `PEcAn.benchmark` R package components for data loading, alignment, and metric calculation.

### Phase 3: Automated Reporting & Visualization
- **Standardized Visualizations**: Built modernized `ggplot2` plotting routines:
  - `metric_timeseries_plot()`: Time-series comparisons with prediction intervals.
  - `metric_scatter_plot()`: Observed vs. modeled scatter plots with regression lines.
  - `metric_residual_plot()`: Residual analysis over time and model values.
  - `metric_lmDiag_plot()`: Linear model diagnostics.
- **Automated Quarto Reports**: Created `Validation_report.qmd` Quarto template that dynamically compiles computed metrics and plots into interactive HTML scorecards (ILAMB-style).
- **Deliverable**: `Validation_report.qmd` template and `generate_validation_report()` API wrapper.

### Phase 4: Biogeochemistry MVP Integration & Initial Benchmarks
- **Salinas SOC Ensemble Benchmark**: Constructed an end-to-end multi-site ensemble benchmark (`examples/benchmarks/salinas_soc_ensemble/`) evaluating 50-member model predictions across 8 agricultural management systems against observed Soil Organic Carbon (SOC) stocks (`white_salinas_2020`).
- **AmeriFlux Tower Benchmark**: Built tower observation benchmarking workflow (`examples/benchmarks/ameriflux/`) evaluating carbon and water flux predictions against AmeriFlux eddy-covariance observations.
- **EFI Matrix Formatting**: Developed `efi_long_to_array()` to parse EFI long-format probabilistic predictions into multidimensional matrices for CRPS and coverage evaluations.
- **Deliverable**: Functional, real-world benchmark examples and data ingestion helpers.

### Phase 5: Testing, Documentation & Final Handoff
- **Unit Testing**: Expanded `testthat` test suite across 16 test files covering loaders, alignment, metrics, reporting, and vignette execution.
- **Roxygen2 Documentation**: Documented all exported functions with standardized Roxygen headers, parameters, return value descriptions, and examples.
- **Tutorial Vignette**: Authored `validation_framework_tutorial.Rmd` demonstrating offline data intake, alignment, scorecard generation, and report rendering using reproducible fixtures (`small_salinas_obs.csv` and `small_salinas_ensemble.csv`).
- **Deliverable**: Complete documentation, tutorial vignette, and PR submissions.
