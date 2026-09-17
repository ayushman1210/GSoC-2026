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

### 1. Ingest Data & Reshape Ensemble Matrix
Load synthetic simulation or real-world fixtures located in package `extdata`:

```r
library(PEcAn.benchmark)

# Locate built-in simulation fixtures in package extdata
extdata_dir <- system.file("extdata", package = "PEcAn.benchmark")
if (extdata_dir == "" || !file.exists(file.path(extdata_dir, "simulated_model_ensemble.csv"))) {
  extdata_dir <- "modules/benchmark/inst/extdata"
}

model_csv <- file.path(extdata_dir, "simulated_model_ensemble.csv")
obs_csv   <- file.path(extdata_dir, "simulated_observations.csv")

raw_model <- read.csv(model_csv)
raw_obs   <- read.csv(obs_csv)

# Extract ensemble member columns into a (time x member) matrix
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
```

### 2. Align Timesteps & Space
Align observational data and model predictions using binary search matching (`findInterval`) with configurable temporal tolerance:

```r
aligned_df <- align_by_time(model_summary, obs_summary, tolerance_secs = 1800)
aligned_indices <- match(aligned_df$time, model_summary$time)
attr(aligned_df, "ensemble_matrix") <- ens_mat[aligned_indices, , drop = FALSE]
```

### 3. Compute Statistical & UQ Validation Metrics
Compute statistical skill and UQ metrics on the aligned dataframe using `compute_metrics()`:

```r
metrics_results <- compute_metrics(
  aligned_df,
  metrics = c("BIAS", "RMSE", "MAE", "R2", "COVERAGE", "CRPS")
)

print(metrics_results)
```

### 4. Render Plots
Generate clean `ggplot2` time-series and diagnostic scatter visualizations:

```r
# Plot time-series with prediction ribbon & ensemble trajectories
p_ts <- metric_timeseries_plot(aligned_df, var = "NEE (umol m-2 s-1)")

# Plot observed vs modeled scatter
p_sc <- metric_scatter_plot(aligned_df, var = "NEE")
```

### 5. Generate Automated Quarto Report
Compile all results into an interactive HTML scorecard:

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
