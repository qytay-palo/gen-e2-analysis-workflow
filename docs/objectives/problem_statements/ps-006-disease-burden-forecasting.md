# PS-006: Five-Year Disease Burden Forecasting and Mortality Projection

```yaml
problem_statement_id: PS-006
title: Five-Year Disease Burden Forecasting and Mortality Projection
analysis_category: Predictive Analytics
dependencies: PS-002 (recommended but not required)
platform: HEALIX/Databricks
primary_language: Python (Polars)
estimated_sprints: 5-7
priority: P1 (High)
```

---

## Executive Summary

Currently, **MOH strategic planners and healthcare capacity planners** face **uncertainty about future disease burden magnitude and resource requirements for the next 5 years (2020-2025)**, which prevents **proactive healthcare capacity planning, clinical service expansion, and public health program scaling**. By **developing time series forecasting models using 30-year historical mortality trends (1990-2019) for major diseases with confidence intervals and scenario analysis**, we can **enable evidence-based resource allocation and early preparation for shifting disease burden patterns** resulting in **optimized healthcare infrastructure investments and prevention program sizing aligned with projected population health needs**.

---

## Problem Statement Hypothesis (Value Proposition)

> We believe that **5-year mortality and disease burden forecasting for healthcare strategic planners and policy makers** will achieve **proactive capacity planning and optimized resource allocation before demand materializes**. We'll know we're successful when we see **healthcare facility expansion plans, clinical service budgets, and public health program scales explicitly referencing our disease burden projections and capacity planning decisions made 12-18 months ahead of projected demand increases**.

---

## Objectives

**Objective 1**: Develop 5-year mortality projection models for Singapore's major disease burdens (2020-2025)
- Build time series forecasting models for cancer, stroke, and ischemic heart disease mortality
- Generate point forecasts with 80% and 95% confidence intervals
- Incorporate historical trend patterns, inflection points, and acceleration/deceleration dynamics
- Validate models using backtesting on 2015-2019 data

**Objective 2**: Quantify future disease burden scenarios under different intervention assumptions
- Create baseline scenario (continuation of historical trends)
- Model optimistic scenario (accelerated improvement from enhanced interventions)
- Model pessimistic scenario (trend reversal from aging population/lifestyle changes)
- Quantify the impact of specific interventions (e.g., expanded screening programs, prevention campaigns)

**Objective 3**: Project healthcare capacity requirements based on forecasted disease burden
- Translate mortality projections into clinical service demand (hospital beds, specialist workforce)
- Forecast demand for disease-specific healthcare services (oncology, cardiology, neurology)
- Identify capacity gaps between projected demand and current infrastructure plans
- Calculate investment requirements to meet projected 2025 disease burden

**Objective 4**: Generate early warning system for emerging disease burden shifts
- Implement monitoring framework comparing actual vs forecasted mortality trends
- Create alert mechanisms for statistically significant deviations from projections
- Enable quarterly or annual forecast updates as new data becomes available
- Build adaptive models that incorporate new data and refine predictions

---

## Stakeholders and Value Proposition

**Primary Stakeholders**:
- **Healthcare Planning Division, MOH** - Infrastructure and capacity planning (hospital beds, facilities)
- **Manpower Planning Unit, MOH** - Clinical workforce requirements for disease-specific specialties
- **Finance & Budget Division, MOH** - Multi-year healthcare budget projections
- **Public Health Policy Unit, MOH** - Prevention program scaling and intervention targeting
- **National Healthcare Group / National University Health System** - Clinical service expansion planning

**Business Value**:
- **Decision enabled**: Proactive infrastructure and workforce planning vs reactive capacity shortages
- **Efficiency gain**: Avoid costly emergency capacity expansions through advance preparation (lead time: 3-5 years for facility builds, 5-10 years for specialist training)
- **Quality improvement**: Reduce wait times and access barriers through capacity-demand alignment
- **Risk reduction**: Mitigate crisis scenarios from unexpected disease burden surges through early warning system
- **Financial optimization**: Better multi-year budget accuracy for healthcare investments (reduce budget variance from ±30% to ±10%)

---

## Data Requirements (High-Level)

**Critical Considerations**:

**✅ Data Availability CONFIRMED** (Reference: `docs/project_context/data-sources.md`):

**Primary Data - Historical Mortality Trends**:
- `age-standardised-mortality-rate-for-cancer.csv` (30 years, 1990-2019) - **390 data points**
- `age-standardised-mortality-rate-for-stroke.csv` (30 years, 1990-2019) - **390 data points**
- `age-standardised-mortality-rate-for-ischaemic-heart-disease.csv` (30 years, 1990-2019) - **390 data points**

**Supplementary Data - Contextual Factors**:
- `hospital-admission-rate-by-age-and-sex.csv` (216 records, 2006-2020) - Correlate mortality with morbidity trends
- `vaccination-and-immunisation-of-students-annual.csv` (33 records) - Prevention program coverage for intervention scenarios
- `common-health-problems-of-students-examined-obesity-annual.csv` (48 records) - Risk factor trends
- `number-of-doctors.csv` (78 records, 2006-2019) - Supply-side constraints for capacity planning
- `health-facilities-and-beds-in-inpatient-facilities.csv` (180 records) - Current capacity baseline

**External Data (for model enrichment)**:
- Singapore population projections (Department of Statistics) - Demographic context
- WHO/OECD mortality projections - Benchmark validation
- Climate data (if correlations with disease burden exist) - Environmental factors

**✅ Data Completeness**: 100% completeness for mortality time series, sufficient length (30 years) for robust forecasting

**✅ Data Quality**: 
- Age-standardized rates (consistent over time)
- Official MOH source with validated methodology
- Continuous annual observations (no gaps)

**✅ Data Granularity**:
- **Geographic**: National level - APPROPRIATE for national capacity planning
- **Temporal**: Annual data - IDEAL for 5-year strategic forecasting (monthly/weekly not needed for strategic planning)
- **Demographic**: Age-standardized - ENSURES valid projections not confounded by aging population

**⚠️ Data Limitations & Mitigations**:
1. **Limitation**: Latest data is 2019-2020 (pre-COVID)
   - **Mitigation**: Clearly position forecasts as "pre-COVID baseline"; recommend model updates when 2020-2025 data available to incorporate pandemic impacts
   - **Approach**: Create COVID-adjustment scenarios based on international literature

2. **Limitation**: Only 3 major diseases (cancer, stroke, heart disease)
   - **Mitigation**: Focus on these high-burden diseases first (account for ~60% of total mortality); expand model when additional disease data obtained
   - **Coverage**: These 3 diseases represent majority of healthcare capacity demand

3. **Limitation**: No disease incidence data (only mortality)
   - **Mitigation**: Use hospital admission rates as proxy for morbidity trends; mortality projections still valid for burden assessment
   - **Justification**: Mortality is leading indicator for healthcare capacity needs (critical care, specialized treatment)

**Privacy/Security**: Aggregated statistics only - NO privacy concerns

**No Critical Data Blockers** - Core forecasting data confirmed available with acceptable limitations

---

## Initial Considerations

**Analytical Approach**:
- **Type**: Time series forecasting with scenario modeling and sensitivity analysis
- **Methods** (to be selected during sprint planning):
  - **Classical Time Series**: ARIMA, Seasonal ARIMA (SARIMA), Exponential Smoothing (Holt-Winters)
  - **Advanced**: Prophet (Facebook) for trend changepoint detection, Bayesian Structural Time Series
  - **Ensemble**: Combine multiple models for robust projections
  - **Scenario Modeling**: Monte Carlo simulation for uncertainty quantification
- **Model Selection Criteria**: Forecast accuracy (RMSE, MAPE), prediction interval coverage, interpretability for stakeholders
- **Validation**: Backtesting on 2015-2019 (withhold 5 years, forecast, compare to actuals)

**Platform Feasibility** (Reference: `docs/project_context/tech-stack.md`):

**✅ Platform Requirements VERIFIED**:
- **Primary Platform**: HEALIX/Databricks - ✅ Confirmed in tech stack
- **Primary Language**: Python (Polars for data wrangling, statsmodels/Prophet/scikit-learn for forecasting) - ✅ Fully supported
- **Compute Requirements**: Local/single-node sufficient - small dataset (<2,000 records total)
- **Data Access Pattern**: Batch CSV processing with iterative model training - ✅ Standard workflow
- **Forecasting Libraries**: 
  - `statsmodels` (ARIMA, state space models) - ✅ Standard Python
  - `prophet` (Facebook's forecasting library) - ✅ Available via pip
  - `scikit-learn` (ensemble methods) - ✅ Standard Python
  - `scipy`, `numpy` (statistical functions) - ✅ Standard Python
- **Visualization**: Interactive forecast charts with confidence bands (Plotly), publication-quality static charts (matplotlib)
- **Statistical Tools**: STATA available for advanced time series if needed - ✅ Listed in tech stack

**Technical Feasibility**: ✅ **CONFIRMED - No gaps identified**
- Time series data preparation: Polars for efficient reshaping, interpolation, feature engineering
- Forecasting: Multiple Python libraries with mature time series capabilities
- Scenario modeling: NumPy/SciPy for Monte Carlo simulation
- Validation: Standard train-test split and backtesting workflows
- Deployment: Forecasts can be operationalized via scheduled Databricks jobs with model retraining

**Platform Constraints**: None - forecasting is well-supported analytical workflow

**Model Monitoring & Updating**:
- Quarterly comparison of forecasts vs actuals when new data released
- Annual model retraining with expanded historical data
- Automated alerts when actual mortality exceeds forecast confidence intervals

---

## Expected Outcomes and Deliverables

**Stakeholder Outcomes**:
1. **Proactive capacity planning**: Lead time for infrastructure and workforce investments before demand materializes
2. **Budget accuracy**: Improved multi-year healthcare expenditure forecasting (disease burden drives ~40% of healthcare costs)
3. **Risk management**: Early detection of deviation from expected disease trajectories
4. **Intervention prioritization**: Quantified impact of prevention programs on future disease burden
5. **Strategic foresight**: Evidence-based scenarios for best-case, expected-case, worst-case disease burden futures

**Concrete Deliverables**:

**1. 🔮 Predictive Models: Disease Burden Forecasting Models (2020-2025)**
- Trained time series models for each major disease (cancer, stroke, heart disease)
- Model artifacts: Serialized Python pickle/joblib files for operationalization
- Model documentation: Algorithm selection rationale, hyperparameters, validation metrics
- Backtesting results: Historical forecast accuracy (2015-2019 predictions vs actuals)
- Location: `shared/models/disease_burden_forecasting/`
- Access: Models callable via Python API for downstream applications

**2. 📊 Analytical Report: "Singapore Disease Burden Projections 2020-2025"**
- Executive summary with key forecast insights and capacity planning implications
- Disease-by-disease 5-year projections with uncertainty quantification (80%/95% CI)
- Scenario analysis: Baseline, optimistic (enhanced interventions), pessimistic (trend reversals)
- Capacity requirements: Projected clinical service demand (beds, specialists) by disease
- Intervention impact analysis: Quantified effect of prevention programs on future burden
- Early warning indicators: Metrics to monitor for forecast deviations
- Format: PDF report (40-50 pages) with embedded forecast visualizations
- Access: Published on MOH Strategic Planning portal, updated annually

**3. 📈 Interactive Dashboard: "Disease Burden Forecast Explorer"**
- Multi-disease forecast visualization with confidence intervals
- Scenario comparison view (baseline vs optimistic vs pessimistic)
- Capacity planning calculator: Translate projected mortality into service demand
- Actual-vs-forecast tracker: Monitor forecast accuracy as new data arrives
- Sensitivity analysis tool: Adjust intervention assumptions and see impact on forecasts
- Platform: Plotly Dash or Databricks Dashboard with annual automated updates
- Access: MOH strategic planning and finance divisions

**4. 📋 Curated Dataset: "Disease Burden Forecasts 2020-2025"**
- Annual forecasts by disease with prediction intervals
- Scenario-specific projections (baseline, optimistic, pessimistic)
- Derived capacity requirements (hospital beds, clinician FTEs)
- Forecast accuracy metrics (updated quarterly as actuals arrive)
- Format: Parquet file with comprehensive metadata
- Location: `shared/data/4_processed/disease_burden_forecasts_2020_2025.parquet`
- Access: Available for downstream healthcare planning models

**5. 📑 Policy Brief: "Healthcare Capacity Investment Roadmap 2020-2025"**
- Prioritized list of capacity expansion needs by disease and year
- Investment timeline: When to initiate infrastructure/workforce expansion projects
- Risk areas: Diseases with high forecast uncertainty requiring monitoring
- Quick wins: High-confidence forecasts enabling immediate action
- Format: Executive brief (3 pages) + data appendix (2 pages)
- Audience: MOH Permanent Secretary, Chief of Strategic Planning, Finance Division

**6. 🔄 Forecast Monitoring System**
- Automated quarterly reports: Actual vs forecast comparison
- Alert mechanism: Flag when actuals exceed 95% confidence interval
- Model performance dashboard: Track forecast accuracy over time
- Recommendation engine: Suggest model retraining when drift detected
- Platform: Databricks scheduled jobs with email/Slack notifications
- Access: MOH Analytics Team, Strategic Planning Division

---

## Dependencies and Assumptions

**Problem Statement Dependencies**:
- **Recommended (but not required)**: PS-002 (Disease Burden Temporal Trends) provides foundational trend analysis that informs forecasting approach
  - If PS-002 completed: Use identified inflection points and trend periods to improve forecast models
  - If PS-002 not completed: Conduct exploratory trend analysis within forecasting workflow

**Related Problem Statements**:
- **Complements PS-001** (Healthcare Workforce Sustainability): Workforce projections can use disease burden forecasts as demand drivers
- **Complements PS-003** (Healthcare Capacity Optimization): Capacity planning uses forecasted disease burden as input
- **Informs PS-004** (Healthcare Expenditure Drivers): Disease burden projections drive cost forecasting

**Key Assumptions**:

**⚠️ CRITICAL - Analytical Assumptions (NOT Data Assumptions)**:

**Assumption 1**: Historical trends (1990-2019) are reasonable predictors of 2020-2025 trends absent major disruptions
- **Validation**: Test for trend stability; examine 5-year backtests
- **Mitigation**: Create multiple scenarios to bound uncertainty; clearly communicate COVID-19 disruption caveat

**Assumption 2**: Age-standardized mortality rates project future disease burden (not confounded by population aging)
- **Justification**: Age standardization removes demographic effects; valid for projection
- **Validation**: Compare age-standardized vs crude rate projections; verify consistency

**Assumption 3**: 30-year time series (1990-2019) provides sufficient data for 5-year forecasts
- **Justification**: Standard in epidemiological forecasting; allows capture of long-term cycles
- **Validation**: Sensitivity analysis with different training period lengths (20-year vs 30-year)

**Assumption 4**: Mortality projections are valid proxies for healthcare capacity demand
- **Justification**: Mortality represents severe disease burden requiring intensive healthcare resources
- **Enhancement**: Incorporate hospital admission trends to validate and refine capacity projections

**Assumption 5**: Singapore-specific trends are more predictive than international benchmarks
- **Approach**: Lead with Singapore historical trends; use international data for validation and scenario bounds
- **Validation**: Compare Singapore-based forecasts vs adapted international projections

**Assumption 6**: COVID-19 pandemic (2020-2022) represents structural break not fully predictable from historical data
- **Mitigation**: Position forecasts as "pre-COVID baseline"; develop COVID-adjustment scenarios; recommend urgent model update when 2020+ data available
- **Communication**: Clearly state forecast validity period and limitations

**Data Verified Against `data-sources.md`** - ✅ No unverified data assumptions

---

## Risks and Open Questions

**Potential Blockers**:

1. **Risk**: COVID-19 pandemic disrupted disease patterns; 2020-2025 forecasts may have limited validity
   - **Severity**: HIGH - fundamental assumption violation
   - **Mitigation**: 
     - Position as "counterfactual baseline" (what would have happened without COVID)
     - Develop COVID-impact scenarios based on international literature
     - Recommend urgent forecast update when Singapore 2020-2025 data available
     - Focus on 2023-2027 projection period instead (post-COVID normalization)

2. **Risk**: Limited disease coverage (only 3 diseases) may not justify large-scale capacity planning
   - **Severity**: MEDIUM - scope limitation
   - **Mitigation**: These 3 diseases account for ~60% of total mortality and majority of specialized clinical capacity (oncology, cardiology, neurology)
   - **Future expansion**: Framework designed to accommodate additional diseases when data obtained

3. **Risk**: Forecast uncertainty may be too wide for actionable capacity planning
   - **Severity**: MEDIUM - deliverable utility
   - **Mitigation**: 
     - Use ensemble models to narrow confidence intervals
     - Provide scenario-based planning (plan for range, not point estimate)
     - Identify "no-regret" investments robust across scenarios

4. **Risk**: Stakeholders may misinterpret probabilistic forecasts as deterministic predictions
   - **Severity**: MEDIUM - communication challenge
   - **Mitigation**: 
     - Extensive stakeholder training on uncertainty interpretation
     - Always present confidence intervals, never just point forecasts
     - Scenario narratives to make uncertainty tangible
     - Regular forecast-vs-actual reviews to build trust in probabilistic approach

5. **Risk**: Model may not capture structural breaks (e.g., new treatment breakthroughs, policy changes)
   - **Severity**: MEDIUM - model limitation
   - **Mitigation**:
     - Implement forecast monitoring system to detect deviations
     - Conduct sensitivity analysis on intervention effectiveness
     - Build "what-if" scenario capability for stakeholder-specified events

**Open Questions for Stakeholder Clarification**:

1. **Forecast horizon**: Is 5-year projection (2020-2025) the right planning horizon, or should we extend to 2030?
   - **Implications**: Longer horizons increase uncertainty but align better with infrastructure planning cycles

2. **Forecast granularity**: Do stakeholders need disease sub-type forecasts (e.g., lung cancer vs breast cancer vs colorectal cancer)?
   - **Data Check**: Verify if sub-type mortality data exists in dataset

3. **Intervention scenarios**: Which specific prevention/intervention programs should be modeled for scenario analysis?
   - **Examples**: Expanded cancer screening, smoking cessation campaigns, diabetes control programs

4. **Update frequency**: Should forecasts be updated annually, bi-annually, or quarterly as new data arrives?
   - **Resource trade-off**: More frequent updates improve accuracy but require ongoing analytics capacity

5. **Integration with other planning tools**: Are there existing capacity planning or budget models that need to consume these forecasts?
   - **Technical requirement**: May need specific output formats or APIs

6. **Acceptable forecast error**: What level of forecast accuracy is required for capacity planning decisions?
   - **Stakeholder risk tolerance**: Helps set model performance targets and confidence interval widths

---

## Problem Statement Readiness

**This Problem Statement is ready for backlog refinement when**:
- [x] Data domains explicitly verified against `data_sources.md` - ✅ COMPLETE
- [x] Platform and technical feasibility confirmed (`tech_stack.md` reviewed) - ✅ COMPLETE
- [x] Problem statement can be decomposed into 5-10 user stories - ✅ READY (see below)
- [x] Deliverable format and access method defined - ✅ COMPLETE
- [ ] **COVID-19 impact strategy confirmed with stakeholders** - ⚠️ REQUIRES STAKEHOLDER INPUT

**Preliminary User Story Breakdown** (to be refined in Sprint Planning):

**Sprint 1-2: Data Preparation & Exploratory Modeling**
1. As a data scientist, I want to prepare and validate 30-year mortality time series data, so that I can ensure clean inputs for forecasting models
2. As a forecasting analyst, I want to conduct exploratory time series analysis (stationarity, seasonality, trend components), so that I can select appropriate modeling techniques
3. As a modeler, I want to implement multiple baseline forecasting methods (ARIMA, Exponential Smoothing, Prophet), so that I can compare performance

**Sprint 3-4: Model Development & Validation**
4. As a data scientist, I want to perform backtesting on 2015-2019 forecasts, so that I can quantify model accuracy and select best-performing approaches
5. As a forecasting analyst, I want to generate 5-year projections (2020-2025) with confidence intervals, so that stakeholders can understand forecast uncertainty
6. As a capacity planner, I want scenario-based forecasts (baseline/optimistic/pessimistic), so that I can plan for multiple futures

**Sprint 5-6: Capacity Translation & Deliverables**
7. As a healthcare planner, I want mortality projections translated into capacity requirements (beds, clinicians), so that I can size infrastructure investments
8. As a policy maker, I want an interactive forecast dashboard, so that I can explore scenarios and sensitivity to assumptions
9. As a strategic planner, I want a comprehensive forecast report with methodology and limitations, so that I can make informed planning decisions

**Sprint 7: Monitoring & Operationalization**
10. As an analytics manager, I want an automated forecast monitoring system, so that I can track accuracy and detect when models need updating

**Status**: ✅ **READY FOR BACKLOG** (pending COVID-19 strategy clarification)

---

## Relationship to PS-002 (Disease Burden Temporal Trends)

| Aspect | PS-002 (Temporal Trends) | PS-006 (Forecasting) |
|--------|--------------------------|----------------------|
| **Analysis Type** | Descriptive (retrospective) | Predictive (prospective) |
| **Time Focus** | Past 30 years (1990-2019) | Next 5 years (2020-2025 or 2023-2028) |
| **Questions Answered** | What happened? Why? | What will happen? What if? |
| **Methods** | Trend analysis, inflection detection, benchmarking | Time series forecasting, scenario modeling |
| **Stakeholder Value** | Understand disease burden evolution | Plan future capacity and investments |
| **Independence** | Standalone foundational analysis | Can leverage PS-002 insights but not dependent |
| **Deliverables** | Trend report, historical dashboard | Forecasts, projection dashboard, monitoring system |

**Recommended Sequencing**: PS-002 → PS-006 (but not required)
- **Synergy**: PS-002 trend insights inform PS-006 forecasting assumptions
- **Independence**: PS-006 can proceed standalone with self-contained exploratory analysis

---

**Created**: 2026-03-13  
**Last Updated**: 2026-03-13  
**Status**: Draft - Pending Stakeholder Review (COVID-19 Strategy)  
