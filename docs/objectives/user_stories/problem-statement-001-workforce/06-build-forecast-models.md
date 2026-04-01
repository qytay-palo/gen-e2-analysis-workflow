# Build Workforce Supply Forecast Models (Lifecycle Stage: Advanced Analysis & Modeling)

**Story ID**: PS-001-US-06  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: L (8-10 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Workforce Planning Data Scientist**,  
I want **to develop and validate time series forecasting models (ARIMA, Prophet, or regression-based) that project healthcare workforce supply through 2030 with confidence intervals**,  
So that **workforce planners can use evidence-based projections to inform training program capacity and recruitment strategies over the next decade**.

---

## 🎯 Acceptance Criteria

1. **Forecasting models developed**
   - Models trained: ARIMA/SARIMAX, Prophet, and linear regression with demographic features
   - Separate models per profession (doctors, nurses, pharmacists, etc.)
   - Models estimate workforce counts annually from 2020-2030 (11-year forecast horizon)
   - Confidence intervals provided: 80% and 95% prediction intervals

2. **Model performance evaluated**
   - Backtesting completed: train on 2006-2016, test on 2017-2019
   - Accuracy metrics calculated: MAPE, RMSE, MAE for each model and profession
   - Model comparison: identify best-performing model per profession
   - Residual diagnostics: validate model assumptions (normality, no autocorrelation)

3. **Forecast outputs generated**
   - Annual workforce projections 2020-2030 for all professions
   - Projections by sector (public, private) using sector-specific models or proportions
   - Lower/upper confidence bounds for each projection
   - Forecast summary table with point estimates and uncertainty ranges

4. **Data output requirement**
   - Output file: `shared/data/4_processed/workforce_forecasts_2020_2030.parquet`
   - Format: Parquet with columns (profession, sector, year, forecast_count, lower_80, upper_80, lower_95, upper_95, model_name)
   - Schema documented: Yes
   - Model artifacts saved: `shared/models/workforce/` (serialized models for reproducibility)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (data processing); statsmodels/Prophet (forecasting)
- **Forecasting Libraries**: 
  - `statsmodels>=0.14.0`: ARIMA/SARIMAX models
  - `prophet>=1.1.0`: Facebook Prophet for long-term forecasts
  - `scikit-learn>=1.3.0`: Linear regression with exogenous variables
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for model training/evaluation functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md) - Understanding workforce forecasting context and benchmarks
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#objectives) - Objective 2: Develop forecasting models with confidence intervals

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data preparation for modeling
- `statsmodels>=0.14.0`: ARIMA, SARIMAX, ACF/PACF analysis
- `prophet>=1.1.0`: Prophet time series forecasting (optional)
- `scikit-learn>=1.3.0`: Linear regression, cross-validation
- `matplotlib>=3.8.0`: Forecast visualizations
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-05 (Feature engineering - BLOCKING)
  - PS-001-US-04 (Population ratios for demand features - BLOCKING)
- **Data Sources**: `shared/data/3_interim/workforce_forecast_features.parquet`
- **Config Files**: `config/analysis.yml` (model hyperparameters: ARIMA orders, Prophet seasonality settings)

---

## ✅ Implementation Tasks

### Model Development Setup
- [ ] Create modeling script: `shared/src/models/forecast_workforce.py`
- [ ] Load feature-engineered workforce data using Polars
- [ ] Split data: training (2006-2016), validation (2017-2019), forecast (2020-2030)
- [ ] Define model configurations in `config/analysis.yml` (ARIMA orders, Prophet parameters)

### ARIMA/SARIMAX Models
- [ ] Perform ACF/PACF analysis to determine ARIMA orders (p, d, q)
- [ ] Train ARIMA models per profession (test different orders: (1,1,1), (2,1,1), auto-selection)
- [ ] Train SARIMAX models with exogenous variables (population, demand pressure score)
- [ ] Generate forecasts 2020-2030 with confidence intervals
- [ ] Save model diagnostics: residual plots, ACF of residuals

### Prophet Models
- [ ] Prepare data in Prophet format (ds, y columns)
- [ ] Train Prophet models per profession (no seasonality for annual data)
- [ ] Add demographic regressors (aging index, demand pressure)
- [ ] Generate forecasts 2020-2030 with prediction intervals
- [ ] Tune Prophet hyperparameters: changepoint_prior_scale, seasonality_mode

### Linear Regression with Trends
- [ ] Train linear regression: workforce ~ time_trend + lag_features + demographic_features
- [ ] Use rolling window validation (train on 8 years, validate on 2 years)
- [ ] Generate forecasts by extrapolating trend + feature projections
- [ ] Calculate prediction intervals using residual standard error

### Model Evaluation & Selection
- [ ] Calculate accuracy metrics for 2017-2019 validation period:
   - MAPE: Mean Absolute Percentage Error
   - RMSE: Root Mean Squared Error
   - MAE: Mean Absolute Error
- [ ] Compare models per profession (which model has lowest MAPE?)
- [ ] Select best model per profession for final 2020-2030 forecasts
- [ ] Document model selection rationale in modeling report

### Forecast Generation
- [ ] Generate final forecasts using selected models
- [ ] Combine forecasts from all professions into single dataset
- [ ] Create forecast visualizations: historical + forecast with confidence bands
- [ ] Export forecast data to `shared/data/4_processed/workforce_forecasts_2020_2030.parquet`

### Testing & Validation
- [ ] Unit tests for forecast function: validate output shape and columns
- [ ] Test edge cases: professions with short time series (nurses, allied health)
- [ ] Validate confidence intervals: check coverage on validation set
- [ ] Integration test: end-to-end modeling pipeline
- [ ] Residual diagnostics: test for normality, no autocorrelation (Ljung-Box test)

### Documentation
- [ ] Docstrings for all modeling functions (Google style)
- [ ] Modeling report: `results/workforce_forecasting_methodology.md`
  - Model specifications, hyperparameters
  - Validation results and model selection rationale
  - Forecast uncertainty discussion
  - Assumptions and limitations
- [ ] Model card: `shared/models/workforce/MODEL_CARD.md` (model metadata, performance, ethical considerations)
- [ ] Update README: `shared/src/models/README.md`

---

## 📌 Notes

**ARIMA Model Example (statsmodels)**:
```python
from statsmodels.tsa.arima.model import ARIMA
import polars as pl

# Prepare data
train_data = df.filter(pl.col('year') <= 2016).select('count').to_numpy().flatten()

# Train ARIMA
model = ARIMA(train_data, order=(1, 1, 1))
fitted_model = model.fit()

# Forecast 2020-2030 (11 years after 2019)
forecast = fitted_model.forecast(steps=11)
forecast_ci = fitted_model.get_forecast(steps=11).conf_int(alpha=0.2)  # 80% CI

logger.info(f"ARIMA(1,1,1) - AIC: {fitted_model.aic}, RMSE: {rmse}")
```

**Prophet Model Example**:
```python
from prophet import Prophet
import pandas as pd

# Prepare data (Prophet requires pandas)
df_prophet = df.select(['year', 'count']).to_pandas()
df_prophet.columns = ['ds', 'y']
df_prophet['ds'] = pd.to_datetime(df_prophet['ds'], format='%Y')

# Train Prophet
model = Prophet(yearly_seasonality=False, weekly_seasonality=False, daily_seasonality=False)
model.add_regressor('aging_index')  # demographic feature
model.fit(df_prophet)

# Forecast
future = model.make_future_dataframe(periods=11, freq='Y')
forecast = model.predict(future)
```

**Model Selection Criteria**:
1. **Lowest MAPE on validation set** (2017-2019)
2. **Stable residuals** (no patterns, normally distributed)
3. **Reasonable confidence intervals** (not too wide, not too narrow)
4. **Interpretability** (stakeholders need to understand model assumptions)

**Expected Model Performance** (target metrics):
- MAPE < 10% for professions with stable trends (doctors, nurses)
- MAPE < 20% for professions with short/volatile history (allied health)
- Coverage: 80% CI should contain ~80% of actual values in validation

**Handling Different Data Lengths**:
- Doctors/pharmacists: full 2006-2019 data (13 years) → robust ARIMA
- Nurses: 2008-2019 data (11 years) → ARIMA or Prophet
- Allied health (2014-2019): only 5 years → use simpler trend extrapolation (linear regression)

**Forecast Assumptions to Document**:
- Assumes historical trends continue (no major policy disruptions)
- Does not account for COVID-19 impact (data ends 2019)
- Confidence intervals reflect historical volatility
- Recommend updating forecasts when post-2020 data available

---

## Implementation Plan

### 1. Forecasting Approach

Build ARIMA, Prophet, and regression-based models to forecast workforce counts 2020-2030. Use engineered features (US-05) to enhance predictions. Evaluate models on RMSE, MAPE, and forecast stability. Output 10-year projections with 95% confidence intervals.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Engineered features (US-05), cleaned data (US-02)
**Create**: `shared/src/models/workforce_forecaster.py`, model evaluation tests

### 3. Key Implementation Components

**[CREATE] `shared/src/models/workforce_forecaster.py`**

```python
"""
Workforce Forecasting Models
=============================

ARIMA, Prophet, and regression-based forecasting for healthcare workforce.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import numpy as np
from pathlib import Path
from typing import Dict, List, Tuple, Optional
from dataclasses import dataclass
from loguru import logger
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.statespace.sarimax import SARIMAX
from prophet import Prophet
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, mean_absolute_percentage_error
import yaml


@dataclass
class ForecastResult:
    """Container for forecast outputs."""
    profession: str
    sector: str
    model_type: str
    year_range: List[int]
    forecast_values: np.ndarray
    lower_bound_95: np.ndarray
    upper_bound_95: np.ndarray
    rmse: float
    mape: float
    

class WorkforceForecaster:
    """
    Build and execute workforce forecasting models.
    
    Models:
    - ARIMA: For professions with 8+ years data
    - Prophet: For professions with trend + seasonality
    - Linear Regression: For professions with <8 years data
    """
    
    def __init__(self, features_df: pl.DataFrame, config_path: str):
        """
        Initialize forecaster with engineered features.
        
        Args:
            features_df: DataFrame with lag/rolling features (from US-05)
            config_path: Path to forecast configuration YAML
        """
        self.df = features_df
        self.config = self._load_config(config_path)
        logger.info(f"Initialized forecaster with {len(self.df)} records")
    
    def _load_config(self, config_path: str) -> dict:
        """Load forecasting configuration."""
        with open(config_path, 'r') as f:
            config = yaml.safe_load(f)
        logger.info(f"Loaded config from {config_path}")
        return config
    
    def fit_arima(
        self,
        timeseries: np.ndarray,
        order: Tuple[int, int, int] = (2, 1, 2),
        forecast_horizon: int = 11
    ) -> ForecastResult:
        """
        Fit ARIMA model and forecast.
        
        Args:
            timeseries: Historical workforce counts
            order: ARIMA(p, d, q) order (default: 2,1,2)
            forecast_horizon: Years to forecast (default: 11 for 2020-2030)
            
        Returns:
            ForecastResult with predictions and confidence intervals
        """
        logger.info(f"Fitting ARIMA{order} model...")
        
        # Fit ARIMA model
        model = ARIMA(timeseries, order=order)
        fitted_model = model.fit()
        
        # Generate forecasts
        forecast = fitted_model.forecast(steps=forecast_horizon)
        forecast_results = fitted_model.get_forecast(steps=forecast_horizon)
        conf_int = forecast_results.conf_int(alpha=0.05)  # 95% CI
        
        # Calculate in-sample metrics
        fitted_values = fitted_model.fittedvalues
        rmse = np.sqrt(mean_squared_error(timeseries[1:], fitted_values))
        mape = mean_absolute_percentage_error(timeseries[1:], fitted_values) * 100
        
        logger.info(f"✓ ARIMA model trained (RMSE={rmse:.1f}, MAPE={mape:.2f}%)")
        
        return ForecastResult(
            profession="",  # Fill in caller
            sector="",
            model_type="ARIMA",
            year_range=list(range(2020, 2031)),
            forecast_values=forecast,
            lower_bound_95=conf_int.iloc[:, 0].values,
            upper_bound_95=conf_int.iloc[:, 1].values,
            rmse=rmse,
            mape=mape
        )
    
    def fit_prophet(
        self,
        df_ts: pl.DataFrame,
        forecast_horizon: int = 11
    ) -> ForecastResult:
        """
        Fit Prophet model for trend + seasonality.
        
        Args:
            df_ts: DataFrame with 'ds' (date) and 'y' (value) columns
            forecast_horizon: Years to forecast
            
        Returns:
            ForecastResult with predictions
        """
        logger.info("Fitting Prophet model...")
        
        # Convert to pandas for Prophet
        df_prophet = df_ts.to_pandas()
        
        # Initialize and fit Prophet
        model = Prophet(
            yearly_seasonality=False,
            weekly_seasonality=False,
            daily_seasonality=False,
            changepoint_prior_scale=0.1  # Conservative changepoints
        )
        model.fit(df_prophet)
        
        # Create future dataframe
        future = model.make_future_dataframe(periods=forecast_horizon, freq='Y')
        forecast = model.predict(future)
        
        # Extract forecast period
        forecast_period = forecast.tail(forecast_horizon)
        
        # Calculate in-sample metrics
        fitted_values = forecast['yhat'].iloc[:len(df_prophet)]
        rmse = np.sqrt(mean_squared_error(df_prophet['y'], fitted_values))
        mape = mean_absolute_percentage_error(df_prophet['y'], fitted_values) * 100
        
        logger.info(f"✓ Prophet model trained (RMSE={rmse:.1f}, MAPE={mape:.2f}%)")
        
        return ForecastResult(
            profession="",
            sector="",
            model_type="Prophet",
            year_range=list(range(2020, 2031)),
            forecast_values=forecast_period['yhat'].values,
            lower_bound_95=forecast_period['yhat_lower'].values,
            upper_bound_95=forecast_period['yhat_upper'].values,
            rmse=rmse,
            mape=mape
        )
    
    def fit_linear_regression(
        self,
        years: np.ndarray,
        values: np.ndarray,
        forecast_horizon: int = 11
    ) -> ForecastResult:
        """
        Fit simple linear regression (for short time series).
        
        Args:
            years: Historical years (e.g., 2014-2019)
            values: Historical workforce counts
            forecast_horizon: Years to forecast
            
        Returns:
            ForecastResult with linear trend projections
        """
        logger.info("Fitting Linear Regression model...")
        
        # Fit model
        X = years.reshape(-1, 1)
        y = values
        model = LinearRegression()
        model.fit(X, y)
        
        # Forecast
        future_years = np.arange(2020, 2031).reshape(-1, 1)
        forecast = model.predict(future_years)
        
        # Estimate confidence intervals (±1.96 * RMSE)
        fitted_values = model.predict(X)
        rmse = np.sqrt(mean_squared_error(y, fitted_values))
        mape = mean_absolute_percentage_error(y, fitted_values) * 100
        
        margin = 1.96 * rmse
        lower_bound = forecast - margin
        upper_bound = forecast + margin
        
        logger.info(f"✓ Linear model trained (RMSE={rmse:.1f}, MAPE={mape:.2f}%)")
        
        return ForecastResult(
            profession="",
            sector="",
            model_type="LinearRegression",
            year_range=list(range(2020, 2031)),
            forecast_values=forecast,
            lower_bound_95=lower_bound,
            upper_bound_95=upper_bound,
            rmse=rmse,
            mape=mape
        )
    
    def select_best_model(
        self,
        profession: str,
        sector: str,
        data_years: int
    ) -> str:
        """
        Select appropriate model based on data availability.
        
        Args:
            profession: Profession name (e.g., 'doctors')
            sector: Sector (public/private/total)
            data_years: Number of years of historical data
            
        Returns:
            Model type ('arima', 'prophet', 'linear')
        """
        if data_years >= 10:
            return 'arima'
        elif data_years >= 8:
            return 'prophet'
        else:
            return 'linear'
    
    def forecast_all_professions(self) -> List[ForecastResult]:
        """
        Forecast all professions using appropriate models.
        
        Returns:
            List of ForecastResult objects
        """
        logger.info("=" * 60)
        logger.info("Forecasting all professions 2020-2030")
        logger.info("=" * 60)
        
        results = []
        
        # Group by profession and sector
        groups = self.df.groupby(['profession', 'sector'])
        
        for (profession, sector), group_df in groups:
            logger.info(f"\nForecasting: {profession} ({sector})")
            
            # Sort by year and extract time series
            ts_df = group_df.sort('year')
            years = ts_df['year'].to_numpy()
            counts = ts_df['count'].to_numpy()
            data_years = len(years)
            
            # Select model
            model_type = self.select_best_model(profession, sector, data_years)
            logger.info(f"Selected model: {model_type} ({data_years} years of data)")
            
            # Fit model
            if model_type == 'arima':
                result = self.fit_arima(counts, forecast_horizon=11)
            elif model_type == 'prophet':
                df_prophet = pl.DataFrame({
                    'ds': [f"{year}-01-01" for year in years],
                    'y': counts
                })
                result = self.fit_prophet(df_prophet, forecast_horizon=11)
            else:  # linear
                result = self.fit_linear_regression(years, counts, forecast_horizon=11)
            
            # Update result metadata
            result.profession = profession
            result.sector = sector
            results.append(result)
        
        logger.success(f"✓ Forecasted {len(results)} profession-sector combinations")
        return results
    
    def export_forecasts(
        self,
        results: List[ForecastResult],
        output_path: str
    ) -> None:
        """
        Export forecast results to Parquet.
        
        Args:
            results: List of ForecastResult objects
            output_path: Path to save forecasts
        """
        logger.info(f"Exporting forecasts to {output_path}...")
        
        # Convert results to DataFrame
        forecast_rows = []
        for result in results:
            for i, year in enumerate(result.year_range):
                forecast_rows.append({
                    'profession': result.profession,
                    'sector': result.sector,
                    'year': year,
                    'model_type': result.model_type,
                    'forecast': result.forecast_values[i],
                    'lower_95': result.lower_bound_95[i],
                    'upper_95': result.upper_bound_95[i],
                    'rmse': result.rmse,
                    'mape': result.mape
                })
        
        df_forecasts = pl.DataFrame(forecast_rows)
        df_forecasts.write_parquet(output_path)
        
        logger.success(f"✓ Exported {len(forecast_rows)} forecast records")
```

### 4. Model Configuration

**[CREATE] `shared/config/forecast_config.yml`**

```yaml
# Workforce Forecasting Configuration
# ====================================

forecast_horizon:
  start_year: 2020
  end_year: 2030
  total_years: 11

models:
  arima:
    enabled: true
    default_order: [2, 1, 2]  # (p, d, q)
    min_data_years: 10
    
  prophet:
    enabled: true
    changepoint_prior_scale: 0.1
    min_data_years: 8
    
  linear_regression:
    enabled: true
    confidence_level: 0.95
    min_data_years: 3

profession_overrides:
  doctors:
    model: arima
    arima_order: [1, 1, 1]
  
  nurses:
    model: prophet
  
  allied_health:
    model: linear  # Only 5 years of data

evaluation:
  metrics:
    - rmse
    - mape
    - forecast_stability
  
  validation:
    holdout_years: 2  # Use 2017-2019 for validation
```

### 5. Model Evaluation Tests

```python
# shared/tests/unit/test_workforce_forecaster.py

import pytest
import numpy as np
import polars as pl
from shared.src.models.workforce_forecaster import WorkforceForecaster, ForecastResult


def test_arima_forecast_length():
    """Test ARIMA produces correct forecast horizon."""
    forecaster = WorkforceForecaster(
        features_df=pl.DataFrame(),  # Mock
        config_path="shared/config/forecast_config.yml"
    )
    
    # Mock time series (13 years)
    ts = np.array([100, 105, 110, 115, 120, 125, 130, 135, 140, 145, 150, 155, 160])
    
    result = forecaster.fit_arima(ts, order=(1, 1, 1), forecast_horizon=11)
    
    assert len(result.forecast_values) == 11
    assert len(result.lower_bound_95) == 11
    assert len(result.upper_bound_95) == 11
    assert result.year_range == list(range(2020, 2031))


def test_linear_regression_positive_trend():
    """Test linear regression captures positive trend."""
    forecaster = WorkforceForecaster(
        features_df=pl.DataFrame(),
        config_path="shared/config/forecast_config.yml"
    )
    
    years = np.array([2014, 2015, 2016, 2017, 2018, 2019])
    values = np.array([100, 110, 120, 130, 140, 150])
    
    result = forecaster.fit_linear_regression(years, values, forecast_horizon=11)
    
    # Forecast should continue upward trend
    assert result.forecast_values[0] > 150  # 2020 > 2019
    assert result.forecast_values[-1] > result.forecast_values[0]  # 2030 > 2020


def test_confidence_interval_validity():
    """Test forecast CI bounds are sensible."""
    forecaster = WorkforceForecaster(
        features_df=pl.DataFrame(),
        config_path="shared/config/forecast_config.yml"
    )
    
    ts = np.array([100, 105, 110, 115, 120, 125, 130, 135, 140, 145, 150, 155, 160])
    result = forecaster.fit_arima(ts, order=(1, 1, 1), forecast_horizon=11)
    
    # Lower bound < forecast < upper bound
    assert np.all(result.lower_bound_95 < result.forecast_values)
    assert np.all(result.forecast_values < result.upper_bound_95)
```

### 6. Implementation Steps

**Phase 1: Model Infrastructure (Days 1-2)**
- [ ] Create `WorkforceForecaster` class
- [ ] Implement ARIMA fitting function
- [ ] Implement Prophet fitting function
- [ ] Implement linear regression function
- [ ] Create model selection logic

**Phase 2: Forecast Execution (Days 2-3)**
- [ ] Load engineered features (US-05 output)
- [ ] Forecast all profession-sector combinations
- [ ] Calculate forecast metrics (RMSE, MAPE)
- [ ] Generate 95% confidence intervals

**Phase 3: Validation (Days 3-4)**
- [ ] Holdout validation (2017-2019)
- [ ] Compare model performance (ARIMA vs Prophet vs Linear)
- [ ] Validate forecast stability (should not have wild swings)
- [ ] Unit tests for all model functions

**Phase 4: Output & Documentation (Day 4)**
- [ ] Export forecasts: `shared/data/4_processed/workforce_forecasts_2020_2030.parquet`
- [ ] Generate forecast comparison charts
- [ ] Document model assumptions and limitations
- [ ] Update data dictionary with forecast schema

### 7. Success Metrics

- ✅ All professions forecasted 2020-2030
- ✅ MAPE < 15% for professions with 10+ years data
- ✅ MAPE < 20% for professions with 8-10 years data
- ✅ Confidence intervals do not overlap zero (workforce cannot be negative)
- ✅ Forecasts validated on holdout period

### 8. References

- [statsmodels ARIMA Documentation](https://www.statsmodels.org/stable/generated/statsmodels.tsa.arima.model.ARIMA.html)
- [Prophet Forecasting Documentation](https://facebook.github.io/prophet/)
- US-05 (engineered features), US-09 (validation strategy)

---

**Implementation Plan Complete for US-06**  
**Dependencies**: US-05 (engineered features)  
**Enables**: US-07 (demand gap analysis), US-09 (forecast validation)
