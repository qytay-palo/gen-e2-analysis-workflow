# User Story: 8 - Healthcare Capacity Requirement Translation

**As a** healthcare infrastructure planner,  
**I want** mortality projections translated into capacity requirements (hospital beds, specialist workforce),  
**so that**I can size infrastructure investments and workforce plans for projected disease burden.

## 1. 🎯 Acceptance Criteria

1. **Capacity Translation Model** - Convert mortality forecasts to service demand (beds, clinicians)
2. **Disease-Specific Requirements** - Calculate capacity needs by disease (oncology, cardiology, neurology)
3. **Gap Analysis** - Compare projected demand vs current capacity plans
4. **Investment Roadmap** - Prioritized capacity expansion needs by year (2020-2025)
5. **Policy Brief** - "Healthcare Capacity Investment Roadmap 2020-2025" delivered

## 2. 🔒 Technical Constraints

- **Translation methodology**: Use standard ratios (e.g., deaths per bed, deaths per specialist)
- **Data integration**: Link mortality forecasts to workforce/facility datasets
- **Stakeholder-ready**: Output must be actionable for planners (not just forecasts)

## 3. 📚 Domain Knowledge References

- [Healthcare Workforce Metrics KPIs](../../../domain_knowledge/healthcare-workforce-metrics-kpis.md) - Workforce planning ratios
- MOH planning standards: Bed-to-population ratios, specialist-to-population ratios

**Key Translation Ratios** (Singapore context):
- Cancer care: ~15-20 oncologists per 1M population
- Cardiology: ~10-15 cardiologists per 1M population
- Hospital beds: ~2-3 acute care beds per 1,000 population

## 4. 📦 Dependencies

**Input**:
- Mortality forecasts from User Story 6
- Workforce data: `number-of-doctors.csv`
- Facilities data: `health-facilities-and-beds-in-inpatient-facilities.csv`

**Output**:
- Capacity requirements table: `results/tables/problem-statement-006/capacity_requirements_2020_2025.csv`
- Policy brief: `reports/figures/problem-statement-006/capacity_investment_roadmap.pdf`

## 5. ✅ Implementation Tasks

### Translation Modeling
- ⬜ **Define mortality-to-capacity conversion formulas** (literature + stakeholder input)
- ⬜ **Apply conversion to forecasts** (estimate bed needs, specialist FTEs)
- ⬜ **Calculate scenario range** (baseline/optimistic/pessimistic capacity needs)

### Gap Analysis
- ⬜ **Load current capacity data** (beds, specialist counts)
- ⬜ **Project current capacity growth** (assume linear expansion or zero growth)
- ⬜ **Calculate capacity gaps** (projected need - projected supply)

### Investment Roadmap
- ⬜ **Prioritize capacity needs** (high gap, high certainty)
- ⬜ **Create investment timeline** (when to initiate projects)
- ⬜ **Identify "no-regret" investments** (robust across scenarios)

### Deliverables
- ⬜ **Generate capacity requirements table** by disease and year
- ⬜ **Create policy brief** (3-page executive brief + 2-page data appendix)
- ⬜ **Visualize capacity gaps** (projected demand vs supply charts)

## 6. Notes

**Translation Methodology**:
- Mortality is proxy for severe disease burden (requires intensive healthcare resources)
- Use disease-specific bed utilization rates and specialist workload standards
- Acknowledge uncertainty: capacity planning should use range (optimistic-pessimistic), not point estimate

**Stakeholder Value**:
- **Lead time for action**: Infrastructure takes 3-5 years, specialist training 5-10 years
- **Budget justification**: Evidence-based capacity needs for funding requests
- **Risk mitigation**: Avoid capacity crises from underestimation

**Next Steps**:
- Capacity requirements integrated into dashboard (User Story 9)
- Investment roadmap guides MOH strategic planning decisions

---

## Implementation Plan

### 1. Feature Overview

Translate mortality projections into healthcare capacity requirements (hospital beds, specialists) to size infrastructure investments.

### 2. Component Analysis & Reuse Strategy

**Input**: Forecasts from US-06, scenarios from US-07, workforce data from Kaggle

**New**: `shared/src/analysis/capacity_planning.py`

### 4-6. Code Specifications

```python
# shared/src/analysis/capacity_planning.py

import polars as pl
from loguru import logger
from typing import Dict


# Singapore capacity planning ratios (MOH standards)
BED_RATIOS = {
    'cancer': 2.5,  # Acute care beds per 1,000 population
    'stroke': 1.8,
    'heart_disease': 2.0
}

SPECIALIST_RATIOS = {
    'cancer': 18,  # Oncologists per 1M population
    'stroke': 12,  # Neurologists per 1M population  
    'heart_disease': 15  # Cardiologists per 1M population
}

POPULATION_SG = 5_686_000  # 2023 estimate


def translate_to_bed_capacity(
    mortality_rate: float,
    disease: str,
    population: int = POPULATION_SG
) -> float:
    """Estimate required hospital bed capacity.
    
    Simplified formula: Higher mortality → more severe cases → more beds
    
    Args:
        mortality_rate: ASMR per 100k
        disease: Disease type
        population: Total population
        
    Returns:
        Estimated bed requirement
    """
    # Convert mortality rate to absolute cases
    annual_cases = (mortality_rate / 100_000) * population
    
    # Apply disease-specific bed ratio
    beds_needed = annual_cases * BED_RATIOS.get(disease, 2.0)
    
    return beds_needed


def translate_to_specialist_capacity(
    mortality_rate: float,
    disease: str,
    population: int = POPULATION_SG
) -> int:
    """Estimate required specialist FTEs."""
    
    base_specialists = (population / 1_000_000) * SPECIALIST_RATIOS.get(disease, 15)
    
    # Adjust based on mortality burden (higher burden = more specialists)
    sg_avg_mortality = 100  # Approximate baseline
    burden_factor = mortality_rate / sg_avg_mortality
    
    specialists_needed = int(base_specialists * burden_factor)
    
    return specialists_needed


def calculate_capacity_requirements(
    forecast_df: pl.DataFrame
) -> pl.DataFrame:
    """Calculate capacity requirements from forecasts."""
    
    logger.info("Translating forecasts to capacity requirements")
    
    capacity_df = forecast_df.with_columns([
        # Beds (vectorized calculation)
        pl.struct(['forecast_mean', 'disease'])
        .apply(lambda row: translate_to_bed_capacity(row['forecast_mean'], row['disease']))
        .alias('beds_required'),
        
        # Specialists
        pl.struct(['forecast_mean', 'disease'])
        .apply(lambda row: translate_to_specialist_capacity(row['forecast_mean'], row['disease']))
        .alias('specialists_required')
    ])
    
    logger.info("Capacity calculations complete")
    
    return capacity_df
```

### 7. Implementation Steps

- [ ] Define mortality-to-capacity translation formulas
- [ ] Load forecasts from US-06 and scenarios from US-07
- [ ] Calculate bed requirements by disease and year
- [ ] Calculate specialist FTE requirements
- [ ] Load current capacity data (beds, specialists from Kaggle)
- [ ] Calculate capacity gaps (demand - supply)
- [ ] Prioritize investments (high gap, high certainty)
- [ ] Generate policy brief PDF
- [ ] Save to `results/tables/problem-statement-006/capacity_requirements_2020_2025.csv`

✅ **PLAN READY**
