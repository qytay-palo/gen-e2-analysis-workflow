# User Story: 6 - Five-Year Mortality Projections Generation

**As a** healthcare strategic planner,  
**I want** 5-year mortality projections (2020-2025) with confidence intervals for each disease,  
**so that** I can plan healthcare capacity and infrastructure investments with quantified uncertainty.

## 1. 🎯 Acceptance Criteria

1. **Production Forecasts Generated** - Retrain best models on full dataset (1990-2019), forecast 2020-2025
2. **Prediction Intervals Included** - 80% and 95% confidence intervals for all forecasts
3. **Point Estimates Provided** - Best estimate (median or mean forecast) by disease and year
4. **Forecast Dataset Created** - Structured table with year, disease, forecast, lower/upper bounds
5. **Forecast Report Produced** - "Singapore Disease Burden Projections 2020-2025" analytical report

## 2. 🔒 Technical Constraints

- **Model retraining**: Use full 1990-2019 data (no hold-out for production forecasts)
- **Uncertainty quantification**: Generate both 80% and 95% prediction intervals
- **Output format**: Parquet file for data, PDF for stakeholder report

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Production forecasting workflow example
- [Disease Burden Feature Engineering Guide](../../../domain_knowledge/disease-burden-feature-engineering-guide.md) - Mortality rate interpretation

## 4. 📦 Dependencies

**Input**:
- Best models from User Story 5 (validation-selected models)
- Full mortality dataset 1990-2019

**Output**:
- Forecast dataset: `shared/data/4_processed/disease_burden_forecasts_2020_2025.parquet`
- Report: `reports/figures/problem-statement-006/disease_burden_projections_2020-2025.pdf`

## 5. ✅ Implementation Tasks

### Production Model Training
- ⬜ **Retrain best models** on full 1990-2019 data
- ⬜ **Verify model stability** (check parameters similar to validation training)

### Forecast Generation
- ⬜ **Generate 2020-2025 forecasts** (6-year horizon)
- ⬜ **Extract 80% prediction intervals** (10th-90th percentile)
- ⬜ **Extract 95% prediction intervals** (2.5th-97.5th percentile)

### Dataset Creation
- ⬜ **Structure forecast table** (year, disease, forecast_mean, lower_80, upper_80, lower_95, upper_95)
- ⬜ **Add metadata columns** (model_type, forecast_date, data_version)
- ⬜ **Save to processed data** (`disease_burden_forecasts_2020_2025.parquet`)

### Report Generation
- ⬜ **Write executive summary** (key projections and capacity implications)
- ⬜ **Create disease-by-disease projection sections**
- ⬜ **Include forecast visualizations** (line charts with PIs)
- ⬜ **Document methodology** and limitations (COVID-19 caveat)
- ⬜ **Publish PDF report** to `reports/figures/problem-statement-006/`

## 6. Notes

**COVID-19 Caveat** (CRITICAL):
- Data ends 2019-2020 (pre-COVID)
- Forecasts represent "baseline" (no pandemic scenario)
- Report must clearly state: "These projections assume continuation of historical trends without major disruptions like COVID-19"

**Success Criteria**:
- Forecasts available for all 3 diseases (cancer, stroke, heart disease)
- Uncertainty bounds provided (stakeholders can plan for range, not just point estimate)
- Report accessible to non-technical audiences (visualizations, plain language)

**Next Steps**:
- Forecasts used in User Story 7: Scenario Modeling (optimistic/pessimistic variants)
- Forecasts translated to capacity requirements in User Story 8

---

## Implementation Plan

### 1. Feature Overview

Generate production 5-year mortality projections (2020-2025) with confidence intervals using best models retrained on full 1990-2019 dataset.

### 2. Component Analysis & Reuse Strategy

**Input**: Best models from US-05, full 1990-2019 data from US-03

**New Components**:
- `problem-statements/ps-006-forecasting/scripts/generate_production_forecasts.py` - Production forecast pipeline
- `problem-statements/ps-006-forecasting/notebooks/06_production_forecasts.ipynb`

### 4-6. Code Specifications

```python
# problem-statements/ps-006-forecasting/scripts/generate_production_forecasts.py

import polars as pl
import joblib
from pathlib import Path
from datetime import datetime
from loguru import logger
import yaml

logger.add('logs/etl/production_forecasts_{time}.log')


def generate_production_forecasts():
    """Generate 2020-2025 forecasts using best models."""
    
    logger.info("=" * 60)
    logger.info("Production Forecast Generation: 2020-2025")
    logger.info("=" * 60)
    
    # Load configuration
    with open('problem-statements/ps-006-forecasting/config/config.yml', 'r') as f:
        config = yaml.safe_load(f)
    
    # Load full training data (1990-2019)
    df_full = pl.read_parquet('shared/data/3_interim/forecasting/mortality_time_series_clean.parquet')
    logger.info(f"Loaded {df_full.height} rows (1990-2019)")
    
    # Load best model selections from validation
    with open('results/tables/problem-statement-006/best_models.json', 'r') as f:
        import json
        best_models = json.load(f)
    
    forecast_results = []
    
    for disease in ['cancer', 'stroke', 'heart_disease']:
        logger.info(f"\n--- Forecasting {disease} ---")
        
        df_disease = df_full.filter(pl.col('disease') == disease).sort('year')
        y_train = df_disease['asmr'].to_numpy()
        
        # Load and retrain best model on full data
        model_type = best_models[disease]['model_type']
        model_params = best_models[disease]['parameters']
        
        if model_type == 'arima':
            from shared.src.models.forecasting_models import ARIMAForecaster
            model = ARIMAForecaster(order=tuple(model_params['order']))
            model.fit(y_train)
            
            mean, lower, upper = model.forecast(steps=6)  # 2020-2025 (6 years)
            
            # Save model
            model.save(Path(f'shared/models/disease_burden_forecasting/{disease}_arima.pkl'))
        
        # Create forecast dataframe
        forecast_years = list(range(2020, 2026))
        forecast_df = pl.DataFrame({
            'year': forecast_years,
            'disease': [disease] * 6,
            'forecast_mean': mean,
            'lower_95': lower,
            'upper_95': upper,
            'model_type': [model_type] * 6,
            'forecast_date': [datetime.now().isoformat()] * 6
        })
        
        forecast_results.append(forecast_df)
        logger.info(f"{disease} forecast complete: mean={mean[0]:.1f} (2020)")
    
    # Combine all forecasts
    df_forecasts = pl.concat(forecast_results, how='vertical')
    
    # Save to processed data
    output_path = Path('shared/data/4_processed/disease_burden_forecasts_2020_2025.parquet')
    output_path.parent.mkdir(parents=True, exist_ok=True)
    df_forecasts.write_parquet(output_path)
    
    logger.info(f"\n✅ Production forecasts saved: {output_path}")
    logger.info(f"Total forecasts: {df_forecasts.height} rows")
    
    return df_forecasts


if __name__ == '__main__':
    generate_production_forecasts()
```

### 7. Implementation Steps

- [ ] Load best model selections from US-05
- [ ] Retrain models on full 1990-2019 data
- [ ] Generate 2020-2025 forecasts (6 years)
- [ ] Extract 80% and 95% prediction intervals
- [ ] Save to `shared/data/4_processed/disease_burden_forecasts_2020_2025.parquet`
- [ ] Generate executive report PDF with projections
- [ ] Document COVID-19 caveat (pre-pandemic baseline)

✅ **PLAN READY**
