# Disease Burden Prioritization Framework (Lifecycle Stage: Feature Engineering & Insights)

**Story ID**: PS-002-US-07  
**Epic**: National Disease Burden Temporal Trends Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Public Health Policy Director allocating program budgets**,  
I want **to rank diseases by burden magnitude, trend urgency, and interventionability to create an evidence-based prioritization framework**,  
So that **I can justify resource allocation decisions with objective criteria and focus investments on highest-impact opportunities**.

---

## 🎯 Acceptance Criteria

1. **Multi-criteria prioritization model developed**
   - Burden magnitude score: based on absolute mortality rate and years of life lost
   - Trend urgency score: based on annual percentage change and acceleration
   - Interventionability score: expert assessment of preventability and treatment effectiveness
   - Composite priority score: weighted combination of three dimensions

2. **Disease rankings generated**
   - Overall priority ranking: diseases ordered by composite score
   - Dimension-specific rankings: separate rankings for burden, trend, interventionability
   - Priority quadrants: high burden + increasing vs high burden + declining, etc.
   - Top 5 priority diseases identified for immediate action

3. **Resource allocation recommendations**
   - Investment increase recommendations: diseases needing more resources
   - Investment maintenance: diseases with adequate resources
   - Investment reallocation opportunities: over-resourced diseases
   - Quantified budget implications: suggested % changes based on priority scores

4. **Deliverables produced**
   - Output file: `results/tables/disease_burden_prioritization_matrix.csv`
   - Priority quadrant visualization: scatter plot (burden vs trend urgency)
   - Executive scorecard: top priorities with justification
   - Policy brief: Evidence-based resource allocation recommendations (8-10 pages)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for scoring and ranking
- **Multi-Criteria Analysis**: Custom scoring framework or AHP (Analytic Hierarchy Process)
- **Visualization**: Matplotlib/Plotly for priority matrices
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for scoring functions

---

## 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#disability-adjusted-life-years-dalys) - Understanding DALY calculations for burden assessment
- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md) - Prioritization frameworks in public health
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md#objective-4) - Objective: Generate actionable prioritization insights

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Scoring and ranking analytics
- `scikit-learn>=1.3.0`: Normalization and scoring utilities
- `matplotlib>=3.8.0`: Priority matrix visualizations
- `seaborn>=0.13.0`: Heatmaps and quadrant plots
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: PS-002-US-03, PS-002-US-04, PS-002-US-05 (All trend analyses - BLOCKING)
- **Data Sources**: `shared/data/3_interim/mortality_trends_integrated_clean.parquet`
- **Config Files**: `config/analysis.yml` (scoring weights, priority thresholds)

---

## ✅ Implementation Tasks

### Dimension 1: Burden Magnitude Scoring
- [ ] Calculate absolute burden: latest year (2019) mortality rate
- [ ] Estimate years of life lost (YLL): mortality rate × average years lost per death
- [ ] Normalize burden scores to 0-100 scale (min-max normalization)
- [ ] Weight by population impact: consider prevalence if data available

### Dimension 2: Trend Urgency Scoring
- [ ] Extract annual percentage change (APC) from previous analysis
- [ ] Assign urgency points: increasing trends = higher urgency
- [ ] Factor in trend acceleration: rapidly increasing > slowly increasing
- [ ] Penalty for declining trends: stable or improving = lower urgency
- [ ] Normalize urgency scores to 0-100 scale

### Dimension 3: Interventionability Scoring
- [ ] Define interventionability dimensions: preventability, treatability, cost-effectiveness
- [ ] Expert assessment: gather public health expert ratings (0-10 scale) for each disease
- [ ] Preventability factors: lifestyle modifiable (high) vs genetic (low)
- [ ] Treatment effectiveness: availability of proven interventions
- [ ] Normalize interventionability scores to 0-100 scale

### Composite Priority Score
- [ ] Define scoring weights: burden (40%), trend urgency (30%), interventionability (30%)
- [ ] Calculate weighted composite score: `Priority = 0.4*Burden + 0.3*Urgency + 0.3*Interventionability`
- [ ] Rank diseases by composite priority score (1 = highest priority)
- [ ] Sensitivity analysis: test different weighting schemes

### Priority Quadrants & Segmentation
- [ ] Create 2×2 matrix: High/Low burden × Increasing/Declining trend
- [ ] Assign diseases to quadrants
- [ ] Quadrant interpretations:
  - **High burden + increasing**: URGENT PRIORITY - immediate action needed
  - **High burden + declining**: MAINTAIN EFFORTS - current programs working
  - **Low burden + increasing**: EMERGING THREAT - early intervention opportunity
  - **Low burden + declining**: SUCCESS STORY - document and sustain
- [ ] Generate quadrant assignments for all diseases

### Resource Allocation Recommendations
- [ ] Current resource allocation: gather existing program budgets (if available)
- [ ] Recommended allocation: proportional to priority scores
- [ ] Gap analysis: over-resourced vs under-resourced diseases
- [ ] Reallocation scenarios: simulate budget shifts
- [ ] Impact projections: estimate mortality reduction from reallocation

### Visualization
- [ ] Priority matrix scatter plot: burden (x-axis) vs urgency (y-axis), point size = interventionability
- [ ] Ranking table: all diseases with scores and ranks
- [ ] Quadrant plot: 2×2 matrix with disease labels
- [ ] Budget allocation chart: current vs recommended
- [ ] Save figures: `reports/figures/disease_prioritization_*.png/pdf`

### Policy Brief Development
- [ ] Executive summary: top 3-5 priority diseases
- [ ] Methodology explanation: scoring framework and weights
- [ ] Detailed findings: scores and rankings for all diseases
- [ ] Recommendations: specific resource allocation actions
- [ ] Implementation roadmap: phased approach to reallocation

### Testing & Validation
- [ ] Unit tests for scoring functions
- [ ] Validate normalization: all scores 0-100
- [ ] Test ranking logic: ties handled appropriately
- [ ] Sensitivity analysis: how do results change with different weights?
- [ ] Expert review: validate interventionability assessments

---

## 📌 Notes

**Prioritization Framework Components**:

1. **Burden Magnitude** (0-100 score)
   - Based on 2019 mortality rate + YLL estimates
   - Normalized: Highest burden disease = 100, lowest = proportional score

2. **Trend Urgency** (0-100 score)
   - Increasing trend = higher urgency (100 for fastest increase)
   - Declining trend = lower urgency (0 for fastest decline)
   - Based on APC from US-03

3. **Interventionability** (0-100 score)
   - Preventability: Lifestyle modifiable (smoking, diet) = high score
   - Treatability: Evidence-based interventions available = high score
   - Cost-effectiveness: $ per DALY averted = inverse score

**Scoring Example**:
```python
import polars as pl
from sklearn.preprocessing import MinMaxScaler

# Normalize burden to 0-100
scaler = MinMaxScaler(feature_range=(0, 100))
df['burden_score'] = scaler.fit_transform(df[['mortality_rate']])

# Urgency score: map APC to 0-100
# Increasing trend (positive APC) = high urgency
# Declining trend (negative APC) = low urgency
df = df.with_columns([
    pl.when(pl.col('apc') > 0)
    .then(pl.col('apc') / pl.col('apc').max() * 100)
    .when(pl.col('apc') < 0)
    .then(0)
    .otherwise(50)
    .alias('urgency_score')
])

# Composite priority score
df = df.with_columns([
    (0.4 * pl.col('burden_score') + 
     0.3 * pl.col('urgency_score') + 
     0.3 * pl.col('interventionability_score'))
    .alias('priority_score')
])
```

**Priority Quadrants**:

| Quadrant | Burden | Trend | Interpretation |
|----------|--------|-------|----------------|
| Q1: URGENT PRIORITY | High | Increasing | Immediate action needed |
| Q2: MAINTAIN EFFORTS | High | Declining | Current programs working, sustain |
| Q3: EMERGING THREAT | Low | Increasing | Early intervention opportunity |
| Q4: SUCCESS STORY | Low | Declining | Document best practices |

**Interventionability Assessment Table**:

| Disease | Preventability | Treatability | Cost-Effectiveness | Score |
|---------|---------------|--------------|-------------------|-------|
| Cancer | Moderate (smoking, diet) | High (screening, surgery) | Moderate ($$) | 70 |
| Stroke | High (BP control, diet) | Moderate (acute care) | High ($) | 85 |
| IHD | High (cholesterol, exercise) | High (statin, surgery) | High ($) | 90 |

---

## Implementation Plan

### 1. Feature Overview & Component Analysis

**Purpose**: Create evidence-based disease prioritization framework combining burden magnitude, trend urgency, and interventionability to guide resource allocation.

**Primary User Role**: Public Health Policy Director

**Reuse Strategy**:
- ✅ **US-03 outputs**: Trend analysis (APC, slopes) from `results/tables/mortality_trend_analysis.csv`
- ✅ **US-05 outputs**: Burden comparisons from `results/tables/disease_burden_comparative_analysis.csv`
- 🆕 **`shared/src/analysis/prioritization.py`** - Priority scoring module
- 🆕 **`config/prioritization_weights.yml`** - Scoring weights and thresholds
- 🆕 **`shared/data/2_external/disease_interventionability_scores.csv`** - Expert assessments
- 🆕 **`results/tables/disease_burden_prioritization_matrix.csv`** - Final priority rankings

### 3. ML Model Evaluation & Selection

**N/A** - Multi-criteria decision analysis (MCDA), not ML. Uses weighted scoring framework.

### 4-6. Code Specifications

**Prioritization Functions** (`shared/src/analysis/prioritization.py`):

```python
import polars as pl
import numpy as np
from sklearn.preprocessing import MinMaxScaler
from loguru import logger
from pathlib import Path
from typing import Dict, Tuple

def normalize_scores(
    df: pl.DataFrame,
    column: str,
    ascending: bool = True
) -> pl.DataFrame:
    """Normalize scores to 0-100 scale.
    
    Args:
        df: DataFrame with scores
        column: Column to normalize
        ascending: If True, higher raw value = higher score
        
    Returns:
        DataFrame with added normalized column
    """
    values = df[column].to_numpy().reshape(-1, 1)
    scaler = MinMaxScaler(feature_range=(0, 100))
    normalized = scaler.fit_transform(values).flatten()
    
    if not ascending:
        normalized = 100 - normalized  # Reverse scale
    
    df_normalized = df.with_columns([
        pl.Series(name=f'{column}_normalized', values=normalized)
    ])
    
    logger.info(f"Normalized {column}: min={normalized.min():.1f}, max={normalized.max():.1f}")
    return df_normalized


def calculate_burden_score(df: pl.DataFrame) -> pl.DataFrame:
    """Calculate burden magnitude score (0-100).
    
    Args:
        df: DataFrame with [disease_category, mortality_rate_2019]
        
    Returns:
        DataFrame with burden_score column
    """
    df_scored = normalize_scores(df, 'mortality_rate_2019', ascending=True)
    df_result = df_scored.rename({'mortality_rate_2019_normalized': 'burden_score'})
    
    logger.info(f"Calculated burden scores for {df_result.height} diseases")
    return df_result


def calculate_urgency_score(df: pl.DataFrame) -> pl.DataFrame:
    """Calculate trend urgency score (0-100).
    
    Increasing trends = high urgency (toward 100)
    Declining trends = low urgency (toward 0)
    
    Args:
        df: DataFrame with [disease_category, apc]
        
    Returns:
        DataFrame with urgency_score column
    """
    # Map APC to 0-100 scale
    # Positive APC (increasing) -> high score
    # Negative APC (declining) -> low score
    max_apc = df['apc'].abs().max()
    
    df_scored = df.with_columns([
        pl.when(pl.col('apc') > 0)
        .then((pl.col('apc') / max_apc) * 100)
        .when(pl.col('apc') < 0)
        .then(((pl.col('apc') / max_apc) * -50) + 50)  # Scale -max to 0, 0 to 50
        .otherwise(50)
        .alias('urgency_score')
    ])
    
    logger.info(f"Calculated urgency scores for {df_scored.height} diseases")
    return df_scored


def load_interventionability_scores(filepath: str) -> pl.DataFrame:
    """Load expert-assessed interventionability scores.
    
    Args:
        filepath: Path to interventionability CSV
        
    Returns:
        DataFrame with [disease_category, interventionability_score]
    """
    df = pl.read_csv(filepath)
    
    required_cols = ['disease_category', 'interventionability_score']
    if not all(col in df.columns for col in required_cols):
        raise ValueError(f"Interventionability file must contain: {required_cols}")
    
    # Validate scores 0-100
    invalid = df.filter((pl.col('interventionability_score') < 0) | (pl.col('interventionability_score') > 100))
    if invalid.height > 0:
        raise ValueError(f"Interventionability scores must be 0-100, found {invalid.height} invalid")
    
    logger.info(f"Loaded interventionability scores for {df.height} diseases")
    return df


def calculate_composite_priority_score(
    df: pl.DataFrame,
    burden_weight: float = 0.4,
    urgency_weight: float = 0.3,
    interventionability_weight: float = 0.3
) -> pl.DataFrame:
    """Calculate weighted composite priority score.
    
    Args:
        df: DataFrame with burden_score, urgency_score, interventionability_score
        burden_weight: Weight for burden magnitude
        urgency_weight: Weight for trend urgency
        interventionability_weight: Weight for interventionability
        
    Returns:
        DataFrame with priority_score and priority_rank
    """
    # Validate weights sum to 1.0
    total_weight = burden_weight + urgency_weight + interventionability_weight
    if abs(total_weight - 1.0) > 0.001:
        raise ValueError(f"Weights must sum to 1.0, got {total_weight}")
    
    df_scored = df.with_columns([
        (burden_weight * pl.col('burden_score') +
         urgency_weight * pl.col('urgency_score') +
         interventionability_weight * pl.col('interventionability_score'))
        .alias('priority_score')
    ]).with_columns([
        pl.col('priority_score').rank(method='ordinal', descending=True).alias('priority_rank')
    ])
    
    logger.info(f"Calculated composite priority scores (weights: {burden_weight}/{urgency_weight}/{interventionability_weight})")
    return df_scored


def assign_priority_quadrants(df: pl.DataFrame) -> pl.DataFrame:
    """Assign diseases to priority quadrants based on burden and trend.
    
    Args:
        df: DataFrame with burden_score, urgency_score
        
    Returns:
        DataFrame with quadrant column
    """
    # Quadrant thresholds
    burden_threshold = df['burden_score'].median()
    urgency_threshold = 50.0  # Neutral point
    
    df_quadrants = df.with_columns([
        pl.when((pl.col('burden_score') >= burden_threshold) & (pl.col('urgency_score') >= urgency_threshold))
        .then(pl.lit('Q1_URGENT_PRIORITY'))
        .when((pl.col('burden_score') >= burden_threshold) & (pl.col('urgency_score') < urgency_threshold))
        .then(pl.lit('Q2_MAINTAIN_EFFORTS'))
        .when((pl.col('burden_score') < burden_threshold) & (pl.col('urgency_score') >= urgency_threshold))
        .then(pl.lit('Q3_EMERGING_THREAT'))
        .otherwise(pl.lit('Q4_SUCCESS_STORY'))
        .alias('quadrant')
    ])
    
    quadrant_counts = df_quadrants.group_by('quadrant').agg(pl.count().alias('count'))
    logger.info(f"Quadrant distribution: {quadrant_counts.to_dicts()}")
    
    return df_quadrants


def sensitivity_analysis(
    df: pl.DataFrame,
    weight_scenarios: List[Tuple[float, float, float]]
) -> Dict[str, pl.DataFrame]:
    """Test priority rankings with different weight scenarios.
    
    Args:
        df: DataFrame with component scores
        weight_scenarios: List of (burden_wt, urgency_wt, interventionability_wt) tuples
        
    Returns:
        Dictionary of scenario_name: ranked_dataframe
    """
    results = {}
    
    for i, (b_wt, u_wt, i_wt) in enumerate(weight_scenarios):
        scenario_name = f"scenario_{i+1}_B{int(b_wt*100)}_U{int(u_wt*100)}_I{int(i_wt*100)}"
        
        df_scenario = calculate_composite_priority_score(df, b_wt, u_wt, i_wt)
        results[scenario_name] = df_scenario.select([
            'disease_category', 'priority_score', 'priority_rank'
        ])
        
        logger.info(f"{scenario_name}: Top priority = {df_scenario.sort('priority_rank')['disease_category'][0]}")
    
    return results
```

**Configuration File** (`config/prioritization_weights.yml`):

```yaml
# Disease Burden Prioritization Configuration

# Default scoring weights (must sum to 1.0)
scoring_weights:
  burden_magnitude: 0.4
  trend_urgency: 0.3
  interventionability: 0.3

# Quadrant thresholds
quadrant_thresholds:
  burden_cutoff: median  # Or specific value like 50
  urgency_cutoff: 50

# Sensitivity analysis scenarios
sensitivity_scenarios:
  - name: "equal_weights"
    burden: 0.33
    urgency: 0.33
    interventionability: 0.34
  
  - name: "burden_heavy"
    burden: 0.5
    urgency: 0.25
    interventionability: 0.25
  
  - name: "urgency_heavy"
    burden: 0.25
    urgency: 0.5
    interventionability: 0.25
```

### 10. Testing Strategy

```python
# shared/tests/unit/test_prioritization.py

def test_normalize_scores():
    """Test score normalization to 0-100."""
    df = pl.DataFrame({'values': [10, 50, 100]})
    
    result = normalize_scores(df, 'values', ascending=True)
    
    assert result['values_normalized'].min() == 0.0
    assert result['values_normalized'].max() == 100.0
    assert result['values_normalized'][1] == pytest.approx(44.44, rel=0.1)


def test_calculate_composite_priority_score():
    """Test composite scoring calculation."""
    df = pl.DataFrame({
        'disease_category': ['A', 'B', 'C'],
        'burden_score': [100, 50, 0],
        'urgency_score': [100, 50, 0],
        'interventionability_score': [100, 50, 0]
    })
    
    result = calculate_composite_priority_score(df, 0.4, 0.3, 0.3)
    
    assert 'priority_score' in result.columns
    assert 'priority_rank' in result.columns
    assert result['priority_rank'][0] == 1  # Disease A = highest priority
    assert result['priority_score'][0] == 100.0


def test_invalid_weights_raises_error():
    """Test that invalid weights raise ValueError."""
    df = pl.DataFrame({
        'burden_score': [100],
        'urgency_score': [50],
        'interventionability_score': [80]
    })
    
    with pytest.raises(ValueError, match="Weights must sum to 1.0"):
        calculate_composite_priority_score(df, 0.5, 0.5, 0.5)  # Sums to 1.5


def test_assign_priority_quadrants():
    """Test quadrant assignment."""
    df = pl.DataFrame({
        'disease_category': ['A', 'B', 'C', 'D'],
        'burden_score': [80, 80, 20, 20],
        'urgency_score': [80, 20, 80, 20]
    })
    
    result = assign_priority_quadrants(df)
    
    assert result['quadrant'][0] == 'Q1_URGENT_PRIORITY'
    assert result['quadrant'][1] == 'Q2_MAINTAIN_EFFORTS'
    assert result['quadrant'][2] == 'Q3_EMERGING_THREAT'
    assert result['quadrant'][3] == 'Q4_SUCCESS_STORY'
```

### 11. Implementation Steps

**Phase 1: Data Integration**
- [ ] Load burden metrics from US-05 output
- [ ] Load trend metrics (APC) from US-03 output
- [ ] Create interventionability assessment CSV with expert scores
- [ ] Validate all input data

**Phase 2: Scoring Implementation**
- [ ] Implement burden score normalization
- [ ] Implement urgency score calculation
- [ ] Load interventionability scores
- [ ] Implement composite priority scoring
- [ ] Unit test all scoring functions

**Phase 3: Prioritization Analysis**
- [ ] Calculate priority scores with default weights
- [ ] Assign priority quadrants
- [ ] Perform sensitivity analysis with alternative weights
- [ ] Generate priority rankings table

**Phase 4: Visualization & Reporting**
- [ ] Create priority matrix scatter plot (burden vs urgency)
- [ ] Create ranking table visualization
- [ ] Create sensitivity analysis comparison chart
- [ ] Generate 8-10 page policy brief
- [ ] Document methodology and recommendations

**Phase 5: Testing & Validation**
- [ ] Run all unit tests
- [ ] Integration test full pipeline
- [ ] Expert review of interventionability scores
- [ ] Stakeholder review of priority rankings

### 14. Data Quality & Validation

**Critical Validations**:
- All scores normalized to 0-100 range
- Weights sum to 1.0 in composite calculation
- Priority ranks have no gaps (1, 2, 3 for 3 diseases)
- Quadrant assignments logical (high burden + increasing = urgent)

### 18. Success Metrics

- ✅ All diseases scored on 3 dimensions (burden, urgency, interventionability)
- ✅ Composite priority scores calculated and ranked
- ✅ Priority quadrants assigned
- ✅ Sensitivity analysis completed with ≥3 weight scenarios
- ✅ Policy brief generated and reviewed

---

✅ **US-07 Implementation Plan Complete** - Multi-criteria prioritization framework defined

**Polars Prioritization Example**:
```python
import polars as pl
from sklearn.preprocessing import MinMaxScaler
from loguru import logger

# Load mortality trends with APC
df = pl.read_parquet("shared/data/3_interim/mortality_trends_integrated_clean.parquet")

# Get latest year data
df_latest = df.filter(pl.col('year') == 2019)

# Burden magnitude score (normalize to 0-100)
df_scored = df_latest.with_columns([
    ((pl.col('mortality_rate') - pl.col('mortality_rate').min()) / 
     (pl.col('mortality_rate').max() - pl.col('mortality_rate').min()) * 100)
    .alias('burden_score')
])

# Add trend urgency from previous analysis
df_scored = df_scored.join(df_apc, on='disease_category', how='left')
df_scored = df_scored.with_columns([
    pl.when(pl.col('apc') > 2).then(100)
      .when(pl.col('apc') > 0).then(70)
      .when(pl.col('apc') > -2).then(40)
      .otherwise(20)
      .alias('urgency_score')
])

# Interventionability (manual scoring or from config)
interventionability = {
    'cancer': 60,  # Moderate - screening helps but genetics important
    'stroke': 90,   # High - preventable via hypertension control
    'ischemic_heart_disease': 85  # High - lifestyle and medication
}
df_scored = df_scored.with_columns([
    pl.col('disease_category').map_dict(interventionability).alias('interventionability_score')
])

# Composite priority score
df_scored = df_scored.with_columns([
    (0.4 * pl.col('burden_score') + 
     0.3 * pl.col('urgency_score') + 
     0.3 * pl.col('interventionability_score'))
    .alias('priority_score')
])

# Rank by priority
df_priority = df_scored.with_columns([
    pl.col('priority_score').rank(descending=True).alias('priority_rank')
])

logger.info(f"✓ Priority scoring complete for {len(df_priority)} diseases")
```

**Interventionability Assessment Criteria**:

**High Interventionability (70-100)**:
- Strong evidence base for prevention (e.g., smoking cessation → lung cancer)
- Cost-effective interventions available (e.g., statins for IHD)
- Controllable risk factors (e.g., hypertension → stroke)

**Moderate Interventionability (40-69)**:
- Some prevention opportunities but limited effectiveness
- Treatment available but expensive or limited reach
- Mix of controllable and non-controllable factors

**Low Interventionability (0-39)**:
- Largely genetic or unavoidable risk factors
- Limited proven prevention strategies
- Expensive treatments with modest effectiveness

**Expected Priority Rankings** (hypotheses):
1. **High Priority**: Diseases with high burden + increasing trend + high interventionability
2. **Moderate Priority**: High burden but declining (maintain programs) OR low burden but rapidly increasing
3. **Low Priority**: Low burden + declining trends

**Scoring Weight Sensitivity**:
- Test alternative weights: equal weights (33/33/33), burden-heavy (50/25/25), urgency-heavy (25/50/25)
- Document robustness: do top priorities remain stable across weighting schemes?

**Output Schema**:
```yaml
# results/tables/disease_burden_prioritization_matrix.csv
columns:
  disease_category: categorical
  burden_score: float  # 0-100
  urgency_score: float  # 0-100
  interventionability_score: float  # 0-100
  priority_score: float  # 0-100 composite
  priority_rank: int  # 1 = highest priority
  quadrant: string  # high_burden_increasing, etc.
  recommendation: string  # increase_resources, maintain, etc.
```

**Policy Brief Structure**:
1. Executive summary (1-2 pages)
2. Methodology & scoring framework (2 pages)
3. Findings: rankings and scores (2-3 pages)
4. Recommendations: resource allocation (2-3 pages)
5. Appendix: sensitivity analysis, data sources
