# User Story: 5 - Model Validation and Backtesting

**As a** data scientist,  
**I want** to rigorously validate forecast model performance on historical data (2015-2019),  
**so that** I can quantify forecast accuracy, select the best model, and communicate prediction reliability to stakeholders.

## 1. 🎯 Acceptance Criteria

1. **Out-of-Sample Validation** - Evaluate all models on withheld test set (2015-2019)
2. **Performance Metrics Calculated** - RMSE, MAE, MAPE, MASE for each model
3. **Prediction Interval Coverage Validated** - 95% PIs should contain ~95% of actuals
4. **Best Model Selected** - Choose optimal model per disease based on MAPE and coverage
5. **Validation Report Generated** - Summary documenting accuracy and model selection rationale

## 2. 🔒 Technical Constraints

- **Metrics library**: `scikit-learn.metrics`, `statsmodels`
- **Visualization**: Actual vs forecast comparison charts
- **Statistical rigor**: Test prediction interval calibration

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Model validation section, forecast metrics table

**Key Metrics**:
- **MAPE**: <10% excellent, 10-20% good, >20% poor (rule of thumb)
- **Coverage**: 95% PI should contain ~95% of actual values (calibration check)
- **MASE**: <1 means model beats naïve forecast

## 4. 📦 Dependencies

**External Packages**:
- `scikit-learn>=1.3.0` - Metrics (MAE, MAPE, RMSE)
- `numpy>=1.24.0` - Array operations

**Internal Dependencies**:
- Trained models from User Story 4
- Test data (2015-2019)

## 5. ✅ Implementation Tasks

### Forecast Generation
- ⬜ **Generate out-of-sample forecasts** from all models (2015-2019)
- ⬜ **Extract prediction intervals** (80%, 95%)

### Evaluation
- ⬜ **Calculate RMSE, MAE, MAPE, MASE** for each model
- ⬜ **Check prediction interval coverage** (% of actuals within PI)
- ⬜ **Rank models by performance** (prioritize MAPE + coverage)

### Model Selection
- ⬜ **Select best model per disease** based on multi-criteria evaluation
- ⬜ **Document selection rationale** (why this model chosen)

### Deliverables
- ⬜ **Generate validation report** with performance tables and charts
- ⬜ **Create actual-vs-forecast visualizations** for all models
- ⬜ **Save validation metrics** to `results/tables/problem-statement-006/model_validation_results.csv`

## 6. Notes

**Success Criteria**:
- MAPE < 15% for at least one model per disease (good forecast accuracy)
- 95% PI coverage between 90-100% (well-calibrated uncertainty)

**Decision Rules**:
- If ARIMA and Prophet perform similarly → Choose simpler model (ARIMA)
- If Prophet significantly better → Use Prophet (likely due to changepoints)
- If all models poor (MAPE > 25%) → Flag disease as "hard to forecast", use ensemble

**Next Steps**:
- Best models used for production forecasts in User Story 6
- Validation insights inform stakeholder communication about forecast reliability

---

## Implementation Plan

### 1. Feature Overview

Rigorously validate forecast model performance on historical test set (2015-2019) to quantify accuracy, select best model per disease, and communicate prediction reliability.

### 2. Component Analysis & Reuse Strategy

**Input**: Trained models from US-04, test data (2015-2019) from US-03

**New Components**:
- `shared/src/models/model_evaluation.py` - Metrics calculation, validation
- `problem-statements/ps-006-forecasting/notebooks/05_model_validation.ipynb`

### 4-6. Code Specifications

```python
# shared/src/models/model_evaluation.py

import numpy as np
from sklearn.metrics import mean_absolute_error, mean_squared_error
from loguru import logger
from typing import Dict


def calculate_mape(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """Mean Absolute Percentage Error."""
    return np.mean(np.abs((y_true - y_pred) / y_true)) * 100


def calculate_mase(y_true: np.ndarray, y_pred: np.ndarray, y_train: np.ndarray) -> float:
    """Mean Absolute Scaled Error (relative to naïve forecast)."""
    mae_model = mean_absolute_error(y_true, y_pred)
    mae_naive = mean_absolute_error(y_train[1:], y_train[:-1])  # Naïve MAE on training
    return mae_model / mae_naive if mae_naive > 0 else np.inf


def validate_prediction_interval_coverage(
    y_true: np.ndarray,
    lower_bound: np.ndarray,
    upper_bound: np.ndarray
) -> float:
    """Calculate % of actuals within prediction interval."""
    within_interval = (y_true >= lower_bound) & (y_true <= upper_bound)
    coverage = np.mean(within_interval) * 100
    logger.info(f"PI coverage: {coverage:.1f}%")
    return coverage


def evaluate_model(
    y_true: np.ndarray,
    y_pred: np.ndarray,
    y_train: np.ndarray,
    lower: np.ndarray = None,
    upper: np.ndarray = None
) -> Dict:
    """Comprehensive model evaluation."""
    
    metrics = {
        'rmse': np.sqrt(mean_squared_error(y_true, y_pred)),
        'mae': mean_absolute_error(y_true, y_pred),
        'mape': calculate_mape(y_true, y_pred),
        'mase': calculate_mase(y_true, y_pred, y_train)
    }
    
    if lower is not None and upper is not None:
        metrics['pi_coverage_95'] = validate_prediction_interval_coverage(y_true, lower, upper)
    
    logger.info(f"RMSE={metrics['rmse']:.2f}, MAPE={metrics['mape']:.1f}%, MASE={metrics['mase']:.2f}")
    
    return metrics
```

### 7. Implementation Steps

- [ ] Load trained models from US-04
- [ ] Generate out-of-sample forecasts (2015-2019)
- [ ] Calculate RMSE, MAE, MAPE, MASE for each model
- [ ] Validate 95% PI coverage
- [ ] Rank models by MAPE and coverage
- [ ] Select best model per disease
- [ ] Generate validation report with visualizations
- [ ] Save results to `results/tables/problem-statement-006/model_validation_results.csv`

✅ **PLAN READY**
