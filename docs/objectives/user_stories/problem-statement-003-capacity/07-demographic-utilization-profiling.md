# Profile High-Burden Demographic Segments (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-003-US-07  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P1 (High)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Service Planner targeting interventions**,  
I want **to profile demographic segments with disproportionately high utilization to understand their specific capacity needs**,  
So that **I can design targeted capacity expansions and care programs for high-burden populations (e.g., elderly, chronic disease patients)**.

---

## 🎯 Acceptance Criteria

1. **High-burden segments identified**
   - Top 10 demographic groups by admission rate
   - Segments with >2x average utilization rate
   - Growth rates for high-burden segments
   - Demographic concentration metrics

2. **Segment profiling completed**
   - Utilization characteristics per segment
   - Temporal trends for each high-burden group
   - Facility type preferences/needs by demographic
   - Care intensity indicators (if data available)

3. **Recommendations generated**
   - Targeted capacity needs for high-burden segments
   - Specialized service requirements
   - Priority populations for intervention programs

4. **Deliverables**
   - Output: `results/tables/high_burden_demographic_profiles.csv`
   - Figures: Demographic burden distribution, segment trends
   - Policy brief: Targeted capacity recommendations

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Seaborn
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#demographic-health-patterns) - Demographic utilization patterns
- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objective-2) - Demographic utilization drivers

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `scikit-learn>=1.3.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-003-US-04 (Utilization demographics - BLOCKING)
- **Data Sources**: `shared/data/3_interim/capacity_utilization_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### High-Burden Identification
- [ ] Calculate average utilization rate across all demographics
- [ ] Identify segments with >2x average rate
- [ ] Rank demographics by absolute utilization
- [ ] Calculate utilization concentration (% from top segments)

### Segment Profiling
- [ ] Profile each high-burden segment: age, sex, trends
- [ ] Analyze growth rates for high-burden groups
- [ ] Cross-reference with facility utilization patterns
- [ ] Assess care intensity needs

### Recommendation Development
- [ ] Map demographic needs to facility types
- [ ] Estimate capacity requirements for high-burden groups
- [ ] Identify specialized service needs (geriatric care, chronic disease)
- [ ] Priority ranking for interventions

### Visualization
- [ ] Demographic burden distribution charts
- [ ] Trend lines for high-burden segments
- [ ] Concentration curve (Lorenz-style)
- [ ] Save figures: `reports/figures/demographic_profiling_*.png/pdf`

### Policy Brief
- [ ] Executive summary: key high-burden segments
- [ ] Detailed profiles for top 5 segments
- [ ] Capacity and service recommendations
- [ ] Implementation priorities

### Testing & Documentation
- [ ] Unit tests for profiling logic
- [ ] Validate burden calculations
- [ ] Docstrings
- [ ] Policy brief generation

---

## 📌 Notes

**Polars Profiling Example**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/capacity_utilization_integrated.parquet")

# Calculate average utilization
avg_rate = df['admission_rate'].mean()

# Identify high-burden segments
df_high_burden = (
    df.filter(pl.col('admission_rate') > 2 * avg_rate)
    .sort('admission_rate', descending=True)
)

# Profile top segments
top_segments = df_high_burden.head(10)
```

**Expected High-Burden Segments**:
- **Elderly (75+)**: Likely highest utilization
- **Elderly males**: May have even higher rates
- **Middle-aged chronic disease patients**: Growing segment

**Capacity Recommendations**:
- **Geriatric care facilities**: Specialized elderly care
- **Chronic disease management**: Outpatient clinics
- **Rehabilitation**: Post-acute step-down care

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Profile demographic segments with disproportionately high hospital utilisation (> 2× average admission rate) to generate targeted, evidence-based capacity expansion recommendations for service planners.

**Primary User Role**: Healthcare Service Planner targeting interventions

**Key Success Metric**:
- `results/tables/problem-statement-003/high_burden_demographic_profiles_{timestamp}.csv` listing top 10 high-burden demographics with growth trajectories
- `results/tables/problem-statement-003/capacity_recommendations_{timestamp}.csv` with actionable facility expansion priorities per segment
- 3+ figures saved to `reports/figures/problem-statement-003/`

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| `demographic_utilization_analysis.py` | `src/analysis/` (US-04) | ✅ Reuse | `aggregate_demographic_profiles()`, `rank_demographics_by_utilization()` — BLOCKING |
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream | US-02 — BLOCKING |

**New Components**:
- `src/analysis/demographic_burden_profiling.py` — high-burden identification, concentration curve, segment growth profiling, recommendation engine
- `src/visualization/burden_plots.py` — Lorenz-style concentration curve, segment trend comparison chart

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/demographic_burden_profiling.py
  Functions:
    identify_high_burden_segments(df, threshold_multiplier) -> pl.DataFrame
    calculate_lorenz_curve_data(df, metric_col) -> pl.DataFrame
    profile_segment_growth(df, top_segments) -> pl.DataFrame
    map_segments_to_facility_needs(high_burden_df) -> pl.DataFrame
    generate_capacity_recommendations(profiled_df) -> pl.DataFrame
    run_burden_profiling(parquet_path, problem_num) -> dict

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/burden_plots.py
  Functions:
    plot_burden_distribution(df, output_path) -> None
    plot_lorenz_curve(lorenz_data, output_path) -> None
    plot_segment_growth_trends(df, top_segments, output_path) -> None

- Results: results/tables/problem-statement-003/high_burden_demographic_profiles_{ts}.csv
           results/tables/problem-statement-003/capacity_recommendations_{ts}.csv
- Figures:  reports/figures/problem-statement-003/burden_distribution_{ts}.png
            reports/figures/problem-statement-003/lorenz_curve_{ts}.png
            reports/figures/problem-statement-003/segment_growth_trends_{ts}.png

- [MODIFY] notebooks/05_demographic_burden_profiling.ipynb — NEW
```

---

### 4. Component Breakdown

#### `demographic_burden_profiling.py`

**Responsibility**: Build on US-04 demographic profiles to identify and characterise high-burden segments. Generate structured recommendations linking segments to facility types.

**Technical Constraints**:
- Memory: < 50 MB
- `threshold_multiplier` must be configurable (default 2.0 from config)
- Handle edge case: if no segment exceeds threshold, return empty DataFrame with warning
- Recommendations are rule-based (no ML needed — insufficient data for supervised learning)

---

### 5. Data Pipeline

```
US-04 outputs OR integrated parquet → filter utilization rows
  │
  ▼  aggregate_demographic_profiles() [reuse from US-04]
  │     → (year, age_group, sex, total_admission_rate)
  │
  ├─▶ identify_high_burden_segments()
  │     avg_rate = mean(total_admission_rate) across all groups, all years
  │     high_burden = filter(total_admission_rate > avg_rate × threshold_multiplier)
  │     → ranked DataFrame of high-burden (age_group, sex) pairs
  │
  ├─▶ calculate_lorenz_curve_data()
  │     sort by admission_rate ascending
  │     cumulative admissions / total admissions
  │     cumulative population % on x-axis
  │     → Lorenz curve data for Gini coefficient calculation
  │
  ├─▶ profile_segment_growth()
  │     for each high-burden segment, compute YoY growth trends
  │     → growth trajectories 2006→2020
  │
  ├─▶ map_segments_to_facility_needs()
  │     rule-based mapping: age 65+ → long_term_care, geriatric acute
  │                          age 45-64 → chronic disease outpatient, day surgery
  │                          younger with high rates → acute specialised
  │
  ├─▶ generate_capacity_recommendations()
  │     combine burden magnitude + growth trajectory + facility mapping
  │     → ranked recommendations with priority scores
  │
  └─▶ Save CSVs + generate 3 figures
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementations

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/demographic_burden_profiling.py
import polars as pl
from pathlib import Path
from datetime import datetime
from loguru import logger

# Configurable thresholds (also in config.yml)
HIGH_BURDEN_MULTIPLIER: float = 2.0  # >2× average = high burden
ELDERLY_AGE_MARKERS: list[str] = ["65", "70", "75", "80", "85", "90"]
WORKING_AGE_MARKERS: list[str] = ["15", "20", "25", "30", "35", "40", "45", "50", "55", "60"]


def identify_high_burden_segments(
    df: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    threshold_multiplier: float = HIGH_BURDEN_MULTIPLIER,
    group_cols: list[str] = ["age_group", "sex"],
) -> pl.DataFrame:
    """Identify demographic segments with admission rates > threshold × average.

    The average is computed across all demographic groups and all years.
    Segments are sorted by mean admission rate (descending).

    Args:
        df: Aggregated demographic profiles (year, age_group, sex, metric_col).
        metric_col: Column to threshold on.
        threshold_multiplier: Multiples of average rate to define high burden.
        group_cols: Demographic group columns.

    Returns:
        DataFrame with high-burden segments and columns:
        group_cols + [mean_rate, threshold_rate, ratio_to_average, rank].

    Raises:
        ValueError: If no segments identified and threshold is unreasonably high.
    """
    if metric_col not in df.columns:
        raise ValueError(f"Column '{metric_col}' not found. Available: {df.columns}")

    # Compute per-group mean across all years
    mean_by_group = (
        df.group_by(group_cols)
        .agg(pl.col(metric_col).mean().alias("mean_rate"))
    )

    overall_avg = mean_by_group["mean_rate"].mean()
    threshold_rate = overall_avg * threshold_multiplier

    high_burden = (
        mean_by_group
        .filter(pl.col("mean_rate") > threshold_rate)
        .with_columns([
            pl.lit(overall_avg).alias("overall_avg_rate"),
            pl.lit(threshold_rate).alias("threshold_rate"),
            (pl.col("mean_rate") / overall_avg).alias("ratio_to_average"),
        ])
        .sort("mean_rate", descending=True)
        .with_row_index("rank", offset=1)
    )

    if high_burden.is_empty():
        logger.warning(
            f"No segments exceed {threshold_multiplier}× average "
            f"({overall_avg:.1f}). Consider lowering threshold_multiplier."
        )
    else:
        logger.info(
            f"identify_high_burden_segments: {high_burden.height} segments above "
            f"{threshold_rate:.1f} (threshold: {threshold_multiplier}× avg={overall_avg:.1f})"
        )

    return high_burden


def calculate_lorenz_curve_data(
    df: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    group_cols: list[str] = ["age_group", "sex"],
) -> pl.DataFrame:
    """Compute Lorenz curve data for admission rate concentration analysis.

    The curve plots cumulative population share (x) vs cumulative admission
    share (y). Greater deviation from 45° diagonal → higher concentration.

    Args:
        df: Aggregated demographic profiles.
        metric_col: Utilisation metric.
        group_cols: Demographic group columns.

    Returns:
        DataFrame with: rank, cumulative_population_pct, cumulative_admission_pct.
    """
    mean_by_group = (
        df.group_by(group_cols)
        .agg(pl.col(metric_col).mean().alias("mean_rate"))
        .sort("mean_rate")
    )

    n = mean_by_group.height
    total_rate = mean_by_group["mean_rate"].sum()

    lorenz = mean_by_group.with_row_index("rank", offset=1).with_columns([
        (pl.col("rank") / n * 100).alias("cumulative_population_pct"),
        (pl.col("mean_rate").cum_sum() / total_rate * 100).alias("cumulative_admission_pct"),
    ])

    # Gini coefficient approximation
    gini = 1 - 2 * (lorenz["cumulative_admission_pct"].sum() / (100 * n))
    logger.info(f"Lorenz curve computed. Approximate Gini coefficient: {gini:.3f}")

    return lorenz


def profile_segment_growth(
    df: pl.DataFrame,
    top_segments: pl.DataFrame,
    metric_col: str = "total_admission_rate",
    group_cols: list[str] = ["age_group", "sex"],
) -> pl.DataFrame:
    """Analyse temporal growth trajectories for high-burden segments.

    Args:
        df: Full time-series demographic profiles with 'year' column.
        top_segments: High-burden segments from identify_high_burden_segments().
        metric_col: Utilisation metric.
        group_cols: Grouping columns for filtering.

    Returns:
        DataFrame with YoY growth and total period growth per high-burden segment.
    """
    # Build filter keys
    segment_keys = set(
        zip(top_segments[group_cols[0]].to_list(), top_segments[group_cols[1]].to_list())
    )

    df_filtered = df.filter(
        pl.struct(group_cols).map_elements(
            lambda row: (row[group_cols[0]], row[group_cols[1]]) in segment_keys,
            return_dtype=pl.Boolean,
        )
    )

    df_growth = df_filtered.sort(group_cols + ["year"]).with_columns([
        pl.col(metric_col).shift(1).over(group_cols).alias("prev_year_rate"),
    ]).with_columns([
        pl.when(pl.col("prev_year_rate").is_not_null() & (pl.col("prev_year_rate") > 0))
        .then(
            (pl.col(metric_col) - pl.col("prev_year_rate")) / pl.col("prev_year_rate") * 100
        )
        .otherwise(None)
        .cast(pl.Float64)
        .alias("yoy_growth_pct"),
    ])

    logger.info(f"profile_segment_growth: {df_growth.height} segment-year records")
    return df_growth


def map_segments_to_facility_needs(
    high_burden_df: pl.DataFrame,
) -> pl.DataFrame:
    """Apply domain-knowledge rules to map high-burden demographics to facility types.

    Rules:
      - Age 65+ → long_term_care, acute_care (geriatric)
      - Age 45-64 → community_hospital (chronic disease), day surgery
      - Age < 45 with high admissions → acute_care (emergency/maternity)
      - Female high-admissions → maternity/women's health (note in output)

    Args:
        high_burden_df: Output from identify_high_burden_segments().

    Returns:
        DataFrame with added recommended_facility_type and intervention_type columns.
    """
    def classify_facility(age_group: str, sex: str) -> str:
        is_elderly = any(marker in age_group for marker in ELDERLY_AGE_MARKERS)
        is_working_age = any(marker in age_group for marker in WORKING_AGE_MARKERS)
        if is_elderly:
            return "long_term_care, acute_care (geriatric)"
        elif is_working_age:
            return "community_hospital, outpatient_clinics"
        else:
            return "acute_care (emergency, paediatric)"

    def classify_intervention(age_group: str, sex: str) -> str:
        is_elderly = any(marker in age_group for marker in ELDERLY_AGE_MARKERS)
        if is_elderly:
            return "Expand geriatric care, step-down facilities"
        elif sex.lower() == "female" and not is_elderly:
            return "Women's health, chronic disease management"
        else:
            return "Chronic disease management, preventive care"

    facility_col = pl.Series(
        "recommended_facility_type",
        [classify_facility(ag, sex)
         for ag, sex in zip(
             high_burden_df["age_group"].to_list(),
             high_burden_df["sex"].to_list()
         )],
    )
    intervention_col = pl.Series(
        "intervention_type",
        [classify_intervention(ag, sex)
         for ag, sex in zip(
             high_burden_df["age_group"].to_list(),
             high_burden_df["sex"].to_list()
         )],
    )

    return high_burden_df.with_columns([
        facility_col,
        intervention_col,
    ])


def generate_capacity_recommendations(
    profiled_df: pl.DataFrame,
) -> pl.DataFrame:
    """Consolidate segment profiles into ranked capacity expansion recommendations.

    Args:
        profiled_df: Output from map_segments_to_facility_needs().

    Returns:
        DataFrame with: rank, age_group, sex, mean_rate, ratio_to_average,
        recommended_facility_type, intervention_type, recommendation_priority.
    """
    recommendations = profiled_df.with_columns([
        pl.when(pl.col("ratio_to_average") >= 3.0)
        .then(pl.lit("Critical"))
        .when(pl.col("ratio_to_average") >= 2.5)
        .then(pl.lit("High"))
        .otherwise(pl.lit("Moderate"))
        .cast(pl.Categorical)
        .alias("recommendation_priority"),
    ]).sort("ratio_to_average", descending=True)

    logger.info(
        f"generate_capacity_recommendations: {recommendations.height} total, "
        f"Critical={recommendations.filter(pl.col('recommendation_priority') == 'Critical').height}"
    )
    return recommendations


def run_burden_profiling(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    raw_admissions_path: str = "shared/data/1_raw/utilization/hospital_admissions.csv",
    problem_num: str = "003",
) -> dict:
    """Orchestrate full demographic burden profiling pipeline.

    Args:
        parquet_path: Integrated dataset path.
        raw_admissions_path: Fallback raw admissions CSV.
        problem_num: Problem statement number for output directory.

    Returns:
        Dict with keys: high_burden, lorenz, segment_growth, recommendations.
    """
    from src.analysis.demographic_utilization_analysis import aggregate_demographic_profiles

    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    # Load data
    try:
        df_raw = pl.read_parquet(parquet_path).filter(
            pl.col("source_table") == "utilization"
        )
        if df_raw.is_empty():
            raise ValueError("No utilization rows in integrated parquet")
    except Exception as e:
        logger.warning(f"Falling back to raw CSV: {e}")
        df_raw = pl.read_csv(raw_admissions_path)

    # Aggregate profiles
    df_profiles = aggregate_demographic_profiles(df_raw)

    results: dict = {}

    # High-burden identification
    high_burden = identify_high_burden_segments(df_profiles)
    results["high_burden"] = high_burden

    # Lorenz curve
    lorenz = calculate_lorenz_curve_data(df_profiles)
    results["lorenz"] = lorenz

    # Segment growth
    if not high_burden.is_empty():
        segment_growth = profile_segment_growth(df_profiles, high_burden)
        results["segment_growth"] = segment_growth

        # Map to facility needs
        mapped = map_segments_to_facility_needs(high_burden)
        recommendations = generate_capacity_recommendations(mapped)
        results["recommendations"] = recommendations

        # Save outputs
        burden_path = tables_dir / f"high_burden_demographic_profiles_{ts}.csv"
        high_burden.write_csv(burden_path)
        logger.info(f"✅ Burden profiles saved: {burden_path}")
        print(f"✅ Saved: {burden_path}")

        rec_path = tables_dir / f"capacity_recommendations_{ts}.csv"
        recommendations.write_csv(rec_path)
        logger.info(f"✅ Recommendations saved: {rec_path}")
        print(f"✅ Saved: {rec_path}")

    return results
```

#### 6.2 Visualization Module

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/burden_plots.py
import polars as pl
import matplotlib.pyplot as plt
import numpy as np
from pathlib import Path
from datetime import datetime
from loguru import logger

_FIG_DIR = "reports/figures/problem-statement-{num}"


def _save_fig(fig: plt.Figure, path: Path) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(path, dpi=300, bbox_inches="tight")
    logger.info(f"✅ Figure saved: {path}")
    print(f"✅ Saved: {path}")
    plt.show()
    plt.close(fig)


def plot_burden_distribution(
    df_high_burden: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Horizontal bar chart of admission rates for high-burden segments.

    Args:
        df_high_burden: Output from identify_high_burden_segments().
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_high_burden.sort("mean_rate").to_pandas()
    labels = pdf["age_group"] + " (" + pdf["sex"] + ")"

    fig, ax = plt.subplots(figsize=(10, max(6, len(labels) * 0.5 + 2)))
    bars = ax.barh(labels, pdf["mean_rate"], color="#E53935", alpha=0.8)
    ax.axvline(pdf["threshold_rate"].iloc[0], color="orange", linewidth=2,
               linestyle="--", label=f"Threshold ({pdf['threshold_rate'].iloc[0]:.0f})")
    ax.axvline(pdf["overall_avg_rate"].iloc[0], color="green", linewidth=2,
               linestyle=":", label=f"Average ({pdf['overall_avg_rate'].iloc[0]:.0f})")
    ax.bar_label(bars, fmt="%.0f", padding=3, fontsize=9)
    ax.set_xlabel("Mean Admission Rate (per 100k)")
    ax.set_title("High-Burden Demographic Segments", fontweight="bold")
    ax.legend()
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"burden_distribution_{ts}.png"
    _save_fig(fig, out)


def plot_lorenz_curve(
    lorenz_data: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Lorenz curve for admission rate concentration.

    Shows x = cumulative % of demographics, y = cumulative % of admissions.
    Diagonal line = perfect equality.

    Args:
        lorenz_data: Output from calculate_lorenz_curve_data().
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = lorenz_data.to_pandas()

    fig, ax = plt.subplots(figsize=(8, 8))
    ax.plot(pdf["cumulative_population_pct"], pdf["cumulative_admission_pct"],
            color="#1565C0", linewidth=2.5, label="Lorenz Curve")
    ax.plot([0, 100], [0, 100], color="gray", linewidth=1.5,
            linestyle="--", label="Perfect Equality")

    # Shade Gini area
    ax.fill_between(
        pdf["cumulative_population_pct"],
        pdf["cumulative_admission_pct"],
        pdf["cumulative_population_pct"],
        alpha=0.15, color="#1565C0",
    )

    ax.set_xlabel("Cumulative % of Demographic Groups")
    ax.set_ylabel("Cumulative % of Total Admissions")
    ax.set_title("Lorenz Curve — Hospital Admission Rate Concentration", fontweight="bold")
    ax.legend()
    ax.set_xlim(0, 100)
    ax.set_ylim(0, 100)
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"lorenz_curve_{ts}.png"
    _save_fig(fig, out)
```

---

### 7. Domain-Driven Feature Engineering

**Validated features**:

| Feature | Formula | Rationale | Available Data |
|---------|---------|---------|---------|
| `mean_rate` | `mean(admission_rate)` per demographic | Stable indicator across years | ✅ US-04 output |
| `ratio_to_average` | `mean_rate / overall_avg` | Relative burden magnitude | ✅ Derived |
| `yoy_growth_pct` | YoY % change per segment | Identifies accelerating burden | ✅ Time-series |
| Lorenz / Gini | Cumulative share analysis | Inequality of utilisation distribution | ✅ Derived |
| `recommendation_priority` | Rule-based from ratio | Actionable for planners | ✅ Rule-based |

**Explicitly rejected**:
- Comorbidity index — not in dataset
- Care intensity score — no discharge/procedure data
- Geographic concentration — no facility-level geographic data
- Predicted future utilisation per segment — insufficient time-series for reliable ML

---

### 8. Testing Strategy

```python
# tests/unit/test_demographic_burden_profiling.py
import polars as pl
import pytest
from src.analysis.demographic_burden_profiling import (
    identify_high_burden_segments,
    calculate_lorenz_curve_data,
    classify_gap_severity,
    generate_capacity_recommendations,
    map_segments_to_facility_needs,
)


@pytest.fixture
def sample_profiles_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2010, 2010, 2010, 2010], dtype=pl.Int32),
        "age_group": ["75-84", "65-74", "25-34", "15-24"],
        "sex": ["Male", "Female", "Male", "Female"],
        "total_admission_rate": [800.0, 700.0, 100.0, 80.0],
    })


def test_identify_high_burden_threshold_correct(sample_profiles_df):
    # avg = (800+700+100+80)/4 = 420, threshold at 2× = 840
    # Only 800 > 840 is False → but 800 close — adjust fixture
    result = identify_high_burden_segments(sample_profiles_df, threshold_multiplier=1.5)
    # avg=420, threshold=630 → only 75-84(800) and 65-74(700) are high burden
    assert result.height == 2
    assert "75-84" in result["age_group"].to_list()
    assert "65-74" in result["age_group"].to_list()


def test_identify_high_burden_empty_when_threshold_too_high(sample_profiles_df):
    result = identify_high_burden_segments(sample_profiles_df, threshold_multiplier=10.0)
    assert result.is_empty()


def test_lorenz_curve_endpoints(sample_profiles_df):
    lorenz = calculate_lorenz_curve_data(sample_profiles_df)
    # Last cumulative_population_pct should be 100
    assert lorenz["cumulative_population_pct"].max() == pytest.approx(100.0)
    # Last cumulative_admission_pct should be 100
    assert lorenz["cumulative_admission_pct"].max() == pytest.approx(100.0, rel=1e-3)


def test_lorenz_curve_monotonic_increasing(sample_profiles_df):
    lorenz = calculate_lorenz_curve_data(sample_profiles_df)
    pct = lorenz["cumulative_admission_pct"].to_list()
    assert all(pct[i] <= pct[i + 1] for i in range(len(pct) - 1))


def test_map_elderly_to_long_term_care():
    df = identify_high_burden_segments(
        pl.DataFrame({
            "year": pl.Series([2010, 2010], dtype=pl.Int32),
            "age_group": ["75-84", "75-84"],
            "sex": ["Male", "Female"],
            "total_admission_rate": [900.0, 850.0],
        }),
        threshold_multiplier=0.5,
    )
    mapped = map_segments_to_facility_needs(df)
    assert "long_term_care" in mapped["recommended_facility_type"][0]


def test_recommendations_have_correct_priority_levels():
    df = pl.DataFrame({
        "age_group": ["75-84"],
        "sex": ["Male"],
        "mean_rate": [900.0],
        "ratio_to_average": [4.5],
        "recommended_facility_type": ["long_term_care"],
        "intervention_type": ["Expand geriatric care"],
    })
    recs = generate_capacity_recommendations(df)
    assert recs["recommendation_priority"][0] == "Critical"
```

---

### 9. Implementation Steps

#### Phase 1: Setup
- [ ] Create `src/analysis/demographic_burden_profiling.py`
- [ ] Create `src/visualization/burden_plots.py`

#### Phase 2: Core Analysis
- [ ] Implement `identify_high_burden_segments()` — verify threshold logic against manual calculation
- [ ] Implement `calculate_lorenz_curve_data()` — verify endpoints = 100
- [ ] Implement `profile_segment_growth()` — build on output from US-04
- [ ] Implement `map_segments_to_facility_needs()` — rule-based mapping
- [ ] Implement `generate_capacity_recommendations()` with priority levels

#### Phase 3: Orchestration
- [ ] Run `run_burden_profiling()` end-to-end
- [ ] Verify top 3 recommended segments are elderly groups

#### Phase 4: Visualizations
- [ ] Generate burden distribution horizontal bar chart
- [ ] Generate Lorenz curve
- [ ] Generate segment growth trend lines for top 5 segments
- [ ] Save all with timestamps

#### Phase 5: Policy Brief Notebook
- [ ] Create `notebooks/05_demographic_burden_profiling.ipynb`
- [ ] Document top segments, growth trajectories, recommendations in markdown cells

#### Phase 6: Tests
- [ ] Write `tests/unit/test_demographic_burden_profiling.py`
- [ ] `pytest tests/unit/test_demographic_burden_profiling.py -v --cov`
- [ ] Coverage ≥ 80%

---

### 10. Code Generation Order

1. `src/analysis/demographic_burden_profiling.py`
2. `tests/unit/test_demographic_burden_profiling.py`
3. `src/visualization/burden_plots.py`
4. `notebooks/05_demographic_burden_profiling.ipynb`

---

### 11. Data Quality & Validation

**Input**:
- Assert profiles DataFrame has `age_group`, `sex`, `total_admission_rate`
- Warn if < 6 distinct age groups (may indicate data truncation)

**High-burden output**:
- Warn if > 50% of demographics flagged as high-burden (threshold too low)
- Assert `ratio_to_average >= threshold_multiplier` for all returned rows

**Lorenz curve**:
- Monotonically increasing: `cumulative_admission_pct[i] ≤ cumulative_admission_pct[i+1]`
- Final values must equal 100% (tolerance ± 0.01%)

**Recommendations**:
- Must have at least 1 "Critical" priority when data includes elderly segments
- `recommendation_priority` ∈ {Critical, High, Moderate}

---

**Implementation Plan Checklist**
- [x] Feature overview with quantitative success metrics
- [x] Reuse strategy specifying US-04 as BLOCKING dependency
- [x] All affected files with CREATE indicators and function signatures
- [x] Complete executable function implementations
- [x] Lorenz curve with Gini coefficient calculation
- [x] Rule-based recommendation engine mapped to facility types
- [x] Tests covering threshold logic, Lorenz endpoints, mapping rules
- [x] Edge cases: empty high-burden result, threshold too high
- [x] Explicit list of rejected features (comorbidity, care intensity)
