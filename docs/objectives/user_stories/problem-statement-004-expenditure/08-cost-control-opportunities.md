# Identify Cost Control Opportunities (Lifecycle Stage: Feature Engineering & Insights)

**Story ID**: PS-004-US-08  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Policy Director designing cost control initiatives**,  
I want **to identify and quantify specific cost control opportunities based on driver analysis, inefficiencies, and international benchmarks**,  
So that **I can recommend evidence-based interventions with estimated savings potential and implementation priorities**.

---

## 🎯 Acceptance Criteria

1. **Opportunities identified**
   - Efficiency improvement opportunities (reduce cost per service)
   - Utilization optimization opportunities (appropriate care settings)
   - Administrative cost reduction opportunities
   - Procurement/supply chain improvements (if data supports)

2. **Savings quantified**
   - Potential savings per opportunity (SGD millions)
   - Savings as % of total expenditure
   - Quick wins vs long-term initiatives
   - Cumulative savings potential

3. **Implementation prioritized**
   - Priority matrix: savings potential vs implementation difficulty
   - Quick wins identified (high savings, low difficulty)
   - Phased implementation roadmap
   - Risk assessment per opportunity

4. **Deliverables**
   - Output: `results/tables/cost_control_opportunities.csv`
   - Figures: Opportunity matrix, savings potential charts
   - Policy brief: Cost control recommendations (8-10 pages)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#cost-control-strategies) - Healthcare cost containment frameworks
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-5) - Cost control priorities

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `scikit-learn>=1.3.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-06, PS-004-US-07 (Driver analysis and benchmarking - BLOCKING)
- **Data Sources**: All PS-004 analysis outputs
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Opportunity Identification
- [ ] Efficiency opportunities: gaps vs benchmarks
- [ ] Utilization optimization: shift to lower-cost settings
- [ ] Administrative streamlining opportunities
- [ ] Compare with international best practices

### Savings Estimation
- [ ] Calculate savings per opportunity
- [ ] Estimate as % of total expenditure
- [ ] Conservative vs optimistic scenarios
- [ ] Time horizon: 1-year vs 5-year savings

### Prioritization
- [ ] Create priority matrix: savings vs difficulty
- [ ] Classify: quick wins, major initiatives, long-term
- [ ] Risk assessment (implementation barriers)
- [ ] Phased roadmap

### Recommendation Development
- [ ] Detailed opportunity descriptions
- [ ] Implementation approaches
- [ ] Success metrics
- [ ] Responsible stakeholders

### Visualization
- [ ] Priority matrix scatter plot
- [ ] Savings waterfall chart
- [ ] Implementation timeline
- [ ] Save figures

### Policy Brief
- [ ] Executive summary: top 5 opportunities
- [  ] Detailed opportunity analyses
- [ ] Implementation roadmap
- [ ] Monitoring framework

### Testing & Documentation
- [ ] Validate savings calculations
- [ ] Docstrings
- [ ] Policy brief generation

---

## 📌 Notes

**Opportunity Categories**:
1. **Efficiency Gains**: Reduce cost per admission to peer levels (e.g., 10% reduction)
2. **Care Setting Shifts**: Move appropriate cases from acute to community care (lower cost)
3. **Generic Substitution**: Increase generic drug utilization
4. **Preventive Care**: Invest in prevention to reduce downstream costs

**Savings Calculation Example**:
```python
# Opportunity: Reduce cost per admission by 10%
current_cost_per_admission = 5000  # SGD
target_reduction = 0.10
annual_admissions = 100000
annual_savings = current_cost_per_admission * target_reduction * annual_admissions
# = 50M SGD annual savings
```

**Priority Scoring**:
```python
priority_score = (savings_potential / max_savings) * 0.6 + (1 - implementation_difficulty / 10) * 0.4
```

---

## Implementation Plan

### 1. Feature Overview

Synthesize driver analysis (US-06) and benchmarking (US-07) to identify quantified cost control opportunities, estimate savings, build a priority matrix, and produce an evidence-based policy brief. Primary role: **Healthcare Financial Policy Director**. Blocking: US-06 and US-07.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `ps-004/src/analysis/cost_driver_identifier.py` | US-06 | **Reuse** driver report |
| `results/tables/problem-statement-004/international_expenditure_benchmarking_*.csv` | US-07 | **Reuse** gap data |
| `ps-004/src/analysis/cost_control_analyzer.py` | Missing | **Create** |
| `results/tables/problem-statement-004/cost_control_opportunities.csv` | Missing | **Create** |

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/cost_control_analyzer.py`**
  - `CostControlAnalyzer` class
  - `identify_opportunities()` → `pl.DataFrame`
  - `estimate_savings()` → `pl.DataFrame`
  - `build_priority_matrix()` → `pl.DataFrame`
  - `plot_priority_matrix()` → `None`
  - `save_results()` → `None`

- **[CREATE] `ps-004/tests/unit/test_cost_control_analyzer.py`**

- **[CREATE] `ps-004/notebooks/08-cost-control-opportunities.ipynb`**

---

### 5. Data Pipeline

```
Inputs:
  - cost_driver report dict (US-06): primary driver, controllable %
  - international_benchmarking_*.csv (US-07): gap_vs_singapore_usd
  - shared/data/3_interim/expenditure_drivers_integrated.parquet: cost_per_admission

  → identify_opportunities():
      4 standard categories:
        1. Efficiency: reduce cost_per_admission to peer median (benchmark gap)
        2. Care setting shift: redirect 5% acute admissions to primary care
        3. Generic substitution: modelled 10% drug cost reduction (standard assumption)
        4. Preventive care: modelled 3% long-term reduction in elective admissions

  → estimate_savings():
      For each opportunity:
        annual_savings_sgd_m = current_exp * reduction_factor
      Conservative (50% of optimistic) and optimistic scenarios

  → build_priority_matrix():
      priority_score = savings_pct * 0.6 + (1 - difficulty_score/10) * 0.4
      classify: quick_win (score >0.7), major_initiative (0.4-0.7), long_term (<0.4)

  → plot_priority_matrix(): scatter savings % vs difficulty (bubble = 5yr savings)

  → SAVE: results/tables/problem-statement-004/cost_control_opportunities_*.csv
  → SAVE: reports/figures/problem-statement-004/cost_control_priority_matrix_*.png
  → SAVE: reports/figures/problem-statement-004/cost_control_savings_waterfall_*.png
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `cost_control_analyzer.py`

```python
"""
Cost Control Opportunity Analyzer
===================================

Identifies, quantifies, and prioritises healthcare expenditure cost-control
opportunities for Singapore based on driver analysis and international benchmarks.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from datetime import datetime
from pathlib import Path
from typing import Optional

import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import polars as pl

logger = logging.getLogger(__name__)


# Default opportunity templates (can be overridden via config)
DEFAULT_OPPORTUNITIES = [
    {
        "opportunity_id": "OPP-01",
        "name": "Efficiency: Reduce Cost per Admission to Peer Median",
        "category": "Efficiency",
        "controllability": "controllable",
        "reduction_factor": 0.10,   # 10% reduction in cost per admission
        "difficulty_score": 6,       # 1 (easy) – 10 (hard)
        "time_horizon_years": 3,
        "rationale": "Singapore cost per admission vs peer median gap from US-07 benchmarking",
    },
    {
        "opportunity_id": "OPP-02",
        "name": "Care Setting Shift: Acute → Primary Care (5% admissions)",
        "category": "UtilizationOptimization",
        "controllability": "moderately_controllable",
        "reduction_factor": 0.05,
        "difficulty_score": 7,
        "time_horizon_years": 5,
        "rationale": "5% of acute admissions redirected to GP/polyclinic (lower unit cost)",
    },
    {
        "opportunity_id": "OPP-03",
        "name": "Generic Drug Substitution (10% drug cost reduction)",
        "category": "ProcurementReform",
        "controllability": "controllable",
        "reduction_factor": 0.03,   # ~30% of 10% of ~10% drug share
        "difficulty_score": 4,
        "time_horizon_years": 2,
        "rationale": "Standard WHO recommendation; 10% drug cost reduction ≈ 3% total reduction",
    },
    {
        "opportunity_id": "OPP-04",
        "name": "Preventive Care Investment (3% elective admission reduction)",
        "category": "PreventiveCare",
        "controllability": "moderately_controllable",
        "reduction_factor": 0.03,
        "difficulty_score": 8,
        "time_horizon_years": 10,
        "rationale": "Long-term downstream savings from chronic disease prevention programs",
    },
]


class CostControlAnalyzer:
    """
    Identify and prioritise cost control opportunities for Singapore healthcare.

    Example:
        >>> analyzer = CostControlAnalyzer(total_expenditure_sgd_m=6000.0)
        >>> df = analyzer.run()
    """

    def __init__(
        self,
        total_expenditure_sgd_m: float,
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
        opportunities: Optional[list] = None,
    ) -> None:
        """
        Args:
            total_expenditure_sgd_m: Latest total expenditure (SGD M) from integrated data.
            results_dir: Output directory for tables.
            figures_dir: Output directory for figures.
            opportunities: Optional override list of opportunity dicts.
        """
        self.total_exp = total_expenditure_sgd_m
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.opportunities = opportunities or DEFAULT_OPPORTUNITIES
        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)
        logger.info(
            f"CostControlAnalyzer initialised. Base expenditure: SGD {self.total_exp:.1f}M"
        )

    def identify_opportunities(self) -> pl.DataFrame:
        """
        Return structured opportunity table with metadata.

        Returns:
            DataFrame with opportunity definitions and categories.
        """
        df = pl.DataFrame(self.opportunities)
        logger.info(f"Identified {len(df)} cost control opportunities")
        return df

    def estimate_savings(
        self, df_opportunities: pl.DataFrame
    ) -> pl.DataFrame:
        """
        Estimate annual and 5-year savings for each opportunity.

        Conservative = 50% of optimistic estimate.

        Args:
            df_opportunities: From identify_opportunities().

        Returns:
            DataFrame with savings_sgd_m_annual_optimistic,
            savings_sgd_m_annual_conservative, savings_pct_total_exp columns.
        """
        df = df_opportunities.with_columns([
            (pl.col("reduction_factor") * self.total_exp)
            .alias("savings_sgd_m_annual_optimistic"),
            (pl.col("reduction_factor") * self.total_exp * 0.5)
            .alias("savings_sgd_m_annual_conservative"),
            (pl.col("reduction_factor") * 100)
            .alias("savings_pct_total_exp"),
            (pl.col("reduction_factor") * self.total_exp * pl.col("time_horizon_years") * 0.75)
            .alias("savings_sgd_m_5yr_cumulative"),
        ])
        logger.info(
            f"Estimated savings range: SGD "
            f"{df['savings_sgd_m_annual_conservative'].sum():.1f}M – "
            f"{df['savings_sgd_m_annual_optimistic'].sum():.1f}M annually"
        )
        return df

    def build_priority_matrix(
        self, df_savings: pl.DataFrame
    ) -> pl.DataFrame:
        """
        Score and classify opportunities using priority formula:
          score = savings_pct * 0.6 + (1 - difficulty/10) * 0.4

        Args:
            df_savings: From estimate_savings().

        Returns:
            DataFrame with priority_score and priority_class columns.
        """
        max_savings_pct = df_savings["savings_pct_total_exp"].max()

        df = df_savings.with_columns(
            (
                (pl.col("savings_pct_total_exp") / max_savings_pct) * 0.6
                + (1 - pl.col("difficulty_score") / 10) * 0.4
            ).alias("priority_score")
        ).with_columns(
            pl.when(pl.col("priority_score") >= 0.7)
            .then(pl.lit("quick_win"))
            .when(pl.col("priority_score") >= 0.4)
            .then(pl.lit("major_initiative"))
            .otherwise(pl.lit("long_term"))
            .alias("priority_class")
        ).sort("priority_score", descending=True)

        logger.info(
            "Priority classification:\n%s",
            df.select(["opportunity_id", "name", "priority_score", "priority_class"])
            .to_pandas().to_string()
        )
        return df

    def plot_priority_matrix(
        self, df_matrix: pl.DataFrame, output_path: Path
    ) -> None:
        """
        Scatter plot: savings % (x) vs difficulty (y), bubble size = 5yr cumulative savings.

        Args:
            df_matrix: From build_priority_matrix().
            output_path: Path to save PNG.
        """
        colors = {"quick_win": "#2ecc71", "major_initiative": "#f39c12", "long_term": "#e74c3c"}

        x = df_matrix["savings_pct_total_exp"].to_list()
        y = df_matrix["difficulty_score"].to_list()
        sizes = [
            max(100, v * 20)
            for v in df_matrix["savings_sgd_m_5yr_cumulative"].to_list()
        ]
        labels = df_matrix["opportunity_id"].to_list()
        classes = df_matrix["priority_class"].to_list()
        point_colors = [colors.get(c, "gray") for c in classes]

        fig, ax = plt.subplots(figsize=(9, 6))
        for xi, yi, si, li, ci in zip(x, y, sizes, labels, point_colors):
            ax.scatter(xi, yi, s=si, color=ci, alpha=0.75, edgecolors="white", linewidth=0.5)
            ax.annotate(li, (xi, yi), textcoords="offset points", xytext=(6, 4), fontsize=8)

        ax.set_xlabel("Savings Potential (% Total Expenditure)", fontsize=11)
        ax.set_ylabel("Implementation Difficulty (1-10)", fontsize=11)
        ax.set_title("Cost Control Opportunity Priority Matrix", fontsize=13, fontweight="bold")
        ax.invert_yaxis()  # Easier to implement at bottom
        ax.grid(alpha=0.3)

        legend_handles = [
            mpatches.Patch(color=c, label=l.replace("_", " ").title())
            for l, c in colors.items()
        ]
        ax.legend(handles=legend_handles, fontsize=9, loc="upper right")
        plt.tight_layout()
        output_path.parent.mkdir(parents=True, exist_ok=True)
        fig.savefig(str(output_path), dpi=150, bbox_inches="tight")
        plt.close(fig)
        logger.info(f"✓ Priority matrix saved: {output_path}")
        print(f"✓ Figure saved: {output_path}")

    def save_results(self, df_matrix: pl.DataFrame) -> None:
        """Persist opportunity matrix CSV with timestamp."""
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        out_path = self.results_dir / f"cost_control_opportunities_{ts}.csv"
        df_matrix.write_csv(str(out_path))
        logger.info(f"✓ Opportunities saved: {out_path}")
        print(f"✓ Results saved: {out_path}")

    def run(self) -> pl.DataFrame:
        """Full pipeline: identify → estimate → prioritise → plot → save."""
        df_opp = self.identify_opportunities()
        df_savings = self.estimate_savings(df_opp)
        df_matrix = self.build_priority_matrix(df_savings)
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        self.plot_priority_matrix(
            df_matrix,
            self.figures_dir / f"cost_control_priority_matrix_{ts}.png"
        )
        self.save_results(df_matrix)
        logger.info("Cost control analysis complete")
        return df_matrix
```

---

### 10. Testing Strategy

```python
# ps-004/tests/unit/test_cost_control_analyzer.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.cost_control_analyzer import (
    CostControlAnalyzer,
    DEFAULT_OPPORTUNITIES,
)


@pytest.fixture
def analyzer(tmp_path: Path) -> CostControlAnalyzer:
    return CostControlAnalyzer(
        total_expenditure_sgd_m=6000.0,
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
    )


def test_identify_opportunities_count(analyzer: CostControlAnalyzer) -> None:
    df = analyzer.identify_opportunities()
    assert len(df) == len(DEFAULT_OPPORTUNITIES)
    assert "opportunity_id" in df.columns


def test_estimate_savings_positive(analyzer: CostControlAnalyzer) -> None:
    df_opp = analyzer.identify_opportunities()
    df_sav = analyzer.estimate_savings(df_opp)
    assert (df_sav["savings_sgd_m_annual_optimistic"] > 0).all()
    assert (df_sav["savings_sgd_m_annual_conservative"] < df_sav["savings_sgd_m_annual_optimistic"]).all()


def test_savings_calculation_known_value(analyzer: CostControlAnalyzer) -> None:
    """OPP-01 reduction_factor=0.10 on 6000M → 600M optimistic."""
    df_opp = analyzer.identify_opportunities()
    df_sav = analyzer.estimate_savings(df_opp)
    opp01 = df_sav.filter(pl.col("opportunity_id") == "OPP-01")
    assert abs(opp01["savings_sgd_m_annual_optimistic"][0] - 600.0) < 0.01


def test_priority_matrix_has_required_columns(analyzer: CostControlAnalyzer) -> None:
    df_opp = analyzer.identify_opportunities()
    df_sav = analyzer.estimate_savings(df_opp)
    df_mat = analyzer.build_priority_matrix(df_sav)
    assert "priority_score" in df_mat.columns
    assert "priority_class" in df_mat.columns


def test_priority_score_range(analyzer: CostControlAnalyzer) -> None:
    df_opp = analyzer.identify_opportunities()
    df_sav = analyzer.estimate_savings(df_opp)
    df_mat = analyzer.build_priority_matrix(df_sav)
    scores = df_mat["priority_score"].to_list()
    assert all(0 <= s <= 1 for s in scores)


def test_save_creates_csv(analyzer: CostControlAnalyzer, tmp_path: Path) -> None:
    df_opp = analyzer.identify_opportunities()
    df_sav = analyzer.estimate_savings(df_opp)
    df_mat = analyzer.build_priority_matrix(df_sav)
    analyzer.save_results(df_mat)
    files = list(Path(analyzer.results_dir).glob("cost_control_opportunities_*.csv"))
    assert len(files) == 1
```

---

### 11. Implementation Steps

#### Phase 1: Setup
- [ ] Load latest `total_expenditure` value from integrated Parquet (2018 value)
- [ ] Load benchmarking gap CSV from US-07 to calibrate OPP-01 reduction factor

#### Phase 2: Opportunity Generation
- [ ] Implement `identify_opportunities()` with 4 default categories
- [ ] If gap from US-07 > 10% → adjust OPP-01 `reduction_factor` accordingly

#### Phase 3: Savings Estimation
- [ ] Run `estimate_savings()` — log conservative vs optimistic range
- [ ] Run `build_priority_matrix()` — log quick wins

#### Phase 4: Visualisations
- [ ] Priority matrix scatter plot → `cost_control_priority_matrix_*.png`
- [ ] Savings waterfall bar chart → `cost_control_savings_waterfall_*.png`

#### Phase 5: Output & Tests
- [ ] Save `results/tables/problem-statement-004/cost_control_opportunities_*.csv`
- [ ] Write unit tests; confirm ≥80% coverage
- [ ] Create `notebooks/08-cost-control-opportunities.ipynb` with narrative policy brief

---

### 12. Adaptive Implementation Strategy

- **If benchmark gap (US-07) < 5%** → lower OPP-01 `reduction_factor` to 5%; document that Singapore is already near peer median
- **If driver analysis (US-06) shows volume dominates** → increase OPP-02 weight; add admission management initiative
- **If 5-year savings < SGD 100M for all opportunities** → flag as insufficient; recommend sector-level analysis with DRG/procedure data for more granular opportunities

---

### 21. Version Control

```
Branch: feature/ps-004-us-08-cost-control-opportunities
Commits:
  feat(ps-004): add CostControlAnalyzer with 4 opportunity categories
  feat(ps-004): priority matrix visualisation and savings estimation
  test(ps-004): unit tests for savings calculations and priority scoring
```
