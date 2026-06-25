# SVR-ARIMA

Standalone ARIMA forecasting implementation for the Sentiment–Volatility Ratio, developed from the data science capstone project:

**Predictive Models for the Diagnostic Ratio of Consumer Sentiment and Volatility**

This repository contains the ARIMA modeling workflow only. The broader capstone compared AR, ARIMA, MLP, and LSTM models, but this repository preserves and documents the ARIMA implementation separately for review, reproducibility, and future cleanup.

## Project Overview

The project models a diagnostic ratio constructed from two macro-financial indicators:

* **UMCSENT**: University of Michigan Consumer Sentiment Index
* **VIXCLS**: CBOE Volatility Index

The modeled series is the log-transformed Sentiment–Volatility Ratio:

```text
log(SVR) = log(UMCSENT) - log(monthly mean VIXCLS)
```

The goal is to evaluate manually specified ARIMA models for forecasting the transformed ratio over held-out test observations.

## Data Sources

The script retrieves data from FRED using `pipewelder::get_fred()`.

The source series are:

* `UMCSENT`, monthly consumer sentiment
* `VIXCLS`, daily VIX close values

Daily VIX observations are aggregated to monthly averages before the ratio is constructed.

## Modeling Window

The intended modeling window is:

```text
January 1990 through December 2025
```

The workflow uses:

* 432 monthly observations
* 348 training observations
* 84 held-out test observations

The split is chronological. No random train/test split is used.

## Forecasting Design

The ARIMA workflow evaluates forecasts for:

* `h = 1`, one month ahead
* `h = 3`, three months ahead

The script uses a leakage-safe expanding-window rolling forecast setup.

For each target observation in the test period:

1. the forecast target is identified;
2. the forecast origin is set to `target - h`;
3. the model is fit only on observations available through the forecast origin;
4. an `h`-step forecast is generated;
5. the `h`-th forecast value is aligned with the target observation;
6. the residual is calculated as actual minus forecast.

This means the model is repeatedly refit as the forecast origin advances through the test period.

## ARIMA Grid

The script evaluates manually specified ARIMA models using the `forecast` package.

Candidate values are:

```text
p = 1, 2, 3
d = 0, 1
q = 1, 2, 3
```

This produces 18 ARIMA specifications.

Models with `d = 0` are estimated with a mean term.

Models with `d = 1` are estimated without a mean term.

Model identifiers use the compact format:

```text
ARIMApdq
```

For example:

```text
ARIMA103 = ARIMA(1, 0, 3)
ARIMA213 = ARIMA(2, 1, 3)
```

## Evaluation Metrics

Forecast accuracy is evaluated on the held-out test period using:

* MSE, mean squared error
* RMSE, root mean squared error
* MAE, mean absolute error

MAE is the primary metric used for comparing model performance.

Metrics are calculated on the log-transformed SVR scale.

## Output

The script produces a wide-format metrics table with one row per model and forecast horizon.

The final table includes:

* `model_id`
* `horizon`
* `test_mse`
* `test_rmse`
* `test_mae`

The script also exports the results to:

```text
ARIMA Metrics FINAL.csv
```

## Repository Scope

This repository is intentionally limited to the ARIMA workflow.

It does not include:

* AR model implementation
* MLP model implementation
* LSTM model implementation
* Shiny dashboard code
* deployment configuration
* website integration
* automated model refresh
* production forecasting services

The current goal is to preserve, document, and validate the original ARIMA research workflow before making larger cleanup or refactoring decisions.

## Reproducibility Notes

Exact numeric reproduction may depend on:

* current FRED data availability
* source data revisions
* R version
* package versions
* `forecast::Arima()` behavior
* numerical optimization behavior

The purpose of this repository is to preserve the ARIMA algorithm and evaluation design clearly. If results differ from historical capstone tables, those differences should be documented rather than hidden.

## Main Dependencies

The script uses:

```r
library(pipewelder)
library(tidyverse)
library(lubridate)
library(forecast)
```

## How to Run

Open the project in RStudio or another R environment and run the ARIMA script.

The script performs the full workflow:

1. retrieve raw FRED data;
2. aggregate daily VIX data to monthly averages;
3. construct the log-transformed Sentiment–Volatility Ratio;
4. create the chronological train/test split;
5. define the ARIMA parameter grid;
6. generate rolling forecasts for each model and horizon;
7. calculate forecast accuracy metrics;
8. reshape the results table;
9. export the final CSV.

## License

See the repository license for usage terms.
