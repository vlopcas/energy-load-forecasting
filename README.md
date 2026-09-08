# Brazilian Energy Load Forecasting

A production-oriented machine learning system for short-term electrical load forecasting in the Brazilian power grid using public ONS data.

## Status

> Planned — development has not started yet.

## Overview

Electrical load forecasting is a time-dependent machine learning problem in which predictions must be generated using only information available at the forecast origin.

This project aims to build a production-oriented forecasting system around public data from the Brazilian National Electric System Operator (ONS), covering the machine learning lifecycle from historical backtesting and reproducible training to batch inference, monitoring, and retraining.

The initial problem will focus on short-term hourly load forecasting by subsystem, with a forecast horizon of up to 24 hours.

## Initial Data Source

The initial source will be public operational data published by **ONS — Operador Nacional do Sistema Elétrico**.

The first version will primarily use historical hourly electrical load data.

Additional ONS datasets and external sources may be introduced later when they provide measurable value to the forecasting system.

## Initial Scope

The first versions will focus on:

- Historical hourly load ingestion
- Temporal data validation
- Seasonal forecasting baselines
- Time-aware feature engineering
- Rolling and walk-forward backtesting
- Reproducible model training
- Experiment tracking
- Forecast generation
- Prediction and ground-truth storage
- Model performance monitoring

The initial forecasting target is:

```text
Electrical load by subsystem
Frequency: hourly
Forecast horizon: 1–24 hours ahead
```

## Engineering Goals

The project is designed to explore production-oriented Machine Learning Engineering and MLOps practices, including:

- Time-series forecasting
- Temporal validation
- Leakage prevention
- Feature pipelines
- Reproducible training
- Experiment tracking
- Model versioning
- Model registry
- Batch inference
- Prediction monitoring
- Model performance monitoring
- Data drift and regime changes
- Champion/challenger evaluation
- Retraining strategies
- CI/CD for machine learning systems

## Forecasting Strategy

Model complexity will be introduced incrementally.

The project will begin with simple and defensible baselines:

```text
Naive
  ↓
Daily Seasonal Naive
  ↓
Weekly Seasonal Naive
  ↓
Statistical / Linear Models
  ↓
Tree-Based Models
  ↓
Specialized Forecasting Models
  ↓
Deep Learning — only if justified
```

More complex models will only be adopted when they demonstrate meaningful improvements under proper temporal backtesting.

## System Architecture

The system will evolve toward the following lifecycle:

```text
ONS Data
   ↓
Ingestion
   ↓
Raw Historical Data
   ↓
Temporal Validation
   ↓
Feature Pipeline
   ├───────────────┐
   ↓               ↓
Training       Batch Inference
   ↓               ↓
Experiments      Forecasts
   ↓               │
Model Registry     │
   └───────┬───────┘
           ↓
    Ground Truth Join
           ↓
       Evaluation
           ↓
       Monitoring
           ↓
    Retraining Policy
```

The architecture will remain intentionally simple in the early versions and evolve according to measured requirements.

## Temporal Correctness

A central requirement of the project is preventing temporal leakage.

Every feature used for a prediction must have been available at the prediction time.

The system will explicitly distinguish concepts such as:

```text
event_time
prediction_time
target_time
forecast_horizon
ingested_at
```

Point-in-time correctness will be treated as part of the model contract rather than only as a preprocessing concern.

## Evaluation

Models will be evaluated using time-aware backtesting rather than randomly shuffled train/test splits.

Candidate metrics include:

- MAE
- RMSE
- WAPE
- Forecast bias

Performance will also be analyzed across dimensions such as:

- Forecast horizon
- Subsystem
- Hour of day
- Day of week
- Time period

## Model Lifecycle

The project is expected to evolve toward a champion/challenger workflow:

```text
Production Model
      ↓
New Observations
      ↓
Candidate Training
      ↓
Temporal Backtesting
      ↓
Evaluation Gate
      ↓
Candidate Better?
   ┌──────┴──────┐
   No            Yes
   ↓              ↓
Reject         Promote
```

Automatic model promotion will not be assumed initially.

## Monitoring

Monitoring will eventually cover three areas.

### Data

- Freshness
- Missing timestamps
- Missing values
- Unexpected ranges
- Schema changes
- Distribution changes

### Predictions

- Forecast generation failures
- Prediction latency
- Forecast coverage
- Model version
- Forecast horizon

### Model Performance

Once observed load becomes available:

- Forecast vs. actual
- MAE
- RMSE
- WAPE
- Bias
- Performance degradation over time

## Potential Evolution

Future versions may explore:

- Semi-hourly forecasting
- ONS verified load data
- ONS programmed load as an operational benchmark
- Weather forecast features
- Calendar and holiday effects
- Generation context
- Drift detection
- Automated retraining
- Model registry and promotion workflows
- Forecast serving APIs

These capabilities will be introduced only when justified by the problem and supported by point-in-time-correct data.

## Potential Technology Stack

The initial implementation is expected to explore technologies such as:

- Python
- Parquet
- DuckDB or Polars
- scikit-learn
- LightGBM or XGBoost
- MLflow
- Docker

Orchestration, monitoring infrastructure, model serving, and additional technologies will be selected as the system evolves.

The stack is intentionally not fixed.

## Development Strategy

Development will follow incremental vertical slices:

```text
Research Baseline
      ↓
Temporal Backtesting
      ↓
Reproducible ML Pipeline
      ↓
Experiment Tracking
      ↓
Batch Forecasting
      ↓
Ground Truth Evaluation
      ↓
Monitoring
      ↓
Model Registry
      ↓
Retraining
      ↓
Semi-Hourly Forecasting
      ↓
Exogenous Features
```

The objective is not to maximize the number of MLOps tools used, but to build a reliable forecasting system whose infrastructure is justified by real machine learning lifecycle requirements.

## Roadmap

A detailed technical roadmap will guide the implementation as development begins.

## License

This project is licensed under the MIT License.
