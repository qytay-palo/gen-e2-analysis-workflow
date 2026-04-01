# User Story: 3 - Temporal Feature Engineering for Forecasting

**As a** data scientist,  
**I want** to engineer temporal features and transformations from mortality time series,  
**so that** I can prepare analysis-ready datasets optimized for forecasting model training.

## 1. 🎯 Acceptance Criteria

1. **Lag Features Created**
   - Generate lag variables (t-1, t-2, t-3, t-5 years) for each disease
   - Create rolling averages (3-year, 5-year windows) for trend smoothing
   - Calculate year-over-year change metrics (absolute and percentage)
   - Ensure no data leakage (lag features only use past observations)

2. **Stationary Transformations Applied**
   - Apply first-differencing where ADF test indicated non-stationarity
   - Apply log transformation if variance increases with level
   - Document transformation strategy by disease
   - Validate transformed series are stationary (re-run ADF tests)

3. **Temporal Indicators Created**
   - Year index (1990=0, 1991=1, etc.) for trend modeling
   - Decade indicators (1990s, 2000s, 2010s) for structural change analysis
   - Time since baseline (years from 1990start)
   - Period indicators for inflection points identified in User Story 2

4. **External Regression Features** (Optional Enhancement)
   - Integrate population aging metrics if available
   - Include obesity prevalence trends (from supplementary data)
   - Add vaccination coverage indicators where relevant
   - Document correlation with mortality trends

5. **Feature Dataset Saved**
   - Consolidated feature matrix for each disease
   - Train-test split markers (pre-2015 train, 2015-2019 test)
   - Comprehensive data dictionary documenting all engineered features
   - Location: `shared/data/3_interim/forecasting/mortality_features_{timestamp}.parquet`

## 2. 🔒 Technical Constraints

- **Primary library**: Polars for feature engineering (leverage lazy evaluation)
- **Transformation library**: `statsmodels` for differencing operations
- **Feature validation**: Ensure no future information leakage in lag variables
- **Memory optimization**: Use Polars' lazy evaluation for large feature sets
- **Documentation**: Each feature must have clear business interpretation

## 3. 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../domain_knowledge/disease-burden-feature-engineering-guide.md) - Temporal features section (rolling averages, lag variables, growth metrics)
- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Stationarity transformations, feature engineering patterns

**Key Feature Patterns**:
- **Lag features**: Capture temporal autocorrelation for AR models
- **Rolling averages**: Smooth short-term noise, highlight trends
- **Differencing**: Remove trend and achieve stationarity
- **YoY changes**: Interpretable measure of trend magnitude

## 4. 📦 Dependencies

**External Packages**:
- `polars>=0.20.0` - Feature engineering
- `statsmodels>=0.14.0` - Differencing, transformations
- `numpy>=1.24.0` - Mathematical operations

**Internal Dependencies**:
- Input: Cleaned data from User Story 1 (`mortality_time_series_clean.parquet`)
- Input: Stationarity test results from User Story 2
- Utility: `shared/src/data_processing/feature_engineering.py` (to be created)

## 5. ✅ Implementation Tasks

### Lag Feature Creation
- ⬜ **Create lag-1 feature** (previous year mortality rate)
- ⬜ **Create lags 2, 3, 5** (2, 3, 5 years prior)
- ⬜ **Generate rolling 3-year average** using Polars `rolling_mean`
- ⬜ **Generate rolling 5-year average** (smoothed long-term trend)
- ⬜ **Calculate year-over-year absolute change** (ASMR_t - ASMR_t-1)
- ⬜ **Calculate year-over-year percent change** (APC formula)

### Stationarity Transformations
- ⬜ **Apply first-differencing** to non-stationary series (based on User Story 2 findings)
- ⬜ **Test log transformation** if heteroskedasticity detected
- ⬜ **Validate transformed series stationarity** (re-run ADF test)
- ⬜ **Save transformation metadata** (which transformations applied per disease)

### Temporal Indicators
- ⬜ **Create year index** (1990=0, normalize time trend)
- ⬜ **Create decade dummies** (1990s, 2000s, 2010s)
- ⬜ **Add inflection point indicators** (binary flags for trend breaks from User Story 2)

### External Features (if data available)
- ⬜ **Load obesity prevalence data** (from `common-health-problems-of-students-examined-obesity-annual.csv`)
-⬜ **Load vaccination coverage data** (from `vaccination-and-immunisation-of-students-annual.csv`)
- ⬜ **Align temporal grain** (match annual frequency)
- ⬜ **Calculate correlation** with mortality trends

### Train-Test Split
- ⬜ **Create split indicator** (train: <2015, test: 2015-2019)
- ⬜ **Validate no data leakage** (test set features only use past data)

### Feature Documentation & Saving
- ⬜ **Create feature data dictionary** (name, formula, interpretation, type)
- ⬜ **Save feature dataset** to `shared/data/3_interim/forecasting/mortality_features.parquet`
- ⬜ **Generate feature summary statistics** (mean, std, range per feature)

## 6. Notes

**Critical Success Factors**:
- No future information leakage in lag features (critical for valid backtesting)
- Transformed series achieve stationarity (validated via ADF tests)
- Feature set enhances forecast accuracy vs raw data alone

**Feature Engineering Strategy by Disease** (adapt based on User Story 2 findings):
- If strong autocorrelation → Prioritize lag features
- If trend-dominated → Prioritize differencing and smoothed trends
- If structural breaks → Include period indicators

**Next Steps**:
- Engineered features feed into User Story 4: Baseline Model Development
- Train-test split enables User Story 5: Model Validation & Backtesting

---

## Implementation Plan

### 1. Feature Overview

Engineer temporal features and transformations from mortality time series to prepare analysis-ready datasets optimized for forecasting model training. Primary user: data scientist preparing feature matrices for ARIMA, Prophet, and other models.

### 2. Component Analysis & Reuse Strategy

**Existing Components**:
- Input data: `shared/data/3_interim/forecasting/mortality_time_series_clean.parquet`
- Stationarity results: `results/tables/problem-statement-006/stationarity_tests.csv` (from US-02)
- Analysis functions: `shared/src/analysis/time_series_analysis.py`

**Components to Create**:
- `shared/src/data_processing/feature_engineering.py` - NEW: Lag features, rolling averages, transformations
- `problem-statements/ps-006-forecasting/notebooks/03_temporal_feature_engineering.ipynb`

### 3. ML Model Evaluation & Selection

Not applicable - feature engineering only.

### 4. Affected Files

**[CREATE] `shared/src/data_processing/feature_engineering.py`**
- Function: `create_lag_features(df: pl.DataFrame, value_col: str, lags: list[int], group_col: str = None) -> pl.DataFrame`
- Function: `create_rolling_features(df: pl.DataFrame, value_col: str, windows: list[int], group_col: str = None) -> pl.DataFrame`
- Function: `apply_differencing(df: pl.DataFrame, value_col: str, order: int = 1, group_col: str = None) -> pl.DataFrame`
- Function: `create_temporal_indicators(df: pl.DataFrame, time_col: str) -> pl.DataFrame`
- Function: `create_train_test_split(df: pl.DataFrame, split_year: int = 2015) -> pl.DataFrame`

### 5-6. Code Specifications

```python
# shared/src/data_processing/feature_engineering.py

import polars as pl
from loguru import logger
from typing import List, Optional
import numpy as np


def create_lag_features(
    df: pl.DataFrame,
    value_col: str,
    lags: List[int],
    group_col: Optional[str] = None
) -> pl.DataFrame:
    """Create lag features for time series forecasting.
    
    Args:
        df: DataFrame with time series
        value_col: Column to create lags for
        lags: List of lag periods (e.g., [1, 2, 3, 5])
        group_col: Optional grouping column
        
    Returns:
        DataFrame with lag features added
    """
    logger.info(f"Creating lag features: {lags}")
    
    df_lagged = df.clone()
    
    for lag in lags:
        lag_col_name = f"{value_col}_lag{lag}"
        
        if group_col:
            df_lagged = df_lagged.with_columns([
                pl.col(value_col).shift(lag).over(group_col).alias(lag_col_name)
            ])
        else:
            df_lagged = df_lagged.with_columns([
                pl.col(value_col).shift(lag).alias(lag_col_name)
            ])
        
        logger.debug(f"Added {lag_col_name}")
    
    return df_lagged


def create_rolling_features(
    df: pl.DataFrame,
    value_col: str,
    windows: List[int],
    group_col: Optional[str] = None
) -> pl.DataFrame:
    """Create rolling average features.
    
    Args:
        df: DataFrame
        value_col: Column to compute rolling averages
        windows: Window sizes (e.g., [3, 5])
        group_col: Optional grouping
        
    Returns:
        DataFrame with rolling features
    """
    logger.info(f"Creating rolling features: windows={windows}")
    
    df_rolling = df.clone()
    
    for window in windows:
        roll_col_name = f"{value_col}_roll{window}"
        
        if group_col:
            df_rolling = df_rolling.with_columns([
                pl.col(value_col).rolling_mean(window_size=window).over(group_col).alias(roll_col_name)
            ])
        else:
            df_rolling = df_rolling.with_columns([
                pl.col(value_col).rolling_mean(window_size=window).alias(roll_col_name)
            ])
    
    return df_rolling


def apply_differencing(
    df: pl.DataFrame,
    value_col: str,
    order: int = 1,
    group_col: Optional[str] = None
) -> pl.DataFrame:
    """Apply differencing transformation.
    
    Args:
        df: DataFrame
        value_col: Column to difference
        order: Differencing order (1 or 2)
        group_col: Optional grouping
        
    Returns:
        DataFrame with differenced column
    """
    logger.info(f"Applying order-{order} differencing to {value_col}")
    
    df_diff = df.clone()
    diff_col_name = f"{value_col}_diff{order}"
    
    if group_col:
        df_diff = df_diff.with_columns([
            pl.col(value_col).diff(n=order).over(group_col).alias(diff_col_name)
        ])
    else:
        df_diff = df_diff.with_columns([
            pl.col(value_col).diff(n=order).alias(diff_col_name)
        ])
    
    return df_diff


def create_temporal_indicators(
    df: pl.DataFrame,
    time_col: str = 'year'
) -> pl.DataFrame:
    """Create temporal indicator features.
    
    Args:
        df: DataFrame with time column
        time_col: Time column name
        
    Returns:
        DataFrame with temporal indicators
    """
    logger.info("Creating temporal indicators")
    
    df_temporal = df.with_columns([
        # Year index (0-based from min year)
        (pl.col(time_col) - pl.col(time_col).min()).alias('year_index'),
        
        # Decade indicators
        (pl.col(time_col) < 2000).alias('decade_1990s'),
        ((pl.col(time_col) >= 2000) & (pl.col(time_col) < 2010)).alias('decade_2000s'),
        (pl.col(time_col) >= 2010).alias('decade_2010s')
    ])
    
    return df_temporal


def create_train_test_split(
    df: pl.DataFrame,
    time_col: str = 'year',
    split_year: int = 2015
) -> pl.DataFrame:
    """Add train/test split indicator.
    
    Args:
        df: DataFrame
        time_col: Time column
        split_year: Year to split train/test
        
    Returns:
        DataFrame with 'split' column
    """
    logger.info(f"Creating train/test split at year {split_year}")
    
    df_split = df.with_columns([
        pl.when(pl.col(time_col) < split_year)
        .then(pl.lit('train'))
        .otherwise(pl.lit('test'))
        .alias('split')
    ])
    
    train_count = df_split.filter(pl.col('split') == 'train').height
    test_count = df_split.filter(pl.col('split') == 'test').height
    
    logger.info(f"Split: {train_count} train, {test_count} test")
    
    return df_split
```

### 7. Implementation Steps

- [ ] Create `shared/src/data_processing/feature_engineering.py`
- [ ] Load stationarity results from US-02 to determine differencing needs
- [ ] Create lag features (1, 2, 3, 5 years)
- [ ] Create rolling averages (3, 5 years)
- [ ] Apply differencing to non-stationary series
- [ ] Create temporal indicators (year_index, decade dummies)
- [ ] Add train/test split (pre-2015 = train)
- [ ] Validate no data leakage
- [ ] Save to `shared/data/3_interim/forecasting/mortality_features.parquet`
- [ ] Generate feature data dictionary

### 10. Testing

```python
def test_lag_features_no_leakage():
    df = pl.DataFrame({'year': [1990, 1991, 1992], 'value': [10, 20, 30]})
    result = create_lag_features(df, 'value', lags=[1])
    
    assert result['value_lag1'][0] is None  # First row has no prior
    assert result['value_lag1'][1] == 10
    assert result['value_lag1'][2] == 20
```

✅ **PLAN READY**
