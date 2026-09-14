# Tutorial Guide & Quickstart: PEcAn Validation Framework

This document outlines the workflow demonstrated in the newly authored GSoC 2026 tutorial vignette: [`validation_framework_tutorial.Rmd`](./vignettes/validation_framework_tutorial.Rmd). 

The vignette provides a step-by-step walkthrough for users looking to perform offline benchmarking of model outputs against observational datasets without needing active database connections to BETYdb.

---

## Reproducible Fixtures

The tutorial includes lightweight, self-contained CSV fixtures located in [`examples/benchmarks/salinas_soc_ensemble/`](./examples/benchmarks/salinas_soc_ensemble/):
- **`small_salinas_obs.csv`**: Sample observed Soil Organic Carbon (SOC) measurements across agricultural fields.
- **`small_salinas_ensemble.csv`**: Sample 50-member model ensemble predictions.

---

## Step-by-Step Pipeline Walkthrough

### 1. Ingest Data
Use `load_data()` or `load_csv()` with native variable mapping to load observation data:

```r
library(PEcAn.benchmark)

# Load observed data with custom column mapping
obs_df <- load_data(
  file_path = "examples/benchmarks/salinas_soc_ensemble/small_salinas_obs.csv",
  format = "csv",
  var_map = list(time = "date", AbvGrndWood = "soc_stock")
)

# Load model ensemble output
model_df <- load_data(
  file_path = "examples/benchmarks/salinas_soc_ensemble/small_salinas_ensemble.csv",
  format = "csv"
)
```

### 2. Align Timesteps & Space
Align observational data and model predictions using binary search matching with configurable tolerance:

```r
aligned_df <- align_by_time(
  model_data = model_df,
  obs_data   = obs_df,
  time_col   = "posix",
  var_col    = "AbvGrndWood",
  tolerance  = 86400 # 1 day tolerance in seconds
)
```

### 3. Compute Statistical & UQ Validation Metrics
Compute statistical skill metrics independently or register custom metrics with `pecan_metric_registry`:

```r
# Compute standard numeric metrics
rmse_val <- metric_RMSE(aligned_df$model_mean, aligned_df$obs)
r2_val   <- metric_R2(aligned_df$model_mean, aligned_df$obs)
bias_val <- metric_Bias(aligned_df$model_mean, aligned_df$obs)

# Compute UQ calibration metrics (PMU & Coverage)
coverage <- metric_Coverage(
  obs       = aligned_df$obs,
  model_q05 = aligned_df$model_q05,
  model_q95 = aligned_df$model_q95
)

message(sprintf("RMSE: %.2f | R2: %.2f | Coverage: %.1f%%", rmse_val, r2_val, coverage * 100))
```

### 4. Render Plots
Generate clean `ggplot2` time-series and scatter visualizations:

```r
# Plot time-series with prediction interval
p1 <- metric_timeseries_plot(
  df        = aligned_df,
  time_col  = "posix",
  var_name  = "AbvGrndWood",
  title     = "Salinas SOC Stock Validation"
)

# Plot observed vs modeled scatter
p2 <- metric_scatter_plot(
  df       = aligned_df,
  obs_col  = "obs",
  pred_col = "model_mean"
)
```

### 5. Generate Automated Quarto Report
Compile all results into an interactive HTML scorecard using `generate_validation_report()`:

```r
generate_validation_report(
  aligned_data = aligned_df,
  output_file  = "Salinas_SOC_Validation_Report.html",
  title        = "Salinas SOC Benchmark Scorecard"
)
```

---

## Verifying the Test Suite

All functions in the toolkit are tested via `testthat`. To execute the entire suite of 16 test files:

```r
testthat::test_dir("tests")
```
