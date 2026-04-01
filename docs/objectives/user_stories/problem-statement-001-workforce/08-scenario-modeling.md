# Model Policy Intervention Scenarios (Lifecycle Stage: Advanced Analysis & Modeling)

**Story ID**: PS-001-US-08  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P1 (High)  
**Effort Estimate**: M (6-7 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Policy Strategist evaluating workforce interventions**,  
I want **to model the impact of policy interventions (training program expansion, retention improvements, immigration quotas) on 2020-2030 workforce supply to quantify which interventions most effectively close identified gaps**,  
So that **policy makers can prioritize interventions with highest ROI and understand the workforce impact of different policy choices before implementation**.

---

## 🎯 Acceptance Criteria

1. **Policy intervention scenarios defined**
   - Scenario 1: Training expansion (increase annual graduates by 10%, 20%, 30%)
   - Scenario 2: Retention improvement (reduce turnover by 5%, 10%, 15%)
   - Scenario 3: Immigration quota increase (add X new workers annually)
   - Scenario 4: Combined interventions (optimize mix of training + retention + immigration)

2. **Scenario modeling completed**
   - Each scenario modeled for 2020-2030 (11-year horizon)
   - Impact quantified: additional workforce supply vs baseline forecast
   - Gap closure calculated: % of shortage eliminated by intervention
   - Cost-effectiveness considered: effort/cost per worker added (qualitative or quantitative)

3. **Scenario comparison & recommendations**
   - Scenarios ranked by gap closure effectiveness
   - Trade-off analysis: speed of impact vs sustainability (immigration = fast, training = slow but sustainable)
   - Recommended intervention mix for each priority profession
   - Timeline: when do interventions need to start to meet 2030 targets?

4. **Data output requirement**
   - Output file: `results/tables/workforce_scenario_analysis.csv`
   - Format: CSV with columns (profession, year, scenario_name, workforce_count, gap_vs_baseline, gap_closure_pct, intervention_parameters)
   - Schema documented: Yes
   - Scenario comparison chart: `reports/figures/workforce_scenario_comparison.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for scenario calculations)
- **Optimization**: scipy.optimize or OR-Tools (optional for combined scenario optimization)
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for scenario modeling functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#4-workforce-turnover-rate) - Understanding turnover impact on workforce supply
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#objectives) - Objective 4: Provide scenario analysis for policy interventions

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Scenario calculations and simulations
- `scipy>=1.11.0`: Optimization (optional for combined scenarios)
- `matplotlib>=3.8.0` or `plotly>=5.18.0`: Scenario comparison charts
- `loguru>=0.7.0`: Structured logging
- `pyyaml>=6.0`: Scenario configuration management

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-06 (Forecast models - BLOCKING for baseline)
  - PS-001-US-07 (Gap analysis - BLOCKING for intervention targets)
- **Data Sources**: 
  - `shared/data/4_processed/workforce_forecasts_2020_2030.parquet` (baseline)
  - `results/tables/workforce_gap_analysis.csv` (target gaps to close)
- **Config Files**: `config/analysis.yml` (scenario parameters: training increase %, turnover reduction %)

---

## ✅ Implementation Tasks

### Scenario Definition
- [ ] Define scenario parameters in `config/analysis.yml`:
  - Training expansion: +10%, +20%, +30% annual graduates
  - Retention improvement: -5%, -10%, -15% turnover reduction
  - Immigration: +50, +100, +200 workers/year per profession
- [ ] Document scenario assumptions and feasibility (e.g., can training realistically expand 30%?)

### Scenario Modeling Implementation
- [ ] Create scenario modeling script: `shared/src/models/scenario_modeling.py`
- [ ] Implement training expansion model:
  - Calculate annual graduates from historical data
  - Project increased graduates entering workforce (with time lag: 4-8 years for doctors)
  - Accumulate expanded workforce over 2020-2030
  
- [ ] Implement retention improvement model:
  - Estimate baseline turnover rate from historical data
  - Model reduced attrition: `new_supply = baseline_supply * (1 + retention_improvement_rate)`
  - Account for cumulative effect over multiple years

- [ ] Implement immigration quota model:
  - Add fixed number of new workers annually
  - Model immediate impact (no training lag)
  - Consider sustainability (quota limits)

- [ ] Implement combined scenario optimization:
  - Define target: minimize gap by 2030
  - Decision variables: training%, retention%, immigration#
  - Constraints: budget, feasibility bounds
  - Use scipy.optimize to find optimal intervention mix

### Gap Closure Analysis
- [ ] Calculate gap closure for each scenario: `gap_closure = (baseline_gap - scenario_gap) / baseline_gap * 100%`
- [ ] Identify scenarios that fully close gap (gap ≤ 0 by 2030)
- [ ] Identify scenarios with partial closure (gap reduced but not eliminated)
- [ ] Calculate timeline: year when gap is eliminated under each scenario

### Cost-Effectiveness & Trade-off Analysis
- [ ] Qualitative assessment: rank scenarios by implementation difficulty
  - Training expansion: medium difficulty, long lag, highly sustainable
  - Retention: high difficulty (requires systemic changes), medium lag, sustainable
  - Immigration: low difficulty (policy change), immediate impact, less sustainable
- [ ] Quantitative assessment (if cost data available):
  - Cost per worker added: training cost, retention program cost, immigration costs
  - ROI: workforce gain per dollar invested
- [ ] Trade-off matrix: speed vs sustainability vs cost

### Scenario Comparison & Recommendations
- [ ] Rank scenarios by gap closure effectiveness (% gap eliminated)
- [ ] Identify profession-specific recommendations (different professions may need different interventions)
- [ ] Create intervention roadmap: which interventions to start when?
- [ ] Sensitivity analysis: how robust are recommendations to parameter changes?

### Visualization
- [ ] Create scenario comparison charts: baseline vs intervention scenarios (line plots)
- [ ] Create gap closure waterfall chart: contribution of each intervention
- [ ] Create heat map: gap closure % by profession and scenario
- [ ] Create decision matrix: scenario effectiveness vs implementation difficulty
- [ ] Export figures in PNG and PDF formats

### Testing & Validation
- [ ] Unit tests for scenario calculation logic
- [ ] Test edge cases: extreme intervention parameters (e.g., 100% training increase)
- [ ] Validate gap closure formula
- [ ] Integration test: end-to-end scenario modeling pipeline
- [ ] Sensitivity test: ±20% parameter variation

### Documentation
- [ ] Docstrings for scenario modeling functions (Google style)
- [ ] Scenario analysis report: `results/workforce_scenario_recommendations.md`
  - Scenario definitions and assumptions
  - Gap closure results per scenario
  - Trade-off analysis
  - Recommended intervention strategy per priority profession
  - Implementation roadmap and timeline
- [ ] Update README: `shared/src/models/README.md`

---

## 📌 Notes

**Training Expansion Scenario (Polars)**:
```python
import polars as pl

# Baseline annual graduates (estimate from historical growth)
annual_graduates = df.filter(pl.col('year') == 2019).select('count').item() * 0.05  # assume 5% are new graduates

# Training expansion scenario (+20%)
expansion_rate = 0.20
additional_graduates_per_year = annual_graduates * expansion_rate

# Account for training lag (e.g., 5 years for doctors)
training_lag = 5

# Project workforce with expanded training
df_scenario = df_baseline.with_columns([
    pl.when(pl.col('year') >= (2020 + training_lag))
      .then(pl.col('forecast_count') + additional_graduates_per_year * (pl.col('year') - 2020 - training_lag + 1))
      .otherwise(pl.col('forecast_count'))
      .alias('scenario_training_20pct')
])
```

**Retention Improvement Scenario**:
```python
# Baseline turnover rate (estimate ~15-20% for healthcare)
baseline_turnover = 0.15

# Retention improvement (-10% turnover → 5% turnover)
improved_turnover = baseline_turnover - 0.10

# Annual workforce retention gain
retention_gain_rate = baseline_turnover - improved_turnover

# Cumulative retention effect
df_scenario = df_baseline.with_columns([
    (pl.col('forecast_count') * (1 + retention_gain_rate) ** (pl.col('year') - 2020))
    .alias('scenario_retention_10pct')
])
```

**Combined Scenario Optimization** (conceptual):
```python
from scipy.optimize import minimize

def gap_objective(params, baseline_supply, demand):
    """Minimize 2030 gap given intervention parameters."""
    training_pct, retention_pct, immigration_annual = params
    
    # Calculate 2030 supply under combined interventions
    supply_2030 = (
        baseline_supply +
        training_impact(training_pct) +
        retention_impact(retention_pct) +
        immigration_annual * 10  # 10 years of immigration
    )
    
    # Return absolute gap (minimize this)
    return abs(supply_2030 - demand)

# Optimize
result = minimize(
    gap_objective,
    x0=[0.1, 0.05, 50],  # initial guess
    bounds=[(0, 0.3), (0, 0.15), (0, 200)]  # feasibility constraints
)
```

**Expected Insights**:
- **Nurses**: Likely need combined approach (high demand, large gap)
- **Doctors**: Training expansion critical (long lag, need to start immediately)
- **Pharmacists**: Immigration may be sufficient (smaller gap)
- **Allied health**: Retention + immigration (shorter training, flexible supply)

**Policy Recommendations Framework**:
| Profession | Priority | Recommended Intervention | Timeline |
|-----------|----------|-------------------------|----------|
| Nurses | P0 | Training +20%, Immigration +100/yr | Start 2026 |
| Doctors | P0 | Training +30%, Retention -10% | Start 2026 |
| Pharmacists | P1 | Immigration +50/yr | Start 2027 |

**Scenario Assumptions to Document**:
- Training expansion feasible within existing infrastructure constraints
- Retention improvements require systemic policy changes (salary, work conditions)
- Immigration quotas subject to government approval and labor market policies
- No major disruptions (pandemics, economic crises) alter workforce dynamics

---

## Implementation Plan

### 1. Scenario Modeling Overview

Model policy interventions (training expansion, retention improvement, immigration) to project impact on workforce supply 2020-2030. Compare scenarios against demand gaps (US-07). Identify optimal intervention combinations per profession.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Gap analysis (US-07), forecasts (US-06)
**Create**: `shared/src/analysis/scenario_modeler.py`, scenario evaluation tests

### 3. Key Implementation Components

**[CREATE] `shared/src/analysis/scenario_modeler.py`**

```python
"""
Workforce Scenario Modeling
============================

Model policy interventions to address workforce shortages.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import numpy as np
from pathlib import Path
from typing import Dict, List, Tuple
from dataclasses import dataclass
from loguru import logger
import yaml


@dataclass
class ScenarioConfig:
    """Configuration for a policy intervention scenario."""
    name: str
    description: str
    training_expansion_pct: float  # % increase in annual graduates
    retention_improvement_pct: float  # % reduction in turnover
    immigration_annual: int  # Annual imported workforce
    start_year: int


@dataclass
class ScenarioResult:
    """Result of scenario simulation."""
    scenario_name: str
    profession: str
    sector: str
    year: int
    baseline_supply: float
    scenario_supply: float
    demand: float
    gap_reduction: float
    gap_closed_pct: float


class WorkforceScenarioModeler:
    """
    Model policy intervention scenarios for workforce planning.
    
    Interventions:
    - Training capacity expansion
    - Retention rate improvements
    - Immigration/overseas recruitment
    """
    
    def __init__(
        self,
        forecast_df: pl.DataFrame,
        gap_df: pl.DataFrame,
        config_path: str
    ):
        """
        Initialize scenario modeler.
        
        Args:
            forecast_df: Baseline workforce forecasts (US-06)
            gap_df: Demand-supply gaps (US-07)
            config_path: Path to scenario configurations
        """
        self.forecasts = forecast_df
        self.gaps = gap_df
        self.scenarios = self._load_scenarios(config_path)
        logger.info("Initialized WorkforceScenarioModeler")
    
    def _load_scenarios(self, config_path: str) -> List[ScenarioConfig]:
        """Load scenario definitions from YAML."""
        with open(config_path, 'r') as f:
            config = yaml.safe_load(f)
        
        scenarios = []
        for name, params in config['scenarios'].items():
            scenarios.append(ScenarioConfig(
                name=name,
                description=params['description'],
                training_expansion_pct=params.get('training_expansion_pct', 0),
                retention_improvement_pct=params.get('retention_improvement_pct', 0),
                immigration_annual=params.get('immigration_annual', 0),
                start_year=params.get('start_year', 2026)
            ))
        
        logger.info(f"Loaded {len(scenarios)} scenario configurations")
        return scenarios
    
    def calculate_training_impact(
        self,
        baseline_supply: np.ndarray,
        expansion_pct: float,
        training_lag: int = 4,
        start_year: int = 2026
    ) -> np.ndarray:
        """
        Calculate impact of training capacity expansion.
        
        Args:
            baseline_supply: Baseline workforce forecasts (2020-2030)
            expansion_pct: % increase in annual graduates (e.g., 0.20 = 20%)
            training_lag: Years before new graduates enter workforce
            start_year: Year intervention starts
            
        Returns:
            Array of cumulative workforce additions
        """
        years = np.arange(2020, 2031)
        impact = np.zeros(len(years))
        
        # Estimate annual baseline graduates (~5% of workforce)
        baseline_graduates_annual = baseline_supply[0] * 0.05
        additional_graduates_annual = baseline_graduates_annual * expansion_pct
        
        # Apply impact after training lag
        for i, year in enumerate(years):
            if year >= (start_year + training_lag):
                years_active = year - start_year - training_lag + 1
                impact[i] = additional_graduates_annual * years_active
        
        return impact
    
    def calculate_retention_impact(
        self,
        baseline_supply: np.ndarray,
        retention_improvement_pct: float,
        baseline_turnover: float = 0.15,
        start_year: int = 2026
    ) -> np.ndarray:
        """
        Calculate impact of retention rate improvements.
        
        Args:
            baseline_supply: Baseline workforce forecasts
            retention_improvement_pct: % reduction in turnover (e.g., 0.10 = 10%)
            baseline_turnover: Current annual turnover rate (default: 15%)
            start_year: Year intervention starts
            
        Returns:
            Array of cumulative retention gains
        """
        years = np.arange(2020, 2031)
        impact = np.zeros(len(years))
        
        improved_turnover = baseline_turnover - retention_improvement_pct
        retention_gain_rate = baseline_turnover - improved_turnover
        
        for i, year in enumerate(years):
            if year >= start_year:
                years_active = year - start_year + 1
                # Cumulative compound effect
                impact[i] = baseline_supply[i] * (
                    (1 + retention_gain_rate) ** years_active - 1
                )
        
        return impact
    
    def calculate_immigration_impact(
        self,
        immigration_annual: int,
        start_year: int = 2026
    ) -> np.ndarray:
        """
        Calculate impact of immigration/overseas recruitment.
        
        Args:
            immigration_annual: Number of imported workers per year
            start_year: Year intervention starts
            
        Returns:
            Array of cumulative immigration additions
        """
        years = np.arange(2020, 2031)
        impact = np.zeros(len(years))
        
        for i, year in enumerate(years):
            if year >= start_year:
                years_active = year - start_year + 1
                impact[i] = immigration_annual * years_active
        
        return impact
    
    def simulate_scenario(
        self,
        scenario: ScenarioConfig,
        profession: str,
        sector: str = 'total'
    ) -> pl.DataFrame:
        """
        Simulate a single policy scenario for a profession.
        
        Args:
            scenario: Scenario configuration
            profession: Profession name
            sector: Sector (default: 'total')
            
        Returns:
            DataFrame with scenario results
        """
        logger.info(f"Simulating scenario '{scenario.name}' for {profession}")
        
        # Get baseline forecasts
        baseline = (
            self.forecasts
            .filter(
                (pl.col('profession') == profession) &
                (pl.col('sector') == sector)
            )
            .sort('year')
        )
        
        baseline_supply = baseline['forecast'].to_numpy()
        
        # Calculate intervention impacts
        training_impact = self.calculate_training_impact(
            baseline_supply,
            scenario.training_expansion_pct,
            start_year=scenario.start_year
        )
        
        retention_impact = self.calculate_retention_impact(
            baseline_supply,
            scenario.retention_improvement_pct,
            start_year=scenario.start_year
        )
        
        immigration_impact = self.calculate_immigration_impact(
            scenario.immigration_annual,
            start_year=scenario.start_year
        )
        
        # Calculate scenario supply
        scenario_supply = (
            baseline_supply +
            training_impact +
            retention_impact +
            immigration_impact
        )
        
        # Create results DataFrame
        years = np.arange(2020, 2031)
        df_scenario = pl.DataFrame({
            'scenario_name': [scenario.name] * len(years),
            'profession': [profession] * len(years),
            'sector': [sector] * len(years),
            'year': years,
            'baseline_supply': baseline_supply,
            'scenario_supply': scenario_supply,
            'training_impact': training_impact,
            'retention_impact': retention_impact,
            'immigration_impact': immigration_impact
        })
        
        # Join with demand and calculate gap reduction
        demand = (
            self.gaps
            .filter(
                (pl.col('profession') == profession) &
                (pl.col('sector') == sector)
            )
            .select(['year', 'demand_baseline', 'gap_baseline'])
        )
        
        df_scenario = df_scenario.join(demand, on='year', how='left')
        
        df_scenario = df_scenario.with_columns([
            (pl.col('scenario_supply') - pl.col('demand_baseline'))
            .alias('scenario_gap'),
            
            (pl.col('gap_baseline') - 
             (pl.col('scenario_supply') - pl.col('demand_baseline')))
            .alias('gap_reduction'),
            
            ((pl.col('gap_baseline') - 
              (pl.col('scenario_supply') - pl.col('demand_baseline'))) / 
             pl.col('gap_baseline') * 100)
            .alias('gap_closed_pct')
        ])
        
        logger.info(f"✓ Scenario simulation complete")
        return df_scenario
    
    def simulate_all_scenarios(
        self,
        professions: List[str]
    ) -> pl.DataFrame:
        """
        Simulate all scenarios for all professions.
        
        Args:
            professions: List of professions to model
            
        Returns:
            Combined DataFrame with all scenario results
        """
        logger.info("=" * 60)
        logger.info("Simulating all policy scenarios")
        logger.info("=" * 60)
        
        all_results = []
        
        for profession in professions:
            for scenario in self.scenarios:
                df_scenario = self.simulate_scenario(scenario, profession)
                all_results.append(df_scenario)
        
        df_all = pl.concat(all_results)
        
        logger.success(f"✓ Simulated {len(all_results)} scenario-profession combinations")
        return df_all
    
    def identify_optimal_scenarios(
        self,
        df_scenarios: pl.DataFrame,
        target_year: int = 2030
    ) -> pl.DataFrame:
        """
        Identify best-performing scenario per profession.
        
        Args:
            df_scenarios: Scenario simulation results
            target_year: Year to evaluate performance
            
        Returns:
            DataFrame with optimal scenarios
        """
        logger.info(f"Identifying optimal scenarios for {target_year}")
        
        # Filter to target year
        target_scenarios = df_scenarios.filter(pl.col('year') == target_year)
        
        # Rank scenarios by gap closed %
        optimal = (
            target_scenarios
            .sort(['profession', 'gap_closed_pct'], descending=[False, True])
            .groupby('profession')
            .first()
        )
        
        logger.info(f"✓ Identified optimal scenarios")
        return optimal
```

### 4. Scenario Configuration

**[CREATE] `shared/config/scenario_config.yml`**

```yaml
# Policy Intervention Scenarios
# ==============================

scenarios:
  baseline:
    description: "No intervention (business as usual)"
    training_expansion_pct: 0.0
    retention_improvement_pct: 0.0
    immigration_annual: 0
    start_year: 2026
  
  training_20pct:
    description: "Expand training capacity by 20%"
    training_expansion_pct: 0.20
    retention_improvement_pct: 0.0
    immigration_annual: 0
    start_year: 2026
  
  retention_10pct:
    description: "Reduce turnover by 10 percentage points"
    training_expansion_pct: 0.0
    retention_improvement_pct: 0.10
    immigration_annual: 0
    start_year: 2026
  
  immigration_100:
    description: "Import 100 workers per year"
    training_expansion_pct: 0.0
    retention_improvement_pct: 0.0
    immigration_annual: 100
    start_year: 2026
  
  combined_moderate:
    description: "Moderate combined intervention"
    training_expansion_pct: 0.15
    retention_improvement_pct: 0.05
    immigration_annual: 50
    start_year: 2026
  
  combined_aggressive:
    description: "Aggressive combined intervention"
    training_expansion_pct: 0.30
    retention_improvement_pct: 0.10
    immigration_annual: 150
    start_year: 2026

assumptions:
  training_lag_years: 4  # Years from enrollment to graduation
  baseline_turnover_rate: 0.15  # 15% annual turnover
  baseline_graduate_rate: 0.05  # 5% of workforce are new graduates
```

### 5. Scenario Evaluation Tests

```python
# shared/tests/unit/test_scenario_modeler.py

import pytest
import numpy as np
import polars as pl
from shared.src.analysis.scenario_modeler import WorkforceScenarioModeler, ScenarioConfig


def test_training_impact_calculation():
    """Test training expansion impact with 4-year lag."""
    modeler = WorkforceScenarioModeler(
        forecast_df=pl.DataFrame(),
        gap_df=pl.DataFrame(),
        config_path="shared/config/scenario_config.yml"
    )
    
    baseline_supply = np.array([1000] * 11)  # 2020-2030
    
    # 20% expansion, 4-year lag, start 2026
    impact = modeler.calculate_training_impact(
        baseline_supply, 0.20, training_lag=4, start_year=2026
    )
    
    # Impact should be 0 before 2030 (2026 + 4 years)
    assert impact[:10].sum() == 0  # 2020-2029
    # Impact should be > 0 in 2030
    assert impact[10] > 0


def test_retention_impact_positive():
    """Test retention improvement increases workforce."""
    modeler = WorkforceScenarioModeler(
        pl.DataFrame(), pl.DataFrame(),
        "shared/config/scenario_config.yml"
    )
    
    baseline_supply = np.array([1000] * 11)
    
    # 10% retention improvement
    impact = modeler.calculate_retention_impact(
        baseline_supply, 0.10, start_year=2026
    )
    
    # Impact should increase over time (compound effect)
    assert impact[-1] > impact[-2]  # 2030 > 2029
    assert impact[10] > 0  # 2030 has positive impact


def test_combined_scenario_closes_gap():
    """Test combined scenario reduces gap."""
    # Mock data
    forecast_df = pl.DataFrame({
        'profession': ['doctors'] * 11,
        'sector': ['total'] * 11,
        'year': list(range(2020, 2031)),
        'forecast': [12000 + i*100 for i in range(11)]
    })
    
    gap_df = pl.DataFrame({
        'profession': ['doctors'] * 11,
        'sector': ['total'] * 11,
        'year': list(range(2020, 2031)),
        'demand_baseline': [13000 + i*150 for i in range(11)],
        'gap_baseline': [-1000] * 11
    })
    
    modeler = WorkforceScenarioModeler(
        forecast_df, gap_df,
        "shared/config/scenario_config.yml"
    )
    
    scenario = ScenarioConfig(
        name='test', description='test',
        training_expansion_pct=0.20,
        retention_improvement_pct=0.10,
        immigration_annual=100,
        start_year=2026
    )
    
    result = modeler.simulate_scenario(scenario, 'doctors', 'total')
    
    # Scenario supply should be > baseline supply in later years
    row_2030 = result.filter(pl.col('year') == 2030)
    assert row_2030['scenario_supply'][0] > row_2030['baseline_supply'][0]
```

### 6. Implementation Steps

**Phase 1: Scenario Infrastructure (Days 1-2)**
- [ ] Create `WorkforceScenarioModeler` class
- [ ] Implement training impact calculator
- [ ] Implement retention impact calculator
- [ ] Implement immigration impact calculator
- [ ] Load scenario configurations from YAML

**Phase 2: Scenario Simulation (Days 2-3)**
- [ ] Load baseline forecasts and gaps (US-06, US-07)
- [ ] Simulate all scenarios for all professions
- [ ] Calculate gap reduction metrics
- [ ] Identify optimal scenarios per profession

**Phase 3: Validation & Comparison (Day 3)**
- [ ] Compare scenarios (training vs retention vs combined)
- [ ] Validate assumptions (turnover rates, training lags)
- [ ] Sensitivity analysis (±20% parameter variation)
- [ ] Unit tests for all impact calculators

**Phase 4: Output & Recommendations (Days 3-4)**
- [ ] Export scenario results: `shared/data/4_processed/scenario_results.parquet`
- [ ] Generate scenario comparison charts
- [ ] Create policy recommendation matrix
- [ ] Document assumptions and limitations

### 7. Success Metrics

- ✅ All scenarios simulated for all professions
- ✅ Optimal scenarios identified for critical professions (from US-07)
- ✅ Gap reduction ≥50% for priority professions under optimal scenario
- ✅ Policy recommendations generated for top 3 priority professions
- ✅ Scenario assumptions validated against Singapore policy constraints

### 8. References

- [Singapore Healthcare Manpower Development Plan](https://www.moh.gov.sg/)
- [OECD Workforce Policy Toolkit](https://www.oecd.org/)
- US-06 (forecasts), US-07 (gap analysis)

---

**Implementation Plan Complete for US-08**  
**Dependencies**: US-06 (forecasts), US-07 (gaps)  
**Enables**: US-10 (dashboard will compare scenarios)
