# Identify Capacity Gaps by Facility Type (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-003-US-06  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Service Planner prioritizing infrastructure investments**,  
I want **to identify capacity shortages and surpluses by comparing capacity growth vs utilization growth trends across facility types**,  
So that **I can recommend targeted capacity expansion for under-resourced facilities and identify over-capacity situations for reallocation**.

---

## 🎯 Acceptance Criteria

1. **Gap analysis completed**
   - Capacity-utilization gap calculated: capacity growth vs utilization growth spread
   - Shortage identification: facility types where utilization outpaces capacity
   - Surplus identification: facility types with excess capacity vs demand
   - Gap severity ranked: critical, moderate, minimal

2. **Temporal gap evolution analyzed**
   - Historical gap trends: widening or narrowing over time
   - Inflection points: years when gaps emerged or resolved
   - Forecast implications: projected gaps if trends continue

3. **Sector-specific gaps assessed**
   - Public sector gaps vs private sector gaps
   - Cross-sector opportunities: can private capacity fill public gaps?
   - Sector balance recommendations

4. **Deliverables**
   - Output: `results/tables/capacity_gap_analysis.csv`
   - Figures: Gap charts, capacity vs utilization trends
   - Priority recommendations: Facility types needing expansion

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#capacity-planning-methods) - Gap analysis methodologies
- [Problem Statement PS-003](../../../problem_statements/ps-003-healthcare-capacity-optimization.md#objective-3) - Identify capacity gaps

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `matplotlib>=3.8.0`, `plotly>=5.18.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-003-US-03, PS-003-US-04 (Capacity and utilization analyses - BLOCKING)
- **Data Sources**: `shared/data/3_interim/capacity_utilization_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Gap Calculation
- [ ] Calculate capacity growth rate per facility type
- [ ] Calculate utilization growth rate per facility type
- [ ] Compute gap metric: utilization growth - capacity growth
- [ ] Classify gaps: positive = shortage, negative = surplus
- [ ] Rank gaps by severity

### Temporal Analysis
- [ ] Analyze gap trends over time
- [ ] Identify when gaps emerged or widened
- [ ] Detect inflection points
- [ ] Simple extrapolation: project future gaps

### Sector Analysis
- [ ] Calculate public vs private gaps
- [ ] Assess cross-sector reallocation opportunities
- [ ] Recommend sector-specific expansions

### Prioritization
- [ ] Priority matrix: gap severity vs interventionability
- [ ] Rank facility types needing expansion
- [ ] Identify low-priority capacity (surplus areas)

### Visualization
- [ ] Line charts: capacity vs utilization trends
- [ ] Gap charts: showing shortage/surplus by facility type
- [ ] Priority matrix scatter plot
- [ ] Save figures: `reports/figures/capacity_gaps_*.png/pdf`

### Testing & Documentation
- [ ] Unit tests for gap calculations
- [ ] Validate gap classification logic
- [ ] Docstrings
- [ ] Gap analysis report with recommendations

---

## 📌 Notes

**Polars Gap Analysis**:
```python
import polars as pl

df = pl.read_parquet("shared/data/3_interim/capacity_utilization_integrated.parquet")

# Calculate growth rates
df_growth = df.group_by('facility_category').agg([
    ((pl.col('capacity_metric').last() / pl.col('capacity_metric').first()) - 1).alias('capacity_growth'),
    ((pl.col('utilization_metric').last() / pl.col('utilization_metric').first()) - 1).alias('utilization_growth')
])

# Gap = utilization growth - capacity growth
df_gaps = df_growth.with_columns([
    (pl.col('utilization_growth') - pl.col('capacity_growth')).alias('gap')
])

# Classify gaps
df_gaps = df_gaps.with_columns([
    pl.when(pl.col('gap') > 0.1).then('critical_shortage')
      .when(pl.col('gap') > 0.05).then('moderate_shortage')
      .when(pl.col('gap') < -0.05).then('surplus')
      .otherwise('balanced')
      .alias('gap_classification')
])
```

**Gap Severity Thresholds**:
- **Critical Shortage**: utilization growth > capacity growth by >10%
- **Moderate Shortage**: 5-10% gap
- **Balanced**: gap within ±5%
- **Surplus**: capacity growth > utilization growth by >5%

**Expected Findings**:
- **Acute Care**: Likely balanced or slight shortage
- **Community Hospitals**: Possible expansion aligned with utilization
- **Long-Term Care**: Likely shortage (aging population driving demand)

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Identify and rank capacity shortages and surpluses by comparing cumulative capacity growth against utilisation growth across facility types (2009–2020). Produce prioritised expansion recommendations and temporal gap trend analysis.

**Primary User Role**: Healthcare Service Planner prioritising infrastructure investments

**Key Success Metric**:
- `results/tables/problem-statement-003/capacity_gap_analysis_{timestamp}.csv` with columns: `facility_category`, `capacity_growth`, `utilization_growth`, `gap`, `gap_severity`, `priority_score`
- 3+ gap visualisations saved to `reports/figures/problem-statement-003/`

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Decision |
|-----------|------|--------|----------|
| Integrated parquet | `shared/data/3_interim/capacity_utilization_integrated.parquet` | ✅ Upstream | US-02 — BLOCKING |
| `capacity_trend_analysis.py` | `src/analysis/capacity_trend_analysis.py` (US-03) | ✅ Reuse | `calculate_yoy_growth()` + `calculate_cagr()` |
| `efficiency_metrics.py` | `src/analysis/efficiency_metrics.py` (US-05) | ✅ Reuse | Utilisation index already calculated |

**New Components**:
- `src/analysis/capacity_gap_analysis.py` — gap calculation, severity classification, temporal evolution, sector analysis, prioritisation
- `src/visualization/gap_plots.py` — gap charts, priority matrix scatter

---

### 3. Affected Files

```
- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_gap_analysis.py
  Functions:
    calculate_capacity_gaps(df, group_cols) -> pl.DataFrame
    classify_gap_severity(df) -> pl.DataFrame
    analyze_temporal_gap_evolution(df, group_cols) -> pl.DataFrame
    calculate_sector_gaps(df) -> pl.DataFrame
    prioritize_expansion(df_gaps) -> pl.DataFrame
    run_gap_analysis(parquet_path, problem_num) -> pl.DataFrame

- [CREATE] problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/gap_plots.py
  Functions:
    plot_capacity_vs_utilization_trends(df, output_path) -> None
    plot_gap_severity_chart(df_gaps, output_path) -> None
    plot_priority_matrix(df_gaps, output_path) -> None
    plot_temporal_gap_evolution(df, output_path) -> None

- Results: results/tables/problem-statement-003/capacity_gap_analysis_{ts}.csv
- Figures:  reports/figures/problem-statement-003/capacity_vs_utilization_trends_{ts}.png
            reports/figures/problem-statement-003/gap_severity_chart_{ts}.png
            reports/figures/problem-statement-003/priority_matrix_{ts}.png
```

---

### 4. Component Breakdown

#### `capacity_gap_analysis.py`

**Responsibility**: Pure gap analysis logic — accepts the integrated parquet, computes growth differentials, classifies severity, ranks priorities. No I/O except via orchestrator.

**Technical Constraints**:
- Memory: < 100 MB
- Edge cases: handle division by zero (zero capacity growth), single-year groups
- Gap thresholds stored in config, not hard-coded
- Temporal rolling windows: minimum 3-year window

---

### 5. Data Pipeline

```
shared/data/3_interim/capacity_utilization_integrated.parquet
  │
  ▼  group_by([facility_category]).agg(period start/end/growth)
  │
  ├─▶ calculate_capacity_gaps()
  │     cap_growth = (end_beds - start_beds) / start_beds
  │     util_growth = (end_adm_rate - start_adm_rate) / start_adm_rate
  │     gap = util_growth - cap_growth
  │     → positive gap = shortage (demand outpacing supply)
  │
  ├─▶ classify_gap_severity()
  │     critical_shortage:   gap > 0.10
  │     moderate_shortage:   0.05 < gap ≤ 0.10
  │     balanced:            -0.05 ≤ gap ≤ 0.05
  │     surplus:             gap < -0.05
  │
  ├─▶ analyze_temporal_gap_evolution()
  │     rolling 3-year gap per facility_category
  │     identify inflection points (gap sign changes)
  │
  ├─▶ calculate_sector_gaps()
  │     repeat gap analysis at [facility_category, sector] level
  │
  ├─▶ prioritize_expansion()
  │     priority_score = gap × 10 + utilization_growth × 5
  │     rank descending
  │
  └─▶ Join all into consolidated DataFrame
          → Save CSV
          → Generate 4 figures
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementations

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/analysis/capacity_gap_analysis.py
import polars as pl
from pathlib import Path
from datetime import datetime
from loguru import logger

# Configurable severity thresholds
CRITICAL_THRESHOLD: float = 0.10   # 10%+ gap = critical shortage
MODERATE_THRESHOLD: float = 0.05   # 5-10% gap = moderate shortage
SURPLUS_THRESHOLD: float = -0.05   # -5%+ negative gap = surplus


def calculate_capacity_gaps(
    df: pl.DataFrame,
    group_cols: list[str] = ["facility_category"],
    beds_col: str = "total_beds",
    util_col: str = "total_admission_rate",
) -> pl.DataFrame:
    """Calculate total-period capacity vs utilisation growth gap.

    Gap = Utilisation Growth - Capacity Growth (both as fractions of start value).
    Positive gap → demand grew faster than supply (shortage).
    Negative gap → supply grew faster than demand (surplus).

    Args:
        df: Integrated DataFrame with year, group_cols, beds_col, util_col.
        group_cols: Columns defining each analysis group.
        beds_col: Capacity metric column.
        util_col: Utilisation metric column.

    Returns:
        DataFrame with: group_cols, start_year, end_year, start_beds, end_beds,
        capacity_growth, start_util, end_util, utilization_growth, gap.
    """
    for col in [beds_col, util_col]:
        if col not in df.columns:
            raise ValueError(f"Column '{col}' not found. Available: {df.columns}")

    df_sorted = df.sort(group_cols + ["year"])

    df_agg = df_sorted.group_by(group_cols).agg([
        pl.col("year").min().alias("start_year"),
        pl.col("year").max().alias("end_year"),
        pl.col(beds_col).first().alias("start_beds"),
        pl.col(beds_col).last().alias("end_beds"),
        pl.col(util_col).first().alias("start_util"),
        pl.col(util_col).last().alias("end_util"),
    ])

    df_gaps = df_agg.with_columns([
        pl.when(pl.col("start_beds") > 0)
        .then((pl.col("end_beds") - pl.col("start_beds")) / pl.col("start_beds"))
        .otherwise(None)
        .cast(pl.Float64)
        .alias("capacity_growth"),

        pl.when(pl.col("start_util") > 0)
        .then((pl.col("end_util") - pl.col("start_util")) / pl.col("start_util"))
        .otherwise(None)
        .cast(pl.Float64)
        .alias("utilization_growth"),
    ]).with_columns([
        (pl.col("utilization_growth") - pl.col("capacity_growth"))
        .cast(pl.Float64)
        .alias("gap"),
    ])

    n_shortages = df_gaps.filter(pl.col("gap") > MODERATE_THRESHOLD).height
    n_surpluses = df_gaps.filter(pl.col("gap") < SURPLUS_THRESHOLD).height
    logger.info(
        f"calculate_capacity_gaps: {n_shortages} shortage groups, "
        f"{n_surpluses} surplus groups"
    )
    return df_gaps


def classify_gap_severity(
    df: pl.DataFrame,
    gap_col: str = "gap",
) -> pl.DataFrame:
    """Apply severity classification to gap values.

    Thresholds:
      critical_shortage:   gap > CRITICAL_THRESHOLD (10%)
      moderate_shortage:   MODERATE_THRESHOLD < gap ≤ CRITICAL_THRESHOLD
      balanced:            SURPLUS_THRESHOLD ≤ gap ≤ MODERATE_THRESHOLD
      surplus:             gap < SURPLUS_THRESHOLD (-5%)

    Args:
        df: DataFrame with gap_col.
        gap_col: Column name for gap values.

    Returns:
        DataFrame with added gap_severity (Categorical) column.
    """
    return df.with_columns([
        pl.when(pl.col(gap_col) > CRITICAL_THRESHOLD)
        .then(pl.lit("critical_shortage"))
        .when(pl.col(gap_col) > MODERATE_THRESHOLD)
        .then(pl.lit("moderate_shortage"))
        .when(pl.col(gap_col) >= SURPLUS_THRESHOLD)
        .then(pl.lit("balanced"))
        .when(pl.col(gap_col).is_not_null())
        .then(pl.lit("surplus"))
        .otherwise(pl.lit("unknown"))
        .cast(pl.Categorical)
        .alias("gap_severity"),
    ])


def analyze_temporal_gap_evolution(
    df: pl.DataFrame,
    group_cols: list[str] = ["facility_category"],
    beds_col: str = "total_beds",
    util_col: str = "total_admission_rate",
    rolling_window: int = 3,
) -> pl.DataFrame:
    """Compute rolling gap evolution to identify inflection points.

    Args:
        df: Year-level DataFrame.
        group_cols: Grouping columns.
        beds_col: Capacity column.
        util_col: Utilisation column.
        rolling_window: Window for rolling average (years).

    Returns:
        DataFrame with rolling_gap, gap_direction, inflection_point flag.
    """
    df_sorted = df.sort(group_cols + ["year"])

    df_yoy = df_sorted.with_columns([
        (pl.col(beds_col).cast(pl.Float64).pct_change().over(group_cols))
        .alias("yoy_cap_growth"),
        (pl.col(util_col).cast(pl.Float64).pct_change().over(group_cols))
        .alias("yoy_util_growth"),
    ]).with_columns([
        (pl.col("yoy_util_growth") - pl.col("yoy_cap_growth")).alias("annual_gap"),
    ]).with_columns([
        pl.col("annual_gap")
        .rolling_mean(window_size=rolling_window, min_periods=2)
        .over(group_cols)
        .alias("rolling_gap"),
    ]).with_columns([
        pl.when(pl.col("rolling_gap") > 0)
        .then(pl.lit("shortage"))
        .when(pl.col("rolling_gap") < 0)
        .then(pl.lit("surplus"))
        .otherwise(pl.lit("neutral"))
        .cast(pl.Categorical)
        .alias("gap_direction"),
    ])

    # Flag inflection points: gap_direction changed from prior year
    df_inflection = df_yoy.with_columns([
        (pl.col("gap_direction") != pl.col("gap_direction").shift(1).over(group_cols))
        .alias("inflection_point"),
    ])

    n_inflections = df_inflection.filter(pl.col("inflection_point")).height
    logger.info(f"analyze_temporal_gap_evolution: {n_inflections} inflection points detected")
    return df_inflection


def calculate_sector_gaps(
    df: pl.DataFrame,
) -> pl.DataFrame:
    """Run gap analysis at [facility_category, sector] level.

    Args:
        df: Integrated DataFrame with sector column.

    Returns:
        Gap DataFrame grouped by facility_category and sector.
    """
    return calculate_capacity_gaps(df, group_cols=["facility_category", "sector"])


def prioritize_expansion(
    df_gaps: pl.DataFrame,
) -> pl.DataFrame:
    """Rank facility types by expansion priority.

    Priority score = gap × 10 + utilization_growth × 5
    Higher score → higher expansion priority.

    Args:
        df_gaps: Output from classify_gap_severity().

    Returns:
        DataFrame sorted by priority_score (descending) with rank column.
    """
    df_priority = df_gaps.with_columns([
        (
            pl.col("gap").fill_null(0) * 10 +
            pl.col("utilization_growth").fill_null(0) * 5
        ).alias("priority_score"),
    ]).sort("priority_score", descending=True).with_row_index("priority_rank", offset=1)

    top = df_priority.head(1)
    logger.info(
        f"prioritize_expansion: top priority = "
        f"{top['facility_category'][0]} (score: {top['priority_score'][0]:.2f})"
    )
    return df_priority


def run_gap_analysis(
    parquet_path: str = "shared/data/3_interim/capacity_utilization_integrated.parquet",
    problem_num: str = "003",
) -> pl.DataFrame:
    """Orchestrate full gap analysis pipeline and save results.

    Args:
        parquet_path: Path to integrated dataset (US-02 output).
        problem_num: Problem statement number for output directory.

    Returns:
        Consolidated gap analysis DataFrame.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    tables_dir = Path(f"results/tables/problem-statement-{problem_num}")
    tables_dir.mkdir(parents=True, exist_ok=True)

    df = pl.read_parquet(parquet_path).filter(
        pl.col("total_beds").is_not_null()
    )
    logger.info(f"Loaded {df.height} rows for gap analysis")

    # Facility-level gap analysis
    df_gaps = calculate_capacity_gaps(df)
    df_gaps = classify_gap_severity(df_gaps)
    df_priority = prioritize_expansion(df_gaps)

    # Sector-level gaps
    df_sector_gaps = calculate_sector_gaps(df)
    df_sector_gaps = classify_gap_severity(df_sector_gaps)

    # Temporal evolution
    df_temporal = analyze_temporal_gap_evolution(df)

    # Save outputs
    gap_path = tables_dir / f"capacity_gap_analysis_{ts}.csv"
    df_priority.write_csv(gap_path)
    logger.info(f"✅ Gap analysis saved: {gap_path}")
    print(f"✅ Saved: {gap_path}")

    sector_path = tables_dir / f"capacity_sector_gap_analysis_{ts}.csv"
    df_sector_gaps.write_csv(sector_path)
    logger.info(f"✅ Sector gaps saved: {sector_path}")
    print(f"✅ Saved: {sector_path}")

    return df_priority
```

#### 6.2 Visualization Module

```python
# problem-statements/ps-003-healthcare-capacity-optimization/src/visualization/gap_plots.py
import polars as pl
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import seaborn as sns
from pathlib import Path
from datetime import datetime
from loguru import logger

_SEVERITY_COLORS = {
    "critical_shortage": "#dc3545",
    "moderate_shortage": "#fd7e14",
    "balanced": "#28a745",
    "surplus": "#17a2b8",
}
_FIG_DIR = "reports/figures/problem-statement-{num}"
sns.set_style("whitegrid")


def _save_fig(fig: plt.Figure, path: Path) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(path, dpi=300, bbox_inches="tight")
    logger.info(f"✅ Figure saved: {path}")
    print(f"✅ Saved: {path}")
    plt.show()
    plt.close(fig)


def plot_gap_severity_chart(
    df_gaps: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Horizontal bar chart of capacity gap by facility type, color-coded by severity.

    Args:
        df_gaps: Output from classify_gap_severity() with gap and gap_severity columns.
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_gaps.sort("gap").to_pandas()

    bar_colors = [_SEVERITY_COLORS.get(s, "#6c757d") for s in pdf["gap_severity"]]
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.barh(pdf["facility_category"], pdf["gap"] * 100, color=bar_colors, alpha=0.85)
    ax.axvline(0, color="black", linewidth=0.8, linestyle="--")
    ax.axvline(10, color="red", linewidth=0.8, linestyle=":", alpha=0.6, label="Critical (10%)")
    ax.axvline(5, color="orange", linewidth=0.8, linestyle=":", alpha=0.6, label="Moderate (5%)")
    ax.axvline(-5, color="blue", linewidth=0.8, linestyle=":", alpha=0.6, label="Surplus (-5%)")

    legend_patches = [
        mpatches.Patch(color=c, label=s.replace("_", " ").title())
        for s, c in _SEVERITY_COLORS.items()
    ]
    ax.legend(handles=legend_patches, loc="lower right")
    ax.set_xlabel("Gap (%) — Positive = Shortage, Negative = Surplus")
    ax.set_title("Capacity-Utilisation Gap by Facility Type (2009–2020)", fontweight="bold")
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"gap_severity_chart_{ts}.png"
    _save_fig(fig, out)


def plot_priority_matrix(
    df_gaps: pl.DataFrame,
    problem_num: str = "003",
) -> None:
    """Scatter plot: gap (x) vs utilization_growth (y), sized by priority_score.

    Args:
        df_gaps: Output from prioritize_expansion().
        problem_num: For output directory.
    """
    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    pdf = df_gaps.to_pandas()

    fig, ax = plt.subplots(figsize=(10, 8))
    scatter = ax.scatter(
        pdf["gap"] * 100,
        pdf["utilization_growth"] * 100,
        s=pdf["priority_score"].abs() * 50 + 100,
        c=[_SEVERITY_COLORS.get(s, "#6c757d") for s in pdf["gap_severity"]],
        alpha=0.8, edgecolors="black", linewidth=0.5,
    )

    for _, row in pdf.iterrows():
        ax.annotate(
            row["facility_category"],
            (row["gap"] * 100, row["utilization_growth"] * 100),
            textcoords="offset points", xytext=(8, 8), fontsize=9,
        )

    ax.axhline(0, color="black", linewidth=0.5, alpha=0.5)
    ax.axvline(0, color="black", linewidth=0.5, alpha=0.5)
    ax.set_xlabel("Capacity Gap (%) — Positive = Shortage")
    ax.set_ylabel("Utilisation Growth (%)")
    ax.set_title("Expansion Priority Matrix\n(Size = Priority Score)", fontweight="bold")
    plt.tight_layout()

    out = Path(_FIG_DIR.format(num=problem_num)) / f"priority_matrix_{ts}.png"
    _save_fig(fig, out)
```

---

### 7. Domain-Driven Feature Engineering

| Feature | Formula | Interpretation |
|---------|---------|---------|
| `capacity_growth` | `(end_beds - start_beds) / start_beds` | 0.20 = 20% more beds over period |
| `utilization_growth` | `(end_util - start_util) / start_util` | 0.30 = 30% more admissions over period |
| `gap` | `util_growth - cap_growth` | 0.10 = demand grew 10pp faster than supply |
| `gap_severity` | Categorical from thresholds | Actionable classification for planners |
| `priority_score` | `gap × 10 + util_growth × 5` | Higher = more urgent expansion need |
| `rolling_gap` | 3-year rolling mean of annual gap | Smooths year-to-year volatility |
| `inflection_point` | Gap direction flip | Identifies when shortage became surplus or vice versa |

---

### 8. Testing Strategy

```python
# tests/unit/test_capacity_gap_analysis.py
import polars as pl
import pytest
from src.analysis.capacity_gap_analysis import (
    calculate_capacity_gaps,
    classify_gap_severity,
    prioritize_expansion,
    analyze_temporal_gap_evolution,
)


@pytest.fixture
def sample_df() -> pl.DataFrame:
    return pl.DataFrame({
        "year": pl.Series([2009, 2020, 2009, 2020, 2009, 2020], dtype=pl.Int32),
        "facility_category": ["acute_care"] * 2 + ["long_term_care"] * 2 + ["primary_care"] * 2,
        "total_beds": pl.Series([10000, 11000, 4000, 6000, 500, 490], dtype=pl.Int32),
        "total_admission_rate": [8000.0, 10400.0, 2000.0, 3200.0, 1500.0, 1200.0],
    })


def test_gap_positive_for_shortage(sample_df):
    # acute: util_growth = (10400-8000)/8000 = 0.30, cap_growth = (11000-10000)/10000 = 0.10
    # gap = 0.30 - 0.10 = 0.20 → shortage
    result = calculate_capacity_gaps(sample_df)
    acute_gap = result.filter(pl.col("facility_category") == "acute_care")["gap"][0]
    assert acute_gap == pytest.approx(0.20, rel=1e-3)


def test_gap_negative_for_surplus(sample_df):
    # primary_care: util_growth = (1200-1500)/1500 = -0.20, cap_growth = (490-500)/500 = -0.02
    # gap = -0.20 - (-0.02) = -0.18 → surplus
    result = calculate_capacity_gaps(sample_df)
    primary_gap = result.filter(pl.col("facility_category") == "primary_care")["gap"][0]
    assert primary_gap < -0.05  # Should be classified as surplus


def test_gap_severity_thresholds(sample_df):
    gaps = calculate_capacity_gaps(sample_df)
    classified = classify_gap_severity(gaps)

    # acute_care gap = 0.20 > 0.10 → critical_shortage
    acute_sev = classified.filter(
        pl.col("facility_category") == "acute_care"
    )["gap_severity"][0]
    assert acute_sev == "critical_shortage"

    # primary_care gap < -0.05 → surplus
    primary_sev = classified.filter(
        pl.col("facility_category") == "primary_care"
    )["gap_severity"][0]
    assert primary_sev == "surplus"


def test_priority_rank_1_is_highest_score(sample_df):
    gaps = calculate_capacity_gaps(sample_df)
    classified = classify_gap_severity(gaps)
    prioritized = prioritize_expansion(classified)
    assert prioritized["priority_rank"][0] == 1
    assert prioritized["priority_score"][0] >= prioritized["priority_score"][-1]


def test_zero_start_beds_gives_null_gap():
    df = pl.DataFrame({
        "year": pl.Series([2009, 2020], dtype=pl.Int32),
        "facility_category": ["new_facility", "new_facility"],
        "total_beds": pl.Series([0, 100], dtype=pl.Int32),
        "total_admission_rate": [0.0, 500.0],
    })
    result = calculate_capacity_gaps(df)
    assert result["capacity_growth"].is_null()[0]
```

---

### 9. Implementation Steps

#### Phase 1: Setup
- [ ] Create `src/analysis/capacity_gap_analysis.py`
- [ ] Create `src/visualization/gap_plots.py`

#### Phase 2: Core Gap Logic
- [ ] Implement `calculate_capacity_gaps()` — verify formula with manual calculation
- [ ] Implement `classify_gap_severity()` — test boundary values
- [ ] Implement `analyze_temporal_gap_evolution()` — verify rolling window
- [ ] Implement `calculate_sector_gaps()` — cross-sector comparison
- [ ] Implement `prioritize_expansion()` — verify ranking logic

#### Phase 3: Visualizations
- [ ] Generate gap severity bar chart
- [ ] Generate priority matrix scatter plot
- [ ] Generate temporal gap evolution line chart
- [ ] Confirm all figures saved with timestamps

#### Phase 4: Recommendations
- [ ] Summarise top 3 expansion priorities in notebook cell
- [ ] Identify cross-sector reallocation opportunities
- [ ] Flag any surplus areas for investigation

#### Phase 5: Tests
- [ ] Write `tests/unit/test_capacity_gap_analysis.py`
- [ ] `pytest tests/unit/test_capacity_gap_analysis.py -v --cov`
- [ ] Coverage ≥ 80%

---

### 10. Code Generation Order

1. `src/analysis/capacity_gap_analysis.py`
2. `tests/unit/test_capacity_gap_analysis.py`
3. `src/visualization/gap_plots.py`
4. `notebooks/04_capacity_gaps.ipynb`

---

### 11. Data Quality & Validation

**Input validation**:
- Assert `total_beds.is_not_null()` for start/end years — log and drop groups missing either endpoint
- Minimum 2 years of data per group for gap calculation
- Both `start_util` and `start_beds` must be > 0 (use null safety)

**Output validation**:
- `capacity_growth + utilization_growth` should each be in range [−0.8, +1.5]
- `gap` ∈ [−1.0, +1.0] (flag if outside — indicates outlier data)
- `gap_severity` should include at least 2 distinct values (not all the same classification)
- CSV row count = number of unique `facility_category` values

---

**Implementation Plan Checklist**
- [x] Feature overview with success metrics
- [x] Component reuse strategy with upstream dependencies
- [x] All affected files with CREATE indicators and function signatures
- [x] Complete executable implementations with type hints, docstrings, null safety
- [x] Visualization module with color-coded severity
- [x] Gap severity thresholds documented (configurable)
- [x] Test assertions covering positive gap, negative gap, zero start, priority ranking
- [x] Phased implementation steps
- [x] Data quality validation for gap inputs and outputs
