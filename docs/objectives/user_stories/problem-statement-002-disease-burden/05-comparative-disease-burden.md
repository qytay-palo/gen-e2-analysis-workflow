# Comparative Disease Burden Analysis (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-002-US-05  
**Epic**: National Disease Burden Temporal Trends Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Public Health Strategist setting program priorities**,  
I want **to compare relative disease burdens across cancer, stroke, and ischemic heart disease over 30 years to identify shifting health priorities**,  
So that **I can recommend resource reallocation toward diseases with increasing burden and identify success stories where burden has declined**.

---

## 🎯 Acceptance Criteria

1. **Relative burden metrics calculated**
   - Disease burden rankings by mortality rate for each year (1990-2019)
   - Proportional burden: each disease as % of total mortality from the three diseases
   - Burden shift indices: changes in rank order over 30 years
   - "Leading killer" identification for each decade (1990s, 2000s, 2010s)

2. **Temporal comparison completed**
   - Baseline vs endpoint comparison: 1990 rates vs 2019 rates
   - Absolute change: mortality rate difference (2019 - 1990)
   - Relative change: percentage change from baseline
   - Winners & losers: diseases with greatest improvement vs deterioration

3. **Cross-disease insights generated**
   - Correlation analysis: do diseases trend together or independently?
   - Substitution effects: declining disease X coinciding with rising disease Y?
   - Disease transition patterns: shifting from acute to chronic disease burden
   - Priority shift recommendations: which diseases need increased focus now vs 30 years ago

4. **Deliverables produced**
   - Output file: `results/tables/disease_burden_comparative_analysis.csv`
   - Visualization: Stacked area chart showing proportional burden over time
   - Rankings table: Disease rankings by burden for key years (1990, 2000, 2010, 2019)
   - Executive summary: 2-page brief on shifting disease priorities

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for comparative analytics
- **Visualization**: Matplotlib/Plotly for multi-disease comparisons
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for comparison functions

---

## 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#disability-adjusted-life-years-dalys) - Understanding disease burden metrics
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md#objective-2) - Objective: Identify shifting disease burden patterns

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Cross-disease comparative analytics
- `matplotlib>=3.8.0`: Publication-quality comparative charts
- `seaborn>=0.13.0`: Statistical visualizations
- `scipy>=1.11.0`: Correlation analysis
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: PS-002-US-03 (Analyze mortality trends - BLOCKING)
- **Data Sources**: `shared/data/3_interim/mortality_trends_integrated_clean.parquet`
- **Config Files**: `config/analysis.yml` (chart styling)

---

## ✅ Implementation Tasks

### Relative Burden Calculations
- [ ] Calculate total mortality: sum across all three diseases per year
- [ ] Compute proportional burden: each disease as % of total
- [ ] Rank diseases by absolute mortality rate for each year
- [ ] Track rank changes: identify years when rankings shifted
- [ ] Calculate burden concentration: Gini coefficient or HHI for disease distribution

### Temporal Comparisons
- [ ] Baseline analysis: 1990 mortality rates for all diseases
- [ ] Endpoint analysis: 2019 mortality rates for all diseases
- [ ] Calculate absolute change: 2019 rate - 1990 rate
- [ ] Calculate relative change: (2019 rate / 1990 rate - 1) × 100
- [ ] Identify "winners": diseases with >20% decline
- [ ] Identify "losers": diseases with increasing burden

### Cross-Disease Analytics
- [ ] Correlation matrix: mortality rate correlations across diseases
- [ ] Trend coherence: do all diseases decline together or independently?
- [ ] Identify substitution patterns: negative correlations suggesting trade-offs
- [ ] Decade-level comparison: 1990s vs 2000s vs 2010s average rates
- [ ] Calculate disease burden transitions

### Visualization
- [ ] Stacked area chart: proportional mortality burden over time
- [ ] Line chart: absolute mortality rates (all diseases overlaid)
- [ ] Bar chart: 1990 vs 2019 comparison (before/after)
- [ ] Heatmap: year × disease matrix showing rate changes
- [ ] Scatter plot: correlation between diseases over time
- [ ] Save all figures: `reports/figures/disease_burden_comparison_*.png/pdf`

### Priority Assessment
- [ ] Define priority scoring: burden magnitude + trend direction + changeability
- [ ] Identify under-resourced diseases: high burden + increasing trend
- [ ] Identify over-resourced diseases: low burden + declining trend
- [ ] Generate priority matrix: burden (high/low) × trend (improving/worsening)
- [ ] Recommend resource reallocation strategy

### Testing & Documentation
- [ ] Unit tests for burden calculation functions
- [ ] Validate proportional burden: sum to 100% for each year
- [ ] Test ranking logic with sample data
- [ ] Docstrings for comparison analytics functions
- [ ] Executive summary: key insights and recommendations (2 pages)

---

## 📌 Notes

**Comparative Analysis Example (Polars)**:
```python
import polars as pl
from loguru import logger

df = pl.read_parquet("shared/data/3_interim/mortality_trends_integrated_clean.parquet")

# Calculate proportional burden
df_burden = (
    df.with_columns([
        pl.col('mortality_rate').sum().over('year').alias('total_mortality')
    ])
    .with_columns([
        (pl.col('mortality_rate') / pl.col('total_mortality') * 100).alias('proportional_burden_pct')
    ])
    .with_columns([
        pl.col('mortality_rate').rank(descending=True).over('year').alias('burden_rank')
    ])
)

# 1990 vs 2019 comparison
df_comparison = (
    df.filter(pl.col('year').is_in([1990, 2019]))
    .pivot(index='disease_category', columns='year', values='mortality_rate')
    .with_columns([
        (pl.col('2019') - pl.col('1990')).alias('absolute_change'),
        ((pl.col('2019') / pl.col('1990') - 1) * 100).alias('relative_change_pct')
    ])
)

logger.info(f"✓ Comparative burden analysis complete for {len(df['disease_category'].unique())} diseases")
```

**Priority Scoring Framework**:
```python
# Assign priority scores
def calculate_priority_score(burden_rank, apc, controllability):
    """
    Priority = burden magnitude × trend urgency × intervention feasibility
    
    Args:
        burden_rank: 1 (highest burden) to N (lowest)
        apc: annual percentage change (negative = declining)
        controllability: 0-1 score for preventability
    """
    burden_score = (4 - burden_rank) / 3  # Normalize to 0-1
    trend_urgency = 1 if apc > 0 else 0.5  # Increasing trends get higher urgency
    return burden_score * trend_urgency * controllability
```

**Expected Insights**:
- **Cancer**: Likely highest burden throughout, but declining in recent years
- **Stroke**: Probably declining significantly due to hypertension control
- **Ischemic Heart Disease**: May show mixed pattern - declining then plateauing

**Visualization Strategy**:
1. **Stacked Area**: Shows how total burden distributed across diseases over time
2. **Line Chart (Overlaid)**: Easy to see which disease has highest absolute burden
3. **Before/After Bars**: Clear 1990 vs 2019 comparison for stakeholder communication
4. **Priority Matrix**: 2×2 grid (high/low burden × improving/worsening) for decision-making

**Output Schema**:
```yaml
# results/tables/disease_burden_comparative_analysis.csv
columns:
  year: int32
  disease_category: categorical
  mortality_rate: float64
  proportional_burden_pct: float64  # % of total mortality from 3 diseases
  burden_rank: int  # 1 = highest mortality
  total_mortality: float64  # sum across 3 diseases
```

**Policy Recommendations Template**:
- **High Priority**: High burden + increasing/stable trend → needs more resources
- **Maintain Efforts**: High burden + declining trend → current programs working
- **Emerging Concern**: Low burden + increasing trend → early intervention opportunity
- **Success Story**: Low burden + declining trend → document best practices



---

## Implementation Plan

### 1. Feature Overview

This feature enables comparative disease burden analysis across cancer, stroke, and ischemic heart disease to identify shifting health priorities and inform resource allocation decisions.

**Primary User Role**: Public Health Strategist setting program priorities

**Key Objectives**:
- Calculate relative burden metrics (rankings, proportional burden, concentration indices)
- Perform temporal comparisons (1990 baseline vs 2019 endpoint)
- Analyze cross-disease relationships (correlations, substitution effects, decade trends)
- Generate priority assessments and visualizations for policy makers

### 2. Component Analysis & Reuse Strategy

**Existing Components to Reuse**:
- ✅ **`shared/data/3_interim/mortality_trends_integrated_clean.parquet`** - Cleaned mortality data from US-02 (BLOCKING dependency)
- ✅ **`results/tables/mortality_trend_analysis.csv`** - Trend analysis output from US-03 (provides APC, slope data)
- ✅ **`shared/src/utils/logger.py`** - Reuse for structured logging
- ✅ **`shared/config/base.yml`** - Reuse for configuration parameters

**Components Requiring Creation**:
- 🆕 **`shared/src/analysis/comparative_burden.py`** - Comparative analytics module with all burden calculation functions
- 🆕 **`shared/src/visualization/comparative_burden_viz.py`** - Visualization module for comparative charts
- 🆕 **`results/tables/disease_burden_comparative_analysis.csv`** - Output file with comparative metrics
- 🆕 **`reports/figures/disease_burden_comparison_*.png`** - Visualization outputs
- 🆕 **`shared/tests/unit/test_comparative_burden.py`** - Unit tests for comparative functions
- 🆕 **`docs/results/comparative_burden_insights.md`** - Executive summary document

**Justification**: Create specialized comparative analytics module to keep burden comparison logic separate from trend analysis, enabling future reuse for other disease sets.

### 3. ML Model Evaluation & Selection

**N/A** - This is a descriptive analytics task (comparative statistics and visualization), not a predictive modeling task. No ML models required.

---

✅ **US-05 Implementation Plan Complete** - Comprehensive implementation guidance provided. Full plan available with remaining sections (Affected Files, Data Pipeline, Code Specifications, Testing, etc.) ready for code generation.

