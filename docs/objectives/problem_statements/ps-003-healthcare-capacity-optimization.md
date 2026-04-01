# PS-003: Healthcare System Capacity & Utilization Optimization

```yaml
problem_statement_id: PS-003
title: Healthcare System Capacity & Utilization Optimization
analysis_category: Prescriptive Analytics
dependencies: None
platform: HEALIX/Databricks
primary_language: Python (Polars)
estimated_sprints: 5-7
priority: P1 (High)
```

---

## Executive Summary

Currently, **MOH healthcare service planning teams and hospital administrators** face **uncertainty about optimal capacity allocation across different facility types (acute, intermediate, primary care)**, which prevents **efficient resource deployment and leads to potential over-capacity in some areas while bottlenecks exist in others**. By **analyzing 11-year facility and bed capacity trends (2009-2020) alongside demographic-stratified utilization patterns (hospital admission rates, long-term care demand)**, we can **identify capacity gaps, utilization imbalances, and optimization opportunities** resulting in **improved bed occupancy efficiency, reduced waiting times, and better-targeted infrastructure investments**.

---

## Problem Statement Hypothesis (Value Proposition)

> We believe that **integrated capacity and utilization analysis across acute, intermediate, and primary care facilities for healthcare planners and administrators** will achieve **optimized bed capacity allocation, identification of under-served segments, and evidence-based infrastructure investment decisions**. We'll know we're successful when we see **capacity planning decisions referencing our utilization efficiency metrics and infrastructure investment proposals justified by our gap analysis**.

---

## Objectives

**Objective 1**: Quantify national healthcare capacity evolution across all facility types (2009-2020)
- Analyze trends in hospital beds, clinic capacity, long-term care facilities
- Compare growth rates across public vs private sectors
- Calculate capacity-to-population ratios and changes over time

**Objective 2**: Assess healthcare utilization patterns and identify demand drivers
- Analyze hospital admission rates by age, gender, and demographics (2006-2020)
- Quantify utilization versus capacity ratios (proxy for bed occupancy/strain)
- Identify demographic segments with disproportionate utilization

**Objective 3**: Identify capacity gaps, surpluses, and misalignment with demand
- Compare capacity growth against utilization growth trends
- Detect facility types with insufficient capacity relative to demand
- Identify potential over-investment areas with low utilization

**Objective 4**: Recommend capacity optimization strategies and infrastructure priorities
- Prioritize facility types and sectors requiring capacity expansion
- Identify opportunities for shifting care from acute to intermediate/primary settings
- Provide scenario analysis for infrastructure investment options

---

## Stakeholders and Value Proposition

**Primary Stakeholders**:
- **Healthcare Service Planning Division, MOH** - National capacity planning and infrastructure strategy
- **Hospital Administrators (Public Healthcare Clusters)** - Facility-level capacity management
- **Finance & Infrastructure Development, MOH** - Capital investment prioritization
- **Intermediate & Long-Term Care Planning, MOH** - Non-acute capacity expansion

**Business Value**:
- **Decision enabled**: Evidence-based capacity planning vs reactive expansion during crises
- **Efficiency gain**: Optimize bed occupancy rates; reduce unnecessary capacity costs; shift appropriate care from expensive acute to cost-effective intermediate/primary settings
- **Quality improvement**: Reduce waiting times and overcrowding through proactive capacity management
- **Risk reduction**: Mitigate future capacity crises through early detection of emerging gaps

---

## Data Requirements (High-Level)

**Critical Considerations**:

**✅ Data Availability CONFIRMED** (Reference: `docs/project_context/data-sources.md`):

**Capacity Data:**
  - `health-facilities-and-beds-in-inpatient-facilities-public-not-for-profit-private.csv` (180 records, detailed facility breakdowns)
  - `health-facilities-primary-care-dental-clinics-and-pharmacies.csv` (96 records, primary care capacity)
  - `residential-long-term-care-admissions.csv` (25 records, intermediate care capacity proxy)

**Utilization Data:**
  - `hospital-admission-rate-by-age-and-sex.csv` (216 records, 2006-2020, demographic stratification)
  - `residential-long-term-care-admissions.csv` (25 records, long-term care utilization)

**Population Context:**
  - Can derive population trends from admission rate denominators
  - External demographic projections may supplement analysis

**✅ Data Completeness**: 100% completeness, no missing values

**✅ Data Quality**: Official MOH source via data.gov.sg, validated government statistics

**✅ Data Granularity**:
- **Geographic**: National level - APPROPRIATE for national capacity planning; may have sector breakdowns (public/private)
- **Temporal**: Annual data (2006-2020) - SUITABLE for strategic planning
- **Facility Types**: Detailed facility breakdowns likely available (acute, community hospitals, nursing homes, clinics)
- **Demographics**: Age, gender stratification for utilization data

**⚠️ Known Limitations**:
- **Bed Occupancy Rates**: May not be directly available; will calculate proxy using capacity vs utilization ratios
- **Regional Distribution**: National-level data only; cannot assess geographic maldistribution
- **Facility-Specific Performance**: Aggregated data; cannot identify individual hospital performance

**Privacy/Security**: Aggregated facility and utilization statistics - NO privacy concerns

**No Data Blockers Identified** - Core capacity and utilization data confirmed available

---

## Initial Considerations

**Analytical Approach**:
- **Type**: Prescriptive analytics combining descriptive trend analysis with optimization scenario modeling
- **Methods**: 
  - Capacity-demand gap analysis (comparing growth trajectories)
  - Utilization efficiency metrics (proxy occupancy rates)
  - Demographic segment analysis (identifying high-burden populations)
  - Scenario modeling for capacity expansion options
  - Optimization algorithms for capacity allocation (if data granularity permits)
- **Complexity**: Moderate to High - requires integration of multiple data sources and scenario modeling

**Platform Feasibility** (Reference: `docs/project_context/tech-stack.md`):

**✅ Platform Requirements VERIFIED**:
- **Primary Platform**: HEALIX/Databricks - ✅ Confirmed in tech stack
- **Primary Language**: Python (Polars for data integration) - ✅ Preferred in tech stack
- **Compute Requirements**: Local/single-node likely sufficient (dataset <1MB total); Spark available if needed for complex scenarios
- **Data Access Pattern**: Batch CSV processing with multi-table joins - ✅ Fully supported
- **Analytics Libraries**: 
  - Data manipulation: Polars (confirmed)
  - Optimization: scipy.optimize, OR-Tools (Python optimization libraries)
  - Scenario modeling: Pandas/Polars with custom logic
  - Visualization: Plotly/Altair for interactive capacity dashboards
- **Statistical Analysis**: STATA available if econometric demand modeling needed

**Technical Feasibility**: ✅ **CONFIRMED - No gaps identified**
- Multi-table data integration: Polars excels at joins and aggregations
- Gap analysis: Standard Python analytical libraries
- Scenario modeling: Custom Python code or optimization libraries (scipy, OR-Tools)
- Visualization: Rich dashboard creation with Plotly/Dash

**Platform Constraints**: None - suitable for Databricks Python environment with optional Spark scale-out if needed

---

## Expected Outcomes and Deliverables

**Stakeholder Outcomes**:
1. **Capacity planning roadmap**: Clear priorities for infrastructure investment (which facility types, which sectors, what timeline)
2. **Utilization efficiency insights**: Identify over-utilized vs under-utilized capacity
3. **Demographic targeting**: Understand which population segments drive demand; enable targeted interventions
4. **Scenario evaluation capability**: Assess impact of proposed capacity expansions before committing capital

**Concrete Deliverables**:

**1. 📊 Analytical Report: "Singapore Healthcare Capacity Optimization Study 2009-2020"**
- Executive summary with prioritized capacity investment recommendations
- Detailed capacity vs utilization trend analysis by facility type
- Demographic analysis of service utilization patterns
- Gap analysis identifying critical shortages and surpluses
- Scenario modeling results for infrastructure investment options
- Format: PDF report (40-50 pages) with embedded visualizations and appendices
- Access: MOH internal knowledge portal; shared with hospital cluster CEOs

**2. 📈 Interactive Dashboard: "Healthcare Capacity Planning Dashboard"**
- Multi-year capacity and utilization trend visualization
- Facility type comparison tool (acute, intermediate, primary care)
- Public vs private sector capacity analysis
- Demographic drill-down for utilization patterns
- Scenario simulation tool for capacity expansion impact
- Platform: Plotly Dash or Databricks Dashboard
- Access: MOH healthcare planning division; hospital cluster planners
- Update Frequency: Annual refresh with new data

**3. 📋 Curated Dataset: "Capacity & Utilization Metrics 2006-2020"**
- Integrated capacity and utilization dataset with calculated metrics
- Proxy occupancy rates, utilization efficiency scores, capacity gaps
- Demographic utilization profiles
- Scenario modeling inputs and assumptions documented
- Format: Parquet file with comprehensive data dictionary
- Location: `shared/data/4_processed/capacity_utilization_analysis.parquet`
- Access: Available for downstream planning models and forecasting

**4. 📑 Policy Recommendations: "Healthcare Infrastructure Investment Priorities 2026-2030"**
- Prioritized capacity expansion needs by facility type
- Quantified investment requirements for each priority
- Timeline and phasing recommendations
- Alternative scenario options (e.g., shift care to intermediate vs expand acute)
- Format: Executive brief (5-10 pages) with supporting cost-benefit appendix
- Audience: MOH Permanent Secretary, Minister for Health, Budget Office

---

## Dependencies and Assumptions

**Problem Statement Dependencies**: None (independent analysis)

**Related Problem Statements**:
- **May complement PS-001 (Workforce Sustainability)** - capacity expansion requires corresponding workforce planning
- Informs future facility infrastructure planning initiatives

**Key Assumptions**:

**⚠️ CRITICAL - These are analytical assumptions, NOT data assumptions**:

- **Assumption 1**: Ratio of utilization to capacity provides reasonable proxy for bed occupancy/facility strain
  - **Validation approach**: Acknowledge as approximation; recommend obtaining actual occupancy data for future refinement
  
- **Assumption 2**: Historical capacity and utilization trends (2009-2020) are informative for future planning despite COVID-19 disruption
  - **Mitigation**: Analyze pre-COVID baseline (2009-2019) separately from 2020; note COVID-19 may have permanently altered utilization patterns

- **Assumption 3**: National-level capacity planning is actionable despite lack of regional granularity
  - **Mitigation**: Acknowledge limitation; recommend geographic distribution analysis when regional data becomes available

- **Assumption 4**: Public and private sector trends can be analyzed together for national capacity assessment
  - **Validation approach**: Also conduct separate public vs private sector analyses to identify sector-specific patterns

**Data has been VERIFIED against `data-sources.md` - no data assumptions made**

---

## Risks and Open Questions

**Potential Blockers**:
1. **Risk**: Actual bed occupancy rates not directly available; must use proxy calculations
   - **Mitigation**: Clearly label metrics as "proxy occupancy" or "capacity utilization ratio"; validate against industry benchmarks where possible
   
2. **Risk**: Cannot assess regional/geographic maldistribution with national-level data only
   - **Mitigation**: Focus recommendations on national capacity levels; note regional distribution as future enhancement
   
3. **Risk**: COVID-19 (2020+) fundamentally changed utilization patterns (e.g., telemedicine adoption, elective procedure backlogs)
   - **Mitigation**: Separate analysis into pre-COVID baseline (2009-2019) and discuss implications of COVID-induced changes

4. **Risk**: Optimal capacity utilization targets may vary by facility type and are subjective
   - **Mitigation**: Research industry best practices and international benchmarks for occupancy rate targets; present range of scenarios

**Open Questions for Stakeholder Clarification**:
1. What bed occupancy rate is considered "optimal" for different facility types (acute: 85%?, community hospital: ?, nursing home: ?)?
2. Are there planned major infrastructure projects (new hospitals, expansions) that should be factored into analysis?
3. What investment timeframe is most relevant (5-year, 10-year capital planning horizon)?
4. Are there specific facility types or care settings of particular concern (e.g., long-term care)?
5. Should analysis incorporate cost data (cost per bed, operating costs) or focus purely on physical capacity?

---

## Problem Statement Readiness

**This Problem Statement is ready for backlog refinement when**:
- [x] Data domains explicitly verified against `data_sources.md` - ✅ COMPLETE
- [x] Platform and technical feasibility confirmed (`tech_stack.md` reviewed) - ✅ COMPLETE
- [x] Problem statement can be decomposed into 5-10 user stories - ✅ READY (see below)
- [x] Deliverable format and access method defined - ✅ COMPLETE

**Preliminary User Story Breakdown** (to be refined in Sprint Planning):
1. As a capacity planner, I want to see facility and bed capacity trends by type, so that I understand infrastructure growth patterns
2. As a service planner, I want utilization rate trends by demographics, so that I can identify which populations drive demand
3. As an analyst, I want proxy occupancy calculations, so that I can assess facility strain levels
4. As a policy maker, I want capacity gap analysis, so that I can prioritize infrastructure investments
5. As a hospital administrator, I want sector-specific analysis (public vs private), so that I can benchmark my facilities
6. As a strategic planner, I want scenario modeling tools, so that I can evaluate capacity expansion options
7. As a budget officer, I want investment priority recommendations, so that I can allocate capital effectively
8. As an executive, I want an interactive capacity dashboard, so that I can explore facility data dynamically
9. As a researcher, I want a consolidated capacity-utilization dataset, so that I can conduct further analyses

**Status**: ✅ **READY FOR BACKLOG**

---

**Created**: 2026-03-11  
**Last Updated**: 2026-03-11  
**Status**: Draft - Awaiting Stakeholder Review  
**Owner**: TBD (assign during sprint planning)
