# Calculate Utilization Efficiency Metrics (Lifecycle Stage: Feature Engineering)

**Story ID**: PS-003-US-05  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Service Planner**,  
I want **to calculate proxy bed occupancy rates and utilization efficiency metrics by comparing capacity trends against admission rate trends**,  
So that **I can identify facilities operating at capacity strain vs those with excess capacity and optimize resource allocation accordingly**.

---

## 🎯 Acceptance Criteria

1. **Proxy occupancy metrics calculated**
   - Proxy occupancy rate: `(admission_rate × avg_length_of_stay) / (beds_per_1000_population × 365 days)` (simplified)
   - Utilization index: `utilization_growth / capacity_growth` (if >1, demand growing faster than supply)
   - Capacity strain indicator: flag years/facilities where proxy occupancy >85%

2. **Capacity-to-population ratios calculated**
   - Beds per 1,000 population by facility type (acute, intermediate, long-term care)
   - Trend analysis: is ratio increasing (capacity expansion) or decreasing (population growth outpacing capacity)?
   - Benchmark against international standards (WHO, OECD targets)

3. **Efficiency benchmarks established**
   - Target occupancy: 75-85% (optimal efficiency range per healthcare literature)
   - Under-utilized: <65% occupancy (potential over-investment)
   - Over-utilized: >90% occupancy (strain, quality risk)
   - Classify facilities into efficiency categories

4. **Data output requirement**
   - Output file: `results/tables/capacity_utilization_efficiency.csv`
   - Format: CSV (facility_type, year, beds_per_1000, proxy_occupancy_pct, utilization_index, efficiency_category)
   - Efficiency visualization: `reports/figures/utilization_efficiency_heatmap.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Logging**: loguru
- **Testing**: pytest with ≥80% coverage

---

## 📚 Domain Knowledge References

- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objectives) - Objective 2: Assess utilization patterns
- International benchmarks: WHO bed density standards, OECD health statistics

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Metric calculations
- `matplotlib>=3.8.0`: Heatmap visualizations
- `loguru>=0.7.0`: Logging

### Internal Dependencies
- **Upstream**: 
  - PS-003-US-02 (Integrated capacity data - BLOCKING)
  - PS-003-US-03 (Capacity trends - RECOMMENDED)
  - PS-003-US-04 (Utilization patterns - BLOCKING)
- **Data Sources**: 
  - `shared/data/3_interim/capacity_utilization_integrated.parquet`
  - Population data (from admission rate denominators)
- **Config Files**: `config/analysis.yml` (efficiency thresholds, international benchmarks)

---

## ✅ Implementation Tasks

### Population Data Preparation
- [ ] Extract population estimates from admission rate data
- [ ] Validate population trends (should match Singapore statistics)
- [ ] Create annual population dataset

### Capacity Metrics Calculation
- [ ] Calculate beds per 1,000 population: `(total_beds / population) * 1000`
- [ ] Calculate by facility type: acute, intermediate, long-term care
- [ ] Calculate public vs private sector bed densities
- [ ] Compare against international benchmarks (WHO: 3-5 beds per 1,000)

### Proxy Occupancy Calculation
- [ ] Estimate admissions from admission rate: `rate * population / 1000`
- [ ] Estimate bed-days needed: `admissions * avg_length_of_stay` (assume or derive from data)
- [ ] Calculate proxy occupancy: `bed_days_needed / (total_beds * 365)`
- [ ] Handle missing average length of stay (use literature values: 5-7 days acute, 20-30 days intermediate)

### Utilization Index
- [ ] Calculate utilization growth rate: `(admissions_t - admissions_t-1) / admissions_t-1`
- [ ] Calculate capacity growth rate: `(beds_t - beds_t-1) / beds_t-1`
- [ ] Calculate utilization index: `utilization_growth / capacity_growth`
- [ ] Interpret: index >1 = demand outpacing supply, index <1 = supply outpacing demand

### Efficiency Classification
- [ ] Classify by proxy occupancy:
  - Under-utilized: <65%
  - Optimal: 65-85%
  - High: 85-90%
  - Over-capacity (strain): >90%
- [ ] Flag years and facility types with efficiency concerns

### Visualization
- [ ] Heatmap: proxy occupancy % by facility type and year
- [ ] Line chart: capacity trends vs utilization trends (dual-axis)
- [ ] Scatter plot: beds per 1,000 vs proxy occupancy (identify outliers)
- [ ] Export figures

### Testing & Validation
- [ ] Unit tests for metric calculations
- [ ] Validate beds per 1,000: compare against WHO reported values (if available)
- [ ] Test edge cases: zero beds, missing years

### Documentation
- [ ] Docstrings (Google style)
- [ ] Methodology document: `results/capacity_efficiency_methodology.md`
  - Proxy occupancy calculation rationale
  - Assumptions and limitations
  - International benchmark sources
- [ ] Update data dictionary

---

## 📌 Notes

**Beds per 1,000 Population (Polars)**:
```python
import polars as pl

df_density = (
    df_capacity
    .join(df_population, on='year', how='left')
    .with_columns([
        (pl.col('total_beds') / pl.col('population') * 1000).alias('beds_per_1000')
    ])
)
```

**Proxy Occupancy Calculation** (simplified):
```python
# Assume average length of stay (ALOS) from literature
ALOS_ACUTE = 5.5  # days (Singapore hospital average)

df_occupancy = (
    df_utilization
    .join(df_capacity, on=['year', 'facility_type'], how='inner')
    .with_columns([
        # Estimated bed-days needed
        (pl.col('annual_admissions') * ALOS_ACUTE).alias('bed_days_needed'),
        
        # Available bed-days
        (pl.col('total_beds') * 365).alias('bed_days_available'),
        
        # Proxy occupancy
        ((pl.col('bed_days_needed') / pl.col('bed_days_available')) * 100)
        .alias('proxy_occupancy_pct')
    ])
)
```

**Utilization Index**:
```python
df_index = (
    df.sort(['facility_type', 'year'])
    .with_columns([
        # Growth rates
        ((pl.col('annual_admissions') - pl.col('annual_admissions').shift(1)) 
         / pl.col('annual_admissions').shift(1))
        .over('facility_type')
        .alias('utilization_growth'),
        
        ((pl.col('total_beds') - pl.col('total_beds').shift(1)) 
         / pl.col('total_beds').shift(1))
        .over('facility_type')
        .alias('capacity_growth')
    ])
    .with_columns([
        (pl.col('utilization_growth') / pl.col('capacity_growth')).alias('utilization_index')
    ])
)
```

**International Benchmarks**:
- **WHO Minimum**: 3 beds per 1,000 population (essential services)
- **OECD Average**: ~5 beds per 1,000 population (acute care)
- **Target Occupancy**: 75-85% (balances efficiency and quality)
- **Singapore (2020)**: ~4.5 beds per 1,000 (estimate)

**Limitations to Document**:
- Proxy occupancy is simplified (actual occupancy requires discharge data)
- Average length of stay estimated from literature (not actual data)
- Does not account for seasonal variation or day-of-week patterns
- Aggregate metrics mask facility-level variation

**Expected Insights**:
- Acute care likely shows high occupancy (85-90%) - efficient but strained
- Long-term care likely shows lower occupancy (60-70%) - potential surplus or access issues
- Public sector likely higher occupancy than private (different demand patterns)

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Calculate proxy bed-occupancy rates, beds-per-1,000-population ratios, utilisation index, and efficiency category classifications for each facility type and year (2009–2020). These metrics form the quantitative backbone for gap analysis (US-06) and scenario modelling (US-08).

**Primary User Role**: Healthcare Service Planner

**Key Success Metric**:
- `results/tables/problem-statement-003/capacity_utilization_efficiency_{timestamp}.csv` with columns: `facility_category`, `year`, `beds_per_1000`, `proxy_occupancy_pct`, `utilization_index`, `efficiency_category`
- `reports/figures/problem-statement-003/utilization_efficiency_heatmap_{timestamp}.png`

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream | US-02 — BLOCKING |
| Growth rates from US-03 | `capacity_trend_analysis.py` | ✅ Reuse | `calculate_yoy_growth()` for utilization index |
| Demographic profiles from US-04 | `demographic_utilization_analysis.py` | ✅ Reuse | `aggregate_demographic_profiles()` for admission rate totals |

**New Components**:
- `src/analysis/efficiency_metrics.py` — beds per 1k, proxy occupancy, utilisation index, classification
- `src/visualization/efficiency_plots.py` — heatmap, dual-axis trend chart, scatter

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/efficiency_metrics.py
  Functions:
    calculate_beds_per_1000(df, population) -> pl.DataFrame
    calculate_proxy_occupancy(df, avg_los_days, population) -> pl.DataFrame
    calculate_utilization_index(df, group_cols) -> pl.DataFrame
    classify_efficiency(df) -> pl.DataFrame
    benchmark_against_international(df) -> pl.DataFrame
    run_efficiency_analysis(parquet_path, problem_num) -> pl.DataFrame

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/efficiency_plots.py
  Functions:
    plot_efficiency_heatmap(df, output_path) -> None
    plot_capacity_vs_utilization_dual_axis(df, facility, output_path) -> None
    plot_beds_per_1000_trend(df, output_path) -> None

- Results: results/tables/problem-statement-003/capacity_utilization_efficiency_{ts}.csv
- Figures:  reports/figures/problem-statement-003/utilization_efficiency_heatmap_{ts}.png
            reports/figures/problem-statement-003/capacity_vs_utilization_{ts}.png
            reports/figures/problem-statement-003/beds_per_1000_trend_{ts}.png

- [MODIFY] problem-statements/ps-003-healthcare-capacity-optimization/config/config.yml
  Add: efficiency_thresholds, avg_los_by_facility, international_benchmarks
```

---

### 4. Component Breakdown

#### `efficiency_metrics.py`

**Responsibility**: Derives efficiency metrics from combined capacity and utilization data. Returns DataFrames only — no I/O, no visualisation.

**Technical Constraints**:
- Memory: < 200 MB
- Execution: < 30 seconds
- ALOS (average length of stay) sourced from config file, NOT hard-coded
- Population constant: 5,686,000 (2023 estimate); configurable
- Division-by-zero guard required for utilization index and proxy occupancy

---

### 5. Data Pipeline

```
shared/data/3_interim/capacity_utilization_integrated.parquet
  │
  ▼  group_by([year, facility_category]).agg(total_beds.sum(), total_admission_rate.sum())
  │
  ├─▶ calculate_beds_per_1000()
  │     beds_per_1000 = total_beds / population × 1000
  │     compare vs WHO minimum (3), OECD average (5)
  │
  ├─▶ calculate_proxy_occupancy()
  │     estimated_admissions = total_admission_rate × population / 100_000
  │     bed_days_needed = estimated_admissions × ALOS
  │     bed_days_available = total_beds × 365
  │     proxy_occupancy_pct = bed_days_needed / bed_days_available × 100
  │
  ├─▶ calculate_utilization_index()
  │     utilization_growth = YoY% of total_admission_rate
  │     capacity_growth = YoY% of total_beds
  │     utilization_index = utilization_growth / capacity_growth
  │     (null where capacity_growth == 0)
  │
  ├─▶ classify_efficiency()
  │     <65%  → under_utilized
  │     65-85% → optimal
  │     85-90% → high
  │     >90%  → over_capacity
  │
  └─▶ Join all metrics into single DataFrame
          → Save CSV
          → Generate visualizations
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementations

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/efficiency_metrics.py
import polars as pl
from pathlib import Path
from datetime import datetime
from loguru import logger

# Configurable constants (also in config/config.yml)
SINGAPORE_POPULATION: int = 5_686_000  # 2023 estimate
INTERNATIONAL_BENCHMARKS = {
    "who_minimum_beds_per_1000": 3.0,
    "oecd_average_beds_per_1000": 5.0,
    "target_occupancy_lower": 75.0,
    "target_occupancy_upper": 85.0,
}
# Average length of stay by facility type (from healthcare literature)
DEFAULT_ALOS_DAYS: dict[str, float] = {
    "acute_care": 5.5,
    "community_hospital": 20.0,
    "long_term_care": 60.0,
    "primary_care": 0.5,  # Day visits
}


def calculate_beds_per_1000(
    df: pl.DataFrame,
    population: int = SINGAPORE_POPULATION,
    beds_col: str = "total_beds",
) -> pl.DataFrame:
    """Calculate beds per 1,000 population by facility category and year.

    Args:
        df: Aggregated capacity DataFrame with facility_category, year, beds_col.
        population: National population for rate calculation.
        beds_col: Column containing bed counts.

    Returns:
        DataFrame with added beds_per_1000 column and benchmark comparison columns.
    """
    if beds_col not in df.columns:
        raise ValueError(f"Column '{beds_col}' not found. Available: {df.columns}")

    df_out = df.with_columns([
        (pl.col(beds_col) / population * 1000).cast(pl.Float64).alias("beds_per_1000"),
    ]).with_columns([
        pl.when(pl.col("beds_per_1000") < INTERNATIONAL_BENCHMARKS["who_minimum_beds_per_1000"])
        .then(pl.lit("below_who_minimum"))
        .when(pl.col("beds_per_1000") < INTERNATIONAL_BENCHMARKS["oecd_average_beds_per_1000"])
        .then(pl.lit("below_oecd_average"))
        .otherwise(pl.lit("at_or_above_oecd"))
        .cast(pl.Categorical)
        .alias("who_benchmark_status"),
    ])
    logger.info(
        f"calculate_beds_per_1000: range {df_out['beds_per_1000'].min():.2f} – "
        f"{df_out['beds_per_1000'].max():.2f} beds per 1k"
    )
    return df_out


def calculate_proxy_occupancy(
    df: pl.DataFrame,
    avg_los_days: dict[str, float] | None = None,
    population: int = SINGAPORE_POPULATION,
    admission_rate_col: str = "total_admission_rate",
    beds_col: str = "total_beds",
) -> pl.DataFrame:
    """Calculate proxy bed occupancy rate.

    Formula:
        estimated_admissions = admission_rate_per_100k × population / 100_000
        bed_days_needed = estimated_admissions × avg_LOS
        proxy_occupancy_pct = bed_days_needed / (total_beds × 365) × 100

    Args:
        df: Integrated DataFrame with facility_category, admission_rate, beds.
        avg_los_days: Dict of {facility_category: avg_LOS_in_days}.
        population: National population.
        admission_rate_col: Column with admission rate per 100k.
        beds_col: Column with total bed count.

    Returns:
        DataFrame with estimated_admissions, bed_days_needed, proxy_occupancy_pct.
    """
    if avg_los_days is None:
        avg_los_days = DEFAULT_ALOS_DAYS

    # Map average LOS to each row via facility_category
    los_series = pl.Series(
        "avg_los",
        [avg_los_days.get(fc, 5.5) for fc in df["facility_category"].to_list()],
        dtype=pl.Float64,
    )

    df_out = df.with_columns([
        los_series.alias("avg_los_days"),
        (pl.col(admission_rate_col) * population / 100_000)
        .cast(pl.Float64)
        .alias("estimated_admissions"),
    ]).with_columns([
        (pl.col("estimated_admissions") * pl.col("avg_los_days"))
        .alias("bed_days_needed"),
        (pl.col(beds_col) * 365).cast(pl.Float64).alias("bed_days_available"),
    ]).with_columns([
        pl.when(pl.col("bed_days_available") > 0)
        .then((pl.col("bed_days_needed") / pl.col("bed_days_available") * 100))
        .otherwise(None)
        .cast(pl.Float64)
        .alias("proxy_occupancy_pct"),
    ])

    valid_rows = df_out.filter(pl.col("proxy_occupancy_pct").is_not_null()).height
    logger.info(
        f"calculate_proxy_occupancy: {valid_rows} valid rows, "
        f"occupancy range {df_out['proxy_occupancy_pct'].drop_nulls().min():.1f}% – "
        f"{df_out['proxy_occupancy_pct'].drop_nulls().max():.1f}%"
    )
    return df_out


def calculate_utilization_index(
    df: pl.DataFrame,
    group_cols: list[str] = ["facility_category"],
    admission_col: str = "total_admission_rate",
    beds_col: str = "total_beds",
) -> pl.DataFrame:
    """Calculate utilization index = utilization_growth / capacity_growth.

    Index interpretation:
      > 1.0 → Demand growing faster than supply (shortage risk)
      < 1.0 → Supply growing faster than demand (potential surplus)
      < 0   → Demand and supply moving in opposite directions

    Args:
        df: Integrated DataFrame sorted by year within each group.
        group_cols: Group columns for window function.
        admission_col: Utilization metric column.
        beds_col: Capacity metric column.

    Returns:
        DataFrame with utilization_growth_pct, capacity_growth_pct, utilization_index.
    """
    df_sorted = df.sort(group_cols + ["year"])

    df_growth = df_sorted.with_columns([
        (pl.col(admission_col).pct_change().over(group_cols) * 100)
        .alias("utilization_growth_pct"),
        (pl.col(beds_col).cast(pl.Float64).pct_change().over(group_cols) * 100)
        .alias("capacity_growth_pct"),
    ]).with_columns([
        pl.when(pl.col("capacity_growth_pct") != 0)
        .then(pl.col("utilization_growth_pct") / pl.col("capacity_growth_pct"))
        .otherwise(None)
        .cast(pl.Float64)
        .alias("utilization_index"),
    ])

    shortage_years = df_growth.filter(pl.col("utilization_index") > 1.0).height
    logger.info(
        f"calculate_utilization_index: {shortage_years} year-facility combinations "
        "where demand outpaces supply"
    )
    return df_growth


def classify_efficiency(
    df: pl.DataFrame,
    occupancy_col: str = "proxy_occupancy_pct",
) -> pl.DataFrame:
    """Classify each row into efficiency category based on proxy occupancy.

    Categories (per healthcare literature):
      under_utilized: < 65%
      optimal:        65% – 85%
      high:           85% – 90%
      over_capacity:  > 90%

    Args:
        df: DataFrame with proxy_occupancy_pct column.
        occupancy_col: Column name for occupancy percentage.

    Returns:
        DataFrame with added efficiency_category column.
    """
    return df.with_columns([
        pl.when(pl.col(occupancy_col) < 65)
        .then(pl.lit("under_utilized"))
        .when(pl.col(occupancy_col) < 85)
        .then(pl.lit("optimal"))
        .when(pl.col(occupancy_col) < 90)
        .then(pl.lit("high"))
        .when(pl.col(occupancy_col).is_not_null())
        .then(pl.lit("over_capacity"))
        .otherwise(pl.lit("unknown"))
        .cast(pl.Categorical)
        .alias("efficiency_category"),
    ])


def run_efficiency_analysis(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    problem_num: str = "003",
) -> pl.DataFrame:
    """Orchestrate efficiency metrics pipeline and persist results.

    Args:
        parquet_path: Path to integrated dataset.
        problem_num: Problem statement number for output directory.

    Returns:
        Final efficiency DataFrame.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    df = pl.read_parquet(parquet_path).filter(pl.col("total_beds").is_not_null())
    logger.info(f"Loaded {df.height} rows")

    # Aggregate to facility_category × year level
    df_agg = (
        df.group_by(["year", "facility_category"])
        .agg([
            pl.col("total_beds").sum().alias("total_beds"),
            pl.col("total_admission_rate").sum().alias("total_admission_rate"),
        ])
        .sort(["facility_category", "year"])
    )

    # Compute metrics sequentially
    df_agg = calculate_beds_per_1000(df_agg)
    df_agg = calculate_proxy_occupancy(df_agg)
    df_agg = calculate_utilization_index(df_agg)
    df_agg = classify_efficiency(df_agg)

    output_path = tables_dir / f"capacity_utilization_efficiency_{ts}.csv"
    df_agg.write_csv(output_path)
    logger.info(f"✅ Efficiency metrics saved: {output_path}")
    print(f"✅ Saved: {output_path}")

    return df_agg
```

#### 6.2 Visualization Module

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/efficiency_plots.py
import polars as pl
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
from datetime import datetime
from loguru import logger

_FIG_DIR = "reports/figures/problem-statement-{num}"
sns.set_style("whitegrid")


def _save_fig(fig: plt.Figure, path: Path) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(path, dpi=300, bbox_inches="tight")
    logger.info(f"✅ Figure saved: {path}")
    print(f"✅ Saved: {path}")
    plt.show()
    plt.close(fig)


def plot_efficiency_heatmap(
    df: pl.DataFrame,
    metric_col: str = "proxy_occupancy_pct",
    problem_num: str = "003",
) -> None:
    """Heatmap of proxy occupancy by facility_category (rows) × year (columns).

    Color coding:
      < 65%  → blue (under-utilized)
      65-85% → green (optimal)
      > 90%  → red (over-capacity)

    Args:
        df: Efficiency DataFrame with facility_category, year, proxy_occupancy_pct.
        metric_col: Metric to display.
        problem_num: Output directory number.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pivot = (
        df.select(["facility_category", "year", metric_col])
        .to_pandas()
        .pivot(index="facility_category", columns="year", values=metric_col)
    )

    fig, ax = plt.subplots(figsize=(16, 5))
    sns.heatmap(pivot, annot=True, fmt=".1f", cmap="RdYlGn_r",
                vmin=50, vmax=100, linewidths=0.5,
                cbar_kws={"label": "Proxy Occupancy (%)"}, ax=ax)

    # Add reference lines
    ax.axhline(y=0, color="orange", linewidth=2, linestyle="--", alpha=0.5)
    ax.set_title("Proxy Bed Occupancy Rate by Facility Type (2009–2020)\n"
                 "Green = Optimal (65-85%), Red = Over-capacity (>90%)",
                 fontweight="bold")
    ax.set_xlabel("Year")
    ax.set_ylabel("Facility Category")
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"utilization_efficiency_heatmap_{ts}.png"
    _save_fig(fig, out)


def plot_beds_per_1000_trend(
    df: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Line chart: beds per 1,000 population over time by facility category.

    Includes horizontal reference lines for WHO (3.0) and OECD (5.0) benchmarks.

    Args:
        df: Efficiency DataFrame with beds_per_1000, facility_category, year.
        problem_num: Output directory number.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df.sort(["facility_category", "year"]).to_pandas()

    fig, ax = plt.subplots(figsize=(12, 6))
    for facility, grp in pdf.groupby("facility_category"):
        ax.plot(grp["year"], grp["beds_per_1000"], marker="o", label=facility, linewidth=2)

    ax.axhline(y=3.0, color="red", linestyle="--", linewidth=1.5, alpha=0.7, label="WHO Min (3.0)")
    ax.axhline(y=5.0, color="orange", linestyle="--", linewidth=1.5, alpha=0.7, label="OECD Avg (5.0)")
    ax.set_xlabel("Year")
    ax.set_ylabel("Beds per 1,000 Population")
    ax.set_title("Beds per 1,000 Population by Facility Type (2009–2020)", fontweight="bold")
    ax.legend(bbox_to_anchor=(1.05, 1))
    ax.grid(True, alpha=0.3)
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"beds_per_1000_trend_{ts}.png"
    _save_fig(fig, out)
```

#### 6.3 Configuration Extension

```yaml
# Append to config/config.yml
efficiency_thresholds:
  under_utilized_upper: 65
  optimal_upper: 85
  high_upper: 90

international_benchmarks:
  who_minimum_beds_per_1000: 3.0
  oecd_average_beds_per_1000: 5.0

avg_los_days_by_facility:
  acute_care: 5.5
  community_hospital: 20.0
  long_term_care: 60.0
  primary_care: 0.5

population:
  singapore_2023: 5686000
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 matplotlib>=3.8.0 seaborn>=0.13.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

| Feature | Formula | Limitation | Range |
|---------|---------|---------|---------|
| `beds_per_1000` | `total_beds / pop × 1000` | Per facility type, not total system | 0.1 – 10 |
| `proxy_occupancy_pct` | `(admissions × ALOS) / (beds × 365) × 100` | ALOS from literature, not actual data | 20% – 120% (can exceed 100% if ALOS miscalibrated) |
| `utilization_index` | `util_growth_pct / cap_growth_pct` | Null when cap_growth = 0; sensitive to small denominators | −5 to +5 |
| `efficiency_category` | Categorical from proxy_occupancy_pct | Based on literature thresholds, not Singapore-specific standards | 4 categories |

**Methodology document** must note ALOS limitations prominently.

---

### 8. Testing Strategy

```python
# tests/unit/test_efficiency_metrics.py
import polars as pl
import pytest
from src.analysis.efficiency_metrics import (
    calculate_beds_per_1000,
    calculate_proxy_occupancy,
    calculate_utilization_index,
    classify_efficiency,
)

POPULATION = 5_686_000


@pytest.fixture
def sample_agg_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2015, 2016, 2015, 2016], dtype=pl.Int32),
        "facility_category": ["acute_care", "acute_care", "long_term_care", "long_term_care"],
        "total_beds": pl.Series([10_000, 10_500, 4_000, 4_200], dtype=pl.Int32),
        "total_admission_rate": [8000.0, 8400.0, 2000.0, 2100.0],
    })


def test_beds_per_1000_formula(sample_agg_df):
    result = calculate_beds_per_1000(sample_agg_df, population=POPULATION)
    expected = 10_000 / POPULATION * 1000
    actual = result.filter(
        (pl.col("year") == 2015) & (pl.col("facility_category") == "acute_care")
    )["beds_per_1000"][0]
    assert actual == pytest.approx(expected, rel=1e-4)


def test_proxy_occupancy_non_negative(sample_agg_df):
    result = calculate_proxy_occupancy(sample_agg_df, population=POPULATION)
    assert (result["proxy_occupancy_pct"].drop_nulls() >= 0).all()


def test_proxy_occupancy_zero_beds_gives_null():
    df = pl.DataFrame({
        "facility_category": ["acute_care"],
        "year": pl.Series([2015], dtype=pl.Int32),
        "total_beds": pl.Series([0], dtype=pl.Int32),
        "total_admission_rate": [500.0],
    })
    result = calculate_proxy_occupancy(df, population=POPULATION)
    assert result["proxy_occupancy_pct"].is_null()[0]


def test_classify_efficiency_boundaries():
    df = pl.DataFrame({
        "proxy_occupancy_pct": [60.0, 64.9, 65.0, 84.9, 85.0, 89.9, 90.0, 95.0]
    })
    result = classify_efficiency(df)
    cats = result["efficiency_category"].to_list()
    assert cats[0] == "under_utilized"   # 60
    assert cats[1] == "under_utilized"   # 64.9
    assert cats[2] == "optimal"          # 65
    assert cats[3] == "optimal"          # 84.9
    assert cats[4] == "high"             # 85
    assert cats[5] == "high"             # 89.9
    assert cats[6] == "over_capacity"    # 90
    assert cats[7] == "over_capacity"    # 95


def test_utilization_index_zero_capacity_growth():
    df = pl.DataFrame({
        "year": pl.Series([2015, 2016], dtype=pl.Int32),
        "facility_category": ["acute_care", "acute_care"],
        "total_beds": pl.Series([10_000, 10_000], dtype=pl.Int32),  # No growth
        "total_admission_rate": [8000.0, 8400.0],
    })
    result = calculate_utilization_index(df)
    # capacity_growth_pct = 0, so utilization_index should be null
    assert result.filter(pl.col("year") == 2016)["utilization_index"].is_null()[0]
```

---

### 9. Implementation Steps

#### Phase 1: Configuration
- [ ] Add efficiency thresholds and ALOS values to `config/config.yml`

#### Phase 2: Analysis Module
- [ ] Create `src/analysis/efficiency_metrics.py` with all 5 functions + orchestrator
- [ ] Validate ALOS values against Singapore MOH data where available
- [ ] Run `run_efficiency_analysis()` and inspect output

#### Phase 3: Visualizations
- [ ] Create `src/visualization/efficiency_plots.py`
- [ ] Generate `utilization_efficiency_heatmap` — confirm color coding correct
- [ ] Generate `beds_per_1000_trend` with WHO/OECD reference lines

#### Phase 4: Methodology Documentation
- [ ] Create `results/capacity_efficiency_methodology.md` documenting:
  - ALOS source and assumptions
  - Population denominator
  - Proxy occupancy limitations
  - International benchmark sources

#### Phase 5: Tests
- [ ] Write `tests/unit/test_efficiency_metrics.py`
- [ ] `pytest tests/unit/test_efficiency_metrics.py -v --cov`
- [ ] ≥ 80% coverage

---

### 10. Code Generation Order

1. `config/config.yml` (add efficiency section)
2. `src/analysis/efficiency_metrics.py`
3. `tests/unit/test_efficiency_metrics.py`
4. `src/visualization/efficiency_plots.py`
5. `notebooks/03_efficiency_metrics.ipynb`

---

### 11. Data Quality & Validation

**Pre-analysis**:
- Assert `total_beds > 0` for all rows used in occupancy calculation
- Log count of rows with null `total_admission_rate` (expected for some facility types)

**Post-analysis**:
- `proxy_occupancy_pct` range: 0 – 150% (warn if > 130% — suggests ALOS mis-specified)
- `beds_per_1000` should be > 0.1 for all facility types (flag if < 0.05)
- Efficiency distribution: assert at least one row in each category across all years
- `utilization_index` nulls only where `capacity_growth_pct == 0`

---

**Implementation Plan Checklist**
- [x] Feature overview with quantitative success metrics
- [x] Component reuse strategy with upstream dependencies noted
- [x] All affected files with CREATE/MODIFY indicators and function signatures
- [x] Complete executable implementations with type hints, docstrings, error handling
- [x] Configuration YAML extension for ALOS and thresholds
- [x] Boundary-condition tests with exact assertions
- [x] Methodology documentation requirement specified
- [x] Phased implementation steps
- [x] Data quality validation for proxy occupancy and utilization index edge cases
