# Analyze 15-Year Expenditure Growth Patterns (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-004-US-03  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Analyst assessing cost trends**,  
I want **to analyze 15-year government health expenditure growth patterns (2006-2018) including absolute growth, real vs nominal growth, and acceleration periods**,  
So that **I can understand historical cost dynamics and identify periods of exceptional expenditure growth or restraint**.

---

## 🎯 Acceptance Criteria

1. **Growth rates calculated**
   - Year-over-year growth rates (nominal and real terms)
   - Compound annual growth rate (CAGR) 2006-2018
   - Per capita expenditure growth trends
   - Inflation-adjusted real expenditure growth (if CPI data available)

2. **Trend analysis completed**
   - Identification of high-growth vs low-growth periods
   - Expenditure acceleration/deceleration detection
   - Comparison with GDP growth (if data available)
   - Expenditure as % of GDP trends (if GDP data available)

3. **Category-level analysis** (if expenditure categories available)
   - Growth rates by expenditure category
   - Category share changes over time
   - Fastest-growing expenditure categories identified

4. **Deliverables**
   - Output: `results/tables/expenditure_growth_analysis_2006_2018.csv`
   - Figures: Expenditure trends, growth rate charts
   - Summary report: Key expenditure evolution insights

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#healthcare-expenditure-analysis) - Expenditure trend methodologies
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-1) - Expenditure growth objectives

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-02 (Integrated expenditure data - BLOCKING)
- **Data Sources**: `shared/data/3_interim/expenditure_drivers_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Growth Calculations
- [ ] Calculate YoY expenditure growth: `(exp_t - exp_t-1) / exp_t-1 * 100`
- [ ] Calculate CAGR: `(end_exp / start_exp)^(1/years) - 1`
- [ ] Calculate per capita expenditure growth
- [ ] Adjust for inflation if CPI data available (real growth)

### Trend Analysis
- [ ] Identify high-growth periods (growth >5%)
- [ ] Identify low-growth or decline periods
- [ ] Detect acceleration/deceleration using second derivative
- [ ] Compare with economic indicators (if available)

### Category Analysis (if data supports)
- [ ] Calculate growth rates by category
- [ ] Analyze category share changes
- [ ] Identify fastest-growing categories

### Visualization
- [ ] Line chart: total expenditure over time
- [ ] Bar chart: YoY growth rates
- [ ] Area chart: expenditure by category (stacked)
- [ ] Save figures: `reports/figures/expenditure_trends_*.png/pdf`

### Testing & Documentation
- [ ] Unit tests for growth calculations
- [ ] Validate CAGR formula
- [ ] Docstrings
- [ ] Analysis summary report

---

## 📌 Notes

**Polars Growth Calculation**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/expenditure_drivers_integrated.parquet")

df_growth = df.sort('year').with_columns([
    ((pl.col('total_expenditure') - pl.col('total_expenditure').shift(1)) / 
     pl.col('total_expenditure').shift(1) * 100).alias('yoy_growth_pct')
])

# CAGR
start_exp = df.filter(pl.col('year') == 2006)['total_expenditure'][0]
end_exp = df.filter(pl.col('year') == 2018)['total_expenditure'][0]
years = 2018 - 2006
cagr = ((end_exp / start_exp) ** (1 / years) - 1) * 100
```

**Expected Insights**:
- Steady growth likely (healthcare costs typically rise with GDP)
- Possible acceleration post-2010 (aging population impact)
- Per capita growth may outpace total growth if population growth slow

---

## Implementation Plan

### 1. Feature Overview

Compute 15-year expenditure growth metrics (YoY, CAGR, per capita, acceleration), identify high/low-growth periods, and produce publication-quality visualisations for Singapore government health expenditure 2006-2018. Primary user role: **Healthcare Financial Analyst**. Blocking upstream: PS-004-US-02.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | Created by US-02 | **Reuse** |
| `shared/src/analysis/trend_analysis.py` | Exists (workforce) | **Reference pattern** |
| `shared/src/analysis/feature_engineering.py` | Exists | **Reference for growth calc patterns** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/expenditure_trend_analyzer.py` | Missing | **Create** |
| `results/tables/problem-statement-004/` | Missing | **Create** |
| `reports/figures/problem-statement-004/` | Missing | **Create** |

No ML model required (descriptive statistical analysis).

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/expenditure_trend_analyzer.py`**
  - `ExpenditureTrendAnalyzer` class
  - `calculate_yoy_growth()` → `pl.DataFrame`
  - `calculate_cagr()` → `float`
  - `calculate_acceleration()` → `pl.DataFrame`
  - `identify_growth_periods()` → `pl.DataFrame`
  - `generate_summary_report()` → `dict`
  - `save_results()` → `None`

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/visualization/expenditure_plots.py`**
  - `plot_expenditure_over_time()` → `None`
  - `plot_yoy_growth_rates()` → `None`
  - `plot_expenditure_by_category()` → `None` (if categories available)
  - `plot_per_capita_trend()` → `None`

- **[CREATE] `results/tables/problem-statement-004/expenditure_growth_analysis_2006_2018.csv`**
  - Output: full growth analysis table

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_expenditure_trend_analyzer.py`**

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/03-analyze-expenditure-trends.ipynb`**

---

### 5. Data Pipeline

```
shared/data/3_interim/expenditure_drivers_integrated.parquet  (US-02 output)
  → ExpenditureTrendAnalyzer.load()

  → calculate_yoy_growth()
      • sort by year
      • (exp_t - exp_t-1) / exp_t-1 * 100  [already in integrated, recalculate here for verification]
      • label periods: high_growth (>5%), moderate (2-5%), low (<2%), decline (<0%)

  → calculate_cagr()
      • CAGR 2006-2018 = (exp_2018 / exp_2006)^(1/12) - 1

  → calculate_acceleration()
      • second derivative of YoY growth: Δgrowth = growth_t - growth_t-1
      • acceleration > 0: expenditure speeding up

  → calculate_per_capita_trends()  [if population available]
      • per_capita column already in integrated dataset
      • compute CAGR per capita

  → generate_summary_report()
      • Dict: CAGR, peak growth year, trough year, avg growth rate

  → save_results()
      • results/tables/problem-statement-004/expenditure_growth_analysis_2006_2018.csv
      • reports/figures/problem-statement-004/expenditure_trend_*.png / .pdf
      • logs/analysis/expenditure_trend_YYYYMMDD.log
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `expenditure_trend_analyzer.py`

```python
"""
Expenditure Trend Analyzer
==========================

Calculate 15-year growth metrics (YoY, CAGR, acceleration) and identify
high/low-growth periods in Singapore government health expenditure 2006-2018.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from datetime import datetime
from pathlib import Path
from typing import Optional

import polars as pl

logger = logging.getLogger(__name__)

HIGH_GROWTH_THRESHOLD = 5.0    # % — classify as high-growth if YoY > 5%
LOW_GROWTH_THRESHOLD = 2.0     # % — moderate below 5%, low below 2%
ANALYSIS_START = 2006
ANALYSIS_END = 2018


def setup_logger(log_dir: str = "logs/analysis") -> logging.Logger:
    """
    Configure file and console logging for trend analysis.

    Args:
        log_dir: Directory to store analysis logs.

    Returns:
        Configured root logger.
    """
    Path(log_dir).mkdir(parents=True, exist_ok=True)
    log_file = (
        Path(log_dir)
        / f"expenditure_trend_{datetime.now().strftime('%Y%m%d_%H%M%S')}.log"
    )
    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s | %(levelname)-8s | %(name)s - %(message)s",
        handlers=[
            logging.FileHandler(str(log_file)),
            logging.StreamHandler(),
        ],
    )
    return logging.getLogger(__name__)


class ExpenditureTrendAnalyzer:
    """
    Analyse 15-year expenditure growth patterns for Singapore healthcare.

    Example:
        >>> analyzer = ExpenditureTrendAnalyzer()
        >>> df = analyzer.run()
    """

    def __init__(
        self,
        data_path: str = "shared/data/3_interim/expenditure_drivers_integrated.parquet",
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
        log_dir: str = "logs/analysis",
    ) -> None:
        """
        Initialise analyzer with configurable paths.

        Args:
            data_path: Path to integrated Parquet from US-02.
            results_dir: Directory for CSV output tables.
            figures_dir: Directory for PNG/PDF figures.
            log_dir: Directory for analysis log files.
        """
        self.data_path = Path(data_path)
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.log_dir = Path(log_dir)

        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)
        self.log_dir.mkdir(parents=True, exist_ok=True)

        setup_logger(str(self.log_dir))

        if not self.data_path.exists():
            raise FileNotFoundError(
                f"Integrated dataset not found: {self.data_path}\n"
                "Please run PS-004-US-02 integration first."
            )

        self.df_raw: pl.DataFrame = pl.read_parquet(str(self.data_path))
        logger.info(
            f"Loaded integrated dataset: {len(self.df_raw)} rows from {self.data_path}"
        )

    def load(self) -> pl.DataFrame:
        """
        Return filtered and sorted expenditure dataset within analysis period.

        Returns:
            DataFrame sorted by year, filtered 2006-2018.
        """
        df = (
            self.df_raw
            .filter(
                (pl.col("year") >= ANALYSIS_START)
                & (pl.col("year") <= ANALYSIS_END)
            )
            .sort("year")
        )
        logger.info(f"Analysis window: {df['year'].min()} - {df['year'].max()} ({len(df)} rows)")
        return df

    def calculate_yoy_growth(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Calculate year-over-year growth rates and classify growth periods.

        Args:
            df: Expenditure DataFrame sorted by year.

        Returns:
            DataFrame with additional columns:
            - yoy_growth_pct: YoY % change in total_expenditure
            - growth_period: 'high' / 'moderate' / 'low' / 'decline'
        """
        df = df.with_columns(
            (
                (pl.col("total_expenditure") - pl.col("total_expenditure").shift(1))
                / pl.col("total_expenditure").shift(1)
                * 100
            ).alias("yoy_growth_pct")
        )

        df = df.with_columns(
            pl.when(pl.col("yoy_growth_pct").is_null())
            .then(pl.lit("baseline"))
            .when(pl.col("yoy_growth_pct") < 0)
            .then(pl.lit("decline"))
            .when(pl.col("yoy_growth_pct") < LOW_GROWTH_THRESHOLD)
            .then(pl.lit("low"))
            .when(pl.col("yoy_growth_pct") < HIGH_GROWTH_THRESHOLD)
            .then(pl.lit("moderate"))
            .otherwise(pl.lit("high"))
            .alias("growth_period")
        )

        logger.info(
            "YoY growth calculated. Distribution:\n%s",
            df.group_by("growth_period").agg(pl.len().alias("count")).to_pandas().to_string()
        )
        return df

    def calculate_cagr(self, df: pl.DataFrame) -> float:
        """
        Compute Compound Annual Growth Rate over the full analysis period.

        Args:
            df: Expenditure DataFrame with total_expenditure and year columns.

        Returns:
            CAGR as a percentage (e.g., 5.3 for 5.3%).
        """
        start_row = df.filter(pl.col("year") == ANALYSIS_START)
        end_row = df.filter(pl.col("year") == ANALYSIS_END)

        if len(start_row) == 0 or len(end_row) == 0:
            raise ValueError(
                f"CAGR requires data for {ANALYSIS_START} and {ANALYSIS_END}"
            )

        start_exp = start_row["total_expenditure"][0]
        end_exp = end_row["total_expenditure"][0]
        years = ANALYSIS_END - ANALYSIS_START
        cagr = ((end_exp / start_exp) ** (1 / years) - 1) * 100

        logger.info(
            f"CAGR {ANALYSIS_START}-{ANALYSIS_END}: {cagr:.2f}% "
            f"(from {start_exp:.1f} to {end_exp:.1f} SGD millions)"
        )
        return cagr

    def calculate_acceleration(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Detect expenditure acceleration/deceleration via second derivative.

        Args:
            df: DataFrame containing yoy_growth_pct column.

        Returns:
            DataFrame with additional columns:
            - growth_acceleration: Δ(yoy_growth_pct) year-on-year
            - trend_signal: 'accelerating' / 'decelerating' / 'stable'
        """
        if "yoy_growth_pct" not in df.columns:
            df = self.calculate_yoy_growth(df)

        df = df.with_columns(
            (pl.col("yoy_growth_pct") - pl.col("yoy_growth_pct").shift(1))
            .alias("growth_acceleration")
        )

        df = df.with_columns(
            pl.when(pl.col("growth_acceleration").is_null())
            .then(pl.lit("stable"))
            .when(pl.col("growth_acceleration") > 0.5)
            .then(pl.lit("accelerating"))
            .when(pl.col("growth_acceleration") < -0.5)
            .then(pl.lit("decelerating"))
            .otherwise(pl.lit("stable"))
            .alias("trend_signal")
        )

        logger.info("Acceleration analysis complete")
        return df

    def calculate_per_capita_cagr(self, df: pl.DataFrame) -> Optional[float]:
        """
        Compute CAGR for per-capita expenditure if population data available.

        Args:
            df: Integrated DataFrame potentially containing per_capita_expenditure.

        Returns:
            Per-capita CAGR percentage, or None if column absent.
        """
        if "per_capita_expenditure" not in df.columns:
            logger.warning("per_capita_expenditure column not found — skipping per capita CAGR")
            return None

        start_pc = df.filter(pl.col("year") == ANALYSIS_START)["per_capita_expenditure"][0]
        end_pc = df.filter(pl.col("year") == ANALYSIS_END)["per_capita_expenditure"][0]

        if start_pc is None or end_pc is None:
            logger.warning("Null per_capita_expenditure at boundary years — skipping")
            return None

        years = ANALYSIS_END - ANALYSIS_START
        cagr_pc = ((end_pc / start_pc) ** (1 / years) - 1) * 100
        logger.info(f"Per-capita CAGR {ANALYSIS_START}-{ANALYSIS_END}: {cagr_pc:.2f}%")
        return cagr_pc

    def generate_summary_report(
        self, df: pl.DataFrame, cagr: float, cagr_per_capita: Optional[float]
    ) -> dict:
        """
        Compile key growth insights into a summary dictionary.

        Args:
            df: Enriched DataFrame with yoy_growth_pct and growth_period.
            cagr: Overall CAGR percentage.
            cagr_per_capita: Per-capita CAGR or None.

        Returns:
            Summary dict with CAGR, peak/trough years, average growth, period distribution.
        """
        df_growth = df.filter(pl.col("yoy_growth_pct").is_not_null())
        peak_year = int(df_growth.sort("yoy_growth_pct", descending=True)["year"][0])
        trough_year = int(df_growth.sort("yoy_growth_pct")["year"][0])
        avg_growth = float(df_growth["yoy_growth_pct"].mean())
        period_dist = (
            df.group_by("growth_period")
            .agg(pl.len().alias("count"))
            .sort("count", descending=True)
            .to_dicts()
        )

        report = {
            "analysis_period": f"{ANALYSIS_START}-{ANALYSIS_END}",
            "cagr_pct": round(cagr, 2),
            "cagr_per_capita_pct": round(cagr_per_capita, 2) if cagr_per_capita else None,
            "average_yoy_growth_pct": round(avg_growth, 2),
            "peak_growth_year": peak_year,
            "trough_growth_year": trough_year,
            "growth_period_distribution": period_dist,
            "generated_at": datetime.now().isoformat(),
        }
        logger.info("Summary report generated: %s", report)
        return report

    def save_results(self, df: pl.DataFrame) -> None:
        """
        Persist growth analysis CSV to results/tables/.

        Args:
            df: Enriched analysis DataFrame.
        """
        out_path = self.results_dir / "expenditure_growth_analysis_2006_2018.csv"
        df.write_csv(str(out_path))
        logger.info(f"✓ Growth analysis saved: {out_path}")
        print(f"✓ Results saved to: {out_path}")

    def run(self) -> pl.DataFrame:
        """
        Full analysis pipeline: load → YoY → CAGR → acceleration → save.

        Returns:
            Enriched analysis DataFrame.
        """
        df = self.load()
        df = self.calculate_yoy_growth(df)
        cagr = self.calculate_cagr(df)
        df = self.calculate_acceleration(df)
        cagr_pc = self.calculate_per_capita_cagr(df)
        report = self.generate_summary_report(df, cagr, cagr_pc)
        self.save_results(df)
        logger.info("Expenditure trend analysis complete. Report: %s", report)
        return df
```

#### 6.1b Complete Implementation — `expenditure_plots.py`

```python
"""
Expenditure Visualisation Utilities
=====================================

Publication-quality charts for Singapore health expenditure trend analysis.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from pathlib import Path

import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import polars as pl

logger = logging.getLogger(__name__)

FIGURE_DPI = 150
FIGURE_FORMAT = "png"
COLOR_PRIMARY = "#1f77b4"
COLOR_ACCENT = "#d62728"
COLOR_HIGHLIGHT = "#ff7f0e"


def plot_expenditure_over_time(
    df: pl.DataFrame,
    output_path: Path,
    title: str = "Singapore Government Health Expenditure 2006-2018",
) -> None:
    """
    Line chart of total_expenditure over years.

    Args:
        df: DataFrame with 'year' and 'total_expenditure' columns.
        output_path: Path to save PNG figure.
        title: Plot title.
    """
    fig, ax = plt.subplots(figsize=(10, 5))
    years = df["year"].to_list()
    expenditure = df["total_expenditure"].to_list()

    ax.plot(years, expenditure, marker="o", color=COLOR_PRIMARY, linewidth=2.5,
            markersize=6, label="Total Expenditure (SGD M)")
    ax.fill_between(years, expenditure, alpha=0.15, color=COLOR_PRIMARY)

    ax.set_title(title, fontsize=14, fontweight="bold", pad=12)
    ax.set_xlabel("Year", fontsize=11)
    ax.set_ylabel("Expenditure (SGD Millions)", fontsize=11)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f"{x:,.0f}"))
    ax.set_xticks(years)
    ax.tick_params(axis="x", rotation=45)
    ax.grid(axis="y", alpha=0.3)
    ax.legend()

    plt.tight_layout()
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=FIGURE_DPI, format=FIGURE_FORMAT, bbox_inches="tight")
    plt.close(fig)
    logger.info(f"✓ Figure saved: {output_path}")
    print(f"✓ Figure saved: {output_path}")


def plot_yoy_growth_rates(
    df: pl.DataFrame,
    output_path: Path,
    title: str = "Year-on-Year Health Expenditure Growth Rate",
) -> None:
    """
    Bar chart of YoY growth rates with high/low-growth period colouring.

    Args:
        df: DataFrame with 'year', 'yoy_growth_pct', 'growth_period' columns.
        output_path: Path to save PNG figure.
        title: Plot title.
    """
    df_growth = df.filter(pl.col("yoy_growth_pct").is_not_null())
    years = df_growth["year"].to_list()
    growth = df_growth["yoy_growth_pct"].to_list()
    periods = df_growth["growth_period"].to_list()

    color_map = {
        "high": COLOR_ACCENT,
        "moderate": COLOR_PRIMARY,
        "low": COLOR_HIGHLIGHT,
        "decline": "#9467bd",
        "baseline": "#7f7f7f",
    }
    colors = [color_map.get(p, COLOR_PRIMARY) for p in periods]

    fig, ax = plt.subplots(figsize=(10, 5))
    ax.bar(years, growth, color=colors, edgecolor="white", linewidth=0.5)
    ax.axhline(0, color="black", linewidth=0.8, linestyle="--")
    ax.axhline(5.0, color=COLOR_ACCENT, linewidth=1, linestyle=":", alpha=0.6,
               label="High-growth threshold (5%)")

    for year, val in zip(years, growth):
        ax.text(year, val + 0.1, f"{val:.1f}%", ha="center", va="bottom", fontsize=8)

    ax.set_title(title, fontsize=14, fontweight="bold", pad=12)
    ax.set_xlabel("Year", fontsize=11)
    ax.set_ylabel("YoY Growth (%)", fontsize=11)
    ax.set_xticks(years)
    ax.tick_params(axis="x", rotation=45)
    ax.grid(axis="y", alpha=0.3)
    ax.legend(fontsize=9)

    plt.tight_layout()
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=FIGURE_DPI, format=FIGURE_FORMAT, bbox_inches="tight")
    plt.close(fig)
    logger.info(f"✓ Figure saved: {output_path}")
    print(f"✓ Figure saved: {output_path}")


def plot_per_capita_trend(
    df: pl.DataFrame,
    output_path: Path,
    title: str = "Per-Capita Health Expenditure Trend 2006-2018",
) -> None:
    """
    Line chart of per_capita_expenditure if available.

    Args:
        df: DataFrame with 'year' and 'per_capita_expenditure' columns.
        output_path: Path to save PNG figure.
        title: Plot title.
    """
    if "per_capita_expenditure" not in df.columns:
        logger.warning("per_capita_expenditure column absent — skipping per-capita plot")
        return

    df_pc = df.filter(pl.col("per_capita_expenditure").is_not_null())
    if len(df_pc) == 0:
        logger.warning("All per_capita_expenditure values are null — skipping plot")
        return

    years = df_pc["year"].to_list()
    per_capita = df_pc["per_capita_expenditure"].to_list()

    fig, ax = plt.subplots(figsize=(10, 5))
    ax.plot(years, per_capita, marker="s", color=COLOR_HIGHLIGHT, linewidth=2.5,
            markersize=6, label="Per-Capita Expenditure (SGD M)")
    ax.fill_between(years, per_capita, alpha=0.15, color=COLOR_HIGHLIGHT)

    ax.set_title(title, fontsize=14, fontweight="bold", pad=12)
    ax.set_xlabel("Year", fontsize=11)
    ax.set_ylabel("Per-Capita Expenditure (SGD M)", fontsize=11)
    ax.set_xticks(years)
    ax.tick_params(axis="x", rotation=45)
    ax.grid(axis="y", alpha=0.3)
    ax.legend()

    plt.tight_layout()
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=FIGURE_DPI, format=FIGURE_FORMAT, bbox_inches="tight")
    plt.close(fig)
    logger.info(f"✓ Figure saved: {output_path}")
    print(f"✓ Figure saved: {output_path}")
```

#### 6.3 Validation Rules

```python
ANALYSIS_VALIDATION_RULES = {
    "required_columns": ["year", "total_expenditure"],
    "output_columns": ["yoy_growth_pct", "growth_period", "growth_acceleration", "trend_signal"],
    "year_range": (2006, 2018),
    "cagr_realistic_range_pct": (0.0, 20.0),     # Reject negative or >20% CAGR
    "high_growth_threshold_pct": 5.0,
    "low_growth_threshold_pct": 2.0,
}
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 matplotlib>=3.8.0 seaborn>=0.13.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

**Step 1 — Relevant Domain Knowledge**:
Healthcare costs tend to grow faster than general inflation due to: demographic aging, technology adoption, and coverage expansion. CAGR enables consistent multi-period comparison.

**Step 2 — Data Availability**:

| Feature | Formula | Required Fields | Available |
|---------|---------|----------------|-----------|
| `yoy_growth_pct` | `(exp_t - exp_t-1)/exp_t-1 * 100` | `total_expenditure` | ✅ |
| `cagr` | `(end/start)^(1/n) - 1` | `total_expenditure`, boundary years | ✅ |
| `growth_acceleration` | `Δ(yoy_growth_pct)` | `yoy_growth_pct` | ✅ (derived) |
| `per_capita_cagr` | CAGR of `per_capita_expenditure` | `per_capita_expenditure` | Conditional |

**Step 3 — Growth Rate Calculation**:

```python
import polars as pl

def calculate_growth_rates(
    df: pl.DataFrame,
    value_col: str = "total_expenditure",
    time_col: str = "year",
) -> pl.DataFrame:
    """
    Calculate YoY growth rates and acceleration for any value column.

    Args:
        df: Time-series DataFrame sorted by time_col.
        value_col: Column to compute growth for.
        time_col: Time column (year).

    Returns:
        DataFrame with yoy_growth_pct and growth_acceleration columns added.
    """
    df = df.sort(time_col)
    df = df.with_columns(
        (
            (pl.col(value_col) - pl.col(value_col).shift(1))
            / pl.col(value_col).shift(1)
            * 100
        ).alias("yoy_growth_pct")
    )
    df = df.with_columns(
        (pl.col("yoy_growth_pct") - pl.col("yoy_growth_pct").shift(1))
        .alias("growth_acceleration")
    )
    return df
```

---

### 10. Testing Strategy

```python
# problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_expenditure_trend_analyzer.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.expenditure_trend_analyzer import (
    ExpenditureTrendAnalyzer,
    calculate_growth_rates,
    HIGH_GROWTH_THRESHOLD,
)


@pytest.fixture
def sample_df() -> pl.DataFrame:
    """13-row sample dataset matching integrated schema."""
    years = list(range(2006, 2019))
    expenditures = [3500.0 + i * 250 for i in range(13)]  # Steady 7% growth
    return pl.DataFrame({"year": years, "total_expenditure": expenditures})


@pytest.fixture
def analyzer(tmp_path: Path, sample_df: pl.DataFrame) -> ExpenditureTrendAnalyzer:
    parquet_path = tmp_path / "integrated.parquet"
    sample_df.write_parquet(str(parquet_path))
    return ExpenditureTrendAnalyzer(
        data_path=str(parquet_path),
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
        log_dir=str(tmp_path / "logs"),
    )


def test_calculate_yoy_growth_first_row_null(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    df = analyzer.calculate_yoy_growth(sample_df)
    assert df["yoy_growth_pct"][0] is None


def test_calculate_yoy_growth_values(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    df = analyzer.calculate_yoy_growth(sample_df)
    # (3750 - 3500) / 3500 * 100 = 7.142...
    assert abs(df["yoy_growth_pct"][1] - 7.142) < 0.01


def test_growth_period_high(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    df = analyzer.calculate_yoy_growth(sample_df)
    high_rows = df.filter(pl.col("growth_period") == "high")
    for val in high_rows["yoy_growth_pct"].to_list():
        assert val >= HIGH_GROWTH_THRESHOLD


def test_calculate_cagr_known_value(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    cagr = analyzer.calculate_cagr(sample_df)
    # Expected: (6500/3500)^(1/12) -1 ≈ 5.08%
    expected = ((6500 / 3500) ** (1 / 12) - 1) * 100
    assert abs(cagr - expected) < 0.01


def test_cagr_raises_on_missing_boundary_year(
    analyzer: ExpenditureTrendAnalyzer,
) -> None:
    df_truncated = pl.DataFrame({
        "year": list(range(2007, 2019)),
        "total_expenditure": [3750.0 + i * 250 for i in range(12)],
    })
    with pytest.raises(ValueError, match="CAGR requires"):
        analyzer.calculate_cagr(df_truncated)


def test_calculate_acceleration_produces_column(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    df = analyzer.calculate_yoy_growth(sample_df)
    df = analyzer.calculate_acceleration(df)
    assert "growth_acceleration" in df.columns
    assert "trend_signal" in df.columns


def test_save_results_creates_csv(
    analyzer: ExpenditureTrendAnalyzer, sample_df: pl.DataFrame
) -> None:
    df = analyzer.calculate_yoy_growth(sample_df)
    analyzer.save_results(df)
    out = Path(analyzer.results_dir) / "expenditure_growth_analysis_2006_2018.csv"
    assert out.exists()
    df_check = pl.read_csv(str(out))
    assert len(df_check) == 13


def test_run_returns_dataframe(analyzer: ExpenditureTrendAnalyzer) -> None:
    df = analyzer.run()
    assert isinstance(df, pl.DataFrame)
    assert "yoy_growth_pct" in df.columns
    assert "growth_period" in df.columns
```

---

### 11. Implementation Steps

#### Phase 1: Verify Upstream Dependency
- [ ] Confirm `shared/data/3_interim/expenditure_drivers_integrated.parquet` exists (US-02 output)
- [ ] Load and inspect column list and dtypes
- [ ] Verify year range is 2006-2018 and `total_expenditure` is non-null

#### Phase 2: Growth Calculations
- [ ] Implement `ExpenditureTrendAnalyzer.calculate_yoy_growth()`
- [ ] Run calculation and review YoY series for reasonableness
- [ ] Implement `calculate_cagr()` and log result
- [ ] Implement `calculate_acceleration()` and identify inflection years

#### Phase 3: Per-Capita Analysis
- [ ] Check if `per_capita_expenditure` column is populated
- [ ] If available, implement `calculate_per_capita_cagr()` and compare with total CAGR
- [ ] Log gap if population data absent

#### Phase 4: Visualisations
- [ ] Implement `expenditure_plots.py` with all three chart functions
- [ ] Generate `expenditure_trends_total.png`
- [ ] Generate `expenditure_trends_yoy_growth.png`
- [ ] Generate `expenditure_trends_per_capita.png` (if data available)
- [ ] Save PDF copies alongside PNGs

#### Phase 5: Results & Testing
- [ ] Run `save_results()` → confirm `results/tables/problem-statement-004/expenditure_growth_analysis_2006_2018.csv` exists
- [ ] Write unit tests in `tests/unit/test_expenditure_trend_analyzer.py`
- [ ] Run `pytest` and confirm ≥80% coverage
- [ ] Create `notebooks/03-analyze-expenditure-trends.ipynb`

---

### 12. Adaptive Implementation Strategy

- **If YoY growth is uniformly stable** → extend analysis with 3-year rolling average to reveal underlying trend
- **If CAGR > 15%** → flag as anomaly; verify expenditure units (ensure not switching between millions/billions)
- **If per_capita_expenditure all null** → add commentary in notebook that population data was unavailable; recommend sourcing from WHO or Singstat
- **If fewer than 13 rows in integrated dataset** → re-run US-02 integration; log which years are absent
- **If high acceleration detected post-2010** → add annotation to YoY chart referencing potential aging population drivers

---

### 13. Code Generation Order

1. `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/expenditure_trend_analyzer.py`
2. `problem-statements/ps-004-healthcare-expenditure-drivers/src/visualization/expenditure_plots.py`
3. `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_expenditure_trend_analyzer.py`
4. `problem-statements/ps-004-healthcare-expenditure-drivers/src/scripts/03_analyze_expenditure_trends.py` (CLI)
5. `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/03-analyze-expenditure-trends.ipynb`

---

### 19. References

- [data-sources.md](../../../project_context/data-sources.md) — Expenditure table spec
- [02-integrate-expenditure-drivers.md](02-integrate-expenditure-drivers.md) — Upstream integration spec
- [trend_analysis.py](../../../../shared/src/analysis/trend_analysis.py) — Workforce YoY/CAGR reference implementation

---

### 20. Security & Privacy

- Analysis output is aggregate public expenditure data — no PII
- Results and figures saved locally under `results/` and `reports/` (not committed to version control by default)
- Log files contain only aggregate statistics — safe for audit trail

---

### 21. Version Control

```
Branch: feature/ps-004-us-03-analyze-expenditure-trends
Commits:
  feat(ps-004): add ExpenditureTrendAnalyzer with CAGR and acceleration
  feat(ps-004): add expenditure visualisation utilities
  test(ps-004): unit tests for trend analysis and CAGR calculations
  docs(ps-004): analysis summary for expenditure growth 2006-2018
```
