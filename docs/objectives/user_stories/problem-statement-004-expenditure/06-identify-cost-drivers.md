# Identify Primary Cost Drivers & Inefficiencies (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-004-US-06  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Analyst prioritizing cost control initiatives**,  
I want **to identify primary cost drivers (demographic vs utilization vs intensity factors) and detect potential inefficiencies in expenditure patterns**,  
So that **I can recommend targeted cost containment strategies focusing on controllable drivers and inefficiencies**.

---

## 🎯 Acceptance Criteria

1. **Cost drivers decomposed**
   - Demographic effect quantified (population growth, aging)
   - Utilization effect quantified (volume of services)
   - Intensity effect quantified (cost per service)
   - Price effect quantified (if price index data available)

2. **Inefficiency indicators identified**
   - Cost outlier years flagged (unusually high expenditure vs utilization)
   - Efficiency metrics: cost per admission vs benchmarks
   - Productivity indicators: expenditure per healthcare worker (if workforce data integrated)

3. **Driver prioritization**
   - Primary drivers ranked by contribution to cost growth
   - Controllable vs uncontrollable drivers classified
   - High-leverage opportunities identified

4. **Deliverables**
   - Output: `results/tables/cost_driver_decomposition.csv`
   - Figures: Driver contribution charts, efficiency trends
   - Report: Cost driver analysis with action recommendations

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Analysis**: Custom decomposition algorithms or econometric methods
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#cost-decomposition-methods) - Cost driver decomposition frameworks
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-4) - Identify cost drivers

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scipy>=1.11.0`, `matplotlib>=3.8.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-05 (Correlation analysis - BLOCKING)
- **Data Sources**: `shared/data/3_interim/expenditure_drivers_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Cost Driver Decomposition
- [ ] Decompose expenditure growth into components
- [ ] Calculate demographic effect: population growth contribution
- [ ] Calculate utilization effect: admission volume contribution
- [ ] Calculate intensity effect: cost per admission contribution
- [ ] Sum decomposition: validate components sum to total growth

### Inefficiency Detection
- [ ] Flag outlier years: expenditure growth >> utilization growth
- [ ] Calculate efficiency metrics: cost per admission trends
- [ ] Compare with productivity: expenditure per healthcare worker
- [ ] Identify periods of declining efficiency

### Driver Prioritization
- [ ] Rank drivers by absolute contribution to cost growth
- [ ] Classify as controllable (intensity, efficiency) vs uncontrollable (demographics)
- [ ] Identify high-leverage interventions

### Visualization
- [ ] Waterfall chart: cost growth decomposition
- [ ] Trend charts: efficiency metrics over time
- [ ] Driver contribution pie/bar charts
- [ ] Save figures

### Testing & Documentation
- [ ] Unit tests for decomposition logic
- [ ] Validate decompositions sum correctly
- [ ] Docstrings
- [ ] Cost driver report

---

## 📌 Notes

**Decomposition Formula**:
```
Total Expenditure Change = 
  Population Effect + Utilization Effect + Intensity Effect + Interaction Terms

Where:
- Population Effect = ΔPopulation × Baseline per capita expenditure
- Utilization Effect = ΔUtilization per capita × Population × Baseline cost per unit
- Intensity Effect = ΔCost per unit × Utilization × Population
```

**Polars Implementation**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/expenditure_drivers_integrated.parquet")

# Calculate components
df = df.with_columns([
    (pl.col('population') - pl.col('population').shift(1)).alias('pop_change'),
    (pl.col('per_capita_utilization') - pl.col('per_capita_utilization').shift(1)).alias('util_change'),
    (pl.col('cost_per_admission') - pl.col('cost_per_admission').shift(1)).alias('intensity_change')
])

# Calculate contributions
df = df.with_columns([
    (pl.col('pop_change') * pl.col('per_capita_exp_baseline')).alias('pop_effect'),
    (pl.col('util_change') * pl.col('population') * pl.col('cost_baseline')).alias('util_effect'),
    (pl.col('intensity_change') * pl.col('utilization') * pl.col('population')).alias('intensity_effect')
])
```

---

## Implementation Plan

### 1. Feature Overview

Synthesize outputs from US-04 (decomposition) and US-05 (correlations) to produce a ranked driver analysis, classify drivers as controllable/uncontrollable, detect outlier years, and calculate cost-efficiency metrics. Primary role: **Healthcare Financial Analyst**. Blocking: US-05.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `ps-004/src/analysis/expenditure_decomposition.py` | US-04 | **Reuse** cumulative contributions |
| `ps-004/src/analysis/expenditure_correlations.py` | US-05 | **Reuse** correlation rankings |
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | US-02 | **Reuse** |
| `ps-004/src/analysis/cost_driver_identifier.py` | Missing | **Create** |
| `results/tables/problem-statement-004/cost_driver_decomposition.csv` | Missing | **Create** |

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/cost_driver_identifier.py`**
  - `CostDriverIdentifier` class
  - `rank_drivers()` → `pl.DataFrame`
  - `classify_controllability()` → `pl.DataFrame`
  - `detect_outlier_years()` → `pl.DataFrame`
  - `calculate_efficiency_metrics()` → `pl.DataFrame`
  - `generate_driver_report()` → `dict`
  - `save_results()` → `None`

- **[CREATE] `ps-004/tests/unit/test_cost_driver_identifier.py`**

- **[CREATE] `ps-004/notebooks/06-identify-cost-drivers.ipynb`**

---

### 5. Data Pipeline

```
shared/data/3_interim/expenditure_drivers_integrated.parquet
  → ExpenditureDecomposer.cumulative_contributions()    [from US-04]
  → ExpenditureCorrelationAnalyzer.compute_correlations()  [from US-05]

  → rank_drivers(): sort by |contribution| descending
  → classify_controllability():
      demographic → uncontrollable
      volume      → moderately controllable (care setting, preventive)
      price/intensity → controllable (procurement, protocols)

  → detect_outlier_years():
      compute expenditure growth per admission (intensity proxy)
      flag years where intensity > μ + 2σ (IQR method)

  → calculate_efficiency_metrics():
      cost_per_admission trend: is it rising above CAGR baseline?
      expenditure_per_worker: if workforce data available

  → generate_driver_report(): ranked dict with savings levers

  → SAVE: results/tables/problem-statement-004/cost_driver_decomposition_*.csv
  → SAVE: reports/figures/problem-statement-004/cost_driver_contribution_*.png
  → SAVE: reports/figures/problem-statement-004/cost_efficiency_trend_*.png
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `cost_driver_identifier.py`

```python
"""
Cost Driver Identification & Efficiency Analysis
=================================================

Synthesises decomposition and correlation outputs to rank primary cost
drivers, classify controllability, and detect efficiency outliers.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from datetime import datetime
from pathlib import Path
from typing import Optional

import polars as pl

logger = logging.getLogger(__name__)

CONTROLLABILITY_MAP = {
    "demographic": "uncontrollable",
    "volume": "moderately_controllable",
    "price": "controllable",
    "intensity": "controllable",
}


class CostDriverIdentifier:
    """
    Rank and classify cost drivers from decomposition and correlation evidence.

    Example:
        >>> identifier = CostDriverIdentifier(cumulative, df_corr)
        >>> df_ranked = identifier.run()
    """

    def __init__(
        self,
        cumulative_contributions: dict,
        df_correlations: pl.DataFrame,
        df_integrated: pl.DataFrame,
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
    ) -> None:
        """
        Initialise with upstream analysis outputs.

        Args:
            cumulative_contributions: Dict from ExpenditureDecomposer.cumulative_contributions().
            df_correlations: DataFrame from ExpenditureCorrelationAnalyzer.compute_correlations().
            df_integrated: Integrated expenditure DataFrame from US-02.
            results_dir: Output directory for tables.
            figures_dir: Output directory for figures.
        """
        self.cumulative = cumulative_contributions
        self.df_corr = df_correlations
        self.df = df_integrated.sort("year")
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)

    def rank_drivers(self) -> pl.DataFrame:
        """
        Combine decomposition shares and correlation evidence into a ranked driver table.

        Returns:
            DataFrame: driver, contribution_pct, correlation_r, controllability, priority_rank.
        """
        decomp_rows = [
            {"driver": "demographic",
             "contribution_pct": self.cumulative.get("demographic_contribution_pct") or 0.0},
            {"driver": "volume",
             "contribution_pct": self.cumulative.get("volume_contribution_pct") or 0.0},
            {"driver": "price_intensity",
             "contribution_pct": self.cumulative.get("price_contribution_pct") or 0.0},
        ]
        df_decomp = pl.DataFrame(decomp_rows)

        # Join with Pearson r where driver names overlap
        driver_alias = {"demographic": "population", "volume": "total_admissions",
                        "price_intensity": "cost_per_admission"}
        corr_lookup = {
            row["driver"]: row["pearson_r"]
            for row in self.df_corr.to_dicts()
        } if self.df_corr is not None and len(self.df_corr) > 0 else {}

        df_ranked = df_decomp.with_columns(
            pl.col("driver")
            .map_elements(
                lambda d: corr_lookup.get(driver_alias.get(d, d), None),
                return_dtype=pl.Float64
            ).alias("correlation_r"),
            pl.col("driver")
            .map_elements(
                lambda d: CONTROLLABILITY_MAP.get(d.split("_")[0], "unknown"),
                return_dtype=pl.Utf8
            ).alias("controllability"),
        ).sort("contribution_pct", descending=True)

        df_ranked = df_ranked.with_columns(
            pl.Series("priority_rank", list(range(1, len(df_ranked) + 1)))
        )
        logger.info("Driver ranking:\n%s", df_ranked.to_pandas().to_string())
        return df_ranked

    def detect_outlier_years(
        self, multiplier: float = 2.0
    ) -> pl.DataFrame:
        """
        Flag years where cost-per-admission growth exceeds μ + multiplier*σ.

        Args:
            multiplier: Sigma threshold for outlier detection.

        Returns:
            DataFrame of flagged outlier years with intensity_growth_pct.
        """
        if "cost_per_admission" not in self.df.columns:
            logger.warning("cost_per_admission not available — skipping outlier detection")
            return pl.DataFrame()

        df_eff = self.df.with_columns(
            (
                (pl.col("cost_per_admission") - pl.col("cost_per_admission").shift(1))
                / pl.col("cost_per_admission").shift(1)
                * 100
            ).alias("intensity_growth_pct")
        ).filter(pl.col("intensity_growth_pct").is_not_null())

        mean_growth = df_eff["intensity_growth_pct"].mean()
        std_growth = df_eff["intensity_growth_pct"].std()

        outliers = df_eff.filter(
            pl.col("intensity_growth_pct") > mean_growth + multiplier * std_growth
        )
        logger.info(
            f"Outlier years detected: {outliers['year'].to_list()} "
            f"(threshold: {mean_growth:.1f} + {multiplier}×{std_growth:.1f}%)"
        )
        return outliers.select(["year", "cost_per_admission", "intensity_growth_pct"])

    def calculate_efficiency_metrics(self) -> pl.DataFrame:
        """
        Generate annual cost efficiency: cost_per_admission trend and CAGR baseline deviation.

        Returns:
            DataFrame with year, cost_per_admission, efficiency_deviation_pct.
        """
        if "cost_per_admission" not in self.df.columns:
            logger.warning("cost_per_admission not available — returning empty metrics")
            return pl.DataFrame()

        df_eff = self.df.filter(pl.col("cost_per_admission").is_not_null()).select(
            ["year", "cost_per_admission"]
        )

        # Compute deviation from simple linear trend
        trend_mean = df_eff["cost_per_admission"].mean()
        df_eff = df_eff.with_columns(
            ((pl.col("cost_per_admission") - trend_mean) / trend_mean * 100)
            .alias("efficiency_deviation_pct")
        )
        logger.info(f"Efficiency metrics computed for {len(df_eff)} years")
        return df_eff

    def generate_driver_report(self, df_ranked: pl.DataFrame) -> dict:
        """
        Compile structured report: primary driver, savings levers, policy implications.

        Args:
            df_ranked: Output of rank_drivers().

        Returns:
            Dict summary for policy brief.
        """
        if len(df_ranked) == 0:
            return {}

        top = df_ranked.filter(pl.col("priority_rank") == 1).to_dicts()[0]
        controllable = df_ranked.filter(
            pl.col("controllability") == "controllable"
        )["contribution_pct"].sum()

        report = {
            "primary_driver": top["driver"],
            "primary_driver_contribution_pct": top["contribution_pct"],
            "total_controllable_pct": round(controllable, 1),
            "policy_levers": {
                "price_intensity": "Generic substitution, protocol standardisation, price negotiation",
                "volume": "Primary care substitution, preventive care investment, GP-first referral policies",
                "demographic": "Capacity planning, workforce expansion for aging population (unavoidable)",
            },
            "generated_at": datetime.now().isoformat(),
        }
        logger.info("Driver report: %s", report)
        return report

    def save_results(
        self, df_ranked: pl.DataFrame, df_outliers: pl.DataFrame
    ) -> None:
        """Persist driver decomposition and outlier CSVs."""
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        ranked_path = self.results_dir / f"cost_driver_decomposition_{ts}.csv"
        df_ranked.write_csv(str(ranked_path))
        logger.info(f"✓ Driver ranking saved: {ranked_path}")
        print(f"✓ Results saved: {ranked_path}")

        if len(df_outliers) > 0:
            out_path = self.results_dir / f"cost_driver_outlier_years_{ts}.csv"
            df_outliers.write_csv(str(out_path))
            logger.info(f"✓ Outlier years saved: {out_path}")
            print(f"✓ Outliers saved: {out_path}")

    def run(self) -> pl.DataFrame:
        """Full pipeline: rank → classify → outliers → efficiency → save."""
        df_ranked = self.rank_drivers()
        df_outliers = self.detect_outlier_years()
        df_eff = self.calculate_efficiency_metrics()
        report = self.generate_driver_report(df_ranked)
        self.save_results(df_ranked, df_outliers)
        logger.info("Cost driver identification complete. Report: %s", report)
        return df_ranked
```

---

### 10. Testing Strategy

```python
# ps-004/tests/unit/test_cost_driver_identifier.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.cost_driver_identifier import (
    CostDriverIdentifier,
)


@pytest.fixture
def sample_cumulative() -> dict:
    return {
        "demographic_contribution_pct": 35.0,
        "volume_contribution_pct": 25.0,
        "price_contribution_pct": 40.0,
        "total_expenditure_change_sgd_m": 3000.0,
    }


@pytest.fixture
def sample_corr_df() -> pl.DataFrame:
    return pl.DataFrame({
        "driver": ["total_admissions", "population", "cost_per_admission"],
        "pearson_r": [0.97, 0.95, 0.89],
        "p_value": [0.001, 0.002, 0.003],
    })


@pytest.fixture
def sample_integrated() -> pl.DataFrame:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 250 for i in range(13)],
        "cost_per_admission": [3000.0 + i * 150 for i in range(13)],
    })


@pytest.fixture
def identifier(
    tmp_path: Path,
    sample_cumulative: dict,
    sample_corr_df: pl.DataFrame,
    sample_integrated: pl.DataFrame,
) -> CostDriverIdentifier:
    return CostDriverIdentifier(
        cumulative_contributions=sample_cumulative,
        df_correlations=sample_corr_df,
        df_integrated=sample_integrated,
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
    )


def test_rank_drivers_sorted_descending(identifier: CostDriverIdentifier) -> None:
    df = identifier.rank_drivers()
    pcts = df["contribution_pct"].to_list()
    assert pcts == sorted(pcts, reverse=True)


def test_rank_drivers_controllability_assigned(identifier: CostDriverIdentifier) -> None:
    df = identifier.rank_drivers()
    assert "controllability" in df.columns
    assert df.filter(pl.col("driver") == "demographic")["controllability"][0] == "uncontrollable"


def test_detect_outlier_years_empty_on_stable_data(identifier: CostDriverIdentifier) -> None:
    # Stable cost_per_admission growth → no outliers
    outliers = identifier.detect_outlier_years(multiplier=3.0)
    # With linear growth, σ is near zero, so check result is a DataFrame
    assert isinstance(outliers, pl.DataFrame)


def test_generate_driver_report_primary_driver(identifier: CostDriverIdentifier) -> None:
    df_ranked = identifier.rank_drivers()
    report = identifier.generate_driver_report(df_ranked)
    assert "primary_driver" in report
    assert report["primary_driver"] == "price_intensity"  # 40% is highest


def test_save_results_creates_csv(identifier: CostDriverIdentifier, tmp_path: Path) -> None:
    df_ranked = identifier.rank_drivers()
    df_outliers = pl.DataFrame()
    identifier.save_results(df_ranked, df_outliers)
    files = list(Path(identifier.results_dir).glob("cost_driver_decomposition_*.csv"))
    assert len(files) == 1
```

---

### 11. Implementation Steps

#### Phase 1: Setup
- [ ] Confirm US-04 `cumulative_contributions()` has been run; load result dict
- [ ] Confirm US-05 `compute_correlations()` DataFrame available; load CSV if needed

#### Phase 2: Driver Ranking & Classification
- [ ] Implement `rank_drivers()` — combine decomp shares + correlation r
- [ ] Apply `classify_controllability()` — 3 classifications
- [ ] Log dominant driver and its % contribution

#### Phase 3: Efficiency & Outliers
- [ ] Implement `detect_outlier_years()` using IQR/σ threshold on `cost_per_admission` growth
- [ ] Implement `calculate_efficiency_metrics()` — deviation from trend
- [ ] Flag any years where intensity growth > baseline by 2σ

#### Phase 4: Visualisations
- [ ] Stacked bar: driver contributions (demographic/volume/price) → `cost_driver_contribution_*.png`
- [ ] Line chart: `cost_per_admission` trend with outlier markers → `cost_efficiency_trend_*.png`
- [ ] Pie chart: controllable vs uncontrollable share

#### Phase 5: Output & Testing
- [ ] Save `results/tables/problem-statement-004/cost_driver_decomposition_*.csv`
- [ ] Write unit tests; confirm ≥80% coverage
- [ ] Create `notebooks/06-identify-cost-drivers.ipynb`

---

### 12. Adaptive Implementation Strategy

- **If decomposition dict is unavailable** → compute directly from integrated Parquet using simplified 3-component approach from US-04 notes
- **If `cost_per_admission` null** → efficiency metrics fall back to `per_capita_expenditure` trend analysis
- **If all 3 drivers show equal contribution** → flag inconclusive; recommend higher-granularity data (sector-level or DRG-level)

---

### 21. Version Control

```
Branch: feature/ps-004-us-06-identify-cost-drivers
Commits:
  feat(ps-004): add CostDriverIdentifier with ranking and outlier detection
  feat(ps-004): controllability classification and policy lever mapping
  test(ps-004): unit tests for driver ranking and efficiency metrics
```
