# Analytics Problem Statements - Strategic Initiatives

## Overview

This directory contains strategic analytics problem statements for the Singapore Health Trends Analysis project. Each problem statement represents a complete analytical initiative from data extraction to actionable insights, focused on problems that can be solved end-to-end with available data and technical capabilities.

**Total Problem Statements**: 6  
**Last Updated**: 2026-03-13

---

## Problem Statement Categories

### Predictive Analytics (1)

**[PS-006: Five-Year Disease Burden Forecasting and Mortality Projection](ps-006-disease-burden-forecasting.md)**
- **Description**: Develop time series forecasting models to project mortality and disease burden for major diseases (cancer, stroke, heart disease) over the next 5 years, enabling proactive healthcare capacity planning and resource allocation
- **Priority**: P1 (High)
- **Estimated Sprints**: 5-7
- **Platform**: HEALIX/Databricks
- **Dependencies**: PS-002 (recommended but not required)
- **Key Deliverables**: Forecasting models, 5-year projection report, interactive forecast dashboard, capacity planning tool, forecast monitoring system

---

### Descriptive Analytics (2)

**[PS-002: National Disease Burden Temporal Trends Analysis](ps-002-disease-burden-temporal-trends.md)**
- **Description**: Analyze 30-year mortality trends (1990-2019) for major diseases to identify shifting disease burden patterns and inform public health program prioritization
- **Priority**: P0 (Critical)
- **Estimated Sprints**: 3-5
- **Platform**: HEALIX/Databricks
- **Dependencies**: None
- **Key Deliverables**: Trend analysis report, interactive trend explorer dashboard, curated disease burden dataset, policy brief

**[PS-005: Healthcare Equity and Disparities Analysis](ps-005-healthcare-equity-disparities.md)**
- **Description**: Quantify healthcare access and outcome disparities across demographic groups and geographic regions to identify underserved populations
- **Priority**: P1 (High)
- **Estimated Sprints**: 4-6
- **Platform**: HEALIX/Databricks
- **Dependencies**: None
- **Key Deliverables**: Equity assessment report, disparity metrics dashboard, priority intervention areas

---

### Diagnostic Analytics (2)

**[PS-001: Healthcare Workforce Sustainability Analysis](ps-001-healthcare-workforce-sustainability.md)**
- **Description**: Analyze healthcare workforce trends and identify factors contributing to sustainability challenges in manpower planning
- **Priority**: P0 (Critical)
- **Estimated Sprints**: 4-6
- **Platform**: HEALIX/Databricks
- **Dependencies**: None
- **Key Deliverables**: Workforce sustainability report, shortage risk assessment, workforce planning dashboard

**[PS-004: Healthcare Expenditure Drivers Analysis](ps-004-healthcare-expenditure-drivers.md)**
- **Description**: Identify and quantify key drivers of healthcare expenditure growth to inform cost containment strategies
- **Priority**: P1 (High)
- **Estimated Sprints**: 4-5
- **Platform**: HEALIX/Databricks
- **Dependencies**: None
- **Key Deliverables**: Expenditure driver analysis report, cost projection models, policy recommendations

---

### Prescriptive Analytics (1)

**[PS-003: Healthcare Capacity Optimization](ps-003-healthcare-capacity-optimization.md)**
- **Description**: Optimize allocation of healthcare resources (beds, facilities, workforce) across the system to maximize access and efficiency
- **Priority**: P1 (High)
- **Estimated Sprints**: 5-7
- **Platform**: HEALIX/Databricks
- **Dependencies**: PS-001 (workforce data), PS-006 (demand forecasts - recommended)
- **Key Deliverables**: Capacity optimization models, resource allocation recommendations, scenario planning tool

---

## Problem Statement Priority Matrix

| Priority | Problem Statements | Rationale |
|----------|-------------------|-----------|
| **P0 (Critical)** | PS-001, PS-002 | Foundational analyses addressing immediate strategic needs and data availability |
| **P1 (High)** | PS-003, PS-004, PS-005, PS-006 | High-value analyses building on foundational work and enabling proactive planning |

---

## Problem Statement Dependencies Graph

```
PS-002 (Disease Trends)
   ↓ (recommended input)
PS-006 (Disease Forecasting)
   ↓ (demand projections)
PS-003 (Capacity Optimization)
   ↑ (workforce data)
PS-001 (Workforce Sustainability)

PS-004 (Expenditure Drivers) ← independent
PS-005 (Equity Analysis) ← independent
```

**Legend**:
- **Solid arrows (↓)**: Recommended sequencing for maximum value
- **Independent**: Can be executed in parallel without dependencies

---

## Recommended Execution Sequence

### Phase 1: Foundational Analysis (Quarters 1-2)
1. **PS-002**: Disease Burden Temporal Trends (3-5 sprints)
   - Establishes baseline understanding of disease landscape
   - Informs forecasting and capacity planning
   
2. **PS-001**: Healthcare Workforce Sustainability (4-6 sprints)
   - Provides workforce supply context for capacity planning
   - Can run in parallel with PS-002

### Phase 2: Predictive & Optimization (Quarters 2-3)
3. **PS-006**: Disease Burden Forecasting (5-7 sprints)
   - Builds on PS-002 trend insights (recommended but not required)
   - Generates demand projections for capacity optimization

4. **PS-005**: Healthcare Equity Analysis (4-6 sprints)
   - Can run in parallel with PS-006
   - Informs equitable capacity allocation

### Phase 3: Strategic Planning (Quarters 3-4)
5. **PS-003**: Healthcare Capacity Optimization (5-7 sprints)
   - Leverages workforce data (PS-001) and demand forecasts (PS-006)
   - Incorporates equity considerations (PS-005)

6. **PS-004**: Healthcare Expenditure Drivers (4-5 sprints)
   - Can run in parallel or after optimization work
   - Provides financial context for capacity decisions

---

## Problem Statement Status

| ID | Title | Status | Priority | Sprints | Platform |
|----|-------|--------|----------|---------|----------|
| PS-001 | Healthcare Workforce Sustainability | Draft | P0 | 4-6 | HEALIX/Databricks |
| PS-002 | Disease Burden Temporal Trends | Draft | P0 | 3-5 | HEALIX/Databricks |
| PS-003 | Healthcare Capacity Optimization | Draft | P1 | 5-7 | HEALIX/Databricks |
| PS-004 | Healthcare Expenditure Drivers | Draft | P1 | 4-5 | HEALIX/Databricks |
| PS-005 | Healthcare Equity Disparities | Draft | P1 | 4-6 | HEALIX/Databricks |
| PS-006 | Disease Burden Forecasting | **Draft** | **P1** | **5-7** | **HEALIX/Databricks** |

---

## Data Sources Reference

All problem statements are constrained by data documented in:
- **Primary Data Source**: [Kaggle Health Dataset - Singapore](../project_context/data-sources.md)
- **Dataset**: `subhamjain/health-dataset-complete-singapore`
- **Coverage**: 1990-2020 (varies by table)
- **Tables**: 35 data tables across workforce, facilities, disease burden, utilization, expenditure

**Key Constraint**: Annual granularity, national-level aggregation (no regional breakdowns, no real-time data)

---

## Technical Stack Reference

All problem statements use:
- **Platform**: HEALIX/Databricks (GCC Cloud Environment)
- **Primary Language**: Python 3.9+ (Databricks Runtime 13.3)
- **Data Processing**: Polars (mandatory), pandas (only when justified)
- **Package Management**: uv (NOT pip/conda)
- **Forecasting Libraries**: statsmodels, prophet, scikit-learn
- **Visualization**: Plotly (interactive), matplotlib/seaborn (static)

See [tech-stack.md](../project_context/tech-stack.md) for complete details.

---

## Problem Statement Lifecycle

### Statuses
- **Draft**: Initial problem statement created, pending stakeholder review
- **Approved**: Stakeholder sign-off obtained, ready for backlog
- **In Progress**: User stories created, sprints active
- **Completed**: All deliverables produced, stakeholder acceptance achieved
- **Archived**: No longer relevant or superseded by other work

### Update Triggers
Problem statements should be reviewed and updated when:
- New data sources become available
- Stakeholder priorities shift
- Technical capabilities change
- Related problem statements complete and provide new insights

---

## Document Management

**Owners**: 
- **Business Analyst**: Problem statement definition, stakeholder alignment
- **Data Analytics Team**: Technical feasibility validation, data verification
- **Project Manager**: Prioritization, sequencing, resource allocation

**Review Frequency**: Quarterly or when significant context changes occur

**Version Control**: All problem statements tracked in Git with change history

---

## Related Documentation

- **User Stories**: [docs/objectives/user_stories/](../user_stories/) - Tactical breakdown of problem statements
- **Data Dictionary**: [docs/data_dictionary/](../../data_dictionary/) - Dataset schemas and definitions
- **Domain Knowledge**: [docs/domain_knowledge/](../../domain_knowledge/) - Healthcare and analytics domain expertise
- **Project Context**: [docs/project_context/](../../project_context/) - Business objectives, data sources, tech stack

---

**Document Status**: Active  
**Last Updated**: 2026-03-13  
**Next Review**: 2026-06-13 (Quarterly)
