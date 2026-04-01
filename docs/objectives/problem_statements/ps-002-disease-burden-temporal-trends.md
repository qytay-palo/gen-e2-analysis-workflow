# PS-002: National Disease Burden Temporal Trends Analysis

```yaml
problem_statement_id: PS-002
title: National Disease Burden Temporal Trends Analysis
analysis_category: Descriptive Analytics
dependencies: None
platform: HEALIX/Databricks
primary_language: Python (Polars)
estimated_sprints: 3-5
priority: P0 (Critical)
```

---

## Executive Summary

Currently, **MOH public health officials and policy makers** face **fragmented understanding of how major disease burdens have shifted over three decades**, which prevents **evidence-based prioritization of public health programs and resource allocation**. By **analyzing 30-year mortality trends (1990-2019) for Singapore's leading causes of death (cancer, stroke, heart disease) with demographic stratification**, we can **identify emerging health threats and declining disease burdens** resulting in **optimized public health investments aligned with actual population health needs**.

---

## Problem Statement Hypothesis (Value Proposition)

> We believe that **comprehensive temporal trend analysis of major disease mortality patterns for public health strategists and policy makers** will achieve **data-driven prioritization of disease prevention and control programs**. We'll know we're successful when we see **public health budgets reallocated based on emerging disease trends and policy documents citing our mortality trajectory analysis in program justification**.

---

## Objectives

**Objective 1**: Quantify three-decade mortality trends for Singapore's major disease burdens (1990-2019)
- Analyze age-standardized mortality rates for cancer, stroke, ischemic heart disease
- Calculate annual percentage change and trend acceleration/deceleration
- Identify inflection points and trend shifts over the 30-year period

**Objective 2**: Identify shifting disease burden patterns and emerging public health priorities
- Compare relative burden changes across disease categories
- Detect diseases showing concerning upward trends vs improving downward trends
- Quantify disease burden transitions (e.g., shifting from infectious to chronic diseases)

**Objective 3**: Benchmark Singapore's disease burden trends against international standards
- Compare mortality rate trajectories to WHO global averages
- Assess Singapore's performance relative to high-income country benchmarks
- Identify areas of exceptional success vs areas needing improvement

**Objective 4**: Generate actionable insights for public health program prioritization
- Rank diseases by mortality burden magnitude and trend direction
- Identify under-resourced disease areas given current burden
- Recommend resource reallocation based on evolving disease landscape

---

## Stakeholders and Value Proposition

**Primary Stakeholders**:
- **Disease Prevention & Control Division, MOH** - Public health program planning
- **Public Health Policy Unit, MOH** - Strategic health policy formulation
- **Healthcare Service Planning, MOH** - Clinical service capacity planning for disease-specific treatment
- **Preventive Health Programs, Health Promotion Board** - Prevention campaign targeting

**Business Value**:
- **Decision enabled**: Evidence-based public health program prioritization vs legacy-driven funding
- **Efficiency gain**: Optimize prevention spending by targeting diseases with greatest burden and controllable risk factors
- **Quality improvement**: Improve population health outcomes through targeted interventions
- **Risk reduction**: Early detection of emerging disease threats before becoming crises

---

## Data Requirements (High-Level)

**Critical Considerations**:

**✅ Data Availability CONFIRMED** (Reference: `docs/project_context/data-sources.md`):
- **Mortality Tables Available**:
  - `age-standardised-mortality-rate-for-cancer.csv` (30 years, 1990-2019)
  - `age-standardised-mortality-rate-for-stroke.csv` (30 years, 1990-2019)
  - `age-standardised-mortality-rate-for-ischaemic-heart-disease.csv` (30 years, 1990-2019)
  - Additional mortality tables for other diseases (if available in dataset)

**✅ Data Completeness**: 100% completeness, 30-year continuous time series (1990-2019)

**✅ Data Quality**: 
- Age-standardized rates (controls for demographic changes)
- Official MOH source via data.gov.sg
- Internationally comparable methodology

**✅ Data Granularity**:
- **Geographic**: National level - APPROPRIATE for national disease burden assessment
- **Temporal**: Annual data - IDEAL for long-term trend analysis
- **Demographic**: Age-standardized - CRITICAL for valid temporal comparisons

**Contextual Data for Richer Analysis**:
- **Public Health Programs**: Use `vaccination-and-immunisation-of-students-annual.csv` (33 records) and obesity data to link interventions to outcomes
- **Healthcare Utilization**: Use `hospital-admission-rate-by-age-and-sex.csv` (216 records) to correlate with mortality trends

**Privacy/Security**: Aggregated mortality statistics only - NO privacy concerns

**No Data Blockers Identified** - Core mortality data confirmed available

---

## Initial Considerations

**Analytical Approach**:
- **Type**: Descriptive time series analysis with comparative benchmarking
- **Methods**: 
  - Trend analysis (linear regression, segmented regression for inflection points)
  - Annual percentage change calculations
  - Joinpoint regression for trend change detection
  - Comparative analysis against external benchmarks (WHO, OECD)
- **Complexity**: Moderate - standard epidemiological trend analysis

**Platform Feasibility** (Reference: `docs/project_context/tech-stack.md`):

**✅ Platform Requirements VERIFIED**:
- **Primary Platform**: HEALIX/Databricks - ✅ Confirmed in tech stack
- **Primary Language**: Python (Polars for data wrangling) - ✅ Preferred in tech stack
- **Compute Requirements**: Local/single-node sufficient - small dataset (<100 records per disease)
- **Data Access Pattern**: Batch CSV processing - ✅ Fully supported
- **Analytics Libraries**: scipy, statsmodels for trend analysis; matplotlib/plotly for visualization - ✅ Standard Python libraries
- **Statistical Tools**: STATA available if advanced epidemiological models needed - ✅ Listed in tech stack

**Technical Feasibility**: ✅ **CONFIRMED - No gaps identified**
- Polars: Efficient time series manipulation and reshaping
- Statistical analysis: Standard Python (scipy, statsmodels) or STATA for advanced models
- Visualization: Publication-quality trend charts using matplotlib/seaborn or interactive Plotly dashboards
- Benchmarking: Simple data joins with external reference datasets

**Platform Constraints**: None - straightforward analytical workflow

---

## Expected Outcomes and Deliverables

**Stakeholder Outcomes**:
1. **Clear disease burden prioritization**: Objective evidence for which diseases require increased focus
2. **Trend awareness**: Early detection of emerging or worsening disease problems
3. **Performance transparency**: Understand Singapore's standing relative to international peers
4. **Investment justification**: Data-backed evidence for public health funding requests

**Concrete Deliverables**:

**1. 📊 Analytical Report: "Singapore Disease Burden Evolution: 30-Year Trends (1990-2019)"**
- Executive summary highlighting key trend shifts and policy implications
- Disease-by-disease trend analysis with statistical significance testing
- Comparative benchmarking section (Singapore vs global/high-income countries)
- Public health recommendations with prioritized action areas
- Format: PDF report (30-40 pages) with embedded visualizations
- Access: Published on MOH internal knowledge portal

**2. 📈 Interactive Dashboard: "Disease Burden Trends Explorer"**
- Multi-disease trend visualization with selectable time ranges
- Comparative view: Singapore vs WHO/OECD benchmarks
- Demographic drill-down (if sub-group data available)
- Exportable charts for presentations and policy briefs
- Platform: Plotly Dash or Databricks Dashboard
- Access: MOH public health division web portal, updated annually

**3. 📋 Curated Dataset: "Disease Burden Metrics 1990-2019"**
- Consolidated mortality trends across all major diseases
- Calculated metrics: annual percentage change, trend periods, inflection points
- Benchmark comparisons integrated
- Format: Parquet file with comprehensive data dictionary
- Location: `shared/data/4_processed/disease_burden_trends.parquet`
- Access: Available for downstream epidemiological research

**4. 📑 Policy Brief: "Public Health Program Prioritization Recommendations"**
- Top 5 disease areas requiring immediate attention
- Top 5 disease areas showing successful control (continue current strategy)
- Emerging disease threats requiring new programs
- Format: 2-page executive brief with 1-page supporting data appendix
- Audience: MOH senior leadership, Minister for Health

---

## Dependencies and Assumptions

**Problem Statement Dependencies**: None (independent foundational analysis)

**Related Problem Statements**:
- May inform **PS-004: Public Health Program Effectiveness** (if that problem is created)
- Complements workforce planning by identifying disease-driven clinical service needs

**Key Assumptions**:

**⚠️ CRITICAL - These are analytical assumptions, NOT data assumptions**:

- **Assumption 1**: Age-standardized mortality rates are valid measures of disease burden trends
  - **Validation approach**: Standard epidemiological methodology; widely accepted in public health literature
  
- **Assumption 2**: Mortality trends reflect underlying disease burden (not just treatment improvements)
  - **Validation approach**: Acknowledge limitation in report; recommend pairing with incidence data when available

- **Assumption 3**: International benchmarks (WHO, OECD) are appropriate comparators for Singapore
  - **Validation approach**: Compare only against high-income countries with similar healthcare systems

- **Assumption 4**: 1990-2019 trends are informative for 2020+ planning despite COVID-19 disruption
  - **Mitigation**: Clearly note that COVID-19 pandemic (2020+) may alter disease burden landscape; recommend update when 2020+ data available

**Data has been VERIFIED against `data-sources.md` - no data assumptions made**

---

## Risks and Open Questions

**Potential Blockers**:
1. **Risk**: Limited disease coverage (only cancer, stroke, heart disease confirmed)
   - **Mitigation**: Focus analysis on available diseases; clearly state scope limitations; recommend expansion when additional mortality data obtained
   
2. **Risk**: No sub-group mortality breakdowns (age, gender, ethnicity) confirmed
   - **Mitigation**: Conduct initial analysis with available data; flag as enhancement opportunity if demographic stratification exists

3. **Risk**: COVID-19 pandemic (2020-2022) drastically changed mortality patterns
   - **Mitigation**: Clearly communicate that analysis covers pre-COVID era; position as "baseline" for understanding post-COVID shifts

4. **Risk**: Mortality may not capture full disease burden (non-fatal conditions)
   - **Mitigation**: Acknowledge limitation; recommend future analyses incorporating morbidity/disability-adjusted life years (DALYs) if data available

**Open Questions for Stakeholder Clarification**:
1. Are there specific diseases beyond cancer/stroke/heart disease that should be prioritized?
2. What international benchmarks are most relevant to MOH (WHO, OECD, specific countries)?
3. What is the intended use for this analysis (budget planning, policy justification, academic publication)?
4. Should analysis incorporate healthcare intervention milestones (e.g., new screening programs launched)?

---

## Problem Statement Readiness

**This Problem Statement is ready for backlog refinement when**:
- [x] Data domains explicitly verified against `data_sources.md` - ✅ COMPLETE
- [x] Platform and technical feasibility confirmed (`tech_stack.md` reviewed) - ✅ COMPLETE
- [x] Problem statement can be decomposed into 5-10 user stories - ✅ READY (see below)
- [x] Deliverable format and access method defined - ✅ COMPLETE

**Preliminary User Story Breakdown** (to be refined in Sprint Planning):
1. As a public health analyst, I want to visualize 30-year mortality trends for each major disease, so that I can identify long-term patterns
2. As a policy maker, I want to see annual percentage change calculations, so that I can quantify trend magnitudes
3. As a strategic planner, I want comparative benchmarking against international standards, so that I can assess Singapore's performance
4. As a program director, I want disease burden ranking by magnitude and trend, so that I can prioritize interventions
5. As an epidemiologist, I want inflection point detection, so that I can identify when disease trends shifted
6. As an executive, I want an interactive trend explorer dashboard, so that I can dynamically analyze disease patterns
7. As a researcher, I want publication-quality trend charts, so that I can communicate findings effectively
8. As a data consumer, I want a consolidated disease burden dataset, so that I can conduct further analyses

**Status**: ✅ **READY FOR BACKLOG**

---

**Created**: 2026-03-11  
**Last Updated**: 2026-03-11  
**Status**: Draft - Awaiting Stakeholder Review  
**Owner**: TBD (assign during sprint planning)
