# PS-001: Healthcare Workforce Sustainability Analysis

```yaml
problem_statement_id: PS-001
title: Healthcare Workforce Sustainability Analysis
analysis_category: Predictive Analytics
dependencies: None
platform: HEALIX/Databricks
primary_language: Python (Polars)
estimated_sprints: 4-6
priority: P0 (Critical)
```

---

## Executive Summary

Currently, **MOH policy makers and workforce planners** face **uncertainty about future healthcare manpower needs** as Singapore's aging population drives increasing demand, which prevents **proactive recruitment and training strategies**. By **analyzing 14-year workforce trends (2006-2019) across all healthcare sectors and forecasting future supply gaps through 2030**, we can **enable evidence-based workforce planning decisions** resulting in **reduced manpower shortages and improved healthcare system resilience**.

---

## Problem Statement Hypothesis (Value Proposition)

> We believe that **forecasting healthcare workforce supply and demand across all sectors (doctors, nurses, allied health) for workforce planners and policy makers** will achieve **proactive manpower planning and targeted recruitment strategies**. We'll know we're successful when we see **workforce planners using forecasts to justify training program expansion and policy makers citing projections in budget allocation decisions**.

---

## Objectives

**Objective 1**: Quantify historical workforce growth patterns across all healthcare sectors (2006-2019)
- Analyze annual growth rates for doctors, nurses, pharmacists, dentists, and allied health professionals
- Identify sector-specific trends and variations in public vs private sectors
- Calculate workforce-to-population ratios over time

**Objective 2**: Develop forecasting models for future workforce supply through 2030
- Build time series models projecting workforce numbers under current growth scenarios
- Incorporate demographic trends (aging population, healthcare demand growth)
- Generate confidence intervals for workforce projections

**Objective 3**: Identify critical workforce gaps and shortage risks by sector
- Compare projected supply against estimated demand based on population aging
- Quantify magnitude and timing of potential shortages
- Prioritize sectors requiring immediate intervention

**Objective 4**: Provide scenario analysis for policy interventions
- Model impact of increased training program capacity
- Simulate effects of retention improvement initiatives
- Quantify workforce import requirements under different scenarios

---

## Stakeholders and Value Proposition

**Primary Stakeholders**:
- **Workforce Planning Division, MOH** - Strategic manpower planning
- **Healthcare Professional Regulatory Bodies** - Training program capacity decisions
- **Finance & Budget Office, MOH** - Budget allocation for training programs
- **Healthcare Institutions (Public Hospitals)** - Long-term staffing strategies

**Business Value**:
- **Decision enabled**: Proactive manpower planning vs reactive crisis management
- **Efficiency gain**: Reduce time lag between identifying shortages and producing qualified professionals (typically 4-8 years for doctors)
- **Quality improvement**: Prevent understaffing-related burnout and quality issues
- **Risk reduction**: Mitigate healthcare capacity crises during demand surges

---

## Data Requirements (High-Level)

**Critical Considerations**:

**✅ Data Availability CONFIRMED** (Reference: `docs/project_context/data-sources.md`):
- **Workforce Tables Available**:
  - `number-of-doctors.csv` (78 records, 2006-2019, by sector)
  - `number-of-nurses-and-midwives.csv` (126 records, 2008-2019, by sector)
  - `number-of-pharmacists.csv` (42 records, 2006-2019, by sector)
  - `number-of-dentists.csv` (available in dataset)
  - Additional allied health professional tables

**✅ Data Completeness**: 100% completeness, no missing values, standardized annual data

**✅ Data Quality**: Official MOH source via data.gov.sg, validated government statistics

**⚠️ Data Granularity Constraints**:
- **Geographic**: National level only (no regional breakdowns) - ACCEPTABLE for national workforce planning
- **Temporal**: Annual data only (not monthly/quarterly) - SUFFICIENT for long-term forecasting
- **Latest Data**: Through 2019 (dataset from April 2020) - Use 2019 as baseline, forecast 2020-2030

**Privacy/Security**: No individual-level data, aggregated statistics only - NO privacy concerns

**Proxy Data for Demand Estimation**:
- **Population Demographics**: Use `hospital-admission-rate-by-age-and-sex.csv` (216 records) as proxy for demand growth
- **Facilities Growth**: Use `health-facilities-and-beds-in-inpatient-facilities.csv` (180 records) to validate demand trends

**No Data Blockers Identified** - All required data domains present in Kaggle dataset

---

## Initial Considerations

**Analytical Approach**:
- **Type**: Time series forecasting with scenario analysis
- **Methods**: ARIMA/SARIMAX for trend projection, linear regression for sector-specific growth, Monte Carlo simulation for confidence intervals
- **Complexity**: Moderate - standard forecasting techniques appropriate for annual data

**Platform Feasibility** (Reference: `docs/project_context/tech-stack.md`):

**✅ Platform Requirements VERIFIED**:
- **Primary Platform**: HEALIX/Databricks - ✅ Confirmed in tech stack
- **Primary Language**: Python (Polars for data processing) - ✅ Preferred in tech stack
- **Compute Requirements**: Local/single-node sufficient - dataset <5MB, 1,521 total records
- **Data Access Pattern**: Batch CSV processing - ✅ Fully supported
- **Analytics Libraries**: statsmodels, prophet, or scikit-learn for forecasting - ✅ Available in Python ecosystem

**Technical Feasibility**: ✅ **CONFIRMED - No gaps identified**
- Polars: Efficient data manipulation for joining workforce tables
- Time series libraries: Standard Python libraries (statsmodels, prophet)
- Visualization: Plotly/Altair for interactive dashboards (approved in tech stack)
- No specialized tools required beyond standard Python data science stack

**Platform Constraints**: None - standard analytical workflow suitable for Databricks Python environment

---

## Expected Outcomes and Deliverables

**Stakeholder Outcomes**:
1. **Evidence-based workforce planning**: Replace ad-hoc estimates with data-driven forecasts
2. **Early warning system**: Identify emerging shortages 3-5 years in advance
3. **Scenario planning capability**: Evaluate policy intervention effectiveness before implementation
4. **Budget justification**: Quantified evidence for training program investments

**Concrete Deliverables**:

**1. 📊 Analytical Report: "Singapore Healthcare Workforce Sustainability 2020-2030"**
- Executive summary with key findings and recommendations
- Detailed sector-by-sector workforce projections with confidence intervals
- Gap analysis identifying critical shortage risks
- Policy recommendations with prioritized interventions
- Format: PDF report with embedded visualizations
- Access: Shared via MOH internal portal

**2. 📈 Interactive Dashboard: "Workforce Planning Dashboard"**
- Real-time visualization of workforce trends and projections
- Sector drill-down functionality (doctors, nurses, allied health)
- Scenario comparison tool (baseline vs policy interventions)
- Export capability for presentations and reports
- Platform: Plotly Dash or Databricks Dashboard
- Access: MOH workforce planning division web portal

**3. 📋 Curated Dataset: "Workforce Projections 2020-2030"**
- Annual workforce projections by sector through 2030
- Upper/lower bounds for each projection
- Demand estimates and supply-demand gap calculations
- Format: Parquet files with data dictionary
- Location: `shared/data/4_processed/workforce_projections.parquet`
- Access: Available for downstream analyses and integration into existing MOH systems

**4. 📑 Policy Recommendations Document**
- Prioritized list of sectors requiring immediate intervention
- Quantified impact of proposed training program expansions
- Timeline and milestones for addressing critical gaps
- Format: Executive brief (5-10 pages) with supporting appendices
- Audience: Senior MOH leadership and Ministers

---

## Dependencies and Assumptions

**Problem Statement Dependencies**: None (foundational analysis)

**Related Problem Statements**:
- May inform future capacity planning analyses
- Complements facility infrastructure planning

**Key Assumptions**:

**⚠️ CRITICAL - These are analytical assumptions, NOT data assumptions**:
- **Assumption 1**: Historical workforce growth patterns (2006-2019) are reasonably predictive of future trends absent major policy changes
  - **Validation approach**: Test forecasts against 2017-2019 actuals using 2006-2016 training data
  
- **Assumption 2**: Population aging trends follow Singapore Department of Statistics projections
  - **Validation approach**: Reference official demographic forecasts; sensitivity test with alternative scenarios

- **Assumption 3**: Workforce-to-population ratios provide reasonable demand estimates
  - **Validation approach**: Benchmark against international standards (WHO, OECD)

**Data has been VERIFIED against `data-sources.md` - no data assumptions made**

---

## Risks and Open Questions

**Potential Blockers**:
1. **Risk**: Forecasting accuracy may be limited by data ending in 2019 (now 7-year lag)
   - **Mitigation**: Clearly communicate forecast uncertainty; update models when newer data becomes available
   
2. **Risk**: COVID-19 pandemic (2020-2022) may have disrupted historical patterns
   - **Mitigation**: Develop multiple scenarios incorporating pandemic impact; recommend updating analysis when post-COVID data available

3. **Risk**: Policy interventions already implemented (2020-2026) not captured in historical data
   - **Mitigation**: Engage stakeholders to document recent initiatives; incorporate as scenario adjustments

**Open Questions for Stakeholder Clarification**:
1. What retention rate assumptions should we use for current workforce?
2. Are there planned training program expansions we should incorporate?
3. What international recruitment targets exist?
4. What workforce productivity improvements are anticipated (technology, task-shifting)?

---

## Problem Statement Readiness

**This Problem Statement is ready for backlog refinement when**:
- [x] Data domains explicitly verified against `data_sources.md` - ✅ COMPLETE
- [x] Platform and technical feasibility confirmed (`tech_stack.md` reviewed) - ✅ COMPLETE
- [x] Problem statement can be decomposed into 5-10 user stories - ✅ READY (see below)
- [x] Deliverable format and access method defined - ✅ COMPLETE

**Preliminary User Story Breakdown** (to be refined in Sprint Planning):
1. As a workforce planner, I want to see historical trends for each healthcare sector, so that I understand past growth patterns
2. As a policy maker, I want workforce projections through 2030, so that I can plan training program expansions
3. As a budget officer, I want to quantify workforce gaps by sector, so that I can allocate funding appropriately
4. As a strategy analyst, I want scenario analysis of policy interventions, so that I can evaluate options before implementation
5. As an executive, I want an interactive dashboard, so that I can explore workforce data dynamically
6. As a communications officer, I want export-ready visualizations, so that I can create presentations for Ministers
7. As an analyst, I want a reusable forecasting pipeline, so that I can update projections as new data arrives
8. As a data consumer, I want validated projection datasets, so that I can integrate into downstream analyses

**Status**: ✅ **READY FOR BACKLOG**

---

**Created**: 2026-03-11  
**Last Updated**: 2026-03-11  
**Status**: Draft - Awaiting Stakeholder Review  
**Owner**: TBD (assign during sprint planning)
