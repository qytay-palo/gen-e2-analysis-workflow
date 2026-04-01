# Analyze 11-Year Capacity Evolution by Facility Type (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-003-US-03  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Service Planner assessing infrastructure needs**,  
I want **to analyze how facility capacity evolved from 2009-2020 across acute care, community hospitals, long-term care, and primary care sectors**,  
So that **I can identify which facility types expanded rapidly, stagnated, or declined and understand capacity distribution between public and private sectors**.

---

## 🎯 Acceptance Criteria

1. **Capacity growth trends calculated**
   - Year-over-year capacity growth rates for each facility type
   - Compound annual growth rates (CAGR) 2009-2020
   - Absolute capacity changes: 2020 capacity - 2009 baseline
   - Growth volatility: standard deviation of annual growth rates

2. **Sector comparisons completed**
   - Public vs private capacity growth analyzed
   - Sector share trends: % of total beds in public vs private
   - Divergence analysis: facility types where public/private growth differs

3. **Capacity distribution analysis**
   - Capacity per capita: beds per 10,000 population (if population data available)
   - Facility type proportions: % of total capacity in acute vs community vs LTC
   - Identification of capacity concentration or diversification trends

4. **Deliverables produced**
   - Output: `results/tables/capacity_growth_analysis_2009_2020.csv`
   - Figures: Capacity trends by facility type, sector comparisons
   - Summary report: Key capacity evolution insights

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#healthcare-capacity-metrics) - Capacity benchmarks
- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objective-1) - Capacity quantification objectives

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-003-US-02 (Clean capacity data - BLOCKING)
- **Data Sources**: `shared/data/3_interim/capacity_utilization_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Growth Rate Calculations
- [ ] Calculate YoY growth: `(capacity_t - capacity_t-1) / capacity_t-1 * 100`
- [ ] Calculate CAGR: `(end_capacity / start_capacity)^(1/years) - 1`
- [ ] Compute growth statistics per facility type
- [ ] Identify years with exceptional growth or contraction

### Sector Analysis
- [ ] Compare public vs private growth rates
- [ ] Calculate sector share over time
- [ ] Statistical tests for growth differences
- [ ] Identify sector-specific trends

### Capacity Distribution
- [ ] Calculate total capacity by facility type per year
- [ ] Compute proportional distribution
- [ ] Analyze shifts in capacity mix
- [ ] Assess concentration vs diversification

### Visualization
- [ ] Line charts: capacity over time by facility type
- [ ] Stacked area: proportional capacity by facility type
- [ ] Bar charts: CAGR comparisons
- [ ] Sector comparison charts
- [ ] Save figures: `reports/figures/capacity_trends_*.png/pdf`

### Testing & Documentation
- [ ] Unit tests for growth calculations
- [ ] Validation: growth rates calculated correctly
- [ ] Docstrings for all functions
- [ ] Analysis summary report

---

## 📌 Notes

**Polars Growth Calculation**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/capacity_utilization_integrated.parquet")

df_growth = (
    df.sort(['facility_category', 'sector', 'year'])
    .with_columns([
        ((pl.col('capacity_metric') - pl.col('capacity_metric').shift(1)) / 
         pl.col('capacity_metric').shift(1) * 100)
        .over(['facility_category', 'sector'])
        .alias('yoy_growth_pct')
    ])
)

# CAGR
df_cagr = (
    df.group_by(['facility_category', 'sector']).agg([
        pl.col('capacity_metric').first().alias('start_capacity'),
        pl.col('capacity_metric').last().alias('end_capacity'),
        (pl.col('year').max() - pl.col('year').min()).alias('years')
    ])
    .with_columns([
        ((pl.col('end_capacity') / pl.col('start_capacity')) ** (1 / pl.col('years')) - 1) * 100
        .alias('cagr_pct')
    ])
)
```

**Expected Insights**:
- **Acute Care**: Likely moderate growth (established infrastructure)
- **Community Hospitals**: Possibly rapid growth (policy push for intermediate care)
- **Long-Term Care**: Likely high growth (aging population)
- **Primary Care**: Steady growth (population increase)

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Quantify 11-year capacity evolution (2009–2020) across acute care, community hospitals, long-term care, and primary care using YoY growth rates, compound annual growth rates (CAGR), and public-vs-private sector comparisons.

**Primary User Role**: Healthcare Service Planner assessing infrastructure needs

**Key Success Metric**: `results/tables/problem-statement-003/capacity_growth_analysis_2009_2020_{timestamp}.csv` containing CAGR % and absolute growth per facility type and sector, with 4+ publication-quality figures saved to `reports/figures/problem-statement-003/`.

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| `trend_analysis.py` | `shared/src/analysis/trend_analysis.py` | 🔧 Extend | Add capacity-specific growth/CAGR helpers if not already present |
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream output | US-02 output — BLOCKING dependency |
| Visualization patterns from PS-002 | `problem-statements/ps-002-*/src/visualization/` | 🔧 Reference | Adapt `plot_line_trends()` pattern for capacity charts |

**New Components**:
- `problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_trend_analysis.py` — YoY growth, CAGR, sector comparisons, diversification metrics
- `problem-statements/ps-003-healthcare-capacity-optimization/notebooks/01_capacity_trends.ipynb` — reproducible analysis notebook

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_trend_analysis.py
  Functions:
    calculate_yoy_growth(df, capacity_col, group_cols) -> pl.DataFrame
    calculate_cagr(df, capacity_col, group_cols) -> pl.DataFrame
    calculate_growth_volatility(df, yoy_col, group_cols) -> pl.DataFrame
    calculate_sector_shares(df, capacity_col) -> pl.DataFrame
    analyze_sector_divergence(df, capacity_col) -> pl.DataFrame
    calculate_capacity_distribution(df, capacity_col) -> pl.DataFrame
  Dependencies: polars, loguru, pathlib
  Config: problem-statements/ps-003-healthcare-capacity-optimization/config/config.yml

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/capacity_plots.py
  Functions:
    plot_capacity_trends(df, output_path) -> None
    plot_stacked_area_proportions(df, output_path) -> None
    plot_cagr_comparison(df_cagr, output_path) -> None
    plot_sector_comparison(df_sector, output_path) -> None
  Dependencies: polars, matplotlib, seaborn, pathlib

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/notebooks/01_capacity_trends.ipynb
  - Reproducible EDA notebook using the analysis module

- [CREATE] results/tables/problem-statement-003/ (directory)
- [CREATE] reports/figures/problem-statement-003/ (directory)

- [MODIFY] shared/src/analysis/trend_analysis.py
  - Add: calculate_cagr_generalized(start, end, years) -> float
    (if not already present as a reusable helper)
```

---

### 4. Component Breakdown

#### `capacity_trend_analysis.py`

**Responsibility**: Pure analytical functions — no I/O, no visualization. Returns DataFrames.

**Technical Constraints**:
- Memory: < 100 MB (184 rows in parquet)
- Execution: < 10 seconds per analysis function
- Data volume: trivial — use eager loading
- Edge cases: handle years with only one data point (CAGR requires ≥ 2 years)

---

### 5. Data Pipeline

```
shared/data/3_interim/capacity_utilization_integrated.parquet
  │
  ▼  Filter: drop rows where total_beds is null
  │
  ▼  Sort by [facility_category, sector, year]
  │
  ├─▶ calculate_yoy_growth()  ─────────────────────────────────────────────────────
  │     .shift(1).over([facility_category, sector])                               │
  │     → df with yoy_growth_pct column                                           │
  │                                                                                │
  ├─▶ calculate_cagr()  ──────────────────────────────────────────────────────────
  │     group_by([facility_category, sector]).agg(start, end, years)             │
  │     → df_cagr with cagr_pct, absolute_growth                                 │
  │                                                                                │
  ├─▶ calculate_sector_shares()  ─────────────────────────────────────────────────
  │     group_by([year, sector]).agg(total_sector_beds)                           │
  │     → sector share % per year                                                 │
  │                                                                                │
  ├─▶ calculate_capacity_distribution()  ─────────────────────────────────────────
  │     group_by([year, facility_category]).agg(total_beds)                       │
  │     proportion = facility_beds / total_all_beds                               │
  │                                                                                │
  └─▶ Visualizations (4 figures) → reports/figures/problem-statement-003/
        Results CSV → results/tables/problem-statement-003/
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementations

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_trend_analysis.py
import polars as pl
from pathlib import Path
from loguru import logger
from datetime import datetime


def calculate_yoy_growth(
    df: pl.DataFrame,
    capacity_col: str = "total_beds",
    group_cols: list[str] = ["facility_category", "sector"],
) -> pl.DataFrame:
    """Calculate year-over-year capacity growth rates.

    Args:
        df: Integrated capacity DataFrame sorted by year.
        capacity_col: Column containing the capacity metric (e.g., total_beds).
        group_cols: Columns that define each time series group.

    Returns:
        DataFrame with added columns:
          - {capacity_col}_prev: prior year value
          - yoy_growth_abs: absolute change
          - yoy_growth_pct: percentage change (null for first year per group)
    """
    if capacity_col not in df.columns:
        raise ValueError(f"Column '{capacity_col}' not found. Available: {df.columns}")

    df_sorted = df.sort(group_cols + ["year"])

    df_growth = df_sorted.with_columns([
        pl.col(capacity_col).shift(1).over(group_cols).alias(f"{capacity_col}_prev"),
    ]).with_columns([
        (pl.col(capacity_col) - pl.col(f"{capacity_col}_prev")).alias("yoy_growth_abs"),
        (
            (pl.col(capacity_col) - pl.col(f"{capacity_col}_prev")) /
            pl.col(f"{capacity_col}_prev") * 100
        ).alias("yoy_growth_pct"),
    ])

    non_null = df_growth["yoy_growth_pct"].drop_nulls().len()
    logger.info(f"calculate_yoy_growth: {non_null} non-null growth records computed")
    return df_growth


def calculate_cagr(
    df: pl.DataFrame,
    capacity_col: str = "total_beds",
    group_cols: list[str] = ["facility_category", "sector"],
) -> pl.DataFrame:
    """Calculate Compound Annual Growth Rate (2009–2020) per group.

    Formula: CAGR = (end / start) ^ (1 / years) - 1

    Args:
        df: Integrated capacity DataFrame.
        capacity_col: Column to compute CAGR for.
        group_cols: Group-by columns.

    Returns:
        DataFrame with columns: group_cols + [start_value, end_value,
        years_covered, cagr_pct, absolute_growth, growth_volatility_pct].
    """
    df_sorted = df.sort(group_cols + ["year"])

    df_agg = df_sorted.group_by(group_cols).agg([
        pl.col(capacity_col).first().alias("start_value"),
        pl.col(capacity_col).last().alias("end_value"),
        (pl.col("year").max() - pl.col("year").min()).alias("years_covered"),
        pl.col("year").min().alias("start_year"),
        pl.col("year").max().alias("end_year"),
    ])

    df_cagr = df_agg.with_columns([
        (pl.col("end_value") - pl.col("start_value")).alias("absolute_growth"),
        pl.when(
            (pl.col("years_covered") > 0) & (pl.col("start_value") > 0)
        ).then(
            ((pl.col("end_value") / pl.col("start_value")) **
             (1.0 / pl.col("years_covered")) - 1) * 100
        ).otherwise(None)
        .cast(pl.Float64)
        .alias("cagr_pct"),
    ])

    logger.info(
        f"calculate_cagr: {df_cagr.height} groups — "
        f"CAGR range: {df_cagr['cagr_pct'].min():.1f}% to {df_cagr['cagr_pct'].max():.1f}%"
    )
    return df_cagr


def calculate_growth_volatility(
    df_with_yoy: pl.DataFrame,
    group_cols: list[str] = ["facility_category", "sector"],
) -> pl.DataFrame:
    """Calculate standard deviation of YoY growth rates (volatility proxy).

    Args:
        df_with_yoy: DataFrame from calculate_yoy_growth() with yoy_growth_pct.
        group_cols: Group columns.

    Returns:
        DataFrame with columns: group_cols + [growth_volatility_pct, avg_yoy_pct].
    """
    return df_with_yoy.group_by(group_cols).agg([
        pl.col("yoy_growth_pct").std().alias("growth_volatility_pct"),
        pl.col("yoy_growth_pct").mean().alias("avg_yoy_pct"),
        pl.col("yoy_growth_pct").count().alias("n_periods"),
    ])


def calculate_sector_shares(
    df: pl.DataFrame,
    capacity_col: str = "total_beds",
) -> pl.DataFrame:
    """Calculate each sector's share of total capacity per year.

    Args:
        df: Integrated DataFrame with year, sector, capacity_col.
        capacity_col: Capacity metric column.

    Returns:
        DataFrame with added sector_share_pct column.
    """
    total_by_year = (
        df.group_by("year")
        .agg(pl.col(capacity_col).sum().alias("total_all_sectors"))
    )

    sector_by_year = (
        df.group_by(["year", "sector"])
        .agg(pl.col(capacity_col).sum().alias("sector_capacity"))
    )

    result = sector_by_year.join(total_by_year, on="year").with_columns(
        (pl.col("sector_capacity") / pl.col("total_all_sectors") * 100)
        .alias("sector_share_pct")
    ).sort(["year", "sector"])

    logger.info(f"calculate_sector_shares: {result.height} year-sector records")
    return result


def calculate_capacity_distribution(
    df: pl.DataFrame,
    capacity_col: str = "total_beds",
) -> pl.DataFrame:
    """Calculate each facility type's share of total capacity per year.

    Args:
        df: Integrated DataFrame.
        capacity_col: Capacity metric column.

    Returns:
        DataFrame with facility_share_pct and cumulative_share_pct.
    """
    total_by_year = df.group_by("year").agg(
        pl.col(capacity_col).sum().alias("total_capacity")
    )

    by_facility_year = df.group_by(["year", "facility_category"]).agg(
        pl.col(capacity_col).sum().alias("facility_capacity")
    )

    result = by_facility_year.join(total_by_year, on="year").with_columns(
        (pl.col("facility_capacity") / pl.col("total_capacity") * 100)
        .alias("facility_share_pct")
    ).sort(["year", "facility_share_pct"])

    logger.info(f"calculate_capacity_distribution: {result.height} records")
    return result


def run_capacity_trend_analysis(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    problem_num: str = "003",
) -> dict[str, pl.DataFrame]:
    """Orchestrate all capacity trend analyses and persist results.

    Args:
        parquet_path: Path to integrated dataset from US-02.
        problem_num: Problem statement number for output directory naming.

    Returns:
        Dict of {analysis_name: DataFrame}.
    """
    from datetime import datetime

    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    df = pl.read_parquet(parquet_path).filter(pl.col("total_beds").is_not_null())
    logger.info(f"Loaded {df.height} rows from {parquet_path}")

    results: dict[str, pl.DataFrame] = {}

    # YoY Growth
    df_yoy = calculate_yoy_growth(df)
    results["yoy_growth"] = df_yoy

    # CAGR
    df_cagr = calculate_cagr(df)
    results["cagr"] = df_cagr
    cagr_path = tables_dir / f"capacity_cagr_{ts}.csv"
    df_cagr.write_csv(cagr_path)
    logger.info(f"✅ Saved CAGR table: {cagr_path}")

    # Volatility
    df_volatility = calculate_growth_volatility(df_yoy)
    results["volatility"] = df_volatility

    # Sector shares
    df_sectors = calculate_sector_shares(df)
    results["sector_shares"] = df_sectors

    # Capacity distribution
    df_dist = calculate_capacity_distribution(df)
    results["distribution"] = df_dist

    # Combined growth analysis output
    growth_summary = df_cagr.join(df_volatility, on=["facility_category", "sector"], how="left")
    summary_path = tables_dir / f"capacity_growth_analysis_2009_2020_{ts}.csv"
    growth_summary.write_csv(summary_path)
    logger.info(f"✅ Growth analysis saved: {summary_path}")
    print(f"✅ Results saved to: {tables_dir}")

    return results
```

#### 6.2 Visualization Module

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/capacity_plots.py
import polars as pl
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
from datetime import datetime
from loguru import logger


_FIG_DIR_TEMPLATE = "reports/figures/problem-statement-{num}"
sns.set_style("whitegrid")
plt.rcParams.update({"figure.dpi": 150, "font.size": 11})


def _save_fig(fig: plt.Figure, path: Path, title: str) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(path, dpi=300, bbox_inches="tight")
    logger.info(f"✅ Figure saved: {path}")
    print(f"✅ Saved: {path}")
    plt.show()
    plt.close(fig)


def plot_capacity_trends(
    df: pl.DataFrame,
    capacity_col: str = "total_beds",
    problem_num: str = "003",
) -> None:
    """Line chart of capacity by facility type over time.

    Args:
        df: Integrated DataFrame with year, facility_category, capacity_col.
        capacity_col: Column to plot on y-axis.
        problem_num: Used for output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df.group_by(["year", "facility_category"]).agg(
        pl.col(capacity_col).sum()
    ).sort(["facility_category", "year"]).to_pandas()

    fig, ax = plt.subplots(figsize=(12, 6))
    for facility, grp in pdf.groupby("facility_category"):
        ax.plot(grp["year"], grp[capacity_col], marker="o", label=facility, linewidth=2)

    ax.set_title("Facility Capacity by Type (2009–2020)", fontweight="bold", fontsize=14)
    ax.set_xlabel("Year")
    ax.set_ylabel("Total Beds")
    ax.legend(title="Facility Type", bbox_to_anchor=(1.05, 1), loc="upper left")
    ax.set_xticks(sorted(pdf["year"].unique()))
    ax.tick_params(axis="x", rotation=45)
    ax.grid(True, alpha=0.3)
    plt.tight_layout()

    out = Path(_FIG_DIR_TEMPLATE.format(num=problem_num)) / f"capacity_trends_{ts}.png"
    _save_fig(fig, out, "capacity_trends")


def plot_cagr_comparison(
    df_cagr: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Horizontal bar chart of CAGR % by facility type and sector.

    Args:
        df_cagr: DataFrame from calculate_cagr() with cagr_pct, facility_category, sector.
        problem_num: Used for output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_cagr.to_pandas().dropna(subset=["cagr_pct"])

    fig, ax = plt.subplots(figsize=(10, 6))
    colors = {"public": "#2196F3", "private": "#FF5722", "not_for_profit": "#4CAF50"}
    
    pdf["label"] = pdf["facility_category"] + " (" + pdf["sector"] + ")"
    pdf_sorted = pdf.sort_values("cagr_pct")

    bar_colors = [colors.get(s, "#9E9E9E") for s in pdf_sorted["sector"]]
    bars = ax.barh(pdf_sorted["label"], pdf_sorted["cagr_pct"], color=bar_colors)
    ax.axvline(0, color="black", linewidth=0.8, linestyle="--")
    ax.set_xlabel("CAGR (%)")
    ax.set_title("Compound Annual Growth Rate by Facility Type (2009–2020)", fontweight="bold")
    ax.bar_label(bars, fmt="%.1f%%", padding=3)
    plt.tight_layout()

    out = Path(_FIG_DIR_TEMPLATE.format(num=problem_num)) / f"capacity_cagr_comparison_{ts}.png"
    _save_fig(fig, out, "cagr_comparison")


def plot_sector_shares(
    df_sector: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Stacked area chart showing sector share of total capacity over time.

    Args:
        df_sector: DataFrame from calculate_sector_shares().
        problem_num: Used for output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_sector.sort(["year", "sector"]).to_pandas()
    pivot = pdf.pivot(index="year", columns="sector", values="sector_share_pct").fillna(0)

    fig, ax = plt.subplots(figsize=(12, 6))
    pivot.plot.area(ax=ax, stacked=True, alpha=0.7,
                    color=["#2196F3", "#FF5722", "#4CAF50"])
    ax.set_xlabel("Year")
    ax.set_ylabel("Share of Total Capacity (%)")
    ax.set_title("Sector Share of Total Capacity (2009–2020)", fontweight="bold")
    ax.legend(title="Sector", bbox_to_anchor=(1.05, 1))
    plt.tight_layout()

    out = Path(_FIG_DIR_TEMPLATE.format(num=problem_num)) / f"capacity_sector_shares_{ts}.png"
    _save_fig(fig, out, "sector_shares")


def plot_capacity_distribution(
    df_dist: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Stacked area chart of facility type share over time.

    Args:
        df_dist: DataFrame from calculate_capacity_distribution().
        problem_num: Used for output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_dist.sort(["year", "facility_category"]).to_pandas()
    pivot = pdf.pivot(index="year", columns="facility_category",
                      values="facility_share_pct").fillna(0)

    fig, ax = plt.subplots(figsize=(12, 6))
    pivot.plot.area(ax=ax, stacked=True, alpha=0.7)
    ax.set_xlabel("Year")
    ax.set_ylabel("Share of Total Capacity (%)")
    ax.set_title("Capacity Distribution by Facility Type (2009–2020)", fontweight="bold")
    ax.legend(title="Facility Type", bbox_to_anchor=(1.05, 1))
    plt.tight_layout()

    out = Path(_FIG_DIR_TEMPLATE.format(num=problem_num)) / f"capacity_distribution_{ts}.png"
    _save_fig(fig, out, "capacity_distribution")
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 matplotlib>=3.8.0 seaborn>=0.13.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

**Validated against available data**:

| Feature | Formula | Data Source | Validation |
|---------|---------|---------|---------|
| YoY growth (%) | `(t - t-1)/t-1 × 100` | `total_beds` from integrated parquet | Null for first year |
| CAGR (%) | `(end/start)^(1/n) - 1` | Same | Requires ≥ 2 years |
| Growth volatility | `std(yoy_growth_pct)` | YoY output | CAGR supplement metric |
| Sector share (%) | `sector_beds / total_beds × 100` | Integrated parquet | Sum = 100% per year |
| Facility type proportion (%) | `facility_beds / total_beds × 100` | Same | Sum = 100% per year |

**Explicitly rejected** (data not available):
- Beds per 1,000 population — requires population data not in main dataset (handled in US-05)
- Real investment amounts — no financial data

---

### 8. Testing Strategy

```python
# problem-statements/ps-003-healthcare-capacity-optimization/tests/unit/test_capacity_trend_analysis.py
import polars as pl
import pytest
from src.analysis.capacity_trend_analysis import (
    calculate_yoy_growth,
    calculate_cagr,
    calculate_sector_shares,
    calculate_capacity_distribution,
)


@pytest.fixture
def sample_capacity_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2009, 2010, 2011, 2009, 2010, 2011], dtype=pl.Int32),
        "facility_category": ["acute_care"] * 3 + ["long_term_care"] * 3,
        "sector": ["public"] * 6,
        "total_beds": pl.Series([1000, 1100, 1210, 500, 550, 605], dtype=pl.Int32),
    })


def test_yoy_growth_first_year_is_null(sample_capacity_df):
    result = calculate_yoy_growth(sample_capacity_df)
    first_year_vals = result.filter(pl.col("year") == 2009)["yoy_growth_pct"]
    assert first_year_vals.is_null().all()


def test_yoy_growth_correct_value(sample_capacity_df):
    result = calculate_yoy_growth(sample_capacity_df)
    acute_2010 = result.filter(
        (pl.col("year") == 2010) & (pl.col("facility_category") == "acute_care")
    )["yoy_growth_pct"][0]
    assert acute_2010 == pytest.approx(10.0, rel=1e-4)


def test_cagr_known_growth():
    # Double in 10 years → CAGR ≈ 7.177%
    df = pl.DataFrame({
        "year": pl.Series([2009, 2019], dtype=pl.Int32),
        "facility_category": ["acute_care", "acute_care"],
        "sector": ["public", "public"],
        "total_beds": pl.Series([1000, 2000], dtype=pl.Int32),
    })
    result = calculate_cagr(df)
    assert result["cagr_pct"][0] == pytest.approx(7.177, abs=0.01)


def test_cagr_absolute_growth(sample_capacity_df):
    result = calculate_cagr(sample_capacity_df)
    acute_row = result.filter(pl.col("facility_category") == "acute_care")
    assert acute_row["absolute_growth"][0] == 210  # 1210 - 1000


def test_sector_shares_sum_to_100(sample_capacity_df):
    result = calculate_sector_shares(sample_capacity_df)
    for year in [2009, 2010, 2011]:
        year_sum = result.filter(pl.col("year") == year)["sector_share_pct"].sum()
        assert year_sum == pytest.approx(100.0, abs=0.1)


def test_capacity_distribution_sum_to_100(sample_capacity_df):
    result = calculate_capacity_distribution(sample_capacity_df)
    for year in [2009, 2010, 2011]:
        year_sum = result.filter(pl.col("year") == year)["facility_share_pct"].sum()
        assert year_sum == pytest.approx(100.0, abs=0.1)
```

---

### 9. Implementation Steps

#### Phase 1: Project Structure Setup
- [ ] Create `problem-statements/ps-003-healthcare-capacity-optimization/` directory tree
- [ ] Create `src/analysis/`, `src/visualization/`, `notebooks/`, `tests/unit/`
- [ ] Create `config/config.yml` with analysis thresholds and constants

#### Phase 2: Analysis Functions
- [ ] Create `capacity_trend_analysis.py` with all functions per Section 6.1
- [ ] Test each function in isolation before integration

#### Phase 3: Visualization Module
- [ ] Create `capacity_plots.py` per Section 6.1
- [ ] Run each plot function on sample data; verify PNG output

#### Phase 4: End-to-End Execution
- [ ] Verify `capacity_utilization_integrated.parquet` from US-02 available
- [ ] Run `run_capacity_trend_analysis()` 
- [ ] Confirm CSV outputs in `results/tables/problem-statement-003/`
- [ ] Generate all 4 figures to `reports/figures/problem-statement-003/`

#### Phase 5: Tests
- [ ] Write all tests per Section 8
- [ ] Run: `pytest tests/unit/test_capacity_trend_analysis.py -v --cov`
- [ ] Coverage ≥ 80%

---

### 10. Code Generation Order

1. `problem-statements/ps-003-healthcare-capacity-optimization/config/config.yml`
2. `src/analysis/capacity_trend_analysis.py`
3. `tests/unit/test_capacity_trend_analysis.py`
4. `src/visualization/capacity_plots.py`
5. `notebooks/01_capacity_trends.ipynb`

---

### 11. Data Quality & Validation

**Input checks** (before analysis):
- Assert `total_beds.is_not_null()` — filter and log dropped rows
- Assert year range 2009 ≤ year ≤ 2020
- Validate minimum 2 data points per group for CAGR

**Output checks** (after analysis):
- Sector shares sum to 100% per year (tolerance ± 0.1%)
- CAGR values within plausible range: −20% to +30% per year
- CAGR null only when `start_value == 0` or `years_covered == 0`

---

**Implementation Plan Checklist**
- [x] Feature overview with key success metrics
- [x] Component reuse strategy with justifications
- [x] All affected files with CREATE/MODIFY indicators
- [x] Complete executable function implementations with type hints
- [x] Visualization module with save-to-disk pattern
- [x] Specific test assertions with expected values
- [x] Implementation steps ordered by phase
- [x] Output directory conventions followed (problem-statement-{num}/)
