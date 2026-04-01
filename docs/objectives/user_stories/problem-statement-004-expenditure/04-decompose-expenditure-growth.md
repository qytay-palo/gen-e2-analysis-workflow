# Decompose Healthcare Expenditure Growth Components (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-004-US-04  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: L (6-7 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financing Policy Analyst**,  
I want **to decompose 15-year expenditure growth (2005-2020) into demographic effects (population aging, growth), volume effects (more services), and price/intensity effects (cost per service)**,  
So that **I can identify whether cost growth is driven by unavoidable factors (aging population) vs controllable factors (utilization intensity, prices) and target interventions accordingly**.

---

## 🎯 Acceptance Criteria

1. **Expenditure decomposition completed**
   - Growth decomposed into 3 components:
     1. **Demographic effect**: Population size and age structure changes
     2. **Volume effect**: Changes in utilization rates (more admissions per capita)
     3. **Price/intensity effect**: Changes in cost per service (residual after accounting for demographic + volume)
   - Annual decomposition calculated for each year 2006-2020
   - Cumulative contribution calculated: % of total growth attributable to each component

2. **Decomposition methodology validated**
   - Method: Index decomposition analysis (IDA) or Laspeyres index approach
   - Cross-validation: sum of components equals total expenditure growth
   - Sensitivity analysis: results stable under alternative decomposition methods

3. **Findings documented**
   - Primary driver identified: which component contributes most to growth?
   - Temporal patterns: has dominant driver changed over time (e.g., aging accelerating post-2015)?
   - Policy implications: where can interventions have most impact?

4. **Data output requirement**
   - Output file: `results/tables/expenditure_decomposition.csv`
   - Format: CSV (year, total_growth_pct, demographic_contribution_pct, volume_contribution_pct, price_contribution_pct)
   - Decomposition visualization: `reports/figures/expenditure_decomposition_waterfall.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Advanced Methods**: Custom decomposition implementation or econometric packages
- **Logging**: loguru
- **Testing**: pytest with ≥80% coverage

---

## 📚 Domain Knowledge References

- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objectives) - Objective 2: Identify primary drivers through decomposition
- Health economics literature: Index decomposition analysis (IDA) methods

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data processing
- `matplotlib>=3.8.0` or `plotly>=5.18.0`: Waterfall charts
- `loguru>=0.7.0`: Logging
- `statsmodels>=0.14.0` (optional): Econometric validation

### Internal Dependencies
- **Upstream**: 
  - PS-004-US-02 (Integrated expenditure data - BLOCKING)
  - PS-004-US-03 (Expenditure trends - BLOCKING)
- **Data Sources**: 
  - `shared/data/3_interim/expenditure_drivers_integrated.parquet` (expenditure + utilization + demographics)
- **Config Files**: `config/analysis.yml` (decomposition method selection)

---

## ✅ Implementation Tasks

### Data Preparation
- [ ] Load integrated dataset: expenditure, population, utilization, age structure
- [ ] Calculate base year values (2005): expenditure, population, utilization rate, age mix
- [ ] Calculate end year values (2020): same variables
- [ ] Prepare annual time series for dynamic decomposition

### Decomposition Method Implementation

**Option 1: Laspeyres Decomposition (Simpler)**
- [ ] Calculate demographic effect:
  - Effect of population growth: `(pop_2020 / pop_2005) * exp_2005 - exp_2005`
  - Effect of aging: Use age-specific utilization × population age structure changes
  
- [ ] Calculate volume effect:
  - Effect of utilization change: `(util_rate_2020 / util_rate_2005) * exp_demographic - exp_demographic`
  
- [ ] Calculate price/intensity effect (residual):
  - `exp_2020 - exp_demographic - exp_volume`

**Option 2: Logarithmic Mean Divisia Index (LMDI, Preferred)**
- [ ] Implement LMDI decomposition:
  - Demographic: `Σ[L(exp_t, exp_t-1) * ln(pop_t / pop_t-1)]`
  - Volume: `Σ[L(exp_t, exp_t-1) * ln(util_t / util_t-1)]`
  - Price: `Σ[L(exp_t, exp_t-1) * ln(price_t / price_t-1)]`
  - Where L(a,b) = (a-b)/ln(a/b) (logarithmic mean)

- [ ] Calculate annually (2006-2020) for dynamic analysis
- [ ] Calculate cumulative contributions over full period

### Component Validation
- [ ] Verify additive property: `sum(components) = total_growth`
- [ ] Handle edge cases: zero values, negative growth
- [ ] Cross-check with alternative methods (Laspeyres vs LMDI)

### Sensitivity Analysis
- [ ] Test alternative base years (2005 vs 2010)
- [ ] Test alternative age groupings (5-year vs 10-year age bands)
- [ ] Quantify uncertainty in decomposition estimates

### Interpretation & Insights
- [ ] Identify dominant driver: rank components by contribution magnitude
- [ ] Temporal pattern analysis: when did each component peak?
- [ ] Policy implications: 
  - High demographic effect → unavoidable (aging population)
  - High volume effect → potential over-utilization (policy target)
  - High price effect → cost control opportunity (negotiate prices, efficiency)

### Visualization
- [ ] Waterfall chart: decomposition showing cumulative contributions
- [ ] Stacked area chart: annual contributions over time
- [ ] Pie chart: cumulative contribution shares 2005-2020
- [ ] Export figures

### Testing & Validation
- [ ] Unit tests for decomposition formulas
- [ ] Validate perfect decomposition: components sum to total
- [ ] Test edge cases: zero growth, negative growth, missing data

### Documentation
- [ ] Docstrings (Google style)
- [ ] Methodology document: `results/expenditure_decomposition_methodology.md`
  - Decomposition method rationale (why LMDI vs Laspeyres)
  - Mathematical formulas
  - Assumptions and limitations
  - Sensitivity analysis results
- [ ] Policy brief: `results/expenditure_drivers_summary.md`
  - Top 3 drivers with evidence
  - Policy recommendations per driver
- [ ] Update data dictionary

---

## 📌 Notes

**Laspeyres Decomposition (Simplified Example)**:
```python
import polars as pl

# Base year: 2005
base_exp = df.filter(pl.col('year') == 2005).select('expenditure').item()
base_pop = df.filter(pl.col('year') == 2005).select('population').item()
base_util = df.filter(pl.col('year') == 2005).select('utilization_rate').item()

# End year: 2020
end_exp = df.filter(pl.col('year') == 2020).select('expenditure').item()
end_pop = df.filter(pl.col('year') == 2020).select('population').item()
end_util = df.filter(pl.col('year') == 2020).select('utilization_rate').item()

# Decomposition
demographic_effect = (end_pop / base_pop) * base_exp - base_exp
volume_effect = (end_util / base_util) * (base_exp + demographic_effect) - (base_exp + demographic_effect)
price_effect = end_exp - (base_exp + demographic_effect + volume_effect)

total_growth = end_exp - base_exp
assert abs((demographic_effect + volume_effect + price_effect) - total_growth) < 0.01

logger.info(f"Total growth: ${total_growth:,.2f}")
logger.info(f"  Demographic: {demographic_effect/total_growth*100:.1f}%")
logger.info(f"  Volume: {volume_effect/total_growth*100:.1f}%")
logger.info(f"  Price: {price_effect/total_growth*100:.1f}%")
```

**LMDI Formula (for reference)**:
```
Logarithmic Mean: L(a, b) = (a - b) / ln(a / b)

Component contribution = L(exp_t, exp_t-1) * ln(factor_t / factor_t-1)
```

**Expected Findings** (hypotheses):
- **Demographic effect**: ~30-40% of growth (Singapore's aging population well-documented)
- **Volume effect**: ~20-30% (healthcare utilization increasing beyond demographic effects)
- **Price/intensity effect**: ~30-50% (technology advancement, treatment intensity, labor costs)

**Policy Implications by Driver**:
| Driver | Contribution | Controllability | Policy Levers |
|--------|-------------|----------------|---------------|
| Demographic | 30-40% | Low (aging unavoidable) | Plan capacity, workforce for aging population |
| Volume | 20-30% | Medium | Reduce over-utilization, preventive care, efficiency |
| Price/Intensity | 30-50% | High | Price negotiation, generic drugs, standardized protocols, technology assessment |

**Limitations**:
- Residual method: Price effect captures all unmodeled drivers (could include quality improvements, new technology, administrative costs)
- Aggregate analysis: Sector-specific drivers (e.g., primary vs specialist care) masked
- Data constraints: Limited granularity for detailed price decomposition

---

## Implementation Plan

### 1. Feature Overview

Decompose 2006-2018 expenditure growth into **demographic**, **volume**, and **price/intensity** effects using a Laspeyres approach (simple, exact additive property) with LMDI validation. Identifies whether cost growth is driven by unavoidable aging vs controllable utilization/price factors. Primary role: **Healthcare Financing Policy Analyst**.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | Created by US-02 | **Reuse** |
| `shared/src/analysis/trend_analysis.py` | Exists | **Reference** CAGR patterns |
| `ps-004/src/analysis/expenditure_trend_analyzer.py` | Created by US-03 | **Reuse** `load()` pattern |
| `ps-004/src/analysis/expenditure_decomposition.py` | Missing | **Create** |
| `results/tables/problem-statement-004/expenditure_decomposition.csv` | Missing | **Create** |
| `reports/figures/problem-statement-004/expenditure_decomposition_*.png` | Missing | **Create** |

No ML model required.

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/expenditure_decomposition.py`**
  - `ExpenditureDecomposer` class
  - `laspeyres_decomposition()` → `pl.DataFrame`
  - `lmdi_decomposition()` → `pl.DataFrame`
  - `validate_additive_property()` → `bool`
  - `sensitivity_analysis()` → `dict`
  - `save_results()` → `None`

- **[CREATE] `ps-004/src/visualization/decomposition_plots.py`**
  - `plot_waterfall()`, `plot_stacked_area()`, `plot_contribution_pie()`

- **[CREATE] `ps-004/tests/unit/test_expenditure_decomposition.py`**

- **[CREATE] `ps-004/notebooks/04-decompose-expenditure-growth.ipynb`**

---

### 5. Data Pipeline

```
shared/data/3_interim/expenditure_drivers_integrated.parquet
  → filter 2006-2018, sort by year
  → extract: year, total_expenditure, total_admissions, population, per_capita_expenditure

  → Laspeyres (annual, 2006-2018):
      demographic_effect_t = (pop_t/pop_t-1 - 1) * exp_t-1
      volume_effect_t      = (util_rate_t/util_rate_t-1 - 1) * (exp_t-1 + demographic_effect_t)
      price_effect_t       = exp_t - exp_t-1 - demographic_effect_t - volume_effect_t

  → validate: sum(d + v + p) == Δexp  (tolerance 1e-6)

  → LMDI cross-check (optional, requires log ratios)

  → cumulative contributions over full 2006-2018 period

  → SAVE: results/tables/problem-statement-004/expenditure_decomposition.csv
  → SAVE: reports/figures/problem-statement-004/expenditure_decomposition_waterfall.png
  → SAVE: reports/figures/problem-statement-004/expenditure_decomposition_stacked_area.png
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `expenditure_decomposition.py`

```python
"""
Expenditure Growth Decomposition
==================================

Decomposes Singapore health expenditure growth (2006-2018) into:
  - Demographic effect (population size & structure changes)
  - Volume effect (utilization rate changes)
  - Price/intensity effect (residual: cost per service unit changes)

Methods: Annual Laspeyres decomposition with LMDI cross-validation.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
import math
from datetime import datetime
from pathlib import Path
from typing import Optional

import polars as pl

logger = logging.getLogger(__name__)

ANALYSIS_START = 2006
ANALYSIS_END = 2018
TOLERANCE = 1e-4   # Acceptable residual in additive check (SGD millions)


class ExpenditureDecomposer:
    """
    Decompose health expenditure growth into demographic, volume, and price effects.

    Example:
        >>> decomposer = ExpenditureDecomposer()
        >>> df = decomposer.run()
    """

    def __init__(
        self,
        data_path: str = "shared/data/3_interim/expenditure_drivers_integrated.parquet",
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
    ) -> None:
        self.data_path = Path(data_path)
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)

        if not self.data_path.exists():
            raise FileNotFoundError(
                f"Integrated dataset not found: {self.data_path}\n"
                "Please run PS-004-US-02 first."
            )
        self.df_raw = pl.read_parquet(str(self.data_path))
        logger.info(f"Loaded {len(self.df_raw)} rows for decomposition")

    def _load_analysis_window(self) -> pl.DataFrame:
        """Filter to 2006-2018 and sort by year."""
        df = (
            self.df_raw
            .filter(
                (pl.col("year") >= ANALYSIS_START)
                & (pl.col("year") <= ANALYSIS_END)
            )
            .sort("year")
        )
        if len(df) == 0:
            raise ValueError("No data in 2006-2018 window after filtering")
        return df

    def laspeyres_decomposition(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Annual Laspeyres decomposition: demographic + volume + price/intensity effects.

        Derivation:
          Let E = expenditure, P = population, U = util_rate, I = intensity (E/P/U)
          E_t = P_t * U_t * I_t

          demographic_effect = (P_t/P_{t-1} - 1) * E_{t-1}
          volume_effect      = (U_t/U_{t-1} - 1) * (E_{t-1} + demographic_effect)
          price_effect       = E_t - E_{t-1} - demographic_effect - volume_effect

        Args:
            df: Integrated DataFrame sorted by year with columns:
                year, total_expenditure, population, total_admissions.

        Returns:
            DataFrame with decomposition columns added per year.
        """
        # Derive utilization rate (admissions per capita) if both columns present
        has_util = "total_admissions" in df.columns and "population" in df.columns
        has_pop = "population" in df.columns

        if has_util and has_pop:
            df = df.with_columns(
                (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
            )
        else:
            logger.warning(
                "population/total_admissions not both available — "
                "volume effect will be null; only price effect computed"
            )
            df = df.with_columns(pl.lit(None).cast(pl.Float64).alias("util_rate"))

        records = []
        rows = df.to_dicts()

        for i, row in enumerate(rows):
            if i == 0:
                records.append({
                    "year": row["year"],
                    "total_expenditure": row["total_expenditure"],
                    "expenditure_change": None,
                    "demographic_effect": None,
                    "volume_effect": None,
                    "price_effect": None,
                    "demographic_share_pct": None,
                    "volume_share_pct": None,
                    "price_share_pct": None,
                })
                continue

            prev = rows[i - 1]
            exp_t = row["total_expenditure"]
            exp_prev = prev["total_expenditure"]
            delta_exp = exp_t - exp_prev

            # Demographic effect
            if has_pop and prev["population"] and row["population"]:
                dem_eff = (row["population"] / prev["population"] - 1) * exp_prev
            else:
                dem_eff = 0.0

            # Volume effect
            if has_util and prev["util_rate"] and row["util_rate"]:
                vol_eff = (row["util_rate"] / prev["util_rate"] - 1) * (exp_prev + dem_eff)
            else:
                vol_eff = 0.0

            # Price/intensity (residual)
            price_eff = delta_exp - dem_eff - vol_eff

            total = abs(delta_exp) if abs(delta_exp) > 1e-9 else 1.0
            records.append({
                "year": row["year"],
                "total_expenditure": exp_t,
                "expenditure_change": delta_exp,
                "demographic_effect": dem_eff,
                "volume_effect": vol_eff,
                "price_effect": price_eff,
                "demographic_share_pct": dem_eff / total * 100 if delta_exp != 0 else None,
                "volume_share_pct": vol_eff / total * 100 if delta_exp != 0 else None,
                "price_share_pct": price_eff / total * 100 if delta_exp != 0 else None,
            })

        result = pl.DataFrame(records)
        logger.info(
            "Laspeyres decomposition complete. Avg shares: "
            "demog=%.1f%%, volume=%.1f%%, price=%.1f%%",
            result["demographic_share_pct"].mean() or 0,
            result["volume_share_pct"].mean() or 0,
            result["price_share_pct"].mean() or 0,
        )
        return result

    @staticmethod
    def _log_mean(a: float, b: float) -> float:
        """Logarithmic mean: L(a, b) = (a - b) / ln(a/b); L(a,a)=a."""
        if abs(a - b) < 1e-12:
            return a
        if a <= 0 or b <= 0:
            return 0.0
        return (a - b) / math.log(a / b)

    def lmdi_decomposition(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        LMDI additive decomposition as cross-validation.

        Component_t = L(E_t, E_{t-1}) * ln(factor_t / factor_{t-1})

        Args:
            df: Integrated DataFrame sorted by year.

        Returns:
            DataFrame with lmdi_ prefixed columns.
        """
        has_util = "total_admissions" in df.columns and "population" in df.columns
        has_pop = "population" in df.columns

        records = []
        rows = df.to_dicts()

        for i, row in enumerate(rows):
            if i == 0:
                records.append({"year": row["year"], "lmdi_demographic": None,
                                  "lmdi_volume": None, "lmdi_price": None})
                continue

            prev = rows[i - 1]
            exp_t = row["total_expenditure"]
            exp_prev = prev["total_expenditure"]

            if exp_t <= 0 or exp_prev <= 0:
                records.append({"year": row["year"], "lmdi_demographic": None,
                                  "lmdi_volume": None, "lmdi_price": None})
                continue

            L = self._log_mean(exp_t, exp_prev)

            # Demographic (population)
            if has_pop and prev.get("population") and row.get("population"):
                lmdi_dem = L * math.log(row["population"] / prev["population"])
            else:
                lmdi_dem = 0.0

            # Volume (util_rate = admissions/population)
            if has_util and prev.get("util_rate") and row.get("util_rate"):
                lmdi_vol = L * math.log(row["util_rate"] / prev["util_rate"])
            else:
                lmdi_vol = 0.0

            # Price/intensity residual approximated via spending intensity
            if prev.get("util_rate") and row.get("util_rate") and has_pop:
                intensity_prev = exp_prev / (prev["population"] * prev["util_rate"] + 1e-12)
                intensity_t = exp_t / (row["population"] * row["util_rate"] + 1e-12)
                lmdi_price = L * math.log(intensity_t / intensity_prev) if intensity_prev > 0 else 0
            else:
                lmdi_price = (exp_t - exp_prev) - lmdi_dem - lmdi_vol

            records.append({
                "year": row["year"],
                "lmdi_demographic": lmdi_dem,
                "lmdi_volume": lmdi_vol,
                "lmdi_price": lmdi_price,
            })

        lmdi_df = pl.DataFrame(records)
        logger.info("LMDI decomposition complete")
        return lmdi_df

    def validate_additive_property(self, df: pl.DataFrame) -> bool:
        """
        Check that demographic + volume + price ≈ total expenditure change.

        Args:
            df: Decomposition DataFrame with effect columns.

        Returns:
            True if all rows within TOLERANCE.

        Raises:
            AssertionError: If any row fails the check.
        """
        df_check = df.filter(pl.col("expenditure_change").is_not_null())
        for row in df_check.to_dicts():
            components = (
                (row["demographic_effect"] or 0)
                + (row["volume_effect"] or 0)
                + (row["price_effect"] or 0)
            )
            delta = abs(components - row["expenditure_change"])
            assert delta <= TOLERANCE, (
                f"Additive property violated in year {row['year']}: "
                f"sum={components:.4f}, actual={row['expenditure_change']:.4f}, diff={delta:.6f}"
            )
        logger.info("✓ Additive property validated for all years")
        return True

    def cumulative_contributions(self, df: pl.DataFrame) -> dict:
        """
        Compute cumulative contribution of each component over full period.

        Args:
            df: Decomposition DataFrame.

        Returns:
            Dict with cumulative shares for demographic, volume, price.
        """
        df_valid = df.filter(pl.col("expenditure_change").is_not_null())
        total_change = df_valid["expenditure_change"].sum()
        dem_tot = df_valid["demographic_effect"].sum()
        vol_tot = df_valid["volume_effect"].sum()
        price_tot = df_valid["price_effect"].sum()

        result = {
            "total_expenditure_change_sgd_m": round(total_change, 2),
            "demographic_contribution_pct": round(dem_tot / total_change * 100, 1) if total_change else None,
            "volume_contribution_pct": round(vol_tot / total_change * 100, 1) if total_change else None,
            "price_contribution_pct": round(price_tot / total_change * 100, 1) if total_change else None,
        }
        logger.info("Cumulative contributions 2006-2018: %s", result)
        return result

    def save_results(self, df: pl.DataFrame) -> None:
        """Save decomposition CSV with timestamp."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        out_path = self.results_dir / f"expenditure_decomposition_{timestamp}.csv"
        df.write_csv(str(out_path))
        logger.info(f"✓ Decomposition saved: {out_path}")
        print(f"✓ Results saved: {out_path}")

    def run(self) -> pl.DataFrame:
        """Full pipeline: load → Laspeyres → LMDI → validate → cumulative → save."""
        df = self._load_analysis_window()

        # Attach util_rate if possible
        if "total_admissions" in df.columns and "population" in df.columns:
            df = df.with_columns(
                (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
            )

        df_decomp = self.laspeyres_decomposition(df)
        df_lmdi = self.lmdi_decomposition(df)

        # Merge LMDI columns for comparison
        df_full = df_decomp.join(df_lmdi, on="year", how="left")
        self.validate_additive_property(df_decomp)
        cumulative = self.cumulative_contributions(df_decomp)
        logger.info("Cumulative summary: %s", cumulative)
        self.save_results(df_full)
        return df_full
```

#### 6.1b Visualization — `decomposition_plots.py`

```python
"""Waterfall and stacked-area charts for expenditure decomposition."""

import logging
from pathlib import Path

import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import polars as pl

logger = logging.getLogger(__name__)

COLORS = {
    "demographic": "#3498db",
    "volume": "#e67e22",
    "price": "#e74c3c",
    "total": "#2c3e50",
}


def plot_waterfall(
    cumulative: dict,
    output_path: Path,
    title: str = "Expenditure Growth Decomposition 2006-2018 (Cumulative)",
) -> None:
    """
    Waterfall chart showing cumulative contributions of each component.

    Args:
        cumulative: Dict from ExpenditureDecomposer.cumulative_contributions().
        output_path: Path to save PNG figure.
        title: Chart title.
    """
    labels = ["Demographic", "Volume", "Price/Intensity", "Total Change"]
    values = [
        cumulative.get("demographic_contribution_pct") or 0,
        cumulative.get("volume_contribution_pct") or 0,
        cumulative.get("price_contribution_pct") or 0,
        100.0,
    ]
    colors_list = [COLORS["demographic"], COLORS["volume"], COLORS["price"], COLORS["total"]]

    fig, ax = plt.subplots(figsize=(9, 5))
    bars = ax.bar(labels, values, color=colors_list, edgecolor="white", linewidth=0.5)
    ax.axhline(0, color="black", linewidth=0.7)

    for bar, val in zip(bars, values):
        if val is not None:
            ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.5,
                    f"{val:.1f}%", ha="center", va="bottom", fontsize=10, fontweight="bold")

    ax.set_title(title, fontsize=13, fontweight="bold", pad=12)
    ax.set_ylabel("% of Total Growth")
    ax.grid(axis="y", alpha=0.3)
    plt.tight_layout()
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=150, bbox_inches="tight")
    plt.close(fig)
    logger.info(f"✓ Waterfall saved: {output_path}")
    print(f"✓ Figure saved: {output_path}")


def plot_stacked_area(
    df: pl.DataFrame,
    output_path: Path,
    title: str = "Annual Decomposition of Expenditure Growth",
) -> None:
    """
    Stacked area chart of annual demographic, volume, price effects.

    Args:
        df: Decomposition DataFrame with year, demographic_effect,
            volume_effect, price_effect columns.
        output_path: Path to save PNG figure.
        title: Chart title.
    """
    df_valid = df.filter(pl.col("expenditure_change").is_not_null()).sort("year")
    years = df_valid["year"].to_list()
    dem = [v or 0 for v in df_valid["demographic_effect"].to_list()]
    vol = [v or 0 for v in df_valid["volume_effect"].to_list()]
    pri = [v or 0 for v in df_valid["price_effect"].to_list()]

    fig, ax = plt.subplots(figsize=(11, 5))
    ax.stackplot(years, dem, vol, pri,
                 labels=["Demographic", "Volume", "Price/Intensity"],
                 colors=[COLORS["demographic"], COLORS["volume"], COLORS["price"]],
                 alpha=0.85)
    ax.set_title(title, fontsize=13, fontweight="bold", pad=12)
    ax.set_xlabel("Year")
    ax.set_ylabel("Contribution (SGD Millions)")
    ax.legend(loc="upper left", fontsize=9)
    ax.set_xticks(years)
    ax.tick_params(axis="x", rotation=45)
    ax.grid(axis="y", alpha=0.3)
    plt.tight_layout()
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=150, bbox_inches="tight")
    plt.close(fig)
    logger.info(f"✓ Stacked area saved: {output_path}")
    print(f"✓ Figure saved: {output_path}")
```

#### 6.3 Validation Rules

```python
DECOMPOSITION_VALIDATION_RULES = {
    "additive_tolerance_sgd_m": 1e-4,
    "required_columns": [
        "year", "total_expenditure", "expenditure_change",
        "demographic_effect", "volume_effect", "price_effect"
    ],
    "year_range": (ANALYSIS_START, ANALYSIS_END),
    "first_row_effects_null": True,
}
```

---

### 10. Testing Strategy

```python
# ps-004/tests/unit/test_expenditure_decomposition.py

import math
import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.expenditure_decomposition import (
    ExpenditureDecomposer,
    TOLERANCE,
)


@pytest.fixture
def sample_integrated(tmp_path: Path) -> pl.DataFrame:
    """Minimal integrated dataset with known values for exact checks."""
    df = pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 250 for i in range(13)],
        "population": [4_500_000.0 + i * 50_000 for i in range(13)],
        "total_admissions": [200_000.0 + i * 3_000 for i in range(13)],
    })
    return df


@pytest.fixture
def decomposer(tmp_path: Path, sample_integrated: pl.DataFrame) -> ExpenditureDecomposer:
    p = tmp_path / "integrated.parquet"
    sample_integrated.write_parquet(str(p))
    return ExpenditureDecomposer(
        data_path=str(p),
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
    )


def test_laspeyres_first_row_null(
    decomposer: ExpenditureDecomposer, sample_integrated: pl.DataFrame
) -> None:
    df = sample_integrated.with_columns(
        (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
    )
    result = decomposer.laspeyres_decomposition(df)
    assert result["expenditure_change"][0] is None
    assert result["demographic_effect"][0] is None


def test_additive_property_holds(
    decomposer: ExpenditureDecomposer, sample_integrated: pl.DataFrame
) -> None:
    df = sample_integrated.with_columns(
        (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
    )
    result = decomposer.laspeyres_decomposition(df)
    assert decomposer.validate_additive_property(result) is True


def test_cumulative_contributions_sums_to_100(
    decomposer: ExpenditureDecomposer, sample_integrated: pl.DataFrame
) -> None:
    df = sample_integrated.with_columns(
        (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
    )
    result = decomposer.laspeyres_decomposition(df)
    cumul = decomposer.cumulative_contributions(result)
    total = (
        (cumul["demographic_contribution_pct"] or 0)
        + (cumul["volume_contribution_pct"] or 0)
        + (cumul["price_contribution_pct"] or 0)
    )
    assert abs(total - 100.0) < 0.5


def test_save_creates_file(
    decomposer: ExpenditureDecomposer, sample_integrated: pl.DataFrame
) -> None:
    df = sample_integrated.with_columns(
        (pl.col("total_admissions") / pl.col("population")).alias("util_rate")
    )
    result = decomposer.laspeyres_decomposition(df)
    decomposer.save_results(result)
    files = list(Path(decomposer.results_dir).glob("expenditure_decomposition_*.csv"))
    assert len(files) == 1


def test_lmdi_log_mean_symmetric() -> None:
    assert abs(ExpenditureDecomposer._log_mean(4, 2) - ExpenditureDecomposer._log_mean(2, 4)) < 1e-10


def test_run_returns_dataframe(decomposer: ExpenditureDecomposer) -> None:
    df = decomposer.run()
    assert isinstance(df, pl.DataFrame)
    assert "price_effect" in df.columns
    assert len(df) == 13
```

---

### 11. Implementation Steps

#### Phase 1: Setup
- [ ] Confirm `shared/data/3_interim/expenditure_drivers_integrated.parquet` exists with `population` and `total_admissions` columns
- [ ] If columns absent, revisit US-02 integration to add population from Kaggle

#### Phase 2: Decomposition
- [ ] Implement `ExpenditureDecomposer.laspeyres_decomposition()`
- [ ] Run and inspect annual demographic/volume/price effects
- [ ] Run `validate_additive_property()` — all years must pass
- [ ] Compute `cumulative_contributions()` and log dominant driver

#### Phase 3: LMDI Cross-validation
- [ ] Implement `lmdi_decomposition()`
- [ ] Compare LMDI vs Laspeyres totals — flag discrepancies > 5%

#### Phase 4: Sensitivity Analysis
- [ ] Re-run with base year = 2010 and compare cumulative shares
- [ ] Document if dominant driver changes under alternative base year

#### Phase 5: Visualisations & Output
- [ ] Generate waterfall chart (`expenditure_decomposition_waterfall_*.png`)
- [ ] Generate stacked area chart (`expenditure_decomposition_stacked_area_*.png`)
- [ ] Save `results/tables/problem-statement-004/expenditure_decomposition_*.csv`
- [ ] Write unit tests and confirm ≥80% coverage

---

### 12. Adaptive Implementation Strategy

- **If population column null/absent** → volume effect = 0, price effect absorbs all residual; document limitation
- **If LMDI diverges > 10% from Laspeyres** → investigate data quality; check for zero-value population years
- **If price/intensity dominates (>60%)** → flag for US-07 benchmarking to check if Singapore's intensity cost is above OECD peers
- **If additive tolerance violated** → inspect floating-point precision; add rounding step before assertion

---

### 21. Version Control

```
Branch: feature/ps-004-us-04-decompose-expenditure
Commits:
  feat(ps-004): add ExpenditureDecomposer with Laspeyres and LMDI methods
  feat(ps-004): add decomposition waterfall and stacked-area plots
  test(ps-004): unit tests for additive property and cumulative contributions
```
