# Engineer Features for Workforce Forecasting (Lifecycle Stage: Feature Engineering)

**Story ID**: PS-001-US-05  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Scientist preparing workforce forecasting models**,  
I want **to create time series features, lag variables, and demographic demand proxies that enhance forecast accuracy**,  
So that **my forecasting models can leverage historical patterns, seasonality (if any), and demand drivers to produce robust 2020-2030 workforce projections**.

---

## 🎯 Acceptance Criteria

1. **Time series features created**
   - Lag features: workforce counts from t-1, t-2, t-3 years (3-year moving history)
   - Rolling statistics: 3-year moving average, 3-year rolling growth rate
   - Trend component: linear time trend (year index from baseline)
   - Growth momentum: acceleration/deceleration indicators

2. **Demographic demand proxies engineered**
   - Population aging index: % population aged 65+ (derived from admission rate data)
   - Hospital utilization trend: admission rate growth as demand proxy
   - Beds-per-capita trend: facility capacity as complementary demand indicator
   - Demand pressure score: composite metric combining demographic and utilization trends

3. **Sector-specific features**
   - Public-private ratio: ratio of public to private workforce
   - Sector growth differential: difference in public vs private growth rates
   - Market share trends: % workforce in public vs private over time

4. **Data output requirement**
   - Output file: `shared/data/3_interim/workforce_forecast_features.parquet`
   - Format: Parquet (optimized for ML workflows)
   - Schema documented: Yes - in `docs/data_dictionary/workforce-forecast-features.md`
   - Feature engineering report: `results/feature_engineering_summary.md` (feature importance preview)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for feature engineering)
- **Time Series**: Use Polars native methods (avoid pandas for consistency)
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for feature engineering functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#calculation-methodologies) - Understanding workforce metrics calculation patterns
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#objectives) - Objective 2: Develop forecasting models with demographic trends

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Feature engineering and transformations
- `loguru>=0.7.0`: Structured logging
- `pyyaml>=6.0`: Configuration management
- `scikit-learn>=1.3.0` (optional): Feature scaling/normalization if needed

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-02 (Clean workforce data - BLOCKING)
  - PS-001-US-03 (EDA and growth analysis - RECOMMENDED for insights)
  - PS-001-US-04 (Population ratios - BLOCKING for demographic features)
- **Data Sources**: 
  - `shared/data/3_interim/workforce_integrated_clean.parquet`
  - `shared/data/1_raw/utilization/hospital-admission-rate-by-age-and-sex.csv`
  - `shared/data/1_raw/facilities/health-facilities-and-beds-in-inpatient-facilities.csv`
- **Config Files**: `config/analysis.yml` (feature engineering settings: lag windows, smoothing parameters)

---

## ✅ Implementation Tasks

### Time Series Feature Engineering
- [ ] Create lag features: workforce counts from t-1, t-2, t-3
- [ ] Calculate rolling statistics: 3-year moving average, 3-year std dev
- [ ] Calculate rolling growth rates: 3-year rolling CAGR
- [ ] Create trend component: linear time index (0 for 2006, 1 for 2007, etc.)
- [ ] Calculate momentum: difference between current growth and rolling average growth
- [ ] Handle edge cases: missing lags for early years (2006-2008)

### Demographic Demand Proxy Features
- [ ] Extract age distribution from admission rate data (% aged 65+)
- [ ] Calculate aging index: ratio of elderly to working-age population
- [ ] Extract hospital admission rate trends (total admissions growth)
- [ ] Calculate beds-per-capita trend from facility data
- [ ] Create demand pressure score: weighted composite of aging + utilization + beds growth

### Sector-Specific Features
- [ ] Calculate public-private workforce ratio per profession
- [ ] Calculate sector growth differential: public growth % - private growth %
- [ ] Calculate public sector market share: public / (public + private)
- [ ] Create sector shift indicator: change in market share over time

### Feature Validation & Selection
- [ ] Validate no data leakage (only use past data for lags)
- [ ] Check for multicollinearity (correlation matrix for features)
- [ ] Handle missing values (early years with incomplete lags)
- [ ] Normalize/scale features if needed (document decision)
- [ ] Create feature importance preview (correlation with workforce growth)

### Testing & Validation
- [ ] Unit tests for lag feature creation: validate correct time alignment
- [ ] Test rolling statistics: compare against manual calculations
- [ ] Validate no future data leakage: assert lag features use only past data
- [ ] Test edge cases: professions with missing years (nurses pre-2008)
- [ ] Integration test: full feature engineering pipeline

### Documentation
- [ ] Docstrings for all feature engineering functions (Google style)
- [ ] Feature dictionary: `docs/data_dictionary/workforce-forecast-features.md`
  - Document each feature: name, formula, interpretation, range
- [ ] Feature engineering report: `results/feature_engineering_summary.md`
  - Feature correlation analysis
  - Feature importance preview
  - Recommendations for model selection
- [ ] Update README: `shared/src/data_processing/README.md`

---

## 📌 Notes

**Lag Features (Polars)**:
```python
import polars as pl

df_features = (
    df.sort(['profession', 'sector', 'year'])
    .with_columns([
        # Lag features
        pl.col('count').shift(1).over(['profession', 'sector']).alias('count_lag1'),
        pl.col('count').shift(2).over(['profession', 'sector']).alias('count_lag2'),
        pl.col('count').shift(3).over(['profession', 'sector']).alias('count_lag3'),
        
        # Rolling average
        pl.col('count')
          .rolling_mean(window_size=3)
          .over(['profession', 'sector'])
          .alias('count_ma3'),
        
        # Trend component
        (pl.col('year') - pl.col('year').min()).alias('time_trend')
    ])
)
```

**Demand Pressure Score Calculation**:
```python
# Composite demand metric (weighted average)
df_demand = df_demand.with_columns([
    (
        0.5 * pl.col('aging_index_normalized') +
        0.3 * pl.col('admission_rate_growth_normalized') +
        0.2 * pl.col('beds_growth_normalized')
    ).alias('demand_pressure_score')
])
```

**Feature Engineering Best Practices**:
- Document rationale for each feature (why it should improve forecasts)
- Avoid creating too many correlated features (check correlation matrix)
- Prioritize interpretable features (stakeholders need to understand drivers)
- Test feature stability across time periods (avoid overfitting to recent trends)

**Expected Features Table** (document in data dictionary):
| Feature Name | Type | Description | Formula | Range |
|-------------|------|-------------|---------|-------|
| `count_lag1` | Numeric | Workforce count 1 year prior | `count(t-1)` | [0, ∞) |
| `count_ma3` | Numeric | 3-year moving average workforce | `mean(count(t-2:t))` | [0, ∞) |
| `time_trend` | Numeric | Linear time index from 2006 | `year - 2006` | [0, 13] |
| `aging_index` | Numeric | % population aged 65+ | `pop_65+ / pop_total * 100` | [0, 100] |
| `demand_pressure_score` | Numeric | Composite demand metric | See formula above | [0, 1] |

**Known Challenges**:
- Nurses data starts 2008 → lags only available from 2011 (need 3 years history)
- Population aging data may need to be estimated/interpolated for missing years
- External validation: confirm aging trends match official Singapore demographic forecasts

---

## Implementation Plan

### 1. Feature Overview

Create time series features, lag variables, and demographic demand proxies to enhance forecast accuracy. Goal is to enable forecasting models to leverage historical patterns, growth momentum, and demographic drivers for robust 2020-2030 projections.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Cleaned workforce data (US-02), population ratios (US-04)
**Create**: `shared/src/analysis/feature_engineering.py`, feature validation tests

### 3. Key Implementation Components

**[CREATE] `shared/src/analysis/feature_engineering.py`**

```python
"""
Workforce Forecast Feature Engineering
=======================================

Create time series and demographic features for forecasting.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
from pathlib import Path
from typing import List
from loguru import logger


class WorkforceFeatureEngineer:
    """
    Engineer features for workforce forecasting models.
    
    Features Created:
    - Lag features (t-1, t-2, t-3)
    - Rolling statistics (MA, growth rates)
    - Time trends
    - Demographic demand proxies
    """
    
    def __init__(self, workforce_df: pl.DataFrame):
        """Initialize with cleaned workforce DataFrame."""
        self.df = workforce_df.sort(['profession', 'sector', 'year'])
        logger.info(f"Initialized with {len(self.df)} workforce records")
    
    def create_lag_features(self, n_lags: int = 3) -> pl.DataFrame:
        """
        Create lag features for workforce counts.
        
        Args:
            n_lags: Number of lag periods (default: 3 years)
            
        Returns:
            DataFrame with lag columns added
        """
        logger.info(f"Creating {n_lags} lag features...")
        
        df_with_lags = self.df.clone()
        
        for lag in range(1, n_lags + 1):
            df_with_lags = df_with_lags.with_columns([
                pl.col('count')
                .shift(lag)
                .over(['profession', 'sector'])
                .alias(f'count_lag{lag}')
            ])
        
        logger.info(f"✓ Created lag features: count_lag1 to count_lag{n_lags}")
        return df_with_lags
    
    def create_rolling_features(self, window: int = 3) -> pl.DataFrame:
        """
        Create rolling window statistics.
        
        Args:
            window: Rolling window size (default: 3 years)
            
        Returns:
            DataFrame with rolling features
        """
        logger.info(f"Creating {window}-year rolling features...")
        
        df_rolling = self.df.with_columns([
            # Rolling mean
            pl.col('count')
            .rolling_mean(window_size=window)
            .over(['profession', 'sector'])
            .alias(f'count_ma{window}'),
            
            # Rolling std dev
            pl.col('count')
            .rolling_std(window_size=window)
            .over(['profession', 'sector'])
            .alias(f'count_std{window}'),
            
            # Rolling growth rate (CAGR over window)
            ((pl.col('count') / pl.col('count').shift(window))
             .over(['profession', 'sector'])
             .pow(1 / window) - 1)
            .alias(f'rolling_cagr{window}')
        ])
        
        logger.info(f"✓ Created rolling features (MA, STD, CAGR)")
        return df_rolling
    
    def create_time_trend(self, baseline_year: int = 2006) -> pl.DataFrame:
        """
        Create linear time trend variable.
        
        Args:
            baseline_year: Year to use as t=0
            
        Returns:
            DataFrame with time_trend column
        """
        logger.info(f"Creating time trend (baseline={baseline_year})...")
        
        df_trend = self.df.with_columns([
            (pl.col('year') - baseline_year).cast(pl.Int16).alias('time_trend')
        ])
        
        logger.info("✓ Created time_trend feature")
        return df_trend
    
    def create_momentum_features(self) -> pl.DataFrame:
        """
        Create growth momentum indicators.
        
        Momentum = current growth rate - average growth rate
        Indicates acceleration (+) or deceleration (-)
        """
        logger.info("Creating growth momentum features...")
        
        df_momentum = (
            self.df
            .sort(['profession', 'sector', 'year'])
            .with_columns([
                # Current YoY growth
                ((pl.col('count') - pl.col('count').shift(1)) / 
                 pl.col('count').shift(1))
                .over(['profession', 'sector'])
                .alias('yoy_growth')
            ])
            .with_columns([
                # Average growth rate (3-year rolling)
                pl.col('yoy_growth')
                .rolling_mean(window_size=3)
                .over(['profession', 'sector'])
                .alias('avg_growth_3y')
            ])
            .with_columns([
                # Momentum = current - average
                (pl.col('yoy_growth') - pl.col('avg_growth_3y'))
                .alias('growth_momentum')
            ])
        )
        
        logger.info("✓ Created momentum features")
        return df_momentum
    
    def integrate_demographic_features(
        self,
        population_df: pl.DataFrame,
        admission_df: pl.DataFrame
    ) -> pl.DataFrame:
        """
        Integrate demographic demand proxy features.
        
        Args:
            population_df: Population estimates by year
            admission_df: Hospital admission data (for aging index)
            
        Returns:
            DataFrame with demographic features added
        """
        logger.info("Integrating demographic features...")
        
        # Calculate aging index (% population aged 65+)
        aging_index = (
            admission_df
            .filter(pl.col('age_group').str.contains('65'))
            .groupby('year')
            .agg([
                pl.col('denominator').sum().alias('pop_65_plus')
            ])
            .join(
                population_df.select(['year', 'total_population']),
                on='year',
                how='inner'
            )
            .with_columns([
                (pl.col('pop_65_plus') / pl.col('total_population') * 100)
                .alias('aging_index')
            ])
            .select(['year', 'aging_index'])
        )
        
        # Join with workforce data
        df_with_demo = self.df.join(aging_index, on='year', how='left')
        
        logger.info("✓ Integrated demographic features")
        return df_with_demo
    
    def create_all_features(
        self,
        population_df: Optional[pl.DataFrame] = None,
        admission_df: Optional[pl.DataFrame] = None
    ) -> pl.DataFrame:
        """
        Create all forecast features in one pipeline.
        
        Returns:
            DataFrame with all engineered features
        """
        logger.info("=" * 60)
        logger.info("Creating all forecast features")
        logger.info("=" * 60)
        
        df_features = self.df.clone()
        
        # Time series features
        df_features = self.create_lag_features(n_lags=3)
        df_features = self.create_rolling_features(window=3)
        df_features = self.create_time_trend()
        df_features = self.create_momentum_features()
        
        # Demographic features (if data provided)
        if population_df is not None and admission_df is not None:
            df_features = self.integrate_demographic_features(
                population_df, admission_df
            )
        
        logger.success(f"✓ Feature engineering complete: {len(df_features.columns)} total columns")
        
        return df_features
```

### 4. Feature Validation

```python
# shared/tests/unit/test_feature_engineering.py

def test_lag_features_correct_shift():
    """Test lag features shift correctly."""
    df = pl.DataFrame({
        'profession': ['doctors'] * 5,
        'sector': ['public'] * 5,
        'year': [2006, 2007, 2008, 2009, 2010],
        'count': [100, 110, 121, 133, 146]
    })
    
    engineer = WorkforceFeatureEngineer(df)
    df_lagged = engineer.create_lag_features(n_lags=2)
    
    # Year 2008: lag1 should be 110 (2007 value), lag2 should be 100 (2006 value)
    row_2008 = df_lagged.filter(pl.col('year') == 2008)
    assert row_2008['count_lag1'][0] == 110
    assert row_2008['count_lag2'][0] == 100


def test_rolling_mean_calculation():
    """Test 3-year rolling mean."""
    df = pl.DataFrame({
        'profession': ['doctors'] * 5,
        'sector': ['public'] * 5,
        'year': [2006, 2007, 2008, 2009, 2010],
        'count': [100, 110, 120, 130, 140]
    })
    
    engineer = WorkforceFeatureEngineer(df)
    df_rolling = engineer.create_rolling_features(window=3)
    
    # Year 2008: MA3 = (100 + 110 + 120) / 3 = 110
    row_2008 = df_rolling.filter(pl.col('year') == 2008)
    assert row_2008['count_ma3'][0] == pytest.approx(110.0, abs=0.1)
```

### 5. Implementation Steps

**Phase 1: Time Series Features (Days 1-2)**
- [ ] Implement lag features (t-1, t-2, t-3)
- [ ] Implement rolling statistics (MA, STD, CAGR)
- [ ] Implement time trend variable
- [ ] Implement growth momentum features
- [ ] Handle edge cases (missing lags for early years)

**Phase 2: Demographic Features (Day 2)**
- [ ] Extract population aging data from admission rates
- [ ] Calculate aging index (% 65+)
- [ ] Extract hospital utilization trends
- [ ] Create demand pressure composite score

**Phase 3: Sector Features (Day 3)**
- [ ] Calculate public-private ratio
- [ ] Calculate sector growth differential
- [ ] Calculate market share trends

**Phase 4: Validation & Output (Days 3-4)**
- [ ] Unit tests for all feature creation functions
- [ ] Validate feature stability (no nulls in training period)
- [ ] Save feature dataset: `shared/data/3_interim/workforce_forecast_features.parquet`
- [ ] Update data dictionary: `docs/data_dictionary/workforce-forecast-features.md`
- [ ] Generate feature engineering summary report

### 6. Success Metrics

- ✅ All features created without errors
- ✅ Lag features available from 2009+ (3-year history requirement)
- ✅ Demographic features integrated successfully
- ✅ Feature dataset saved in optimized Parquet format
- ✅ Data dictionary updated with feature descriptions

### 7. References

- [Workforce Forecast Features Data Dictionary](../../../data_dictionary/workforce-forecast-features.md)
- Scikit-learn Feature Engineering Guide
- US-02 (cleaned data), US-04 (population ratios)

---

**Implementation Plan Complete for US-05**  
**Dependencies**: US-02 (cleaned data), US-04 (population data)  
**Enables**: US-06 (forecasting models consume these features)
