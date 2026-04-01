# User Story: 7 - Scenario-Based Forecasting and Sensitivity Analysis

**As a** capacity planner,  
**I want** multiple forecast scenarios (baseline, optimistic, pessimistic) with sensitivity analysis,  
**so that** I can plan for a range of possible futures and identify robust investment decisions.

## 1. 🎯 Acceptance Criteria

1. **Baseline Scenario** - Continuation of historical trends (from User Story 6)
2. **Optimistic Scenario** - Enhanced intervention effects (e.g., 20% faster mortality decline)
3. **Pessimistic Scenario** - Trend reversal (e.g., aging population, lifestyle risks increase burden)
4. **Intervention Impact Analysis** - Quantify effect of specific programs (e.g., expanded screening, prevention campaigns)
5. **Scenario Comparison Report** - Visualizations showing baseline vs optimistic vs pessimistic trajectories

## 2. 🔒 Technical Constraints

- **Scenario modeling**: Adjust baseline forecasts by intervention multipliers
- **Transparency**: Document all scenario assumptions explicitly
- **Visualization**: Multi-scenario overlay charts for comparison

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Scenario modeling section, Monte Carlo simulation
- Public health literature: Typical intervention effects (e.g., smoking cessation reduces lung cancer 15-20%)

## 4. 📦 Dependencies

**Input**:
- Baseline forecasts from User Story 6
- Intervention effectiveness estimates (literature review or stakeholder input)

**Output**:
- Scenario forecasts: `shared/data/4_processed/disease_burden_scenarios_2020_2025.parquet`

## 5. ✅ Implementation Tasks

### Scenario Development
- ⬜ **Define optimistic scenario parameters** (e.g., 1.2x faster decline)
- ⬜ **Define pessimistic scenario parameters** (e.g., 1.1x trend reversal)
- ⬜ **Generate optimistic forecasts** (apply multipliers to baseline)
- ⬜ **Generate pessimistic forecasts**

### Intervention Modeling
- ⬜ **Identify plausible interventions** (screening, prevention campaigns)
- ⬜ **Estimate intervention effects** from literature
- ⬜ **Model intervention scenarios** (what-if analysis)

### Deliverables
- ⬜ **Create scenario comparison visualizations**
- ⬜ **Generate scenario comparison report**
- ⬜ **Save all scenarios to dataset**

## 6. Notes

**Scenario Assumptions** (adjust based on stakeholder input):
- **Optimistic**: 20% acceleration in mortality decline (new treatments, better prevention)
- **Pessimistic**: 10% increase in burden (aging population, lifestyle factors)
- **Intervention**: Specific program effects (e.g., cancer screening reduces mortality 5-10%)

**Value to Stakeholders**:
- Robust planning: Identify "no-regret" investments that make sense across all scenarios
- Risk management: Understand downside risk (pessimistic scenario)
- Advocacy: Use optimistic scenarios to motivate prevention investments

**Next Steps**:
- Scenarios inform capacity planning ranges in User Story 8
- Multiple trajectories displayed in dashboard (User Story 9)

---

## Implementation Plan

### 1. Feature Overview

Develop multiple forecast scenarios (baseline, optimistic, pessimistic) with sensitivity analysis for robust capacity planning.

### 2. Component Analysis & Reuse Strategy

**Input**: Baseline forecasts from US-06

**New**: `shared/src/models/scenario_modeling.py`

### 4-6. Code Specifications

```python
# shared/src/models/scenario_modeling.py

import polars as pl
from loguru import logger
from typing import Dict


def generate_scenarios(
    baseline_df: pl.DataFrame,
    optimistic_multiplier: float = 0.8,  # 20% faster decline
    pessimistic_multiplier: float = 1.1  # 10% burden increase
) -> pl.DataFrame:
    """Generate optimistic and pessimistic scenarios.
    
    Args:
        baseline_df: Baseline forecasts
        optimistic_multiplier: Multiplier for optimistic (< 1.0 = decline)
        pessimistic_multiplier: Multiplier for pessimistic (> 1.0 = increase)
        
    Returns:
        Combined scenarios dataframe
    """
    logger.info("Generating forecast scenarios")
    
    # Baseline
    baseline = baseline_df.with_columns(pl.lit('baseline').alias('scenario'))
    
    # Optimistic (faster mortality decline)
    optimistic = baseline.clone().with_columns([
        (pl.col('forecast_mean') * optimistic_multiplier).alias('forecast_mean'),
        (pl.col('lower_95') * optimistic_multiplier).alias('lower_95'),
        (pl.col('upper_95') * optimistic_multiplier).alias('upper_95'),
        pl.lit('optimistic').alias('scenario')
    ])
    
    # Pessimistic (slower decline or reversal)
    pessimistic = baseline.clone().with_columns([
        (pl.col('forecast_mean') * pessimistic_multiplier).alias('forecast_mean'),
        (pl.col('lower_95') * pessimistic_multiplier).alias('lower_95'),
        (pl.col('upper_95') * pessimistic_multiplier).alias('upper_95'),
        pl.lit('pessimistic').alias('scenario')
    ])
    
    # Combine all scenarios
    scenarios = pl.concat([baseline, optimistic, pessimistic], how='vertical')
    
    logger.info(f"Generated scenarios: {scenarios.height} rows")
    
    return scenarios
```

### 7. Implementation Steps

- [ ] Load baseline forecasts from US-06
- [ ] Define scenario parameters (optimistic: -20%, pessimistic: +10%)
- [ ] Generate optimistic scenario (enhanced interventions)
- [ ] Generate pessimistic scenario (aging population, risk factors)
- [ ] Model specific interventions (screening programs, prevention)
- [ ] Save to `shared/data/4_processed/disease_burden_scenarios_2020_2025.parquet`
- [ ] Create scenario comparison visualizations

✅ **PLAN READY**
