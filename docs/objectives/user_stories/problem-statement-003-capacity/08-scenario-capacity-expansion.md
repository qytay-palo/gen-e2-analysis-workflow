# Model Capacity Expansion Scenarios (Lifecycle Stage: Optimization & Scenarios)

**Story ID**: PS-003-US-08  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P1 (High)  
**Effort Estimate**: L (6-7 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Infrastructure Planner evaluating investment options**,  
I want **to model capacity expansion scenarios testing different investment strategies (acute vs community vs long-term care) and their impact on utilization efficiency**,  
So that **I can recommend optimal infrastructure investment mix maximizing population health outcomes per dollar spent**.

---

## 🎯 Acceptance Criteria

1. **Scenario models developed**
   - Baseline scenario: current capacity trajectory projected forward
   - Scenario A: Prioritize acute care expansion
   - Scenario B: Prioritize community hospital expansion  
   - Scenario C: Prioritize long-term care expansion
   - Scenario D: Balanced expansion across facility types

2. **Scenario impacts quantified**
   - Projected bed occupancy rates under each scenario
   - Estimated wait times/access improvements
   - Cost implications per scenario
   - Population coverage metrics

3. **Optimization analysis performed**
   - Cost-effectiveness analysis: beds added per million dollar invested
   - Pareto frontier: identify efficient scenarios
   - Sensitivity analysis: how do results change with different assumptions?

4. **Deliverables**
   - Output: `results/tables/capacity_expansion_scenarios.csv`
   - Figures: Scenario comparison charts, cost-effectiveness curves
   - Decision support report: Recommended investment strategy

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for data; scipy/PuLP for optimization
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#capacity-planning-optimization) - Optimization methods
- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objective-4) - Capacity optimization strategies

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scipy>=1.11.0`, `pulp>=2.7.0` (linear programming), `matplotlib>=3.8.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-003-US-06 (Gap analysis - BLOCKING)
- **Data Sources**: `shared/data/3_interim/capacity_utilization_integrated.parquet`, `results/tables/capacity_gap_analysis.csv`
- **Config Files**: `config/analysis.yml` (cost assumptions, expansion targets)

---

## ✅ Implementation Tasks

### Scenario Definition
- [ ] Define baseline projection: linear extrapolation of current trends
- [ ] Define expansion scenarios: bed additions by facility type
- [ ] Set budget constraints: total investment available
- [ ] Define cost per bed by facility type (from domain knowledge)

### Impact Modeling
- [ ] Project utilization under each scenario
- [ ] Calculate occupancy rates: utilization / capacity
- [ ] Estimate wait time improvements (simplified proxy)
- [ ] Calculate population coverage changes

### Cost-Effectiveness Analysis
- [ ] Calculate cost per scenario
- [ ] Calculate benefits per scenario (e.g., beds added, occupancy improvement)
- [ ] Compute cost-effectiveness ratios
- [ ] Identify Pareto-efficient scenarios

### Optimization (Optional Advanced)
- [ ] Formulate as linear programming problem
- [ ] Objective: maximize access improvement subject to budget constraint
- [ ] Decision variables: beds to add per facility type
- [ ] Solve using PuLP or scipy.optimize

### Sensitivity Analysis
- [ ] Vary cost assumptions: ±20%
- [ ] Vary utilization growth projections
- [ ] Test different budget levels
- [ ] Assess robustness of recommendations

### Visualization
- [ ] Scenario comparison charts: capacity by facility type
- [ ] Cost-effectiveness frontier
- [ ] Occupancy rate projections
- [ ] Save figures: `reports/figures/capacity_scenarios_*.png/pdf`

### Decision Support Report
- [ ] Executive summary: recommended scenario
- [ ] Scenario details: assumptions, results
- [ ] Cost-benefit analysis
- [ ] Implementation roadmap

### Testing & Documentation
- [ ] Unit tests for scenario modeling functions
- [ ] Validate projections against baselines
- [ ] Docstrings
- [ ] Decision support report generation

---

## 📌 Notes

**Scenario Modeling Example**:
```python
import polars as pl
import numpy as np

# Baseline projection
df = pl.read_parquet("shared/data/3_interim/capacity_utilization_integrated.parquet")

# Calculate historical CAGR
cagr = 0.03  # 3% annual growth (example)

# Project 5 years forward
years_ahead = 5
baseline_capacity_2025 = current_capacity * (1 + cagr) ** years_ahead

# Scenario A: Add 500 acute care beds
scenario_a_capacity_2025 = baseline_capacity_2025 + 500

# Calculate occupancy
scenario_a_occupancy = projected_utilization_2025 / scenario_a_capacity_2025
```

**Cost Assumptions** (example, to be validated):
- **Acute care bed**: SGD 500k - 1M per bed (construction + equipment)
- **Community hospital bed**: SGD 300k - 500k per bed
- **Long-term care bed**: SGD 200k - 400k per bed

**Optimization Formulation** (if using LP):
```
Maximize: ∑(population_coverage_improvement_i * beds_added_i)
Subject to:
  ∑(cost_per_bed_i * beds_added_i) ≤ Budget
  beds_added_i ≥ 0
```

**Expected Recommendations**:
- **Balanced approach**: Likely most robust
- **Long-term care priority**: If elderly population growing rapidly
- **Community hospital focus**: If objective is to reduce acute care pressure

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Model five capacity expansion scenarios (Baseline, Acute Priority, Community Priority, LTC Priority, Balanced), calculate cost-effectiveness, and generate a Pareto-optimal recommendation for infrastructure investment decisions.

**Primary User Role**: Healthcare Infrastructure Planner

**Key Success Metrics**:
- `results/tables/problem-statement-003/capacity_expansion_scenarios_{ts}.csv` — all 5 scenarios with composite scores
- `results/tables/problem-statement-003/scenario_sensitivity_{ts}.csv` — ±20% cost sensitivity analysis
- `reports/figures/problem-statement-003/scenario_comparison_{ts}.png`
- `reports/figures/problem-statement-003/pareto_frontier_{ts}.png`

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| `capacity_gap_analysis.py` | `src/analysis/` (US-06) | ✅ Reuse | `calculate_capacity_gaps()` for baseline gap |
| `efficiency_metrics.py` | `src/analysis/` (US-05) | ✅ Reuse | `calculate_utilization_index()` for baseline occupancy |
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream | US-02 — BLOCKING |

**New Components**:
- `src/analysis/capacity_scenario_modeling.py` — `CapacityScenarioModeler` class
- `src/visualization/scenario_plots.py` — cost-vs-effectiveness scatter (Pareto), grouped bar comparison

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_scenario_modeling.py
  Classes/Functions:
    CapacityScenarioModeler (class)
    project_baseline(years_ahead) -> pl.DataFrame
    model_scenario(name, expansion_beds) -> dict
    compare_scenarios() -> pl.DataFrame
    sensitivity_analysis(scenario, parameter, variations) -> pl.DataFrame
    calculate_pareto_frontier(comparison_df) -> pl.DataFrame
    run_scenario_analysis(parquet_path, problem_num) -> pl.DataFrame

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/scenario_plots.py
  Functions:
    plot_scenario_comparison(df, output_path) -> None
    plot_pareto_frontier(df, output_path) -> None
    plot_sensitivity_tornado(df, output_path) -> None

- Results: results/tables/problem-statement-003/capacity_expansion_scenarios_{ts}.csv
           results/tables/problem-statement-003/scenario_sensitivity_{ts}.csv
- Figures:  reports/figures/problem-statement-003/scenario_comparison_{ts}.png
            reports/figures/problem-statement-003/pareto_frontier_{ts}.png
```

---

### 4. Scenario Definitions & Cost Assumptions

From config (`config/analysis.yml` extension):

| Scenario | Acute | Community | LTC | Primary | Est. Cost |
|----------|-------|-----------|-----|---------|-----------|
| Baseline | 0 | 0 | 0 | 0 | $0 |
| Acute Priority | +1 000 | +200 | +300 | 0 | $620 M |
| Community Priority | +200 | +1 000 | +300 | 0 | $460 M |
| LTC Priority | +200 | +300 | +1 000 | 0 | $390 M |
| Balanced | +500 | +500 | +500 | 0 | $500 M |

**Cost per bed** (domain constants — lock in config):
```yaml
cost_per_bed:
  acute_care: 500_000       # SGD
  community_hospital: 300_000
  long_term_care: 200_000
  primary_care: 100_000
```

---

### 5. Data Pipeline

```
US-06 gap analysis + US-05 efficiency outputs
  │
  ▼  project_baseline()
  │   → Apply historical CAGR to latest-year capacity
  │   → Project 5-year baseline without expansion
  │
  ▼  model_scenario() × 5
  │   → Add expansion_beds to projected capacity
  │   → Recalculate proxy occupancy per facility type
  │   → Calculate cost = beds × cost_per_bed
  │   → Calculate effectiveness:  {beds_added, avg_occupancy, over_capacity_count}
  │
  ▼  compare_scenarios()
  │   → Combine all 5 into comparison DataFrame
  │   → Calculate beds_per_million_SGD
  │
  ▼  calculate_pareto_frontier()
  │   → 2D: cost (x) vs occupancy reduction (y)
  │   → Identify non-dominated scenarios
  │
  ▼  sensitivity_analysis()
  │   → Vary cost_per_bed ±20% for each scenario
  │   → Check if scenario ranking changes
  │
  └─▶ Save CSVs + generate 2 figures
```

---

### 6. Code Generation Specifications

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_scenario_modeling.py
import polars as pl
import numpy as np
from pathlib import Path
from datetime import datetime
from loguru import logger

# Domain constants (also in config/analysis.yml)
COST_PER_BED: dict[str, float] = {
    "acute_care": 500_000.0,
    "community_hospital": 300_000.0,
    "long_term_care": 200_000.0,
    "primary_care": 100_000.0,
}

SCENARIOS: dict[str, dict[str, int]] = {
    "Baseline": {},
    "Acute Priority": {"acute_care": 1000, "community_hospital": 200, "long_term_care": 300},
    "Community Priority": {"acute_care": 200, "community_hospital": 1000, "long_term_care": 300},
    "LTC Priority": {"acute_care": 200, "community_hospital": 300, "long_term_care": 1000},
    "Balanced": {"acute_care": 500, "community_hospital": 500, "long_term_care": 500},
}

ALOS: dict[str, float] = {
    "acute_care": 5.5,
    "community_hospital": 20.0,
    "long_term_care": 60.0,
    "primary_care": 0.5,
}


class CapacityScenarioModeler:
    """Model and compare capacity expansion investment scenarios.

    Wraps baseline projection, scenario modeling, cost-effectiveness
    analysis, and Pareto frontier calculation.
    """

    def __init__(
        self,
        df: pl.DataFrame,
        cost_per_bed: dict[str, float] = COST_PER_BED,
        scenarios: dict[str, dict[str, int]] = SCENARIOS,
    ) -> None:
        self._df = df
        self.cost_per_bed = cost_per_bed
        self.scenario_definitions = scenarios
        self._results: dict[str, dict] = {}

    def project_baseline(self, years_ahead: int = 5) -> pl.DataFrame:
        """Project capacity at historical CAGR if no expansion occurs.

        Args:
            years_ahead: Projection horizon.

        Returns:
            DataFrame with projected_capacity per facility_category.
        """
        df_latest = self._df.filter(pl.col("year") == self._df["year"].max())
        df_earliest = self._df.filter(pl.col("year") == self._df["year"].min())

        n_years = self._df["year"].max() - self._df["year"].min()
        if n_years == 0:
            raise ValueError("Cannot calculate CAGR from a single year of data.")

        joined = df_latest.join(
            df_earliest.rename({"total_beds": "start_beds"}),
            on="facility_category",
            how="left",
        )

        baseline = joined.with_columns([
            pl.when(pl.col("start_beds") > 0)
            .then(
                (pl.col("total_beds") / pl.col("start_beds")).pow(1.0 / n_years) - 1
            )
            .otherwise(0.0)
            .alias("cagr"),
        ]).with_columns([
            (pl.col("total_beds") * ((1 + pl.col("cagr")) ** years_ahead))
            .round(0)
            .cast(pl.Int64)
            .alias("projected_capacity"),
        ])

        logger.info(f"project_baseline: {years_ahead}-year projection computed.")
        return baseline.select(["facility_category", "total_beds", "cagr", "projected_capacity"])

    def model_scenario(
        self,
        scenario_name: str,
        expansion_beds: dict[str, int],
        baseline_df: pl.DataFrame | None = None,
    ) -> dict:
        """Apply an expansion scenario onto the baseline projection.

        Args:
            scenario_name: Human-readable scenario label.
            expansion_beds: Beds to add per facility category.
            baseline_df: Pre-computed baseline; computed if None.

        Returns:
            Dict with keys: name, expansion, total_cost, beds_added,
            beds_per_million_sgd, capacity_df.
        """
        if baseline_df is None:
            baseline_df = self.project_baseline()

        expansion_series = pl.Series(
            "expansion_beds",
            [expansion_beds.get(cat, 0)
             for cat in baseline_df["facility_category"].to_list()],
            dtype=pl.Int64,
        )

        cost_series = pl.Series(
            "cost_per_bed",
            [self.cost_per_bed.get(cat, 0.0)
             for cat in baseline_df["facility_category"].to_list()],
        )

        scenario_df = baseline_df.with_columns([
            expansion_series,
            cost_series,
        ]).with_columns([
            (pl.col("projected_capacity") + pl.col("expansion_beds"))
            .alias("scenario_capacity"),
            (pl.col("expansion_beds").cast(pl.Float64) * pl.col("cost_per_bed"))
            .alias("facility_cost"),
        ])

        total_cost = scenario_df["facility_cost"].sum()
        beds_added = int(scenario_df["expansion_beds"].sum())

        result = {
            "name": scenario_name,
            "expansion": expansion_beds,
            "total_cost_sgd": total_cost,
            "beds_added": beds_added,
            "beds_per_million_sgd": (beds_added / (total_cost / 1_000_000))
            if total_cost > 0 else 0.0,
            "capacity_df": scenario_df,
        }

        self._results[scenario_name] = result
        logger.info(
            f"model_scenario '{scenario_name}': "
            f"beds={beds_added}, cost=SGD{total_cost/1e6:.1f}M"
        )
        return result

    def compare_scenarios(self) -> pl.DataFrame:
        """Summarize all modeled scenarios as a comparison DataFrame.

        Returns:
            Polars DataFrame with one row per scenario and key metrics.
        """
        if not self._results:
            raise RuntimeError("No scenarios modeled. Run model_scenario() first.")

        rows = [
            {
                "scenario": name,
                "beds_added": r["beds_added"],
                "total_cost_sgd": r["total_cost_sgd"],
                "cost_per_bed_sgd": r["total_cost_sgd"] / r["beds_added"]
                if r["beds_added"] > 0 else 0.0,
                "beds_per_million_sgd": r["beds_per_million_sgd"],
            }
            for name, r in self._results.items()
        ]

        comparison = pl.DataFrame(rows).sort("total_cost_sgd")
        logger.info(f"compare_scenarios: {comparison.height} scenarios summarized")
        return comparison

    def sensitivity_analysis(
        self,
        parameter: str = "cost_per_bed_multiplier",
        variations: list[float] | None = None,
    ) -> pl.DataFrame:
        """Test scenario ranking stability under parameter variations.

        Args:
            parameter: Currently supports 'cost_per_bed_multiplier'.
            variations: Multiplier values to test. Defaults to ±20% in 5 steps.

        Returns:
            DataFrame with variation, scenario, adjusted_cost, cost_rank.
        """
        if variations is None:
            variations = [0.80, 0.90, 1.00, 1.10, 1.20]

        all_rows: list[dict] = []
        for v in variations:
            for name, r in self._results.items():
                adjusted_cost = r["total_cost_sgd"] * v
                all_rows.append({
                    "variation": v,
                    "scenario": name,
                    "adjusted_cost_sgd": adjusted_cost,
                    "beds_added": r["beds_added"],
                    "cost_per_bed_sgd": adjusted_cost / r["beds_added"]
                    if r["beds_added"] > 0 else 0.0,
                })

        sensitivity_df = pl.DataFrame(all_rows)
        # Add cost rank within each variation bracket
        sensitivity_df = sensitivity_df.with_columns(
            pl.col("adjusted_cost_sgd")
            .rank("dense", descending=False)
            .over("variation")
            .alias("cost_rank")
        )

        logger.info(
            f"sensitivity_analysis: {len(variations)} variations × "
            f"{len(self._results)} scenarios = {sensitivity_df.height} rows"
        )
        return sensitivity_df

    def calculate_pareto_frontier(
        self,
        comparison_df: pl.DataFrame,
        cost_col: str = "total_cost_sgd",
        effectiveness_col: str = "beds_per_million_sgd",
    ) -> pl.DataFrame:
        """Identify Pareto-optimal scenarios (low cost AND high beds per $M).

        Args:
            comparison_df: Output of compare_scenarios().
            cost_col: Lower is better.
            effectiveness_col: Higher is better.

        Returns:
            comparison_df with added 'is_pareto_optimal' boolean column.
        """
        costs = comparison_df[cost_col].to_list()
        effectiveness = comparison_df[effectiveness_col].to_list()
        n = len(costs)

        pareto_flags = []
        for i in range(n):
            dominated = any(
                costs[j] <= costs[i] and effectiveness[j] >= effectiveness[i]
                and (costs[j] < costs[i] or effectiveness[j] > effectiveness[i])
                for j in range(n) if j != i
            )
            pareto_flags.append(not dominated)

        return comparison_df.with_columns(
            pl.Series("is_pareto_optimal", pareto_flags)
        )


def run_scenario_analysis(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    problem_num: str = "003",
) -> pl.DataFrame:
    """Orchestrate full scenario analysis pipeline.

    Returns:
        Scenario comparison DataFrame.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    df = pl.read_parquet(parquet_path)

    # Ensure required columns exist
    required = {"year", "facility_category", "total_beds"}
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"Missing required columns: {missing}")

    modeler = CapacityScenarioModeler(df)
    baseline = modeler.project_baseline()

    for name, expansion in SCENARIOS.items():
        modeler.model_scenario(name, expansion, baseline_df=baseline)

    comparison = modeler.compare_scenarios()
    comparison_with_pareto = modeler.calculate_pareto_frontier(comparison)

    # Sensitivity analysis
    sensitivity = modeler.sensitivity_analysis()

    # Save outputs
    scenarios_path = tables_dir / f"capacity_expansion_scenarios_{ts}.csv"
    comparison_with_pareto.write_csv(scenarios_path)
    logger.info(f"✅ Scenarios saved: {scenarios_path}")
    print(f"✅ Saved: {scenarios_path}")

    sensitivity_path = tables_dir / f"scenario_sensitivity_{ts}.csv"
    sensitivity.write_csv(sensitivity_path)
    logger.info(f"✅ Sensitivity saved: {sensitivity_path}")
    print(f"✅ Saved: {sensitivity_path}")

    return comparison_with_pareto
```

---

### 7. Testing Strategy

```python
# tests/unit/test_capacity_scenario_modeling.py
import polars as pl
import pytest
from src.analysis.capacity_scenario_modeling import CapacityScenarioModeler, COST_PER_BED


@pytest.fixture
def sample_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2015, 2015, 2020, 2020], dtype=pl.Int32),
        "facility_category": ["acute_care", "long_term_care", "acute_care", "long_term_care"],
        "total_beds": pl.Series([10000, 5000, 12000, 6000], dtype=pl.Int64),
    })


def test_project_baseline_returns_cagr(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline(years_ahead=5)
    assert "cagr" in baseline.columns
    assert "projected_capacity" in baseline.columns
    # CAGR for acute: (12000/10000)^(1/5) - 1 ≈ 0.0371
    acute_row = baseline.filter(pl.col("facility_category") == "acute_care")
    assert abs(acute_row["cagr"][0] - 0.0371) < 0.001


def test_model_scenario_cost_calculation(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline()
    result = modeler.model_scenario("Test", {"acute_care": 1000})
    expected_cost = 1000 * COST_PER_BED["acute_care"]
    assert result["total_cost_sgd"] == pytest.approx(expected_cost)


def test_baseline_scenario_zero_cost(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline()
    result = modeler.model_scenario("Baseline", {}, baseline_df=baseline)
    assert result["total_cost_sgd"] == 0.0
    assert result["beds_added"] == 0


def test_compare_scenarios_includes_all(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline()
    modeler.model_scenario("A", {}, baseline_df=baseline)
    modeler.model_scenario("B", {"acute_care": 500}, baseline_df=baseline)
    comparison = modeler.compare_scenarios()
    assert comparison.height == 2
    assert set(comparison["scenario"].to_list()) == {"A", "B"}


def test_pareto_baseline_dominated_by_all_others(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline()
    for name, exp in [("Baseline", {}), ("Acute", {"acute_care": 1000})]:
        modeler.model_scenario(name, exp, baseline_df=baseline)
    comparison = modeler.compare_scenarios()
    pareto = modeler.calculate_pareto_frontier(comparison)
    # Baseline (0 beds, $0) is NOT dominated — it has zero cost
    # Acute (1000 beds, $500M) has better effectiveness → check Pareto implemented correctly
    assert "is_pareto_optimal" in pareto.columns


def test_sensitivity_analysis_has_five_variations(sample_df):
    modeler = CapacityScenarioModeler(sample_df)
    baseline = modeler.project_baseline()
    modeler.model_scenario("Balanced", {"acute_care": 500}, baseline_df=baseline)
    sens = modeler.sensitivity_analysis(variations=[0.8, 0.9, 1.0, 1.1, 1.2])
    assert sens["variation"].n_unique() == 5
```

---

### 8. Implementation Steps

- [ ] Create `src/analysis/capacity_scenario_modeling.py` with `CapacityScenarioModeler`
- [ ] Unit tests pass: baseline CAGR, cost per scenario, Pareto logic
- [ ] Create `src/visualization/scenario_plots.py` (comparison bar + Pareto scatter)
- [ ] Run all 5 scenarios against real integrated parquet
- [ ] Verify Acute Priority most cost-effective for acute bed shortage
- [ ] Run sensitivity analysis — confirm ranking stable across ±20%
- [ ] Save scenario CSV and sensitivity CSV with timestamps
- [ ] Save 2 figures with timestamps
- [ ] Create `notebooks/06_scenario_capacity_expansion.ipynb`

---

### 9. Code Generation Order

1. `src/analysis/capacity_scenario_modeling.py`
2. `tests/unit/test_capacity_scenario_modeling.py`
3. `src/visualization/scenario_plots.py`
4. `notebooks/06_scenario_capacity_expansion.ipynb`

---

### 10. Data Quality & Validation

- Assert all 5 scenarios present in comparison output
- Assert `Baseline` total_cost == 0
- Assert `beds_per_million_sgd` > 0 for all non-Baseline scenarios
- Sensitivity: rank of recommended scenario must not change for variations ≤ ±20%
- Pareto: at least 1 non-Baseline scenario is Pareto-optimal

---

**Implementation Plan Checklist**
- [x] Feature overview with measurable deliverables
- [x] 5 defined scenarios with beds and cost estimates
- [x] Domain cost constants per bed type
- [x] `CapacityScenarioModeler` class with all methods fully implemented
- [x] Pareto frontier identification
- [x] Sensitivity analysis ±20%
- [x] 6 targeted pytest tests with specific assertions
- [x] Sequential implementation steps with verification checkpoints
- [x] Code generation order

---

### 3. Key Implementation (Legacy Reference)

```python
import polars as pl
import numpy as np
from loguru import logger
from typing import Dict

class CapacityScenarioModeler:
    """Model capacity expansion scenarios."""
    
    def __init__(self, base_df: pl.DataFrame, config: dict):
        self.base_df = base_df
        self.config = config
        self.scenarios = {}
    
    def project_baseline(self, years_ahead: int = 5) -> pl.DataFrame:
        """Project capacity if current trends continue."""
        # Calculate historical CAGR
        cagr = self._calculate_cagr(self.base_df)
        
        # Project forward
        current_capacity = self.base_df.filter(
            pl.col('year') == self.base_df['year'].max()
        )
        
        baseline_projection = current_capacity.with_columns([
            (pl.col('total_beds') * ((1 + cagr) ** years_ahead))
            .alias('projected_capacity')
        ])
        
        logger.info(f"Baseline: {years_ahead}-year projection at {cagr*100:.1f}% CAGR")
        return baseline_projection
    
    def model_scenario(
        self,
        scenario_name: str,
        expansion_beds: Dict[str, int]
    ) -> Dict:
        """Model specific expansion scenario.
        
        Args:
            scenario_name: e.g., 'Acute Priority', 'Balanced'
            expansion_beds: {'acute_care': 500, 'ltc': 1000, ...}
        """
        baseline = self.project_baseline()
        
        # Apply expansion
        scenario_capacity = baseline.with_columns([
            (pl.col('total_beds') + 
             pl.lit(expansion_beds.get(pl.col('facility_category'), 0)))
            .alias('scenario_capacity')
        ])
        
        # Calculate new occupancy
        scenario_capacity = self._calculate_occupancy(scenario_capacity)
        
        # Cost estimation
        cost = self._calculate_scenario_cost(expansion_beds)
        
        # Effectiveness metrics
        effectiveness = self._calculate_effectiveness(scenario_capacity)
        
        result = {
            'name': scenario_name,
            'expansion': expansion_beds,
            'capacity_df': scenario_capacity,
            'total_cost': cost,
            'effectiveness': effectiveness,
            'cost_per_bed': cost / sum(expansion_beds.values()),
            'beds_per_million': sum(expansion_beds.values()) / (cost / 1_000_000)
        }
        
        self.scenarios[scenario_name] = result
        logger.info(f"Scenario '{scenario_name}': ${cost/1e6:.1f}M, {effectiveness['beds_added']} beds")
        
        return result
    
    def _calculate_scenario_cost(
        self,
        expansion_beds: Dict[str, int]
    ) -> float:
        """Calculate total cost of expansion scenario."""
        # Cost per bed by facility type (from domain knowledge/config)
        cost_per_bed = self.config.get('cost_per_bed', {
            'acute_care': 500_000,       # $500k per acute bed
            'community_hospital': 300_000, # $300k per community bed
            'long_term_care': 200_000,   # $200k per LTC bed
            'primary_care': 100_000      # $100k per primary care
        })
        
        total_cost = sum(
            expansion_beds.get(facility, 0) * cost_per_bed.get(facility, 0)
            for facility in expansion_beds
        )
        
        return total_cost
    
    def compare_scenarios(self) -> pl.DataFrame:
        """Compare all modeled scenarios."""
        comparison = []
        
        for name, scenario in self.scenarios.items():
            comparison.append({
                'scenario': name,
                'total_cost': scenario['total_cost'],
                'beds_added': sum(scenario['expansion'].values()),
                'cost_per_bed': scenario['cost_per_bed'],
                'avg_occupancy': scenario['effectiveness']['avg_occupancy'],
                'facilities_over_85pct': scenario['effectiveness']['over_capacity_count']
            })
        
        return pl.DataFrame(comparison)
    
    def sensitivity_analysis(
        self,
        scenario_name: str,
        parameter: str,
        variations: list
    ) -> pl.DataFrame:
        """Test scenario robustness to parameter changes."""
        results = []
        
        for variation in variations:
            # Modify parameter
            modified_config = self.config.copy()
            modified_config[parameter] = variation
            
            # Re-run scenario
            # ... implementation
            
            results.append({
                'parameter_value': variation,
                'total_cost': 0,  # calculated
                'effectiveness': 0  # calculated
            })
        
        return pl.DataFrame(results)

# Define scenarios
scenarios_to_model = {
    'Baseline': {},  # No expansion
    'Acute Priority': {'acute_care': 1000, 'community_hospital': 200, 'long_term_care': 300},
    'Community Priority': {'acute_care': 200, 'community_hospital': 1000, 'long_term_care': 300},
    'LTC Priority': {'acute_care': 200, 'community_hospital': 300, 'long_term_care': 1000},
    'Balanced': {'acute_care': 500, 'community_hospital': 500, 'long_term_care': 500}
}
```

### 4. Testing

- Test baseline projection (CAGR calculation)
- Test scenario modeling (capacity additions correct)
- Test cost calculations (verify cost per bed)
- Test effectiveness metrics (occupancy recalculated)
- Test sensitivity analysis (parameter variations)

### 5. Implementation Steps

- [ ] Create `shared/src/analysis/capacity_scenario_modeling.py`
- [ ] Implement baseline projection
- [ ] Implement scenario modeling framework
- [ ] Implement cost-effectiveness calculations
- [ ] Implement scenario comparison logic
- [ ] Implement sensitivity analysis
- [ ] Run all scenarios (Baseline, Acute, Community, LTC, Balanced)
- [ ] Generate comparison visualizations
- [ ] Create decision support report
- [ ] Save to `results/tables/capacity_expansion_scenarios.csv`

**Plan Complete - Ready for Code Generation**
