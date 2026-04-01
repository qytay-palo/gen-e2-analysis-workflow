# User Story: 4 - Baseline Forecasting Model Development

**As a** forecasting analyst,  
**I want** to develop and compare multiple baseline forecasting models for each disease,  
**so that** I can identify the best-performing approach and establish forecast benchmarks.

## 1. 🎯 Acceptance Criteria

1. **Multiple Models Implemented** - Develop at least 4 forecasting approaches:
   - Naïve forecast (baseline benchmark)
   - Simple Exponential Smoothing (SES) or Holt's Linear Trend
   - ARIMA with auto-selected parameters
   - Prophet (Facebook's forecasting library)

2. **Model Training Complete** - Train all models on pre-2015 data

3. **In-Sample Performance Evaluated** - Calculate AIC, BIC for model comparison

4. **Preliminary Forecasts Generated** - Produce 5-year forecasts from each model

5. **Model Artifacts Saved** - Serialize trained models for reuse

## 2. 🔒 Technical Constraints

- **Forecasting libraries**: `statsmodels` (ARIMA, SES, Holt), `prophet` (Facebook Prophet)
- **Model storage**: Serialize models using `joblib` or `pickle`
- **Computational**: Local compute sufficient (<2,000 observations per disease)
- **Reproducibility**: Set random seeds for stochastic models

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Complete methodology guide for Naïve, SES, Holt, ARIMA, Prophet

**Key Methods**:
- **Naïve**: Simple baseline (last value carried forward)
- **ARIMA**: Uses ACF/PACF patterns from User Story 2
- **Prophet**: Automatic trend changepoint detection
- **Holt**: Linear trend method for trending series

## 4. 📦 Dependencies

**External Packages**:
- `statsmodels>=0.14.0` - ARIMA, SES, Holt
- `prophet>=1.1` - Prophet forecasting
- `joblib>=1.3.2` - Model serialization
- `scikit-learn>=1.3.0` - Metrics

**Internal Dependencies**:
- Training data from User Story 3 (feature-engineered dataset, pre-2015)
- ARIMA parameters from User Story 2 (ACF/PACF analysis)

## 5. ✅ Implementation Tasks

### Model Implementation
- ⬜ **Implement Naïve forecast** (last observation carried forward)
- ⬜ **Implement Simple Exponential Smoothing** (or Holt's method if trend exists)
- ⬜ **Implement ARIMA** using auto.arima or grid search over (p,d,q)
- ⬜ **Implement Prophet** with yearly_seasonality=False for annual data

### Training & Evaluation
- ⬜ **Train all models** on pre-2015 data (1990-2014)
- ⬜ **Calculate in-sample metrics** (AIC, BIC, RMSE)
- ⬜ **Generate preliminary forecasts** (2015-2019 for validation)

### Model Storage
- ⬜ **Serialize trained models** to `shared/models/disease_burden_forecasting/`
- ⬜ **Save model metadata** (parameters, training period, performance metrics)

## 6. Notes

**Model Selection Criteria**:
- Favor simpler models if performance is comparable (parsimony principle)
- ARIMA typically best for strongly autocorrelated series
- Prophet good for interpretability and automatic changepoint detection

**Next Steps**:
- Models evaluated out-of-sample in User Story 5: Model Validation & Backtesting
- Best-performing model(s) used for final forecasts in User Story 6

---

## Implementation Plan

### 1. Feature Overview

Develop and compare 4 baseline forecasting models (Naïve, Exponential Smoothing, ARIMA, Prophet) for each disease to identify best-performing approach.

### 2. Component Analysis & Reuse Strategy

**Input**: Feature matrix from US-03, ARIMA parameters from US-02

**New Components**:
- `shared/src/models/forecasting_models.py` - Model wrappers for Naïve, SES, Holt, ARIMA, Prophet
- `problem-statements/ps-006-forecasting/notebooks/04_baseline_models.ipynb`

### 3. ML Model Evaluation & Selection

Search standard forecasting libraries (NOT Hugging Face for time series):

**Candidate Models**:
1. **Naïve Forecast**: Last value carried forward (baseline)
   - Library: Custom implementation
   - Pros: Simple, interpretable
   - Cons: Ignores trends

2. **Exponential Smoothing / Holt's Linear**: Weighted averages with trend
   - Library: `statsmodels.tsa.holtwinters`
   - Pros: Handles trends, simple
   - Cons: No seasonality (not needed for annual)

3. **ARIMA**: Autoregressive Integrated Moving Average
   - Library: `statsmodels.tsa.arima.model.ARIMA`
   - Pros: Statistically rigorous, handles autocorrelation
   - Cons: Requires parameter tuning

4. **Prophet**: Facebook's forecasting tool
   - Library: `prophet` (open-source, MIT license)
   - Pros: Automatic changepoint detection, interpretable
   - Cons: May overfit with few data points

**Selection**: Implement all 4, compare in US-05.

### 4-6. Code Specifications

```python
# shared/src/models/forecasting_models.py

import polars as pl
import numpy as np
from statsmodels.tsa.holtwinters import SimpleExpSmoothing, Holt
from statsmodels.tsa.arima.model import ARIMA
from prophet import Prophet
from loguru import logger
import joblib
from pathlib import Path
from typing import Tuple, Dict


class NaiveForecaster:
    """Naïve forecast: last observation carried forward."""
    
    def __init__(self):
        self.last_value = None
    
    def fit(self, y: np.ndarray) -> 'NaiveForecaster':
        self.last_value = y[-1]
        logger.info(f"Naïve model fitted: last_value={self.last_value:.2f}")
        return self
    
    def forecast(self, steps: int) -> np.ndarray:
        return np.array([self.last_value] * steps)


class ARIMAForecaster:
    """ARIMA forecasting wrapper."""
    
    def __init__(self, order: Tuple[int, int, int]):
        self.order = order
        self.model = None
        self.fitted_model = None
    
    def fit(self, y: np.ndarray) -> 'ARIMAForecaster':
        logger.info(f"Fitting ARIMA{self.order}")
        self.model = ARIMA(y, order=self.order)
        self.fitted_model = self.model.fit()
        logger.info(f"ARIMA fitted: AIC={self.fitted_model.aic:.2f}")
        return self
    
    def forecast(self, steps: int) -> Tuple[np.ndarray, np.ndarray, np.ndarray]:
        forecast_result = self.fitted_model.get_forecast(steps=steps)
        mean = forecast_result.predicted_mean
        conf_int = forecast_result.conf_int(alpha=0.05)  # 95% CI
        
        return mean, conf_int.iloc[:, 0].values, conf_int.iloc[:, 1].values
    
    def save(self, path: Path) -> None:
        joblib.dump(self.fitted_model, path)
        logger.info(f"Model saved: {path}")


class ProphetForecaster:
    """Prophet forecasting wrapper."""
    
    def __init__(self, yearly_seasonality: bool = False):
        self.model = Prophet(yearly_seasonality=yearly_seasonality, daily_seasonality=False, weekly_seasonality=False)
    
    def fit(self, df: pl.DataFrame, time_col: str = 'year', value_col: str = 'asmr') -> 'ProphetForecaster':
        # Prophet requires 'ds' and 'y' columns
        prophet_df = df.select([
            pl.col(time_col).cast(str).str.strptime(pl.Date, format='%Y').alias('ds'),
            pl.col(value_col).alias('y')
        ]).to_pandas()
        
        self.model.fit(prophet_df)
        logger.info("Prophet model fitted")
        return self
    
    def forecast(self, years: list[int]) -> pl.DataFrame:
        import pandas as pd
        future_df = pd.DataFrame({'ds': pd.to_datetime([f"{y}-01-01" for y in years])})
        forecast = self.model.predict(future_df)
        
        return pl.from_pandas(forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']])
```

### 7. Implementation Steps

- [ ] Create `shared/src/models/forecasting_models.py`
- [ ] Implement Naïve, Holt, ARIMA, Prophet model classes
- [ ] Load training data (pre-2015) from US-03
- [ ] Train all 4 models on each disease
- [ ] Calculate in-sample metrics (AIC, BIC, RMSE)
- [ ] Generate preliminary 2015-2019 forecasts
- [ ] Serialize models to `shared/models/disease_burden_forecasting/`
- [ ] Save model metadata (parameters, metrics)

✅ **PLAN READY**
