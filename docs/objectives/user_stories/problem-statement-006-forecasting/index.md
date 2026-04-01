# PS-006: Five-Year Disease Burden Forecasting - User Stories Index

## Problem Statement Overview

**Problem Statement**: [PS-006: Five-Year Disease Burden Forecasting and Mortality Projection](../../problem_statements/ps-006-disease-burden-forecasting.md)

**Objective**: Develop time series forecasting models to project mortality and disease burden for major diseases (cancer, stroke, heart disease) over the next 5 years, enabling proactive healthcare capacity planning and resource allocation.

**Analysis Category**: Predictive Analytics  
**Priority**: P1 (High)  
**Estimated Effort**: 5-7 sprints

---

## User Stories (Data Analysis Lifecycle Decomposition)

### Phase 1: Data Foundation (Sprint 1)

**[01 - Historical Mortality Data Extraction and Validation](01-historical-mortality-data-extraction-validation.md)**
- **Stage**: Data Extraction & Understanding
- **Description**: Extract and validate 30-year historical mortality time series for cancer, stroke, and heart disease
- **Acceptance Criteria**: 100% data completeness verified, schema validated, quality report generated
- **Deliverables**: Cleaned mortality dataset (1990-2019), data quality report, validation logs

---

### Phase 2: Exploratory Analysis & Feature Engineering (Sprints 2-3)

**[02 - Exploratory Time Series Analysis and Pattern Identification](02-exploratory-time-series-analysis.md)**
- **Stage**: Exploratory Data Analysis
- **Description**: Conduct comprehensive time series analysis to understand trends, stationarity, and autocorrelation patterns
- **Acceptance Criteria**: Stationarity assessed, ACF/PACF analyzed, ARIMA parameters recommended, trend inflection points identified
- **Deliverables**: EDA report with trend analysis, ACF/PACF plots, stationarity test results, forecasting method recommendations

**[03 - Temporal Feature Engineering for Forecasting](03-temporal-feature-engineering.md)**
- **Stage**: Feature Engineering
- **Description**: Engineer lag features, rolling averages, and transformations optimized for forecasting models
- **Acceptance Criteria**: Lag features created, stationary transformations applied, train-test split prepared
- **Deliverables**: Feature-engineered dataset with temporal variables, feature data dictionary

---

### Phase 3: Model Development & Validation (Sprints 3-4)

**[04 - Baseline Forecasting Model Development](04-baseline-forecasting-models.md)**
- **Stage**: Statistical Modeling
- **Description**: Develop and compare multiple baseline forecasting approaches (Naïve, SES, ARIMA, Prophet)
- **Acceptance Criteria**: 4+ models implemented, trained on pre-2015 data, in-sample performance evaluated
- **Deliverables**: Trained forecasting models, model artifacts, in-sample performance metrics

**[05 - Model Validation and Backtesting](05-model-validation-backtesting.md)**
- **Stage**: Model Evaluation & Validation
- **Description**: Rigorously validate forecast accuracy on historical test period (2015-2019)
- **Acceptance Criteria**: Out-of-sample RMSE/MAE/MAPE calculated, prediction interval coverage validated, best model selected
- **Deliverables**: Model validation report, actual-vs-forecast charts, best model selection rationale

---

### Phase 4: Production Forecasts & Scenarios (Sprints 5-6)

**[06 - Five-Year Mortality Projections Generation](06-five-year-projections-generation.md)**
- **Stage**: Production Forecasting
- **Description**: Generate official 5-year mortality forecasts (2020-2025) with 80% and 95% confidence intervals
- **Acceptance Criteria**: Production forecasts for all 3 diseases, prediction intervals included, forecast dataset saved
- **Deliverables**: "Singapore Disease Burden Projections 2020-2025" report, forecast dataset (Parquet), visualization gallery

**[07 - Scenario-Based Forecasting and Sensitivity Analysis](07-scenario-based-forecasting.md)**
- **Stage**: Scenario Modeling
- **Description**: Create baseline, optimistic, and pessimistic forecast scenarios with intervention impact analysis
- **Acceptance Criteria**: 3 scenarios generated, intervention effects modeled, scenario comparison report delivered
- **Deliverables**: Scenario forecast dataset, scenario comparison visualizations, intervention impact analysis

**[08 - Healthcare Capacity Requirement Translation](08-capacity-requirement-translation.md)**
- **Stage**: Business Impact Translation
- **Description**: Convert mortality projections into healthcare capacity needs (hospital beds, specialist workforce)
- **Acceptance Criteria**: Capacity translation model created, gap analysis completed, investment roadmap delivered
- **Deliverables**: "Healthcare Capacity Investment Roadmap 2020-2025" policy brief, capacity requirements table

---

### Phase 5: Communication & Operationalization (Sprint 7)

**[09 - Interactive Disease Burden Forecast Dashboard](09-interactive-forecast-dashboard.md)**
- **Stage**: Visualization & Communication
- **Description**: Build interactive web dashboard for stakeholders to explore forecasts and scenarios
- **Acceptance Criteria**: Multi-disease forecast visualization, scenario comparison, capacity calculator, export functionality
- **Deliverables**: Plotly Dash dashboard, deployed to Databricks or standalone server, user guide

**[10 - Forecast Monitoring and Model Update System](10-forecast-monitoring-system.md)**
- **Stage**: Operationalization & Monitoring
- **Description**: Implement automated system to track forecast accuracy and trigger model retraining
- **Acceptance Criteria**: Quarterly monitoring automated, deviation alerts configured, retraining workflow documented
- **Deliverables**: Forecast monitoring dashboard, alert system, model update procedure documentation

---

## Lifecycle Stages Covered

| Lifecycle Stage | User Stories | Sprint Phase |
|----------------|--------------|--------------|
| **Data Extraction & Understanding** | US-01 | Sprint 1 |
| **Exploratory Data Analysis** | US-02 | Sprint 2 |
| **Feature Engineering** | US-03 | Sprint 2-3 |
| **Statistical Modeling** | US-04 | Sprint 3-4 |
| **Model Evaluation & Validation** | US-05 | Sprint 4 |
| **Production Forecasting** | US-06, US-07 | Sprint 5 |
| **Business Impact Translation** | US-08 | Sprint 5-6 |
| **Visualization & Communication** | US-09 | Sprint 7 |
| **Operationalization** | US-10 | Sprint 7 |

---

## Domain Knowledge Resources

All user stories leverage the following domain knowledge files:

### Primary References
- **[Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md)** - Complete forecasting methodology guide (Naïve, SES, Holt, ARIMA, SARIMA, Prophet, ensemble methods, validation techniques)
- **[Disease Burden Feature Engineering Guide](../../../domain_knowledge/disease-burden-feature-engineering-guide.md)** - Epidemiological metrics (ASMR, DALY, YLL), trend analysis methods, temporal feature patterns

### Supporting References
- **[Healthcare Workforce Metrics KPIs](../../../domain_knowledge/healthcare-workforce-metrics-kpis.md)** - Workforce planning ratios for capacity translation (US-08)
- **Data Sources Documentation**: `docs/project_context/data-sources.md` - Kaggle dataset specifications and limitations

---

## Reusable Components

### Shared Utilities (to be created in `shared/src/`)

**Data Processing** (`shared/src/data_processing/`):
- `forecasting_data_loader.py` - Mortality time series loading and validation
- `feature_engineering_forecasting.py` - Temporal feature creation (lags, rolling averages, differencing)

**Analysis** (`shared/src/analysis/`):
- `time_series_utils.py` - Stationarity tests, ACF/PACF analysis, decomposition
- `forecasting_models.py` - Wrapper classes for ARIMA, Prophet, ensemble models
- `model_validation.py` - Backtesting, metrics calculation, interval coverage checks

**Visualization** (`shared/src/visualization/`):
- `forecast_plots.py` - Forecast charts with confidence bands, actual-vs-forecast comparisons
- `scenario_plots.py` - Multi-scenario overlay visualizations

**Monitoring** (`shared/src/orchestration/`):
- `forecast_monitor.py` - Automated accuracy tracking and drift detection

---

## Key Data Flows

```
RAW DATA (1990-2019)
  ↓
[US-01: Extraction & Validation]
  ↓
CLEAN TIME SERIES → [US-02: EDA] → PATTERN INSIGHTS
  ↓                                        ↓
[US-03: Feature Engineering]     [ARIMA Parameters]
  ↓                                        ↓
FEATURED DATASET (train/test split)       ↓
  ↓                                        ↓
[US-04: Model Development] ← ─ ─ ─ ─ ─ ─ ─ ┘
  ↓
CANDIDATE MODELS
  ↓
[US-05: Validation] → BEST MODELS
  ↓
[US-06: Production Forecasts (2020-2025)]
  ↓                    ↓
  ├─→ [US-07: Scenarios] ─→ SCENARIO FORECASTS
  └─→ [US-08: Capacity Translation] ─→ CAPACITY REQUIREMENTS
                       ↓                         ↓
                       └─────────┬───────────────┘
                                 ↓
                [US-09: Interactive Dashboard] ←─┐
                                 ↓                │
                                 └────────────────┘
                                 ↓
         [US-10: Monitoring System] (continuous)
```

---

## Dependencies Between User Stories

| User Story | Depends On | Blocks |
|-----------|------------|--------|
| US-01 | None | US-02, US-03 |
| US-02 | US-01 | US-03, US-04 |
| US-03 | US-01, US-02 | US-04 |
| US-04 | US-02, US-03 | US-05 |
| US-05 | US-04 | US-06 |
| US-06 | US-05 | US-07, US-08, US-09 |
| US-07 | US-06 | US-09 |
| US-08 | US-06, US-07 | US-09 |
| US-09 | US-06, US-07, US-08 | US-10 |
| US-10 | US-06, US-09 | None (continuous) |

---

## Success Metrics

**Data Quality** (US-01):
- ✅ 100% completeness for core mortality time series (cancer, stroke, heart disease)
- ✅ 30-year continuous span (1990-2019) confirmed

**Model Performance** (US-04, US-05):
- ✅ MAPE < 15% for at least one model per disease (good accuracy)
- ✅ 95% prediction interval coverage: 90-100% (well-calibrated uncertainty)
- ✅ MASE < 1 (beats naïve forecast)

**Forecast Delivery** (US-06):
- ✅ 5-year projections (2020-2025) for all 3 diseases
- ✅ Confidence intervals (80%, 95%) provided
- ✅ Stakeholder report delivered and accessible

**Scenario Planning** (US-07):
- ✅ 3 scenarios generated (baseline, optimistic, pessimistic)
- ✅ Intervention impact quantified

**Capacity Planning** (US-08):
- ✅ Mortality-to-capacity translation model validated
- ✅ Investment roadmap delivered to MOH planning division

**Stakeholder Engagement** (US-09):
- ✅ Dashboard deployed and accessible to 10+ MOH users
- ✅ Positive feedback from Healthcare Planning Division

**Operational Excellence** (US-10):
- ✅ Monitoring system operational (quarterly runs automated)
- ✅ Alerts tested and functional

---

## Sprint Planning Recommendations

**Sprint 1**: US-01 (Data foundation)  
**Sprint 2**: US-02, start US-03 (Exploratory analysis, feature engineering)  
**Sprint 3**: Complete US-03, US-04 (Feature engineering, model development)  
**Sprint 4**: US-05 (Validation and model selection)  
**Sprint 5**: US-06, US-07 (Production forecasts, scenarios)  
**Sprint 6**: US-08 (Capacity translation)  
**Sprint 7**: US-09, US-10 (Dashboard, monitoring system)

**Optional Sprint 8**: Refinements, stakeholder training, model improvements based on feedback

---

## Risk Management

**Data Risks**:
- **COVID-19 impact**: Latest data is 2019-2020 (pre-COVID) → **Mitigation**: Position forecasts as "baseline", add COVID scenarios, plan urgent update when 2020+ data available
- **Limited disease coverage**: Only 3 diseases → **Mitigation**: These account for ~60% of mortality; expand when more data obtained

**Model Risks**:
- **Forecast uncertainty too wide**: Pessimistic-optimistic range too large for planning → **Mitigation**: Use ensemble methods, identify "no-regret" investments
- **Structural breaks**: Historical trends disrupted by policy changes → **Mitigation**: Prophet for changepoint detection, scenario planning

**Operational Risks**:
- **Stakeholder misinterpretation**: Treat probabilistic forecasts as deterministic → **Mitigation**: Extensive communication, always show uncertainty, scenario narratives

---

**Last Updated**: 2026-03-13  
**Status**: Ready for Sprint Planning  
**Problem Statement Owner**: MOH Strategic Planning Division  
**Technical Lead**: Data Analytics Team
