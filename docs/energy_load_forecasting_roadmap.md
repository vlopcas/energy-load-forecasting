# Energy Load Forecasting — Complete Technical Roadmap

> Production-oriented short-term electrical load forecasting system for the Brazilian power grid using public ONS data.

## 1. Project Purpose

This project is the **ML Engineering / MLOps** project of the portfolio.

The objective is not simply to train a forecasting model. The objective is to design, build, evaluate, deploy, observe, and evolve a machine learning system whose predictions are generated repeatedly as new data arrives.

The system should eventually support the lifecycle:

```text
Historical Data
      ↓
Ingestion
      ↓
Validation
      ↓
Feature Engineering
      ↓
Temporal Backtesting
      ↓
Training
      ↓
Experiment Tracking
      ↓
Model Evaluation
      ↓
Model Registry
      ↓
Batch Inference
      ↓
Prediction Storage
      ↓
Ground Truth Arrival
      ↓
Performance Monitoring
      ↓
Retraining Decision
```

The project should remain incremental. Infrastructure and model complexity are introduced only when an actual requirement justifies them.

---

## 2. Problem Definition

### 2.1 Initial forecasting problem

Build a system that predicts short-term electrical load for Brazilian power-system subsystems using historical public ONS data.

Initial target:

```text
target:
electrical load

frequency:
hourly

forecast horizon:
1–24 hours ahead

forecast unit:
subsystem × timestamp
```

Conceptually:

```text
observations available at time t
            ↓
        ML system
            ↓
ŷ(t+1), ŷ(t+2), ... ŷ(t+24)
```

### 2.2 Initial business/operational framing

Short-term load forecasting is useful for planning and operating electrical systems because expected demand influences operational decisions.

This repository does **not** attempt to reproduce or replace ONS operational forecasting.

It is an engineering and research system built from public data.

Any comparison with ONS-published programmed load should be treated as an analytical benchmark, not as evidence that the project reproduces the information set, constraints, methodology, or operational responsibilities of ONS.

### 2.3 Core engineering question

The central question is:

> How do we operate a forecasting model reliably over time while preserving temporal correctness, reproducibility, traceability, measurable model quality, and controlled model evolution?

---

## 3. Primary Data Source

Initial source:

**ONS — Operador Nacional do Sistema Elétrico**

Primary dataset:

**Curva de Carga Horária**

The discovery phase identified historical hourly load data spanning approximately 2000–2026, with current data continuing to be published.

Before implementation, verify the current:

- dataset schema;
- column definitions;
- subsystem identifiers;
- timestamp semantics;
- timezone;
- publication cadence;
- revision behavior;
- historical coverage;
- file formats;
- download endpoints.

Do not hard-code assumptions from exploratory discovery into production code without verifying the source metadata.

---

## 4. Future Data Sources

Potential later sources include:

### ONS Carga de Energia Verificada

Potential use:

- semi-hourly observations;
- operational evolution of the project;
- richer load decomposition.

### ONS Carga de Energia Programada

Potential use:

- analytical operational benchmark;
- comparison between project forecasts, programmed load, and verified load.

### ONS Balanço de Energia nos Subsistemas

Potential use:

- generation context;
- exploratory exogenous variables.

### Calendar information

Potential features:

- hour;
- weekday;
- weekend;
- month;
- holiday;
- special calendar events.

### Weather

Potential future variables:

- temperature;
- humidity;
- apparent temperature;
- weather forecasts.

Weather must not be introduced naively.

For a production-correct forecast, future weather features must correspond to information that was actually available at the prediction origin.

Observed future weather is not a valid production feature for historical simulation.

---

## 5. Non-Goals

The project should explicitly avoid several traps.

### Not a Kaggle-style modeling repository

The objective is not:

```text
notebook
→ train model
→ print RMSE
→ finish
```

### Not an infrastructure showcase

Do not add technologies merely to increase the stack.

Initially avoid introducing without evidence:

- Kubernetes;
- Kafka;
- Spark;
- feature stores;
- service meshes;
- distributed training;
- complex microservices.

### Not an ONS replacement

Do not claim operational equivalence with ONS forecasting.

### Not a deep-learning benchmark

Neural forecasting models are optional challengers, not predetermined architecture.

### Not a real-time streaming system initially

The source and initial product requirements are compatible with batch processing.

---

# Part I — Foundations

## 6. Repository Strategy

Repository:

```text
energy-load-forecasting
```

Recommended initial structure:

```text
energy-load-forecasting/
├── AGENTS.md
├── README.md
├── LICENSE
├── pyproject.toml
├── .gitignore
├── configs/
├── data/
│   ├── raw/
│   ├── interim/
│   └── processed/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   ├── experiments/
│   └── study/
├── notebooks/
├── src/
│   └── energy_load_forecasting/
│       ├── ingestion/
│       ├── validation/
│       ├── features/
│       ├── evaluation/
│       ├── training/
│       ├── inference/
│       ├── monitoring/
│       └── common/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── regression/
└── artifacts/
```

This is a target structure, not a requirement to create every directory on day one.

Create modules only when corresponding functionality exists.

---

## 7. Development Principles

### 7.1 Temporal correctness first

Every transformation must respect what was known at prediction time.

### 7.2 Baseline before complexity

A complex model has no value if it cannot reliably beat a defensible seasonal baseline.

### 7.3 Reproducibility

A training run should eventually be reproducible from:

```text
code version
+
configuration
+
data version/snapshot
+
feature definition
+
training window
+
random state
```

### 7.4 Explicit contracts

Define contracts for:

- source data;
- normalized observations;
- features;
- forecasts;
- model artifacts;
- metrics.

### 7.5 Batch first

Use batch ingestion and inference unless latency requirements prove otherwise.

### 7.6 Measured evolution

Every architectural addition should answer:

> Which measured limitation or requirement does this solve?

---

# Part II — M0: Source Discovery and Data Contract

## 8. Milestone M0 — Verify the Source

Before modeling, inspect the official source directly.

Tasks:

- download representative historical files;
- inspect current-year data;
- compare multiple years;
- inspect schema changes;
- identify subsystem identifiers;
- determine timestamp timezone;
- check duplicated timestamps;
- inspect missing intervals;
- inspect daylight-saving implications historically;
- identify units;
- identify revision behavior;
- inspect publication metadata.

Deliverable:

```text
docs/data-source.md
```

Document:

- source URL;
- dataset name;
- format;
- columns;
- grain;
- keys;
- units;
- timezone;
- update frequency;
- known limitations;
- revision behavior.

---

## 9. Define the Canonical Observation Contract

A normalized observation could conceptually look like:

```text
subsystem
event_time
load_mw
source
source_file
ingested_at
source_updated_at
```

Exact fields should follow verified source semantics.

Candidate logical key:

```text
(subsystem, event_time)
```

Validate whether that key is actually unique.

---

## 10. Timestamp Semantics

This project must explicitly document:

```text
event_time
```

Time represented by the measurement.

```text
ingested_at
```

When the project retrieved the observation.

```text
source_updated_at
```

When available, when the source says the resource was updated.

Later:

```text
prediction_time
```

When a forecast was generated.

```text
target_time
```

Time being forecast.

```text
horizon
```

Difference between prediction and target time.

These fields must not be conflated.

---

## 11. Data Quality Checks

Initial checks:

- required columns exist;
- timestamps parse correctly;
- load is numeric;
- expected subsystem values;
- duplicate logical keys;
- missing timestamps;
- unexpected time gaps;
- impossible/null values;
- suspicious negative load;
- unexpected schema changes.

Do not automatically delete anomalous values.

Validation should distinguish:

```text
invalid
suspicious
valid
```

when appropriate.

---

# Part III — M1: Historical Ingestion

## 12. Milestone M1 — Build Historical Ingestion

Goal:

Create a reproducible process for retrieving historical hourly load.

Expected interface conceptually:

```bash
python -m energy_load_forecasting.ingestion \
  --start-year 2020 \
  --end-year 2025
```

Exact CLI design can evolve.

Requirements:

- deterministic destination paths;
- retries;
- timeout handling;
- HTTP error handling;
- source metadata capture;
- checksum/hash where useful;
- no unnecessary re-download;
- explicit overwrite behavior.

---

## 13. Raw Layer

Raw data should preserve source fidelity.

Example:

```text
data/raw/ons/hourly_load/
├── year=2023/
├── year=2024/
├── year=2025/
└── year=2026/
```

Prefer source files or a minimally transformed representation.

Do not mix normalized features into raw storage.

---

## 14. Idempotency

Running ingestion twice should not silently produce duplicated logical data.

Test:

```text
run ingestion
↓
run same ingestion again
↓
same resulting logical dataset
```

---

## 15. Source Revisions

Because public operational datasets may be revised, do not assume:

```text
same URL = same bytes forever
```

Consider storing:

```text
source identifier
retrieval timestamp
checksum
content length
source modification metadata
```

Initially, logging changed checksums may be sufficient.

Later, historical snapshots may be justified.

---

# Part IV — M2: Canonical Dataset

## 16. Milestone M2 — Normalize Historical Data

Transform raw source data into a canonical analytical dataset.

Preferred storage initially:

```text
Parquet
```

Potential processing engines:

```text
Polars
DuckDB
Pandas
```

Choose based on ergonomics and measured performance.

---

## 17. Canonical Grain

Target:

```text
one row
=
one subsystem
×
one hourly event timestamp
```

Candidate representation:

```text
event_time
subsystem
load_mw
source_year
ingested_at
```

---

## 18. Partitioning

Potential partition:

```text
year
```

Avoid excessive tiny partitions.

Measure before introducing complex partition strategies.

---

## 19. Data Validation Report

Generate a reproducible report containing:

```text
row count
date range
subsystems
missing timestamps
duplicates
null counts
basic distributions
unexpected values
```

Store results as machine-readable artifacts when possible.

---

# Part V — M3: Exploratory Temporal Analysis

## 20. Milestone M3 — Understand the Series

EDA should answer questions relevant to modeling.

Analyze:

- long-term trend;
- intraday seasonality;
- weekday/weekend behavior;
- weekly seasonality;
- monthly/annual effects;
- subsystem differences;
- missing periods;
- structural breaks;
- unusual periods;
- post-2021/2023 behavior where source methodology changed.

---

## 21. Avoid Notebook-Only Knowledge

Notebooks are acceptable for research.

Important conclusions should migrate into:

```text
docs/
```

or tested production code.

Do not let critical feature logic live only in notebooks.

---

# Part VI — M4: Forecasting Baselines

## 22. Milestone M4 — Build Baselines Before ML

Implement at least:

### Persistence baseline

```text
ŷ(t+h) = y(t)
```

where meaningful.

### Daily seasonal naive

```text
ŷ(t) = y(t - 24h)
```

### Weekly seasonal naive

```text
ŷ(t) = y(t - 168h)
```

Potential combinations can be investigated later.

---

## 23. Why Baselines Matter

Every candidate model must answer:

> Does this model provide meaningful improvement over a cheap, stable, interpretable forecast?

The production champion can legitimately remain a simple baseline if complex models fail to improve sufficiently.

---

# Part VII — M5: Temporal Evaluation Framework

## 24. Milestone M5 — Build Backtesting Infrastructure

This is one of the most important milestones.

Never use random shuffled train/test splits for the primary evaluation.

Implement rolling-origin or walk-forward evaluation.

Concept:

```text
Fold 1
TRAIN ───────────────► TEST

Fold 2
   TRAIN ───────────────► TEST

Fold 3
      TRAIN ───────────────► TEST
```

---

## 25. Forecast Origin

Each evaluation record should explicitly know:

```text
prediction_time
target_time
horizon
```

Example:

```text
prediction_time = 2026-08-01 00:00
target_time     = 2026-08-01 08:00
horizon         = 8h
```

---

## 26. Evaluation Dataset

Store forecast evaluation at granular level:

```text
prediction_time
target_time
subsystem
horizon
model_id
prediction
actual
error
absolute_error
squared_error
```

This enables aggregation later without losing information.

---

## 27. Core Metrics

Initial metrics:

### MAE

Easy to interpret in the target unit.

### RMSE

More sensitive to large errors.

### WAPE

Useful aggregate relative error measure.

### Bias

Detect systematic over/underprediction.

Be cautious with MAPE around small denominators.

---

## 28. Metric Slices

Evaluate by:

```text
model
subsystem
horizon
hour_of_day
weekday
month
evaluation window
```

A single aggregate metric is insufficient.

---

## 29. Backtesting Performance

Backtesting can become computationally expensive.

Before distributing computation:

- cache reusable features;
- vectorize metrics;
- avoid retraining unnecessarily;
- profile bottlenecks;
- use columnar formats;
- parallelize locally where justified.

Only adopt distributed processing after measurement.

---

# Part VIII — M6: Point-in-Time-Correct Features

## 30. Milestone M6 — Feature Pipeline

Initial feature families:

### Calendar

```text
target hour
weekday
weekend
month
day of year
```

Potential cyclic encoding:

```text
sin(hour)
cos(hour)
```

when useful.

### Lag features

Examples:

```text
lag_1h
lag_2h
lag_24h
lag_48h
lag_168h
```

### Rolling features

Examples:

```text
rolling_mean
rolling_std
rolling_min
rolling_max
```

over historical windows.

---

## 31. Leakage Rule

For a forecast generated at `prediction_time = t`:

> No feature may depend on information that became available after t.

This must be tested.

---

## 32. Important Multi-Horizon Trap

Suppose at 10:00 we predict 11:00 through 10:00 tomorrow.

For the +24h forecast, a naive dataframe implementation can accidentally use observations from 11:00, 12:00, etc. on the current day.

Those observations were not known at 10:00.

Feature generation must be conditioned on the **forecast origin**, not merely on the target row.

---

## 33. Feature Contract

Eventually record metadata such as:

```text
feature_name
definition
lookback
availability_rule
dtype
version
```

A full feature store is not necessary initially.

---

## 34. Feature Tests

Tests should verify:

- no future timestamps used;
- rolling windows are shifted correctly;
- lags respect subsystem boundaries;
- features are deterministic;
- expected null warm-up periods;
- target is never included as a feature.

---

# Part IX — M7: First ML Models

## 35. Milestone M7 — Train Simple ML Models

Start with interpretable or efficient candidates.

Potential progression:

```text
linear models
↓
LightGBM / XGBoost
↓
other specialized models
```

Do not begin with transformers or large neural forecasting architectures.

---

## 36. Direct vs Recursive Multi-Horizon Forecasting

Investigate strategies.

### Recursive

Predict one step and feed predictions forward.

Pros:

- one model;
- simple concept.

Cons:

- error propagation.

### Direct

Separate horizon-specific predictions/models.

Concept:

```text
model_h1
model_h2
...
model_h24
```

Pros:

- each horizon optimized independently.

Cons:

- operational/model-management complexity.

### Multi-output

One model/system predicts all horizons.

Evaluate rather than assume.

---

## 37. Global vs Per-Subsystem Models

Compare:

### One model per subsystem

```text
South → model A
Southeast/Central-West → model B
...
```

### Global model

```text
all subsystems
+
subsystem feature
→ one model
```

A global model can share patterns, but must demonstrate value.

---

# Part X — M8: Reproducible Training

## 38. Milestone M8 — Productionize Training

Move training logic from exploratory notebooks into reusable modules.

Conceptual command:

```bash
python -m energy_load_forecasting.training --config configs/train.yaml
```

A run should record:

```text
training window
validation window
feature version
model type
hyperparameters
data fingerprint
code revision
metrics
artifact location
```

---

## 39. Configuration

Separate configuration from code where it improves reproducibility.

Potential configuration:

```yaml
data:
  start_date: ...
  end_date: ...

forecast:
  horizon: 24

features:
  lags: [...]
  rolling_windows: [...]

model:
  type: lightgbm
  params: ...
```

Do not create configuration abstractions for values that never vary.

---

## 40. Determinism

Where supported:

- set random seeds;
- pin dependency versions;
- record model library versions;
- control nondeterministic training behavior.

Perfect bitwise reproducibility is not always achievable, but the limits should be documented.

---

# Part XI — M9: Experiment Tracking

## 41. Milestone M9 — Introduce Experiment Tracking

MLflow is a reasonable candidate.

It should solve a real question:

> Which combination of data, features, model, parameters, and code produced this result?

Track:

```text
parameters
metrics
artifacts
model
training dates
feature set/version
data fingerprint
git commit
```

---

## 42. Experiment Naming

Avoid:

```text
test1
final
final2
final_final
```

Use structured metadata.

Example concepts:

```text
baseline-weekly
lgbm-lags-v1
lgbm-rolling-v2
```

and tags for:

```text
subsystem strategy
training window
feature version
forecast horizon
```

---

## 43. Experiment Comparison

Create reproducible comparison tables.

Example:

| Model | MAE | WAPE | Bias | Train Time |
|---|---:|---:|---:|---:|
| Weekly Naive | ... | ... | ... | ~0 |
| Linear | ... | ... | ... | ... |
| LightGBM | ... | ... | ... | ... |

Also compare by horizon.

---

# Part XII — M10: Model Artifact Contract

## 44. Milestone M10 — Define Deployable Model Artifacts

A deployable model requires more than serialized weights.

Artifact metadata should identify:

```text
model
feature specification
model version
training window
required inputs
output schema
library versions
evaluation summary
```

---

## 45. Prediction Contract

Canonical forecast record:

```text
forecast_id
model_version
prediction_time
target_time
horizon
subsystem
predicted_load_mw
created_at
```

Later append actual/evaluation through a separate evaluation table or relation rather than mutating historical prediction semantics carelessly.

---

# Part XIII — M11: Batch Inference

## 46. Milestone M11 — Generate Operational Forecasts

Create an inference job that:

1. identifies latest valid source observations;
2. determines forecast origin;
3. builds point-in-time features;
4. loads the approved model;
5. generates horizons 1–24;
6. validates outputs;
7. persists predictions.

---

## 47. Idempotent Forecast Generation

Repeated execution for the same:

```text
model_version
prediction_time
subsystem
horizon
```

must have defined behavior.

Options:

- reject duplicate;
- deterministic upsert;
- create explicitly versioned rerun.

Choose and document semantics.

---

## 48. Batch First

Initial inference can run:

```text
scheduled job
→ forecast next 24h
→ persist predictions
```

There is no requirement for an online API yet.

---

# Part XIV — M12: Prediction Storage

## 49. Milestone M12 — Persist Forecast History

Never overwrite the only copy of past forecasts.

Historical predictions are necessary for:

- performance monitoring;
- model comparisons;
- incident investigation;
- regression analysis;
- champion/challenger evaluation.

Potential store initially:

```text
Parquet / DuckDB
```

Later:

```text
PostgreSQL
```

if operational querying justifies it.

---

## 50. Immutable Prediction Principle

A forecast is a historical fact:

> At time X, model version Y predicted value Z for target time T.

Model evolution must not erase this record.

---

# Part XV — M13: Ground Truth Join

## 51. Milestone M13 — Match Predictions with Observations

Once actual load becomes available:

```text
forecast
+
actual observation
↓
evaluation record
```

Join using:

```text
subsystem
target_time
```

while respecting source revisions.

---

## 52. Actual Availability

Do not assume that target time and ground-truth availability time are identical.

Record when the project first observes the actual.

Potential concept:

```text
target_time
actual_ingested_at
```

This matters for delayed evaluation.

---

# Part XVI — M14: Monitoring

## 53. Milestone M14 — Data Monitoring

Monitor:

### Freshness

```text
now - latest_event_time
```

### Completeness

Expected hourly timestamps vs received timestamps.

### Schema

Unexpected columns, missing columns, type changes.

### Distribution

Changes in:

```text
load level
variance
seasonality
subsystem distributions
```

Do not equate distribution change automatically with harmful drift.

---

## 54. Prediction Monitoring

Monitor:

- successful forecast runs;
- failed runs;
- number of forecasts;
- missing horizons;
- missing subsystems;
- inference duration;
- model version;
- invalid outputs.

---

## 55. Model Performance Monitoring

When actuals arrive, calculate:

```text
MAE
RMSE
WAPE
bias
```

over rolling windows.

Example:

```text
last 24h
last 7d
last 30d
```

Compare against:

- historical expected performance;
- seasonal baseline;
- production champion history.

---

## 56. Horizon Monitoring

Track separately:

```text
+1h
+2h
...
+24h
```

A model can degrade at long horizons while remaining healthy at short horizons.

---

## 57. Subsystem Monitoring

Performance should be segmented by subsystem.

Do not hide poor subsystem performance behind national aggregate metrics.

---

# Part XVII — M15: Drift and Regime Changes

## 58. Milestone M15 — Study Drift

Distinguish:

### Data drift

Input distribution changes.

### Target drift

Load distribution changes.

### Concept drift

Relationship between features and target changes.

### Operational/source change

Dataset definition or measurement methodology changes.

These are related but not identical.

---

## 59. Documented Regime Changes

The discovery identified changes in ONS load methodology around 2021–2023, including incorporation of distributed-generation estimates in later periods.

Use this as a real research question.

Compare training windows:

```text
long history
vs
recent history
vs
post-regime-change history
```

Test whether more history actually improves generalization.

---

## 60. Training Window Experiments

Candidates:

```text
expanding window
```

versus:

```text
fixed rolling window
```

Example:

```text
last 1 year
last 2 years
last 5 years
all available history
```

Evaluate computational cost and predictive performance.

---

# Part XVIII — M16: Model Registry

## 61. Milestone M16 — Introduce Registry

A registry becomes useful once multiple deployable model versions exist.

Lifecycle concept:

```text
candidate
↓
validated
↓
champion
```

Terminology should follow the selected registry/tool version.

---

## 62. Promotion Metadata

Before promotion record:

```text
candidate model
current champion
backtest dataset/window
metrics
promotion criteria
decision
decision timestamp
```

---

## 63. Never Promote on One Metric Alone

Promotion should consider:

- aggregate accuracy;
- horizon-specific performance;
- subsystem performance;
- bias;
- stability;
- inference cost;
- regression against baselines;
- operational compatibility.

---

# Part XIX — M17: Champion / Challenger

## 64. Milestone M17 — Controlled Model Competition

Architecture:

```text
              new observations
                     │
                     ▼
             train challenger
                     │
                     ▼
                backtest
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      champion              challenger
          │                     │
          └──────── compare ────┘
                     │
                     ▼
                promotion gate
```

---

## 65. Baseline as Permanent Challenger

Keep seasonal naive models permanently available.

This protects against a sophisticated model silently becoming worse than a trivial forecast.

---

# Part XX — M18: Retraining

## 66. Milestone M18 — Scheduled Retraining

Start with predictable scheduling.

Example concept:

```text
monthly candidate training
```

Do not assume daily retraining is useful.

Measure:

- performance benefit;
- training cost;
- stability;
- data accumulation rate.

---

## 67. Performance-Based Retraining

Later evaluate triggers such as:

```text
persistent WAPE degradation
+
sufficient new observations
↓
train candidate
```

The trigger should start **candidate training**, not automatically guarantee deployment.

---

## 68. Retraining State Machine

Concept:

```text
MONITOR
   ↓
TRIGGER
   ↓
TRAIN
   ↓
BACKTEST
   ↓
VALIDATE
   ↓
REGISTER
   ↓
PROMOTION DECISION
```

Failures should leave the current champion untouched.

---

# Part XXI — M19: Orchestration

## 69. Milestone M19 — Introduce an Orchestrator When Needed

Before orchestration, simple commands/scripts are acceptable.

An orchestrator becomes justified when workflows include dependencies such as:

```text
ingest
↓
validate
↓
features
↓
forecast
↓
wait for actual
↓
evaluate
↓
monitor
```

Potential options:

- Dagster;
- Airflow;
- Prefect.

Choose based on project needs at implementation time.

Do not commit to one before comparing current versions and ergonomics.

---

## 70. Job Separation

Potential jobs:

```text
historical_backfill
daily_ingestion
forecast_generation
actual_evaluation
monitoring_metrics
candidate_training
model_evaluation
```

Avoid one giant pipeline that must execute everything every time.

---

# Part XXII — M20: CI

## 71. Milestone M20 — Continuous Integration

CI should eventually run:

```text
format/lint checks
unit tests
integration tests
feature leakage tests
schema/contract tests
small deterministic model tests
```

Do not run expensive full historical training on every commit.

---

## 72. ML-Specific CI Tests

Examples:

### Training smoke test

Train on a tiny fixture.

### Inference contract test

Load a small artifact and verify prediction schema.

### Leakage regression test

Ensure feature timestamps do not exceed forecast origin.

### Baseline regression test

Ensure metric code produces known results on fixed fixtures.

### Serialization test

Train → save → load → predict.

---

# Part XXIII — M21: CD / Deployment

## 73. Milestone M21 — Deployment Strategy

Initial deployment can be a scheduled containerized batch workload.

Concept:

```text
Docker image
↓
scheduled execution
↓
fetch latest data
↓
forecast
↓
persist
```

This is sufficient to demonstrate production ML behavior.

Kubernetes is not required.

---

## 74. Artifact Promotion vs Code Deployment

Treat separately:

```text
application code version
```

and:

```text
model version
```

A new model should not require rebuilding unrelated application logic when architecture permits separation.

---

# Part XXIV — M22: Observability

## 75. Milestone M22 — Operational Observability

Track:

```text
job status
job duration
records processed
latest source timestamp
forecast count
model version
errors
```

Use structured logs.

Potential later metrics backend:

- Prometheus-compatible metrics;
- dashboarding;
- alerting.

Only introduce infrastructure proportionate to deployment environment.

---

## 76. Correlation IDs / Run IDs

Each pipeline execution should eventually have a run identifier.

Example:

```text
ingestion_run_id
training_run_id
forecast_run_id
```

This improves traceability across logs and artifacts.

---

# Part XXV — M23: Semi-Hourly Evolution

## 77. Milestone M23 — Move Beyond Hourly Forecasting

Only after hourly system is reliable.

Potential target:

```text
frequency:
30 minutes

horizon:
48 steps = 24 hours
```

Potential sources:

- ONS verified load;
- ONS programmed load.

---

## 78. Revisit the Data Model

Semi-hourly data may introduce:

- different geographic grain;
- different source fields;
- different publication behavior;
- richer decomposition.

Do not force it into the hourly schema without evaluating semantic compatibility.

---

# Part XXVI — M24: Operational Benchmark

## 79. Milestone M24 — Compare Against Programmed Load

If timestamp and semantic alignment can be established correctly:

```text
project forecast
vs
ONS programmed load
vs
ONS verified load
```

Questions:

- How does error vary by horizon?
- How does it vary by subsystem/area?
- In which periods do forecasts diverge?
- Does the project model add analytical value relative to simple baselines?

Avoid claims of operational superiority unless methodology and information sets are genuinely comparable.

---

# Part XXVII — M25: Exogenous Features

## 80. Milestone M25 — Calendar Features

Calendar features are relatively safe because future calendar information is known in advance.

Potential:

```text
weekday
weekend
holiday
month
hour
special calendar periods
```

Validate actual predictive contribution through ablation tests.

---

## 81. Weather Features

Treat weather as a separate research milestone.

Offline experiment:

```text
historical observed weather
```

can estimate whether weather is informative.

But production-correct evaluation requires:

```text
historical weather forecast
available at forecast origin
```

not future observed weather.

If historical forecast vintages are unavailable, clearly distinguish:

```text
oracle/research experiment
```

from:

```text
production-valid experiment
```

---

## 82. Geographic Weather Aggregation

If used, define how weather maps to electrical load regions.

Possible approaches:

- representative cities;
- population-weighted weather;
- load-area weighting;
- multiple station features.

This requires justification and documentation.

---

# Part XXVIII — M26: Feature Store Decision

## 83. Milestone M26 — Decide Whether a Feature Store Is Justified

Do not assume one is necessary.

Evaluate whether the project has:

- repeated offline/online feature definitions;
- multiple consumers;
- feature discovery needs;
- point-in-time join complexity;
- feature reuse;
- online serving requirements.

If these are absent, maintain a simpler feature pipeline.

---

# Part XXIX — M27: Online Serving Decision

## 84. Milestone M27 — Decide Whether an API Adds Value

Potential endpoints:

```text
GET /forecasts/latest
GET /forecasts/{subsystem}
GET /models/champion
GET /health
```

An API can expose stored forecasts.

It does not need to calculate expensive forecasts synchronously.

Preferred architecture if API is added:

```text
scheduled forecasting
↓
forecast store
↓
API reads forecast store
```

rather than:

```text
HTTP request
↓
train/model pipeline
↓
forecast
```

---

# Part XXX — M28: Advanced Forecasting Models

## 85. Milestone M28 — Specialized Models

Only after the evaluation framework is mature.

Potential families to investigate:

- classical statistical forecasting;
- gradient-boosted trees;
- generalized additive models;
- specialized time-series libraries;
- probabilistic forecasting;
- neural forecasting.

The exact libraries should be selected when this milestone is reached.

---

## 86. Deep Learning Gate

A neural model must justify itself against the champion.

Compare:

```text
accuracy
training time
inference time
memory
operational complexity
stability
explainability
```

A tiny accuracy gain may not justify significantly higher complexity.

---

# Part XXXI — M29: Probabilistic Forecasting

## 87. Milestone M29 — Prediction Intervals

Point forecasts do not express uncertainty.

Potential evolution:

```text
P10
P50
P90
```

or prediction intervals.

Evaluate:

- coverage;
- interval width;
- calibration.

This is optional and should come after point forecasting is reliable.

---

# Part XXXII — M30: Model Explainability

## 88. Milestone M30 — Explain Model Behavior

Potential analysis:

- feature importance;
- SHAP or equivalent;
- partial dependence where appropriate;
- error slices;
- temporal sensitivity.

Explainability should answer concrete questions, not generate decorative plots.

Examples:

> Which lag signals drive the forecast?

> Does model behavior differ by subsystem?

> What changes during major error periods?

---

# Part XXXIII — Data Versioning and Revisions

## 89. Dataset Fingerprints

A training run should eventually identify its data.

Potential fingerprint inputs:

```text
source file hashes
date range
row count
schema
processing version
```

Avoid copying terabytes solely for versioning if metadata/fingerprints provide sufficient reproducibility.

---

## 90. Revised Ground Truth

If ONS revises historical values:

```text
actual_v1
→ actual_v2
```

the project should decide whether to:

- recompute historical metrics;
- preserve original evaluation;
- store both first-seen and latest actuals.

A mature design may distinguish:

```text
first_seen_actual
latest_actual
```

and retain provenance.

---

# Part XXXIV — Security and Reliability

## 91. Secrets

No credentials in Git.

Use environment variables or secret management for any future external services.

Public ONS data itself may not require credentials.

---

## 92. Dependency Security

Use:

- pinned/controlled dependencies;
- dependency updates;
- vulnerability scanning where appropriate.

Do not introduce large dependency trees without justification.

---

## 93. Failure Behavior

Jobs should fail explicitly on:

- invalid schema;
- corrupt source;
- impossible timestamps;
- incompatible model artifact;
- missing critical features.

Silent fallback can be more dangerous than failure.

---

# Part XXXV — Testing Strategy

## 94. Unit Tests

Targets:

```text
timestamp utilities
lag generation
rolling features
metric calculations
forecast horizon construction
schema validation
```

---

## 95. Integration Tests

Examples:

```text
sample raw file
↓
normalize
↓
features
↓
train
↓
predict
↓
evaluate
```

Use small public/synthetic fixtures.

---

## 96. Regression Tests

Protect behavior such as:

- known feature values;
- known baseline forecasts;
- known metric outputs;
- stable prediction schema;
- no temporal leakage.

---

## 97. Data Tests

Test:

```text
uniqueness
completeness
types
ranges
timestamp continuity
subsystem membership
```

---

# Part XXXVI — Documentation Strategy

## 98. README

README should remain an overview, not become the complete technical specification.

Keep:

- problem;
- architecture;
- scope;
- status;
- major engineering principles;
- how to run when implementation exists.

---

## 99. Architecture Documentation

Potential:

```text
docs/architecture/
├── overview.md
├── data-flow.md
├── training.md
├── inference.md
└── monitoring.md
```

Create progressively.

---

## 100. ADRs

Use Architecture Decision Records for meaningful choices.

Potential ADRs:

```text
ADR-001 Parquet for historical storage
ADR-002 batch inference before online serving
ADR-003 backtesting strategy
ADR-004 global vs per-subsystem model
ADR-005 experiment tracking solution
ADR-006 model registry strategy
ADR-007 orchestration choice
```

Do not create ADRs for trivial decisions.

---

## 101. Experiment Documentation

MLflow tracks machine metadata.

Human reasoning can live in:

```text
docs/experiments/
```

Example:

```text
2026-xx-training-window-comparison.md
```

Document:

```text
hypothesis
setup
result
interpretation
decision
```

---

# Part XXXVII — AGENTS.md Strategy

## 102. Codex/Agent Instructions

The root `AGENTS.md` should eventually tell coding agents to:

- explain non-obvious architectural decisions;
- preserve temporal correctness;
- never introduce future information into features;
- add tests for feature transformations;
- avoid dependencies without justification;
- keep raw data out of Git;
- avoid giant refactors without need;
- prefer incremental vertical slices;
- document meaningful architecture changes;
- avoid premature distributed architecture;
- never claim model quality without evaluation;
- preserve reproducibility;
- use public data and safe fixtures.

---

# Part XXXVIII — Git Strategy

## 103. Commit Philosophy

Prefer small coherent commits.

Examples:

```text
feat(ingestion): add ONS hourly load downloader
test(features): verify lag point-in-time correctness
feat(evaluation): add rolling-origin backtesting
feat(training): add reproducible LightGBM pipeline
feat(inference): persist hourly forecasts
feat(monitoring): compute rolling forecast metrics
docs(architecture): document model lifecycle
```

Avoid:

```text
update stuff
final
changes
fix
```

---

## 104. Branch Strategy

For a personal project, avoid unnecessary Git-flow complexity.

A simple model is sufficient:

```text
main
+
short-lived feature branches when useful
```

CI protects `main`.

---

# Part XXXIX — Technology Evolution

## 105. Stage A — Local Research

Potential stack:

```text
Python
Parquet
DuckDB or Polars
scikit-learn
LightGBM/XGBoost
Matplotlib
```

Goal:

correct forecasting evaluation.

---

## 106. Stage B — Reproducible ML

Add when needed:

```text
configuration
MLflow
structured artifacts
Docker
CI
```

Goal:

reproducible training and inference.

---

## 107. Stage C — Operational ML

Potential:

```text
PostgreSQL
orchestrator
scheduled jobs
monitoring
model registry
```

Goal:

continuous model lifecycle.

---

## 108. Stage D — Scale Only If Required

Potential future technologies only after profiling:

```text
distributed processing
cloud object storage
Kubernetes
feature store
```

No technology is mandatory merely because it is common in MLOps diagrams.

---

# Part XL — Milestone Sequence

## 109. Recommended Implementation Order

### M0 — Source discovery

Deliver:

```text
verified data contract
source documentation
sample files
```

### M1 — Historical ingestion

Deliver:

```text
reproducible ONS downloader
raw storage
```

### M2 — Canonical dataset

Deliver:

```text
validated hourly Parquet dataset
```

### M3 — Temporal EDA

Deliver:

```text
documented seasonality and data issues
```

### M4 — Baselines

Deliver:

```text
persistence
daily seasonal naive
weekly seasonal naive
```

### M5 — Backtesting

Deliver:

```text
rolling temporal evaluation framework
```

### M6 — Features

Deliver:

```text
point-in-time-correct feature pipeline
```

### M7 — First ML model

Deliver:

```text
candidate model compared against baselines
```

### M8 — Reproducible training

Deliver:

```text
configurable training command
```

### M9 — Experiment tracking

Deliver:

```text
tracked experiments
```

### M10 — Artifact contract

Deliver:

```text
deployable model artifact
```

### M11 — Batch inference

Deliver:

```text
1–24h forecasts
```

### M12 — Forecast history

Deliver:

```text
persistent prediction store
```

### M13 — Ground truth

Deliver:

```text
forecast vs actual dataset
```

### M14 — Monitoring

Deliver:

```text
data + prediction + performance monitoring
```

### M15 — Drift

Deliver:

```text
regime/training-window analysis
```

### M16 — Registry

Deliver:

```text
versioned deployable models
```

### M17 — Champion/challenger

Deliver:

```text
controlled comparison process
```

### M18 — Retraining

Deliver:

```text
candidate retraining workflow
```

### M19 — Orchestration

Deliver:

```text
scheduled dependent workflows
```

### M20 — CI

Deliver:

```text
automated quality gates
```

### M21 — Deployment

Deliver:

```text
containerized scheduled inference
```

### M22 — Observability

Deliver:

```text
operational metrics and traceable runs
```

### M23+ — Advanced evolution

Only after the core system works:

```text
semi-hourly forecasts
operational benchmark
weather
advanced models
probabilistic forecasting
API
```

---

# Part XLI — Release Strategy

## 110. V0.1 — Data Foundation

Contains:

- source discovery;
- historical ingestion;
- canonical dataset;
- validation;
- initial EDA.

Exit criteria:

```text
raw ONS data can be reproducibly retrieved
+
canonical hourly dataset can be rebuilt
+
data quality is measured
```

---

## 111. V0.2 — Forecasting Baseline

Contains:

- seasonal baselines;
- temporal backtesting;
- evaluation metrics;
- horizon analysis.

Exit criteria:

```text
we can answer:
"How difficult is this forecasting problem?"
```

---

## 112. V0.3 — ML Candidate

Contains:

- point-in-time feature pipeline;
- first ML models;
- baseline comparison;
- training-window experiments.

Exit criteria:

```text
candidate demonstrates measured value
or
we understand why it does not
```

---

## 113. V0.4 — Reproducible ML

Contains:

- productionized training;
- configuration;
- experiment tracking;
- model artifacts;
- tests.

Exit criteria:

```text
a model run can be reproduced and audited
```

---

## 114. V0.5 — Inference System

Contains:

- batch forecasting;
- prediction contract;
- forecast storage;
- model version attached to predictions.

Exit criteria:

```text
system repeatedly produces traceable forecasts
```

---

## 115. V0.6 — Closed Evaluation Loop

Contains:

- new actual ingestion;
- ground-truth join;
- rolling performance;
- monitoring.

Exit criteria:

```text
predictions are automatically evaluated when actuals arrive
```

This is an important portfolio milestone.

At this point the project is no longer merely a training pipeline.

---

## 116. V0.7 — Model Lifecycle

Contains:

- registry;
- champion/challenger;
- candidate training;
- promotion criteria;
- retraining workflow.

Exit criteria:

```text
models can evolve without losing governance or history
```

---

## 117. V1.0 — Production-Oriented ML System

A strong public V1 should demonstrate:

```text
reproducible ingestion
+
validated data
+
temporal backtesting
+
defensible baselines
+
point-in-time features
+
tracked training
+
versioned model
+
batch inference
+
historical predictions
+
ground-truth evaluation
+
monitoring
+
controlled model lifecycle
+
tests
+
CI
+
documentation
```

This is the primary target.

Do not wait for every advanced milestone before calling the project useful.

---

# Part XLII — V2 and Beyond

## 118. V2 — Semi-Hourly Operational Evolution

Potential:

- verified load API;
- programmed load;
- 30-minute forecasting;
- 48-step horizon;
- richer operational evaluation.

---

## 119. V3 — Exogenous Information

Potential:

- holidays;
- weather;
- historical forecast vintages;
- generation context.

Require point-in-time correctness.

---

## 120. V4 — Advanced Forecasting

Potential:

- probabilistic forecasts;
- specialized forecasting models;
- neural models;
- ensembles.

Only retain complexity that wins under evaluation.

---

# Part XLIII — What Not to Build Early

## 121. Explicit Avoid List

Do not begin with:

```text
Kafka
Spark cluster
Kubernetes
microservices
feature store
real-time API
deep learning
automatic production promotion
complex alerting platform
multiple cloud providers
```

These can become legitimate later.

Starting with them would hide the core ML problem beneath infrastructure.

---

# Part XLIV — Learning Objectives

## 122. Time-Series ML

The project should teach:

- temporal validation;
- seasonality;
- forecasting horizons;
- lag features;
- rolling statistics;
- non-stationarity;
- regime changes;
- leakage;
- multi-horizon evaluation.

---

## 123. ML Engineering

The project should teach:

- feature pipelines;
- training pipelines;
- configuration;
- reproducibility;
- model artifacts;
- batch inference;
- prediction contracts;
- model versioning.

---

## 124. MLOps

The project should teach:

- experiment tracking;
- model registry;
- monitoring;
- drift;
- ground-truth joins;
- retraining;
- champion/challenger;
- promotion gates;
- CI/CD for ML.

---

## 125. Software Engineering

The project should reinforce:

- modular architecture;
- testing;
- typing;
- error handling;
- observability;
- configuration;
- documentation;
- dependency management;
- Git discipline.

---

# Part XLV — Portfolio Narrative

## 126. What This Repository Demonstrates

This project should communicate:

> I can build and operate the lifecycle around a predictive model, not merely train one.

It complements the other portfolio projects rather than duplicating them.

```text
medaudit
→ Applied AI / RAG / LLM Systems

third-party-lifecycle
→ Backend / SaaS / Software Architecture

municipal-fiscal-data-platform
→ Data Engineering / Analytics Engineering

energy-load-forecasting
→ ML Engineering / MLOps

coping_struggles_prediction
→ Data Science / Statistical ML
```

---

# Part XLVI — Definition of Done for Public V1

## 127. Data

- [ ] Official source documented
- [ ] Historical ingestion reproducible
- [ ] Raw data provenance preserved
- [ ] Canonical hourly dataset
- [ ] Data validation implemented
- [ ] Revision behavior documented

## 128. Forecasting

- [ ] Persistence baseline
- [ ] Daily seasonal naive
- [ ] Weekly seasonal naive
- [ ] Rolling/walk-forward backtesting
- [ ] Horizon-specific evaluation
- [ ] Subsystem-specific evaluation
- [ ] At least one ML candidate
- [ ] Candidate compared fairly with baselines

## 129. Temporal Correctness

- [ ] Forecast origin represented explicitly
- [ ] Target time represented explicitly
- [ ] Horizon represented explicitly
- [ ] Feature leakage tests
- [ ] Rolling features shifted correctly
- [ ] Training/test chronology preserved

## 130. ML Engineering

- [ ] Reproducible training entry point
- [ ] Configurable experiments
- [ ] Model artifact contract
- [ ] Data fingerprint/version metadata
- [ ] Batch inference
- [ ] Prediction persistence

## 131. MLOps

- [ ] Experiment tracking
- [ ] Model version attached to forecasts
- [ ] Ground-truth evaluation
- [ ] Rolling performance monitoring
- [ ] Registry
- [ ] Champion/challenger workflow
- [ ] Retraining workflow

## 132. Engineering Quality

- [ ] Unit tests
- [ ] Integration tests
- [ ] Regression tests
- [ ] CI
- [ ] Structured logging
- [ ] Dockerized operational workload
- [ ] Architecture documentation
- [ ] Meaningful ADRs

---

# Part XLVII — First Development Sessions

## 133. Session 1 — Source Verification

Do only:

1. inspect official ONS dataset;
2. download a few representative files;
3. inspect schemas;
4. document timestamps, units, keys, and subsystems;
5. identify inconsistencies between years.

Do **not** train a model yet.

---

## 134. Session 2 — Ingestion

Build:

```text
download
↓
validate response
↓
store raw
↓
record provenance
```

Add tests around file handling and source parsing.

---

## 135. Session 3 — Canonical Data

Build:

```text
raw
↓
normalize
↓
validate
↓
Parquet
```

Confirm:

```text
(subsystem, event_time)
```

grain and uniqueness.

---

## 136. Session 4 — Baseline

Implement weekly seasonal naive first.

Generate a real backtest.

This establishes the first meaningful forecasting benchmark.

---

## 137. Session 5 — Evaluation Framework

Before serious ML, make evaluation reusable.

Output granular forecasts and actuals.

Then aggregate metrics from them.

---

## 138. Session 6 — Feature Pipeline

Add:

```text
calendar
lags
rolling features
```

with explicit leakage tests.

---

## 139. Session 7 — First Candidate

Train a simple tree-based candidate.

Compare against seasonal naive.

Do not optimize aggressively yet.

---

## 140. Session 8 — Reproducibility

Move successful research code into production modules.

Add configuration and experiment tracking.

---

# Part XLVIII — Decision Gates

## 141. Gate A — Is ML Better Than Baseline?

If no:

Do not add infrastructure to hide it.

Investigate:

- features;
- training windows;
- subsystem strategy;
- horizon strategy;
- data quality.

A baseline can remain champion.

---

## 142. Gate B — Is MLflow Needed?

Introduce when manual experiment comparison becomes a real limitation.

Expected answer eventually: probably yes.

But prove the requirement through actual experiments.

---

## 143. Gate C — Is an Orchestrator Needed?

Introduce when multiple recurring jobs and dependencies exist.

Not before.

---

## 144. Gate D — Is PostgreSQL Needed?

Introduce when operational forecast/monitoring queries make local analytical storage insufficient.

Not because “production uses databases.”

---

## 145. Gate E — Is an API Needed?

Introduce if there is a real consumer of forecasts.

A dashboard, frontend, or external client can justify it.

---

## 146. Gate F — Is a Feature Store Needed?

Introduce only if offline/online consistency, reuse, or point-in-time feature serving becomes sufficiently complex.

---

## 147. Gate G — Is Deep Learning Needed?

Only if a properly evaluated challenger provides enough benefit to justify operational cost.

---

# Part XLIX — Success Criteria

## 148. Technical Success

The project succeeds if another engineer can:

```text
clone repository
↓
retrieve public data
↓
rebuild dataset
↓
run backtest
↓
train candidate
↓
inspect experiment
↓
generate forecasts
↓
evaluate them against actuals
```

with documented commands and deterministic behavior where feasible.

---

## 149. ML Success

Success is not defined by a predetermined accuracy number.

Success means:

- evaluation is correct;
- baselines are strong;
- model improvements are measurable;
- uncertainty and limitations are documented;
- model behavior over time is observable.

---

## 150. Portfolio Success

A reviewer should be able to identify evidence of:

```text
time-series modeling
+
ML engineering
+
MLOps
+
software engineering
+
production thinking
```

without needing to infer it from a long technology list.

---

# Part L — Final Architecture Direction

The long-term architecture can evolve toward:

```text
                         ┌────────────────────┐
                         │      ONS DATA      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │     INGESTION      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  RAW / HISTORICAL  │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ VALIDATION / DQ    │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  FEATURE PIPELINE  │
                         └──────┬───────┬─────┘
                                │       │
                   ┌────────────┘       └────────────┐
                   ▼                                 ▼
          ┌────────────────┐                ┌────────────────┐
          │    TRAINING    │                │   INFERENCE    │
          └───────┬────────┘                └───────┬────────┘
                  │                                 │
                  ▼                                 ▼
          ┌────────────────┐                ┌────────────────┐
          │  EXPERIMENTS   │                │   FORECASTS    │
          └───────┬────────┘                └───────┬────────┘
                  │                                 │
                  ▼                                 │
          ┌────────────────┐                        │
          │ MODEL REGISTRY │                        │
          └───────┬────────┘                        │
                  │                                 │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  ACTUALS / GROUND  │
                         │       TRUTH        │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │     EVALUATION     │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │     MONITORING     │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ RETRAINING POLICY  │
                         └─────────┬──────────┘
                                   │
                                   └──────────────► TRAINING
```

The diagram represents the destination, not the starting point.

The correct starting point remains:

```text
verified data
↓
strong baseline
↓
correct temporal evaluation
```

Everything else should grow from that foundation.

---

# Recommended Immediate Next Step

When development begins, do **not** scaffold the final architecture.

Start with Milestone M0:

```text
1. Verify the current ONS hourly-load source.
2. Download representative years.
3. Inspect schema and temporal semantics.
4. Write the source data contract.
5. Commit the discovery before implementing ingestion.
```

Suggested first implementation-era commit:

```text
docs(data): document ONS hourly load source and data contract
```

Then build the smallest end-to-end vertical slice:

```text
one source
→ one historical period
→ canonical data
→ weekly seasonal baseline
→ temporal backtest
→ metrics
```

Once that slice is trustworthy, evolve it into the full ML lifecycle described in this roadmap.
