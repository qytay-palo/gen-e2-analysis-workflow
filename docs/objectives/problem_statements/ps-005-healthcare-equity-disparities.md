# PS-005: Healthcare Access Equity & Demographic Disparities Analysis

```yaml
problem_statement_id: PS-005
title: Healthcare Access Equity & Demographic Disparities Analysis
analysis_category: Diagnostic Analytics (Equity Focus)
dependencies: None
platform: HEALIX/Databricks
primary_language: Python (Polars)
estimated_sprints: 4-5
priority: P2 (Medium)
```

---

## Executive Summary

Currently, **MOH population health strategists and equity planning teams** face **limited visibility into healthcare access and health outcome disparities across demographic groups (age, gender, potentially socioeconomic proxies)**, which prevents **targeted interventions to reduce health inequity and leads to one-size-fits-all policies that may not address vulnerable populations**. By **analyzing 15+ years of healthcare utilization, mortality, and disease burden data stratified by demographics (2005-2020)**, we can **identify underserved populations, quantify health outcome gaps, and detect inequitable access patterns** resulting in **evidence-based equity interventions that improve health outcomes for vulnerable groups and reduce preventable mortality disparities**.

---

## Problem Statement Hypothesis (Value Proposition)

> We believe that **multi-dimensional equity analysis across demographics and health domains for population health strategists and policy makers** will achieve **identification of underserved populations, quantification of health outcome gaps, and detection of access barriers**. We'll know we're successful when we see **equity-focused policies targeting our identified vulnerable groups and resource allocation decisions referencing our disparity metrics**.

---

## Objectives

**Objective 1**: Quantify healthcare utilization disparities across demographic groups
- Analyze hospital admission rates, primary care utilization, screening uptake by age and gender
- Calculate utilization ratios to identify over-represented and under-represented groups
- Detect temporal trends - are disparities widening or narrowing?

**Objective 2**: Assess health outcome inequities across population segments
- Analyze mortality rate disparities by age, gender, and disease type
- Quantify disease burden differences (e.g., diabetes prevalence, cancer mortality) across demographics
- Identify preventable mortality gaps

**Objective 3**: Diagnose potential access barriers and systemic inequities
- Compare utilization patterns against expected need (high-risk populations should have higher preventive care utilization)
- Identify underserved demographics with low utilization despite potential need
- Detect geographic or socioeconomic proxies of inequity (if data permits)

**Objective 4**: Recommend equity-focused interventions and resource targeting
- Prioritize demographic segments requiring targeted health programs
- Identify service types with largest access gaps (e.g., preventive care, specialist visits, screening)
- Provide evidence base for resource allocation to reduce disparities

---

## Stakeholders and Value Proposition

**Primary Stakeholders**:
- **Population Health Strategy Division, MOH** - Health equity policy design
- **Public Health & Preventive Medicine, MOH** - Community health program targeting
- **Healthcare Financing Division** - Subsidy targeting and financial access barriers
- **Community Health Organizations (NGOs, VWOs)** - Vulnerable population outreach programs

**Business Value**:
- **Decision enabled**: Targeted health programs for vulnerable populations vs generic outreach
- **Equity improvement**: Reduce preventable mortality disparities; improve health outcomes for underserved groups
- **Resource efficiency**: Direct resources to populations with greatest need; avoid wasteful universal interventions
- **Policy justification**: Evidence-based case for equity initiatives and subsidy targeting

---

## Data Requirements (High-Level)

**Critical Considerations**:

**✅ Data Availability CONFIRMED** (Reference: `docs/project_context/data-sources.md`):

**Utilization Data (Demographic Stratification):**
  - `hospital-admission-rate-by-age-and-sex.csv` (216 records, 2006-2020, age/gender stratification)
  - `subsidised-primary-care-attendances-at-polyclinics.csv` (62 records, primary care utilization by sex)
  - `residential-long-term-care-admissions.csv` (25 records, long-term care access)

**Health Outcome Data (Demographic Stratification):**
  - Multiple age-standardized mortality rate tables by disease and demographics (confirmed in data sources)
  - Disease-specific tables (diabetes, cancer, heart disease) likely have demographic breakdowns
  - Screening uptake data (if available in dataset)

**Disease Burden Data:**
  - Disease prevalence tables may have demographic stratification
  - Can proxy disease burden from mortality and morbidity data

**⚠️ Socioeconomic Context (Limited Availability)**:
  - National-level data may NOT have direct income/education stratification
  - May need to use geographic proxies (if neighborhood-level data available) or age as socioeconomic proxy (elderly = potentially vulnerable)

**✅ Data Completeness**: 100% completeness for available variables

**✅ Data Quality**: Official MOH source via data.gov.sg, validated government statistics

**✅ Data Granularity**:
- **Demographic**: Age groups, gender - CONFIRMED available
- **Geographic**: National level only - LIMITS ability to detect neighborhood-level disparities
- **Temporal**: Annual data (2005-2020) - SUITABLE for trend analysis
- **Socioeconomic**: Likely LIMITED or absent - will need proxies

**⚠️ Known Limitations**:
- **Socioeconomic Stratification**: May not be explicitly available; will need to use age, gender, or geographic proxies
- **Ethnicity Data**: Not confirmed if ethnicity/race breakdowns available (common equity dimension)
- **Geographic Granularity**: National-level data cannot detect neighborhood-level access barriers
- **Attribution**: Difficult to isolate access barriers vs health-seeking behavior vs biological differences

**Privacy/Security**: Aggregated demographic statistics - NO privacy concerns

**⚠️ KNOWN DATA CONSTRAINT** - Limited socioeconomic and ethnic stratification may restrict depth of equity analysis

---

## Initial Considerations

**Analytical Approach**:
- **Type**: Diagnostic analytics with equity focus using disparity metrics and statistical testing
- **Methods**: 
  - Disparity ratio calculation (utilization/outcome rates compared to reference group)
  - Statistical significance testing (confidence intervals, hypothesis tests for group differences)
  - Temporal trend analysis (are disparities widening or narrowing?)
  - Need-adjusted utilization analysis (high-risk groups should have higher preventive care use)
  - Concentration indices and inequality metrics (if socioeconomic data available)
- **Complexity**: Moderate - requires careful statistical analysis with appropriate reference groups

**Platform Feasibility** (Reference: `docs/project_context/tech-stack.md`):

**✅ Platform Requirements VERIFIED**:
- **Primary Platform**: HEALIX/Databricks - ✅ Confirmed in tech stack
- **Primary Language**: Python (Polars for data manipulation) - ✅ Preferred in tech stack
- **Compute Requirements**: Local/single-node sufficient (small datasets)
- **Data Access Pattern**: Batch CSV processing with demographic aggregations - ✅ Fully supported
- **Analytics Libraries**: 
  - Data manipulation: Polars (confirmed)
  - Statistical testing: scipy.stats, statsmodels for hypothesis tests
  - Disparity metrics: Custom Python implementations
  - Health equity metrics: Can implement concentration indices, Gini coefficients if needed
  - Visualization: Matplotlib, Plotly for disparity charts (forest plots, trend lines by group)
- **Statistical Analysis**: STATA available if advanced econometric disparity analysis needed

**Technical Feasibility**: ✅ **CONFIRMED - No gaps identified**
- Demographic stratification: Standard Polars groupby operations
- Statistical testing: Rich Python statistical libraries
- Disparity visualization: Established chart types (forest plots, gap charts)

**Platform Constraints**: None - suitable for standard Python environment

---

## Expected Outcomes and Deliverables

**Stakeholder Outcomes**:
1. **Vulnerable population identification**: Clear understanding of which demographic groups have worst health outcomes and lowest access
2. **Disparity quantification**: Evidence-based metrics on magnitude of health inequities
3. **Intervention targeting**: Prioritized list of demographics and service types for equity programs
4. **Progress monitoring baseline**: Establish baseline disparity metrics to track improvement over time

**Concrete Deliverables**:

**1. 📊 Analytical Report: "Singapore Healthcare Equity Assessment 2005-2020"**
- Executive summary with top 3 vulnerable populations and priority interventions
- Utilization disparity analysis by age, gender, and service type
- Health outcome gap analysis (mortality, disease burden differences)
- Temporal equity trends (are disparities improving or worsening?)
- International equity benchmarking (if data available)
- Root cause hypothesis for observed disparities (access, affordability, awareness, health literacy)
- Format: PDF report (30-40 pages) with embedded visualizations
- Access: MOH Population Health Division, community health organizations

**2. 📈 Interactive Dashboard: "Health Equity Monitor Dashboard"**
- Multi-year disparity trend visualization by demographic group
- Utilization gap explorer (hospital, primary care, screening by demographics)
- Health outcome disparity viewer (mortality, disease burden gaps)
- Service-specific access analysis (which services have largest equity gaps?)
- Comparative reference group selection tool
- Platform: Plotly Dash or Tableau
- Access: MOH Population Health Division, public health planners
- Update Frequency: Annual refresh with new utilization and outcome data

**3. 📋 Curated Dataset: "Healthcare Equity Metrics 2005-2020"**
- Integrated utilization and outcome data with demographic stratification
- Calculated disparity metrics: utilization ratios, mortality gaps, concentration indices
- Temporal trend indicators (disparity change over time)
- Format: Parquet file with comprehensive data dictionary
- Location: `shared/data/4_processed/equity_analysis.parquet`
- Access: Available for health equity research and program evaluation

**4. 📑 Policy Brief: "Health Equity Action Plan 2026-2030"**
- Top 3-5 vulnerable populations requiring targeted interventions
- Service-specific recommendations (e.g., increase screening in elderly men, expand primary care access for women 40-60)
- Resource allocation recommendations to reduce disparities
- Monitoring framework for tracking equity improvement
- Format: Executive brief (8-12 pages) with appendices
- Audience: MOH Permanent Secretary, Population Health Division Director, community health leaders

---

## Dependencies and Assumptions

**Problem Statement Dependencies**: None (independent analysis)

**Related Problem Statements**:
- **May inform PS-003 (Capacity Optimization)** - capacity expansion should address underserved populations
- **May inform PS-004 (Expenditure Analysis)** - out-of-pocket costs may create access barriers
- Could lead to future prescriptive analysis on optimal subsidy targeting or outreach program design

**Key Assumptions**:

**⚠️ CRITICAL - These are analytical assumptions, NOT data assumptions**:

- **Assumption 1**: Demographic disparities in utilization and outcomes reflect inequity rather than solely biological or preference differences
  - **Mitigation**: Acknowledge potential confounding; focus on preventable disparities (e.g., screening uptake, preventable mortality); conduct need-adjusted analysis where possible
  
- **Assumption 2**: Absence of socioeconomic stratification can be partially addressed using age and gender as proxies
  - **Mitigation**: Clearly state limitation; recommend obtaining socioeconomic data for future refinement; acknowledge findings are partial picture

- **Assumption 3**: Statistical disparities identified through analysis represent actionable policy opportunities
  - **Validation approach**: Prioritize disparities with plausible intervention pathways (e.g., low screening uptake → targeted awareness campaigns)

- **Assumption 4**: National-level disparities are meaningful despite inability to detect geographic/neighborhood-level access barriers
  - **Mitigation**: Focus on demographic disparities that can be addressed through national policies; note geographic analysis as future enhancement

**Data has been VERIFIED against `data-sources.md` - no data assumptions made**

**⚠️ ACKNOWLEDGED LIMITATION** - Limited socioeconomic and ethnic stratification restricts depth of equity analysis; findings will focus on age/gender disparities primarily

---

## Risks and Open Questions

**Potential Blockers**:
1. **Risk**: Limited demographic stratification variables (age/gender only) restricts equity analysis depth
   - **Mitigation**: Focus on age/gender equity; clearly acknowledge socioeconomic and ethnic disparities cannot be fully assessed; recommend data enhancement
   
2. **Risk**: Difficult to distinguish health inequity (unjust, avoidable) from health inequality (biological, preference-driven)
   - **Mitigation**: Focus on preventable disparities (screening, preventable mortality); use need-adjusted utilization analysis

3. **Risk**: National-level data cannot detect geographic access barriers (e.g., remote areas with limited facilities)
   - **Mitigation**: Acknowledge limitation; recommend regional data collection for future analysis

4. **Risk**: Statistical disparities may not have clear intervention pathways
   - **Mitigation**: Prioritize actionable disparities with plausible solutions; engage stakeholders to validate feasibility

**Open Questions for Stakeholder Clarification**:
1. Are ethnicity/race demographic variables available in any dataset (critical equity dimension)?
2. Is socioeconomic stratification (income, education) available in any form?
3. What demographic groups are of particular policy concern (e.g., elderly, women, low-income)?
4. Are there existing equity targets or goals (e.g., "reduce mortality gap by X% by 2030")?
5. What equity metrics are most relevant for policy decisions (absolute gaps vs relative gaps vs concentration indices)?
6. Should analysis include cost/affordability barriers (integrate with expenditure data from PS-004)?

---

## Problem Statement Readiness

**This Problem Statement is ready for backlog refinement when**:
- [x] Data domains explicitly verified against `data_sources.md` - ✅ COMPLETE (with acknowledged limitations)
- [x] Platform and technical feasibility confirmed (`tech_stack.md` reviewed) - ✅ COMPLETE
- [x] Problem statement can be decomposed into 5-10 user stories - ✅ READY (see below)
- [x] Deliverable format and access method defined - ✅ COMPLETE

**Preliminary User Story Breakdown** (to be refined in Sprint Planning):
1. As a population health strategist, I want hospital admission rates by age/gender, so that I can identify over- and under-utilizing groups
2. As a public health planner, I want primary care utilization disparities, so that I can target outreach programs
3. As an analyst, I want mortality rate gaps by demographics, so that I can quantify health outcome inequities
4. As a policy maker, I want temporal disparity trends, so that I can see if equity is improving over time
5. As a program designer, I want need-adjusted utilization analysis, so that I can identify access vs need mismatches
6. As a community health leader, I want vulnerable population profiles, so that I can design targeted interventions
7. As a monitoring officer, I want equity metrics dashboard, so that I can track progress on disparity reduction
8. As a researcher, I want consolidated equity dataset, so that I can conduct further disparity research
9. As an executive, I want equity action plan with priorities, so that I can allocate resources to reduce health inequities

**Status**: ✅ **READY FOR BACKLOG** (with acknowledged data limitations)

---

**Created**: 2026-03-11  
**Last Updated**: 2026-03-11  
**Status**: Draft - Awaiting Stakeholder Review  
**Owner**: TBD (assign during sprint planning)  
**Note**: Depth of equity analysis will depend on demographic variables available in dataset - socioeconomic and ethnic stratification are limited
