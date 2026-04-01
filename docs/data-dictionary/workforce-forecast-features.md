# Workforce Forecast Features Data Dictionary

**Dataset**: `shared/data/3_interim/workforce_forecast_features.parquet`  
**Created**: 2026-03-16  
**User Story**: PS-001-US-05  
**Purpose**: Time series and demographic features for workforce forecasting models

---

## Dataset Overview

| Attribute | Value |
|-----------|-------|
| **Rows** | 1,794 |
| **Columns** | 24 |

**Expected Range**: -10% to 15%  
**Null Handling**: First year (2006) has null for each profession-sector (no previous year)  
**Use Case**: Identify annual growth trends and detect growth rate changes  
**Example**: If doctors in public sector grew from 1,000 to 1,050, growth_rate_yoy = 5.0%

---

### 2. growth_rate_3yr_ma

**Type**: Float32  
**Category**: Growth  
**Description**: 3-year rolling average of year-over-year growth rate (smoothed trend)

**Formula**:
```
growth_rate_3yr_ma = mean(growth_rate_yoy_{t}, growth_rate_yoy_{t-1}, growth_rate_yoy_{t-2})
```

**Expected Range**: -10% to 15%  
**Null Handling**: First 2 years (2006-2007) have nulls (insufficient data for 3-year window)  
**Use Case**: Smooth temporal noise to reveal underlying growth trends  
**Example**: If growth rates are 4%, 5%, 6% for years t-2, t-1, t, then 3yr_ma = 5.0%

---

### 3. growth_acceleration

**Type**: Float32  
**Category**: Growth  
**Description**: Change in growth rate year-over-year (momentum indicator)

**Formula**:
```
growth_acceleration = growth_rate_yoy_t - growth_rate_yoy_{t-1}
```

**Expected Range**: -20% to 20%  
**Null Handling**: First year (2006) has null  
**Use Case**: Detect momentum shifts (growth speeding up or slowing down)  
**Example**: If growth was 5% last year and 7% this year, acceleration = 2.0%

---

### 4. growth_index_2006

**Type**: Float32  
**Category**: Growth  
**Description**: Cumulative growth index with 2006 as base year (2006 = 100)

**Formula**:
```
growth_index_2006 = (count_t / count_2006) * 100
```

**Expected Range**: 90 to 200  
**Null Handling**: No nulls (all years referenced to 2006 baseline)  
**Use Case**: Track cumulative workforce growth over entire time period  
**Example**: If 2006 workforce was 1,000 and 2019 is 1,300, index = 130.0

---

## Lag Features

Historical workforce values used for autocorrelation modeling and time series prediction.

### 5. lag_1_workforce

**Type**: Int32  
**Category**: Lag  
**Description**: Workforce count from previous year (t-1)

**Formula**:
```
lag_1_workforce = total_workforce_{t-1}
```

**Expected Range**: 0 to 50,000  
**Null Handling**: First year (2006) has null  
**Use Case**: Strong autocorrelation - previous year workforce predicts current year  
**Example**: For year 2010, lag_1 = workforce in 2009

---

### 6. lag_2_workforce

**Type**: Int32  
**Category**: Lag  
**Description**: Workforce count from 2 years prior (t-2)

**Formula**:
```
lag_2_workforce = total_workforce_{t-2}
```

**Expected Range**: 0 to 50,000  
**Null Handling**: First 2 years (2006-2007) have nulls  
**Use Case**: Capture 2-year autocorrelation patterns  
**Example**: For year 2010, lag_2 = workforce in 2008

---

### 7. lag_3_workforce

**Type**: Int32  
**Category**: Lag  
**Description**: Workforce count from 3 years prior (t-3)

**Formula**:
```
lag_3_workforce = total_workforce_{t-3}
```

**Expected Range**: 0 to 50,000  
**Null Handling**: First 3 years (2006-2008) have nulls  
**Use Case**: Identify medium-term autocorrelation patterns  
**Example**: For year 2010, lag_3 = workforce in 2007

---

### 8. lag_5_workforce

**Type**: Int32  
**Category**: Lag  
**Description**: Workforce count from 5 years prior (t-5)

**Formula**:
```
lag_5_workforce = total_workforce_{t-5}
```

**Expected Range**: 0 to 50,000  
**Null Handling**: First 5 years (2006-2010) have nulls  
**Use Case**: Aligns with medical training pipeline duration (5-year lags relevant for workforce planning)  
**Example**: For year 2011, lag_5 = workforce in 2006

---

## Trend Features

Linear trend components extracted from time series regression analysis.

### 9. trend_component

**Type**: Float32  
**Category**: Trend  
**Description**: Fitted linear trend values from regression (y = slope * year + intercept)

**Formula**:
```
trend_component = trend_slope * year + trend_intercept
```

**Expected Range**: 0 to 50,000  
**Null Handling**: No nulls (calculated for all years)  
**Use Case**: Represent long-term linear growth trajectory  
**Example**: If slope=50 and intercept=-90,000, then for 2010: trend = 50*2010 - 90,000 = 10,500

---

### 10. trend_slope

**Type**: Float32  
**Category**: Trend  
**Description**: Annual change in workforce from linear regression (workforce per year)

**Formula**:
```
trend_slope = Linear regression coefficient (dY/dX)
```

**Expected Range**: -500 to 2,000 (workforce/year)  
**Null Handling**: No nulls (same slope for all years within profession-sector)  
**Use Case**: Quantify long-term growth rate (annual increase/decrease)  
**Example**: slope=50 means workforce increases by 50 people per year on average

---

### 11. trend_r_squared

**Type**: Float32  
**Category**: Trend  
**Description**: R² value indicating strength of linear trend fit

**Formula**:
```
trend_r_squared = 1 - (SS_residual / SS_total)
```

**Expected Range**: 0 to 1  
**Null Handling**: No nulls  
**Use Case**: Assess how well linear trend explains variance (R²>0.8 = strong trend)  
**Example**: R²=0.95 means 95% of variance explained by linear trend

---

### 12. detrended_count

**Type**: Float32  
**Category**: Trend  
**Description**: Residuals from linear trend (actual - fitted)

**Formula**:
```
detrended_count = total_workforce - trend_component
```

**Expected Range**: -5,000 to 5,000  
**Null Handling**: No nulls  
**Use Case**: Analyze deviations from long-term trend (residual patterns may indicate cyclical components)  
**Example**: If actual=11,000 and trend=10,500, detrended=500 (above trend)

---

### 13. trend_direction

**Type**: Int32  
**Category**: Trend  
**Description**: Binary flag indicating trend direction (1=increasing, 0=decreasing/flat)

**Formula**:
```
trend_direction = 1 if trend_slope > 0 else 0
```

**Expected Range**: 0 or 1  
**Null Handling**: No nulls  
**Use Case**: Categorical trend classification for modeling  
**Example**: Positive slope → trend_direction=1 (growing workforce)

---

## Sector Ratio Features

Features reflecting sector distribution and workforce composition (profession-year level).

### 14. public_private_ratio

**Type**: Float32  
**Category**: Ratio  
**Description**: Ratio of public sector to private sector workforce

**Formula**:
```
public_private_ratio = public_sector_count / private_sector_count
```

**Expected Range**:
- Doctors: 1.2 to 2.5
- Nurses: 1.5 to 3.0
- Pharmacists: 0.8 to 2.0

**Null Handling**: Null if private_sector_count = 0 (division by zero)  
**Use Case**: Track public-private workforce balance shifts over time  
**Example**: If public=1,200 and private=600, ratio=2.0 (public sector is 2x larger)

---

### 15. active_practice_rate

**Type**: Float32  
**Category**: Ratio  
**Description**: Percentage of total workforce actively practicing (employed)

**Formula**:
```
active_practice_rate = ((public + private) / (public + private + not_active)) * 100
```

**Expected Range**: 75% to 95%  
**Null Handling**: No nulls (uses all sectors)  
**Use Case**: Monitor workforce utilization and identify inactive workforce trends  
**Example**: If active=1,800 and not_active=200, rate=90.0%

---

### 16. sector_concentration

**Type**: Float32  
**Category**: Ratio  
**Description**: Herfindahl-Hirschman Index (HHI) measuring sector concentration

**Formula**:
```
HHI = sum((sector_count_i / total_count)^2) for all sectors
```

**Expected Range**: 0.33 (perfectly distributed) to 1.0 (complete concentration)  
**Null Handling**: No nulls  
**Use Case**: Measure workforce concentration across sectors (high HHI = concentrated in one sector)  
**Example**: If all workforce in one sector, HHI=1.0; if evenly split across 3, HHI≈0.33

---

## Feature Validation Summary

### Null Patterns (Expected)

| Feature | Null Years | Reason |
|---------|-----------|--------|
| `growth_rate_yoy` | 2006 | No previous year for comparison |
| `growth_rate_3yr_ma` | 2006-2007 | Insufficient data for 3-year window |
| `growth_acceleration` | 2006 | No previous growth rate |
| `lag_1_workforce` | 2006 | No t-1 data |
| `lag_2_workforce` | 2006-2007 | No t-2 data |
| `lag_3_workforce` | 2006-2008 | No t-3 data |
| `lag_5_workforce` | 2006-2010 | No t-5 data |

### Multicollinearity Warnings

**High Correlation Pairs** (|r| > 0.95):
- `total_workforce` ↔ `trend_component`: r=0.998 (expected - trend fitted to target)
- `lag_1_workforce` ↔ `lag_2_workforce`: High autocorrelation (expected)

**Recommendation**: When modeling, select ONE feature from highly correlated pairs to avoid multicollinearity issues.

### Feature Selection Guidance

**For Modeling**:
1. **Start with**: `lag_1_workforce`, `growth_rate_yoy`, `trend_slope`, `public_private_ratio`
2. **Add selectively**: `lag_2` or `lag_3` (not both), `trend_r_squared`, `active_practice_rate`
3. **Avoid combining**: `total_workforce` + `trend_component` (perfect multicollinearity)

**Model-Specific Considerations**:
- **ARIMA/SARIMA**: Lag features may be redundant (ARIMA handles autoregression)
- **Linear Regression**: Include growth and trend features
- **Random Forest**: Can handle correlated features (feature importance helps select)
- **Neural Networks**: Normalize features, may benefit from all features

---

## Usage Examples

### Load Feature Dataset

```python
import polars as pl

# Load features
df = pl.read_parquet('data/4_processed/workforce_forecast_features.parquet')

# Filter to specific profession-sector
doctors_public = df.filter(
    (pl.col('profession') == 'Doctors') &
    (pl.col('sector') == 'Public Sector')
).sort('year')

# Select feature subset for modeling
features_for_model = df.select([
    'year', 'profession', 'sector',
    'lag_1_workforce', 'growth_rate_yoy', 'trend_slope',
    'public_private_ratio', 'active_practice_rate'
])
```

### Handle Nulls for Modeling

```python
# Option 1: Drop rows with any nulls (loses early years)
df_complete = df.drop_nulls()

# Option 2: Drop only lag-5 feature (keeps more data)
df_no_lag5 = df.drop('lag_5_workforce').drop_nulls()

# Option 3: Filter to years with complete data (2011+)
df_2011_onwards = df.filter(pl.col('year') >= 2011)
```

---

## Metadata

**Feature Engineering Script**: `src/problem-statement-005-workforce-demand-forecasting/scripts/04_engineer_forecast_features.py`  
**Feature Module**: `src/features/workforce_features.py`  
**Configuration**: `config/analysis.yml` (problem_statement_005.feature_engineering)  
**Validation Report**: `data/4_processed/workforce_forecast_features_metadata.json`  
**Unit Tests**: `src/problem-statement-005-workforce-demand-forecasting/tests/test_workforce_features.py`

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-09 | Initial feature set created with 16 features across 4 categories |

---

## Contact

**Data Science Team**  
For questions or feature requests, contact the project team or create an issue in the repository.
