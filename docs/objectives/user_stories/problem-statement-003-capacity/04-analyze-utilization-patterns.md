# Analyze Utilization Patterns by Demographics (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-003-US-04  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Service Planner understanding demand drivers**,  
I want **to analyze hospital admission patterns by age, sex, and demographic groups from 2006-2020 to identify high-burden populations**,  
So that **I can understand which demographic segments drive utilization and inform capacity planning for aging populations or specific groups**.

---

## 🎯 Acceptance Criteria

1. **Demographic utilization profiles created**
   - Admission rates calculated by age group and sex
   - Highest-utilizing demographics identified
   - Age-sex pyramids for admission rates
   - Temporal trends: how demographic utilization changed over time

2. **Growth analysis by demographics**
   - Fastest-growing demographic segments identified
   - Aging population impact quantified
   - Sex differentials analyzed
   - Cohort effects identified (if visible in data)

3. **Utilization concentration metrics**
   - Top demographic segments contributing to total admissions
   - Concentration indices: % of admissions from specific age/sex groups
   - Dependency ratios: elderly utilization vs working-age utilization

4. **Deliverables**
   - Output: `results/tables/utilization_demographic_profiles.csv`
   - Figures: Age-sex pyramids, demographic trends
   - Summary: High-burden populations report

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Seaborn
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#healthcare-utilization-patterns) - Demographic utilization patterns
- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objective-2) - Assess utilization patterns

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-003-US-02 (Clean utilization data - BLOCKING)
- **Data Sources**: `shared/data/3_interim/capacity_utilization_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Demographic Profiling
- [ ] Aggregate admission rates by age group and sex
- [ ] Identify highest-utilizing demographics
- [ ] Calculate demographic shares of total utilization
- [ ] Create age-sex stratified summary tables

### Temporal Trends
- [ ] Analyze utilization trends by demographic group
- [ ] Identify fastest-growing segments
- [ ] Compare 2006 vs 2020 demographic profiles
- [ ] Detect shifts in utilization patterns

### Concentration Analysis
- [ ] Calculate top demographics contributing to admissions
- [ ] Compute concentration indices
- [ ] Assess elderly vs working-age utilization
- [ ] Dependency ratio calculations

### Visualization
- [ ] Create age-sex pyramids for utilization
- [ ] Line charts: utilization trends by age group
- [ ] Heatmaps: age × year utilization matrix
- [ ] Save figures: `reports/figures/utilization_demographics_*.png/pdf`

### Testing & Documentation
- [ ] Unit tests for demographic calculations
- [ ] Validate aggregations
- [ ] Docstrings
- [ ] High-burden populations summary report

---

## 📌 Notes

**Polars Demographic Analysis**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/capacity_utilization_integrated.parquet")

# Demographic profiles
df_demo = (
    df.group_by(['year', 'age_group', 'sex']).agg([
        pl.col('admission_rate').sum().alias('total_admission_rate')
    ])
    .with_columns([
        pl.col('total_admission_rate').rank(descending=True).over('year').alias('rank')
    ])
)

# Identify high-burden demographics
high_burden = df_demo.filter(pl.col('rank') <= 5)
```

**Expected Findings**:
- **Elderly (65+)**: Highest admission rates
- **Sex patterns**: Males may have higher utilization for certain age groups
- **Growth**: Elderly utilization likely growing fastest (population aging)

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Analyse hospital admission patterns (2006–2020) by age group and sex to identify which demographic segments are high-burden users, how utilisation has shifted over 14 years, and which groups are growing fastest.

**Primary User Role**: Healthcare Service Planner understanding demand drivers

**Key Success Metric**: `results/tables/problem-statement-003/utilization_demographic_profiles_{timestamp}.csv` with per-demographic admission totals and growth rates; 4+ figures saved to `reports/figures/problem-statement-003/` including age-sex pyramid and utilization heatmap.

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream | US-02 output — BLOCKING |
| YoY growth helper | `capacity_trend_analysis.py` (US-03) | ✅ Reuse | `calculate_yoy_growth()` works on any numeric column |
| `workforce_plots.py` patterns | `shared/src/visualization/workforce_plots.py` | 🔧 Reference | Line-chart pattern for time series |

**New Components**:
- `src/analysis/demographic_utilization_analysis.py` — admission profiling, concentration metrics, dependency ratios
- `src/visualization/demographic_plots.py` — age-sex pyramids, heatmaps, trend lines

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/demographic_utilization_analysis.py
  Functions:
    aggregate_demographic_profiles(df, group_cols) -> pl.DataFrame
    rank_demographics_by_utilization(df, metric_col) -> pl.DataFrame
    calculate_demographic_growth_rates(df, group_cols) -> pl.DataFrame
    calculate_concentration_index(df, metric_col) -> dict
    calculate_dependency_ratio(df) -> pl.DataFrame

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/demographic_plots.py
  Functions:
    plot_age_sex_pyramid(df, year, output_path) -> None
    plot_utilization_heatmap(df, output_path) -> None
    plot_demographic_trends(df, groups, output_path) -> None

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/notebooks/02_utilization_demographics.ipynb

- Results:  results/tables/problem-statement-003/utilization_demographic_profiles_{ts}.csv
- Figures:  reports/figures/problem-statement-003/age_sex_pyramid_{ts}.png
            reports/figures/problem-statement-003/utilization_heatmap_{ts}.png
            reports/figures/problem-statement-003/demographic_trends_{ts}.png
```

---

### 4. Component Breakdown

#### `demographic_utilization_analysis.py`

**Responsibility**: Pure analytical functions on the hospital_admissions subset of the integrated parquet (columns: year, age_group, sex, admission_rate).

**Technical Constraints**:
- Data: 216 raw admission rows — eager loading, no lazy evaluation needed
- Memory: < 50 MB
- Age group categories come from raw data — do not hard-code, discover from data

---

### 5. Data Pipeline

```
shared/data/3_interim/capacity_utilization_integrated.parquet
  │  filter: source_table == 'utilization' OR use raw hospital_admissions CSV
  ▼
  Aggregate: group_by([year, age_group, sex]).agg(admission_rate.sum())
        │
        ├─▶ rank_demographics_by_utilization()
        │     → top 10 by mean admission_rate across all years
        │
        ├─▶ calculate_demographic_growth_rates()
        │     → YoY + total growth 2006→2020 per (age_group, sex)
        │
        ├─▶ calculate_concentration_index()
        │     → % of total admissions attributable to top quintile demographics
        │
        ├─▶ calculate_dependency_ratio()
        │     → elderly (65+) admission rate / working-age (15-64) admission rate
        │
        └─▶ Visualizations
              age_sex_pyramid (selected years: 2006, 2013, 2020)
              utilization_heatmap (age_group × year)
              trend lines for top 5 demographic groups
        │
        ▼
  results/tables/problem-statement-003/utilization_demographic_profiles_{ts}.csv
  reports/figures/problem-statement-003/
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementations

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/demographic_utilization_analysis.py
import polars as pl
from pathlib import Path
from datetime import datetime
from loguru import logger


def aggregate_demographic_profiles(
    df: pl.DataFrame,
    group_cols: list[str] = ["year", "age_group", "sex"],
    metric_col: str = "admission_rate",
) -> pl.DataFrame:
    """Aggregate admission rates by demographic group and year.

    Args:
        df: DataFrame with demographic columns and admission_rate.
        group_cols: Grouping dimensions.
        metric_col: Column to aggregate.

    Returns:
        DataFrame with total, mean, and count per group.
    """
    required = set(group_cols + [metric_col])
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"Missing columns: {missing}")

    result = df.group_by(group_cols).agg([
        pl.col(metric_col).sum().alias("total_admission_rate"),
        pl.col(metric_col).mean().alias("avg_admission_rate"),
        pl.len().alias("n_records"),
    ]).sort(group_cols)

    logger.info(f"aggregate_demographic_profiles: {result.height} demographic-year records")
    return result


def rank_demographics_by_utilization(
    df: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    top_n: int = 10,
) -> pl.DataFrame:
    """Identify highest-utilising demographic segments across all years.

    Args:
        df: Output from aggregate_demographic_profiles().
        metric_col: Metric to rank by.
        top_n: Number of top segments to return.

    Returns:
        DataFrame with rank column, sorted descending by mean metric.
    """
    mean_by_group = (
        df.group_by(["age_group", "sex"])
        .agg(pl.col(metric_col).mean().alias("mean_rate_all_years"))
        .sort("mean_rate_all_years", descending=True)
        .with_row_index("rank", offset=1)
    )

    top_segments = mean_by_group.head(top_n)
    logger.info(
        f"Top segment: {top_segments['age_group'][0]} ({top_segments['sex'][0]}) "
        f"— {top_segments['mean_rate_all_years'][0]:.1f} adm per 100k"
    )
    return top_segments


def calculate_demographic_growth_rates(
    df: pl.DataFrame,
    group_cols: list[str] = ["age_group", "sex"],
    metric_col: str = "total_admission_rate",
) -> pl.DataFrame:
    """Compute total growth (2006→2020) and YoY growth for each demographic group.

    Args:
        df: Aggregated demographic DataFrame with 'year' column.
        group_cols: Demographic grouping columns.
        metric_col: Column to compute growth for.

    Returns:
        DataFrame with start_value, end_value, absolute_growth, total_growth_pct,
        avg_yoy_pct per demographic group.
    """
    df_sorted = df.sort(group_cols + ["year"])

    # YoY within each demographic
    df_yoy = df_sorted.with_columns([
        pl.col(metric_col).shift(1).over(group_cols).alias("prev_year_rate"),
    ]).with_columns([
        ((pl.col(metric_col) - pl.col("prev_year_rate")) / pl.col("prev_year_rate") * 100)
        .alias("yoy_growth_pct"),
    ])

    # Period summary
    df_summary = df_sorted.group_by(group_cols).agg([
        pl.col(metric_col).first().alias("start_value"),
        pl.col(metric_col).last().alias("end_value"),
        pl.col("year").min().alias("start_year"),
        pl.col("year").max().alias("end_year"),
    ]).with_columns([
        (pl.col("end_value") - pl.col("start_value")).alias("absolute_growth"),
        pl.when(pl.col("start_value") > 0)
        .then((pl.col("end_value") / pl.col("start_value") - 1) * 100)
        .otherwise(None)
        .cast(pl.Float64)
        .alias("total_growth_pct"),
    ])

    logger.info(f"calculate_demographic_growth_rates: {df_summary.height} demographic groups")
    return df_summary


def calculate_concentration_index(
    df: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    top_pct: float = 20.0,
) -> dict:
    """Measure what share of total admissions come from the top X% of demographics.

    Args:
        df: Aggregated demographic profiles (one row per age_group/sex pair,
            using mean across years).
        metric_col: Admission metric column.
        top_pct: Percentile threshold for 'top' demographics (default 20%).

    Returns:
        Dict with total_admission_rate, top_share_pct, n_top_demographics.
    """
    mean_by_group = (
        df.group_by(["age_group", "sex"])
        .agg(pl.col(metric_col).mean().alias("mean_rate"))
        .sort("mean_rate", descending=True)
    )

    n_total = mean_by_group.height
    n_top = max(1, round(n_total * top_pct / 100))
    top_groups = mean_by_group.head(n_top)

    total_rate = mean_by_group["mean_rate"].sum()
    top_rate = top_groups["mean_rate"].sum()
    concentration_pct = top_rate / total_rate * 100 if total_rate > 0 else 0.0

    result = {
        "top_pct_threshold": top_pct,
        "n_total_demographics": n_total,
        "n_top_demographics": n_top,
        "top_share_pct": round(concentration_pct, 2),
    }
    logger.info(
        f"Concentration index: top {top_pct:.0f}% of demographics "
        f"account for {concentration_pct:.1f}% of admissions"
    )
    return result


def calculate_dependency_ratio(
    df: pl.DataFrame,
    elderly_age_groups: list[str] | None = None,
    working_age_groups: list[str] | None = None,
) -> pl.DataFrame:
    """Calculate elderly/working-age utilisation dependency ratio per year.

    Ratio > 1 means elderly groups utilise more than working-age groups.

    Args:
        df: Aggregated demographic profiles with 'year', 'age_group', 'total_admission_rate'.
        elderly_age_groups: Age group labels for elderly (auto-detected if None).
        working_age_groups: Age group labels for working age (auto-detected if None).

    Returns:
        DataFrame with [year, elderly_rate, working_age_rate, dependency_ratio].
    """
    # Auto-detect age groups containing '65', '70', '75', '80', '85'
    all_age_groups = df["age_group"].unique().to_list()

    if elderly_age_groups is None:
        elderly_age_groups = [g for g in all_age_groups
                              if any(str(a) in g for a in range(65, 100, 5))]
    if working_age_groups is None:
        working_age_groups = [g for g in all_age_groups
                              if any(str(a) in g for a in range(15, 65, 5))]

    elderly_by_year = (
        df.filter(pl.col("age_group").is_in(elderly_age_groups))
        .group_by("year")
        .agg(pl.col("total_admission_rate").sum().alias("elderly_rate"))
    )

    working_by_year = (
        df.filter(pl.col("age_group").is_in(working_age_groups))
        .group_by("year")
        .agg(pl.col("total_admission_rate").sum().alias("working_age_rate"))
    )

    dep_ratio = (
        elderly_by_year.join(working_by_year, on="year")
        .with_columns(
            pl.when(pl.col("working_age_rate") > 0)
            .then(pl.col("elderly_rate") / pl.col("working_age_rate"))
            .otherwise(None)
            .cast(pl.Float64)
            .alias("dependency_ratio")
        )
        .sort("year")
    )

    logger.info(
        f"Dependency ratio range: "
        f"{dep_ratio['dependency_ratio'].min():.2f} – "
        f"{dep_ratio['dependency_ratio'].max():.2f}"
    )
    return dep_ratio


def run_utilization_analysis(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    raw_admissions_path: str = "shared/data/1_raw/utilization/hospital_admissions.csv",
    problem_num: str = "003",
) -> dict[str, pl.DataFrame | dict]:
    """Orchestrate demographic utilization analysis and save results.

    Args:
        parquet_path: Path to integrated parquet (preferred).
        raw_admissions_path: Fallback raw admissions CSV.
        problem_num: Problem statement number for output dirs.

    Returns:
        Dict with all analysis DataFrames.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    # Load utilization data — try integrated parquet first
    try:
        df_raw = pl.read_parquet(parquet_path).filter(
            pl.col("source_table") == "utilization"
        )
        if df_raw.height == 0:
            raise ValueError("No utilization rows in integrated parquet")
    except Exception as e:
        logger.warning(f"Falling back to raw CSV: {e}")
        df_raw = pl.read_csv(raw_admissions_path)

    logger.info(f"Loaded {df_raw.height} utilization rows")

    results: dict = {}

    # Profiles
    df_profiles = aggregate_demographic_profiles(df_raw)
    results["profiles"] = df_profiles

    # Ranking
    df_ranked = rank_demographics_by_utilization(df_profiles)
    results["ranked"] = df_ranked

    # Growth rates
    df_growth = calculate_demographic_growth_rates(df_profiles)
    results["growth"] = df_growth

    # Concentration
    concentration = calculate_concentration_index(df_profiles)
    results["concentration"] = concentration
    logger.info(f"Concentration index: {concentration}")

    # Dependency ratio
    df_dep = calculate_dependency_ratio(df_profiles)
    results["dependency_ratio"] = df_dep

    # Save outputs
    output_path = tables_dir / f"utilization_demographic_profiles_{ts}.csv"
    df_profiles.write_csv(output_path)
    logger.info(f"✅ Profiles saved: {output_path}")
    print(f"✅ Saved: {output_path}")

    growth_path = tables_dir / f"utilization_demographic_growth_{ts}.csv"
    df_growth.write_csv(growth_path)
    logger.info(f"✅ Growth rates saved: {growth_path}")
    print(f"✅ Saved: {growth_path}")

    return results
```

#### 6.2 Visualization Module

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/demographic_plots.py
import polars as pl
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
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


def plot_age_sex_pyramid(
    df: pl.DataFrame,
    year: int,
    metric_col: str = "total_admission_rate",
    problem_num: str = "003",
) -> None:
    """Population-style pyramid of admission rates by age and sex.

    Args:
        df: Aggregated demographic profiles with age_group, sex, metric_col.
        year: Year to plot.
        metric_col: Admission rate column.
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    df_year = df.filter(pl.col("year") == year).to_pandas()
    age_groups = sorted(df_year["age_group"].unique())

    males = df_year[df_year["sex"].str.lower() == "male"].set_index("age_group")[metric_col]
    females = df_year[df_year["sex"].str.lower() == "female"].set_index("age_group")[metric_col]

    fig, ax = plt.subplots(figsize=(10, 8))
    y = np.arange(len(age_groups))
    bar_w = 0.4

    ax.barh(y, [-males.get(ag, 0) for ag in age_groups], height=bar_w,
            label="Male", color="#2196F3", alpha=0.85)
    ax.barh(y, [females.get(ag, 0) for ag in age_groups], height=bar_w,
            label="Female", color="#E91E63", alpha=0.85)

    ax.set_yticks(y)
    ax.set_yticklabels(age_groups)
    ax.axvline(0, color="black", linewidth=0.8)
    ax.set_xlabel("Admission Rate (per 100k population)")
    ax.set_title(f"Hospital Admission Rates by Age & Sex ({year})", fontweight="bold")
    ax.legend()
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"age_sex_pyramid_{year}_{ts}.png"
    _save_fig(fig, out)


def plot_utilization_heatmap(
    df: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    problem_num: str = "003",
) -> None:
    """Heatmap of admission rates: age_group (rows) × year (columns).

    Args:
        df: Aggregated demographic profiles.
        metric_col: Metric to display.
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pivot = (
        df.group_by(["year", "age_group"])
        .agg(pl.col(metric_col).sum())
        .sort(["age_group", "year"])
        .to_pandas()
        .pivot(index="age_group", columns="year", values=metric_col)
    )

    fig, ax = plt.subplots(figsize=(16, 8))
    sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.5,
                cbar_kws={"label": "Admission Rate (per 100k)"}, ax=ax)
    ax.set_title("Hospital Admission Rates: Age Group × Year", fontweight="bold")
    ax.set_xlabel("Year")
    ax.set_ylabel("Age Group")
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"utilization_heatmap_{ts}.png"
    _save_fig(fig, out)


def plot_demographic_trends(
    df: pl.DataFrame,
    top_groups: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    problem_num: str = "003",
) -> None:
    """Line trends for the top N demographic groups over time.

    Args:
        df: Aggregated demographic profiles.
        top_groups: DataFrame with age_group and sex columns for top N groups.
        metric_col: Metric to plot.
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    fig, ax = plt.subplots(figsize=(14, 7))

    for row in top_groups.iter_rows(named=True):
        subset = df.filter(
            (pl.col("age_group") == row["age_group"]) &
            (pl.col("sex").str.to_lowercase() == row["sex"].lower())
        ).sort("year").to_pandas()

        if subset.empty:
            continue
        label = f"{row['age_group']} ({row['sex']})"
        ax.plot(subset["year"], subset[metric_col], marker="o", label=label, linewidth=2)

    ax.set_xlabel("Year")
    ax.set_ylabel("Admission Rate (per 100k)")
    ax.set_title("Utilization Trends for High-Burden Demographics (2006–2020)", fontweight="bold")
    ax.legend(bbox_to_anchor=(1.05, 1), loc="upper left")
    ax.grid(True, alpha=0.3)
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"demographic_trends_{ts}.png"
    _save_fig(fig, out)
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 matplotlib>=3.8.0 seaborn>=0.13.0 numpy>=1.24.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

**Validated features** (data available in integrated parquet):

| Feature | Formula | Required Fields | Expected Range |
|---------|---------|---------|---------|
| Total admission rate per demographic | `sum(admission_rate)` over age/sex/year | `age_group`, `sex`, `admission_rate`, `year` | 0 – 5,000 /100k |
| YoY growth per demographic | `(t - t-1)/t-1 × 100` | Above, sorted by year | −50% to +50% |
| Dependency ratio | `elderly_rate / working_age_rate` | Age group labelling | 0.5 – 5.0 |
| Concentration index | `top_20%_rate / total_rate × 100` | All demographics | 40% – 80% |

**Rejected** (not computable):
- Care intensity (discharge data not available)
- Length of stay by demographic (not in dataset)

---

### 8. Testing Strategy

```python
# tests/unit/test_demographic_utilization_analysis.py
import polars as pl
import pytest
from src.analysis.demographic_utilization_analysis import (
    aggregate_demographic_profiles,
    rank_demographics_by_utilization,
    calculate_concentration_index,
    calculate_dependency_ratio,
)


@pytest.fixture
def sample_util_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2006, 2006, 2006, 2007, 2007, 2007], dtype=pl.Int32),
        "age_group": ["65-74", "65-74", "15-24", "65-74", "65-74", "15-24"],
        "sex": ["Male", "Female", "Male", "Male", "Female", "Male"],
        "admission_rate": [500.0, 450.0, 80.0, 520.0, 470.0, 85.0],
    })


def test_aggregate_profiles_row_count(sample_util_df):
    result = aggregate_demographic_profiles(sample_util_df)
    assert result.height == 4  # 2 years × (65-74 M, 65-74 F, 15-24 M)... actually 3 groups × 2 years = 6
    # More precisely: unique (year, age_group, sex) combinations = 6


def test_aggregate_profiles_totals(sample_util_df):
    result = aggregate_demographic_profiles(sample_util_df)
    row_65_74_m_2006 = result.filter(
        (pl.col("year") == 2006) & (pl.col("age_group") == "65-74") & (pl.col("sex") == "Male")
    )
    assert row_65_74_m_2006["total_admission_rate"][0] == pytest.approx(500.0)


def test_rank_demographics_top_segment_is_highest(sample_util_df):
    profiles = aggregate_demographic_profiles(sample_util_df)
    ranked = rank_demographics_by_utilization(profiles)
    assert ranked["rank"][0] == 1
    assert ranked["mean_rate_all_years"][0] >= ranked["mean_rate_all_years"][-1]


def test_concentration_index_returns_valid_percentage(sample_util_df):
    profiles = aggregate_demographic_profiles(sample_util_df)
    concentration = calculate_concentration_index(profiles)
    assert 0 < concentration["top_share_pct"] <= 100


def test_dependency_ratio_elderly_over_working_age(sample_util_df):
    profiles = aggregate_demographic_profiles(sample_util_df)
    dep = calculate_dependency_ratio(
        profiles,
        elderly_age_groups=["65-74"],
        working_age_groups=["15-24"],
    )
    # 65-74 rates >> 15-24 rates, so ratio should be >> 1
    assert dep["dependency_ratio"].min() > 1.0


def test_missing_column_raises_value_error():
    df = pl.DataFrame({"year": [2006], "age_group": ["65-74"]})
    with pytest.raises(ValueError, match="Missing columns"):
        aggregate_demographic_profiles(df)
```

---

### 9. Implementation Steps

#### Phase 1: Setup
- [ ] Create `src/analysis/demographic_utilization_analysis.py`
- [ ] Create `src/visualization/demographic_plots.py`
- [ ] Create `reports/figures/problem-statement-003/` and `results/tables/problem-statement-003/` directories

#### Phase 2: Analysis Functions
- [ ] Implement all 5 analysis functions with logging
- [ ] Run `run_utilization_analysis()` interactively to verify output
- [ ] Inspect top 10 demographics: confirm elderly (65+) dominate rankings

#### Phase 3: Visualizations
- [ ] Generate age-sex pyramids for 2006, 2013, 2020
- [ ] Generate utilization heatmap
- [ ] Generate trend lines for top 5 demographic groups
- [ ] Confirm all figures saved with timestamps

#### Phase 4: Output Validation
- [ ] Verify profiles CSV row count = age_groups × sex_groups × years
- [ ] Check dependency ratio trend (should increase 2006→2020 due to aging)
- [ ] Concentration index: document and interpret

#### Phase 5: Tests
- [ ] Write test file per Section 8
- [ ] `pytest tests/unit/test_demographic_utilization_analysis.py -v --cov`
- [ ] Achieve ≥ 80% coverage

---

### 10. Code Generation Order

1. `src/analysis/demographic_utilization_analysis.py`
2. `tests/unit/test_demographic_utilization_analysis.py`
3. `src/visualization/demographic_plots.py`
4. `notebooks/02_utilization_demographics.ipynb`

---

### 11. Data Quality & Validation

**Input checks**:
- `admission_rate` must be non-negative (warn on any negative values)
- `age_group` and `sex` must have at most 20 and 2 unique values respectively
- Year range: 2006 ≤ year ≤ 2020

**Output checks**:
- Profiles: no null `total_admission_rate` for valid groups
- Growth: first-year values for each group have null `yoy_growth_pct` (expected)
- Dependency ratio: monotonically increasing across years (warn if decreasing)
- Concentration: top 20% should contribute ≥ 40% of total (flag if < 30%)

---

**Implementation Plan Checklist**
- [x] Feature overview with success metric
- [x] Component reuse strategy
- [x] Affected files with CREATE/MODIFY and function signatures
- [x] Complete executable functions with type hints and docstrings
- [x] Visualization module with save-to-disk pattern
- [x] Domain feature engineering table with rejected features documented
- [x] Test assertions with expected values
- [x] Phased implementation steps
- [x] Data quality validation for inputs and outputs
