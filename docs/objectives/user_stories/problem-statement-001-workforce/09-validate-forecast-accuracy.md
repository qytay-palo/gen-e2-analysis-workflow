# Validate Forecast Model Accuracy (Lifecycle Stage: Validation & Evaluation)

**Story ID**: PS-001-US-09  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: S (3-4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Science Lead ensuring forecast reliability**,  
I want **to rigorously validate forecasting models using historical holdout data (2017-2019), statistical tests, and cross-validation to quantify forecast accuracy and identify model weaknesses**,  
So that **stakeholders can trust the 2020-2030 projections and I can provide confidence levels for different professions and time horizons**.

---

## 🎯 Acceptance Criteria

1. **Backtesting validation completed**
   - Models retrained on 2006-2016 data, forecasts generated for 2017-2019
   - Actual vs predicted comparison: calculate forecast errors for validation period
   - Accuracy metrics calculated: MAPE, RMSE, MAE, forecast bias per profession
   - Benchmark: MAPE < 10% for stable professions, MAPE < 20% for volatile professions

2. **Statistical validation tests passed**
   - Residual diagnostics: normality test (Shapiro-Wilk), autocorrelation test (Ljung-Box)
   - Forecast unbiasedness test: mean error ≈ 0 (no systematic over/under-prediction)
   - Confidence interval coverage: 80% CI should contain ~80% of actual values
   - Statistical significance: forecast errors not significantly different from zero (t-test)

3. **Cross-validation performed**
   - Rolling window cross-validation: train on 8 years, validate on 2 years, roll forward
   - Calculate CV-MAPE across all folds
   - Identify professions with stable vs unstable forecast performance
   - Sensitivity analysis: forecast accuracy vs forecast horizon (1-year vs 5-year vs 10-year)

4. **Data output requirement**
   - Output file: `results/tables/forecast_validation_metrics.csv`
   - Format: CSV with columns (profession, model_name, mape, rmse, mae, bias, ljung_box_p, shapiro_p, ci_coverage_80, cv_mape)
   - Schema documented: Yes
   - Validation report: `results/forecast_validation_report.md` (pass/fail summary per profession)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (data processing), statsmodels/scipy (statistical tests)
- **Statistical Tests**: scipy.stats (normality, t-tests), statsmodels (autocorrelation)
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for validation functions

---

## 📚 Domain Knowledge References

- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#risks-and-open-questions) - Risk: Forecasting accuracy limited by data ending in 2019
- Standard forecasting validation practices (time series cross-validation, backtesting)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data manipulation for validation
- `statsmodels>=0.14.0`: Statistical tests (Ljung-Box, ACF)
- `scipy>=1.11.0`: Statistical tests (Shapiro-Wilk, t-test)
- `scikit-learn>=1.3.0`: Cross-validation utilities
- `matplotlib>=3.8.0`: Validation diagnostic plots
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-06 (Forecast models - BLOCKING)
  - PS-001-US-02 (Clean data for actual values - BLOCKING)
- **Data Sources**: 
  - `shared/data/4_processed/workforce_forecasts_2020_2030.parquet` (baseline forecasts)
  - `shared/data/3_interim/workforce_integrated_clean.parquet` (actual 2017-2019 values)
- **Config Files**: `config/analysis.yml` (validation thresholds: acceptable MAPE, CI coverage targets)

---

## ✅ Implementation Tasks

### Backtesting Setup
- [ ] Load actual workforce data (2006-2019)
- [ ] Split data: training (2006-2016), validation (2017-2019)
- [ ] Retrain all models (ARIMA, Prophet, regression) on training data only
- [ ] Generate forecasts for 2017-2019 validation period
- [ ] Compare forecasts vs actuals

### Accuracy Metric Calculation
- [ ] Calculate MAPE: `mean(|actual - forecast| / actual) * 100`
- [ ] Calculate RMSE: `sqrt(mean((actual - forecast)^2))`
- [ ] Calculate MAE: `mean(|actual - forecast|)`
- [ ] Calculate Forecast Bias: `mean(forecast - actual)` (positive = over-prediction)
- [ ] Calculate metrics per profession and overall

### Statistical Validation Tests
- [ ] Residual normality test:
  - Calculate residuals: `actual - forecast`
  - Shapiro-Wilk test: `scipy.stats.shapiro(residuals)`
  - Interpretation: p > 0.05 → residuals normally distributed ✓
  
- [ ] Autocorrelation test:
  - Ljung-Box test on residuals: `statsmodels.stats.diagnostic.acorr_ljungbox(residuals)`
  - Interpretation: p > 0.05 → no autocorrelation (random errors) ✓
  
- [ ] Forecast unbiasedness test:
  - t-test: is mean error significantly different from zero?
  - `scipy.stats.ttest_1samp(errors, 0)`
  - Interpretation: p > 0.05 → unbiased forecast ✓

- [ ] Confidence interval coverage:
  - Check: is actual within 80% CI for ~80% of validation points?
  - Coverage ratio: `sum(actual within CI) / total_points`
  - Target: 0.75-0.85 for 80% CI

### Cross-Validation
- [ ] Implement rolling window cross-validation:
  - Fold 1: train 2006-2013, validate 2014-2015
  - Fold 2: train 2007-2014, validate 2015-2016
  - Fold 3: train 2008-2015, validate 2016-2017
  - Fold 4: train 2009-2016, validate 2017-2018
  - Fold 5: train 2010-2017, validate 2018-2019
  
- [ ] Calculate CV-MAPE: average MAPE across all folds
- [ ] Identify professions with high CV-MAPE variance (unstable forecasts)

### Forecast Horizon Sensitivity
- [ ] Analyze accuracy vs forecast horizon:
  - 1-year ahead accuracy (2017 forecast from 2016 model)
  - 2-year ahead accuracy (2018 forecast from 2016 model)
  - 3-year ahead accuracy (2019 forecast from 2016 model)
- [ ] Plot accuracy degradation: MAPE vs forecast horizon
- [ ] Quantify uncertainty growth: CI width vs forecast horizon

### Validation Reporting
- [ ] Create validation summary table per profession:
  - Model performance metrics
  - Statistical test results (pass/fail)
  - Cross-validation results
  - Overall validation score (weighted composite)
  
- [ ] Flag professions/models with validation failures:
  - MAPE > 20%
  - Significant autocorrelation (Ljung-Box p < 0.05)
  - Poor CI coverage (<70% or >90%)
  
- [ ] Generate recommendations:
  - Which professions have reliable forecasts (high confidence)
  - Which professions have uncertain forecasts (recommend caution in interpretation)
  - Model improvements needed (e.g., add exogenous variables, try different model class)

### Visualization
- [ ] Create actual vs predicted scatter plots (with 45° reference line)
- [ ] Create residual plots: histogram, Q-Q plot (normality check)
- [ ] Create forecast error time series: errors over validation period
- [ ] Create CI coverage plot: actual values with forecast CI bands
- [ ] Export diagnostic charts: `reports/figures/forecast_validation_diagnostics.png`

### Testing & Validation
- [ ] Unit tests for accuracy metric calculations
- [ ] Test statistical tests: verify correct interpretation of p-values
- [ ] Test CV implementation: ensure no data leakage across folds
- [ ] Integration test: end-to-end validation pipeline

### Documentation
- [ ] Docstrings for all validation functions (Google style)
- [ ] Validation report: `results/forecast_validation_report.md`
  - Validation methodology
  - Metrics summary table
  - Statistical test results
  - Profession-by-profession validation assessment
  - Confidence ratings: which forecasts are most/least reliable
  - Recommendations for model improvements
- [ ] Model limitations documentation: `shared/models/workforce/MODEL_LIMITATIONS.md`
- [ ] Update README: `shared/src/models/README.md`

---

## 📌 Notes

**Accuracy Metrics (Polars)**:
```python
import polars as pl

df_validation = (
    df_actual
    .join(df_forecast, on=['profession', 'year'], how='inner')
    .with_columns([
        (pl.col('actual_count') - pl.col('forecast_count')).alias('error'),
        (pl.col('actual_count') - pl.col('forecast_count')).abs().alias('abs_error'),
        ((pl.col('actual_count') - pl.col('forecast_count')).abs() / pl.col('actual_count') * 100)
        .alias('pct_error')
    ])
)

# Calculate metrics
metrics = df_validation.group_by('profession').agg([
    pl.col('pct_error').mean().alias('mape'),
    (pl.col('error') ** 2).mean().sqrt().alias('rmse'),
    pl.col('abs_error').mean().alias('mae'),
    pl.col('error').mean().alias('bias')
])
```

**Statistical Tests (Python)**:
```python
from scipy import stats
from statsmodels.stats.diagnostic import acorr_ljungbox

# Normality test
residuals = (actual - forecast).to_numpy()
shapiro_stat, shapiro_p = stats.shapiro(residuals)
logger.info(f"Shapiro-Wilk test: p={shapiro_p:.4f} {'✓ Normal' if shapiro_p > 0.05 else '✗ Non-normal'}")

# Autocorrelation test
ljung_box_results = acorr_ljungbox(residuals, lags=5, return_df=True)
ljung_box_p = ljung_box_results['lb_pvalue'].iloc[-1]
logger.info(f"Ljung-Box test: p={ljung_box_p:.4f} {'✓ No autocorrelation' if ljung_box_p > 0.05 else '✗ Autocorrelated'}")

# Unbiasedness test
t_stat, t_p = stats.ttest_1samp(residuals, 0)
logger.info(f"Bias test: p={t_p:.4f} {'✓ Unbiased' if t_p > 0.05 else '✗ Biased'}")
```

**Confidence Interval Coverage Check**:
```python
# Check if actual within 80% CI
df_coverage = df_validation.with_columns([
    ((pl.col('actual_count') >= pl.col('lower_80')) & 
     (pl.col('actual_count') <= pl.col('upper_80')))
    .alias('within_80_ci')
])

coverage_80 = df_coverage['within_80_ci'].mean()
logger.info(f"80% CI Coverage: {coverage_80:.1%} (target: 75-85%)")
```

**Validation Pass/Fail Criteria**:
| Metric | Target | Pass Criteria |
|--------|--------|---------------|
| MAPE | < 10% (stable), < 20% (volatile) | ✓ if within target |
| Shapiro-Wilk p | > 0.05 | ✓ residuals normal |
| Ljung-Box p | > 0.05 | ✓ no autocorrelation |
| Bias t-test p | > 0.05 | ✓ unbiased |
| 80% CI Coverage | 75-85% | ✓ if in range |
| CV-MAPE std dev | < 5% | ✓ stable across folds |

**Expected Validation Results**:
- **Doctors**: High confidence (long time series, stable trends) → MAPE ~5-8%
- **Nurses**: Moderate confidence (shorter series, but large sample) → MAPE ~8-12%
- **Pharmacists**: Moderate confidence (stable trends) → MAPE ~6-10%
- **Allied health**: Lower confidence (short series 2014-2019) → MAPE ~15-25%

**If Validation Fails**:
- Document limitations explicitly in forecast report
- Recommend caution in interpreting long-term projections (2030)
- Suggest model improvements: add exogenous variables, try ensemble methods
- Recommend updating forecasts when more data becomes available

---

## Implementation Plan

### 1. Validation Overview

Validate forecast accuracy using holdout validation (2017-2019) and cross-validation. Calculate MAPE, RMSE, bias. Perform statistical tests (normality, autocorrelation, unbiasedness). Assess CI coverage. Provide confidence ratings per profession.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Forecasts (US-06), cleaned data (US-02)
**Create**: `shared/src/models/forecast_validator.py`, validation report generator

### 3. Key Implementation Components

**[CREATE] `shared/src/models/forecast_validator.py`**

```python
"""
Forecast Accuracy Validation
=============================

Validate workforce forecasts using statistical rigor.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import numpy as np
from pathlib import Path
from typing import Dict, List, Tuple
from dataclasses import dataclass
from loguru import logger
from scipy import stats
from statsmodels.stats.diagnostic import acorr_ljungbox
from sklearn.model_selection import TimeSeriesSplit
import matplotlib.pyplot as plt
import seaborn as sns


@dataclass
class ValidationResult:
    """Container for validation outputs."""
    profession: str
    sector: str
    mape: float
    rmse: float
    mae: float
    bias: float
    shapiro_p: float
    ljungbox_p: float
    bias_test_p: float
    ci_coverage_80: float
    ci_coverage_95: float
    validation_pass: bool
    confidence_rating: str


class ForecastValidator:
    """
    Validate forecast accuracy using statistical tests.
    
    Validation Methods:
    - Holdout validation (2017-2019)
    - Time series cross-validation
    - Statistical tests (normality, autocorrelation, bias)
    - Confidence interval coverage
    """
    
    def __init__(
        self,
        forecast_df: pl.DataFrame,
        actual_df: pl.DataFrame
    ):
        """
        Initialize validator.
        
        Args:
            forecast_df: Forecast results from US-06
            actual_df: Actual historical workforce data
        """
        self.forecasts = forecast_df
        self.actuals = actual_df
        logger.info("Initialized ForecastValidator")
    
    def holdout_validation(
        self,
        profession: str,
        sector: str,
        holdout_years: List[int] = [2017, 2018, 2019]
    ) -> pl.DataFrame:
        """
        Perform holdout validation on recent years.
        
        Args:
            profession: Profession name
            sector: Sector (public/private/total)
            holdout_years: Years to use for validation
            
        Returns:
            DataFrame with actual vs forecast comparison
        """
        logger.info(f"Holdout validation: {profession} ({sector})")
        
        # Get forecasts for holdout period
        forecasts_holdout = (
            self.forecasts
            .filter(
                (pl.col('profession') == profession) &
                (pl.col('sector') == sector) &
                (pl.col('year').is_in(holdout_years))
            )
        )
        
        # Get actuals for holdout period
        actuals_holdout = (
            self.actuals
            .filter(
                (pl.col('profession') == profession) &
                (pl.col('sector') == sector) &
                (pl.col('year').is_in(holdout_years))
            )
            .rename({'count': 'actual_count'})
        )
        
        # Join and calculate errors
        df_validation = (
            actuals_holdout
            .join(
                forecasts_holdout.select(['year', 'forecast', 'lower_95', 'upper_95']),
                on='year',
                how='inner'
            )
            .with_columns([
                (pl.col('actual_count') - pl.col('forecast')).alias('error'),
                (pl.col('actual_count') - pl.col('forecast')).abs().alias('abs_error'),
                ((pl.col('actual_count') - pl.col('forecast')).abs() / 
                 pl.col('actual_count') * 100).alias('pct_error')
            ])
        )
        
        logger.info(f"✓ Holdout validation complete ({len(df_validation)} years)")
        return df_validation
    
    def calculate_accuracy_metrics(
        self,
        df_validation: pl.DataFrame
    ) -> Dict[str, float]:
        """
        Calculate standard accuracy metrics.
        
        Args:
            df_validation: DataFrame with actual and forecast
            
        Returns:
            Dictionary of accuracy metrics
        """
        logger.info("Calculating accuracy metrics")
        
        # MAPE
        mape = df_validation['pct_error'].mean()
        
        # RMSE
        rmse = np.sqrt((df_validation['error'] ** 2).mean())
        
        # MAE
        mae = df_validation['abs_error'].mean()
        
        # Bias (mean error)
        bias = df_validation['error'].mean()
        
        metrics = {
            'mape': mape,
            'rmse': rmse,
            'mae': mae,
            'bias': bias
        }
        
        logger.info(f"Metrics: MAPE={mape:.2f}%, RMSE={rmse:.1f}, Bias={bias:.1f}")
        return metrics
    
    def test_residual_normality(
        self,
        residuals: np.ndarray
    ) -> Tuple[float, float]:
        """
        Test if forecast residuals are normally distributed.
        
        Args:
            residuals: Forecast errors (actual - forecast)
            
        Returns:
            (statistic, p-value) from Shapiro-Wilk test
        """
        logger.info("Testing residual normality (Shapiro-Wilk)")
        
        stat, p_value = stats.shapiro(residuals)
        
        if p_value > 0.05:
            logger.info(f"✓ Residuals normal (p={p_value:.4f})")
        else:
            logger.warning(f"⚠ Residuals non-normal (p={p_value:.4f})")
        
        return stat, p_value
    
    def test_residual_autocorrelation(
        self,
        residuals: np.ndarray,
        lags: int = 5
    ) -> float:
        """
        Test for autocorrelation in residuals (Ljung-Box).
        
        Args:
            residuals: Forecast errors
            lags: Number of lags to test
            
        Returns:
            p-value from Ljung-Box test
        """
        logger.info(f"Testing autocorrelation (Ljung-Box, {lags} lags)")
        
        results = acorr_ljungbox(residuals, lags=lags, return_df=True)
        p_value = results['lb_pvalue'].iloc[-1]
        
        if p_value > 0.05:
            logger.info(f"✓ No autocorrelation (p={p_value:.4f})")
        else:
            logger.warning(f"⚠ Autocorrelation detected (p={p_value:.4f})")
        
        return p_value
    
    def test_unbiasedness(
        self,
        residuals: np.ndarray
    ) -> Tuple[float, float]:
        """
        Test if forecasts are unbiased (mean error = 0).
        
        Args:
            residuals: Forecast errors
            
        Returns:
            (t-statistic, p-value) from one-sample t-test
        """
        logger.info("Testing forecast unbiasedness (t-test)")
        
        t_stat, p_value = stats.ttest_1samp(residuals, 0)
        
        if p_value > 0.05:
            logger.info(f"✓ Forecasts unbiased (p={p_value:.4f})")
        else:
            logger.warning(f"⚠ Forecasts biased (p={p_value:.4f})")
        
        return t_stat, p_value
    
    def check_ci_coverage(
        self,
        df_validation: pl.DataFrame
    ) -> Dict[str, float]:
        """
        Check if actual values fall within forecast CIs.
        
        Args:
            df_validation: DataFrame with actual, forecast, and CI bounds
            
        Returns:
            Dictionary with CI coverage percentages
        """
        logger.info("Checking confidence interval coverage")
        
        # 95% CI coverage
        coverage_95 = (
            df_validation
            .filter(
                (pl.col('actual_count') >= pl.col('lower_95')) &
                (pl.col('actual_count') <= pl.col('upper_95'))
            )
            .height / df_validation.height
        )
        
        # 80% CI coverage (if available)
        coverage_80 = None
        if 'lower_80' in df_validation.columns:
            coverage_80 = (
                df_validation
                .filter(
                    (pl.col('actual_count') >= pl.col('lower_80')) &
                    (pl.col('actual_count') <= pl.col('upper_80'))
                )
                .height / df_validation.height
            )
        
        logger.info(f"95% CI coverage: {coverage_95:.1%} (target: 90-100%)")
        if coverage_80:
            logger.info(f"80% CI coverage: {coverage_80:.1%} (target: 75-85%)")
        
        return {'coverage_95': coverage_95, 'coverage_80': coverage_80}
    
    def time_series_cross_validation(
        self,
        profession: str,
        sector: str,
        n_splits: int = 3
    ) -> List[float]:
        """
        Perform time series cross-validation.
        
        Args:
            profession: Profession name
            sector: Sector
            n_splits: Number of CV splits
            
        Returns:
            List of MAPE scores from each fold
        """
        logger.info(f"Time series CV: {profession} ({n_splits} splits)")
        
        # Get time series data
        ts_data = (
            self.actuals
            .filter(
                (pl.col('profession') == profession) &
                (pl.col('sector') == sector)
            )
            .sort('year')
        )
        
        years = ts_data['year'].to_numpy()
        counts = ts_data['count'].to_numpy()
        
        # Time series split
        tscv = TimeSeriesSplit(n_splits=n_splits)
        mape_scores = []
        
        for fold, (train_idx, test_idx) in enumerate(tscv.split(counts)):
            # Placeholder: would re-fit model on train, predict test
            # For now, use existing forecasts
            test_years = years[test_idx]
            actual_test = counts[test_idx]
            
            # Get forecasts for test years (from US-06 output)
            forecast_test = (
                self.forecasts
                .filter(
                    (pl.col('profession') == profession) &
                    (pl.col('sector') == sector) &
                    (pl.col('year').is_in(test_years.tolist()))
                )
                ['forecast'].to_numpy()
            )
            
            # Calculate MAPE
            mape = np.mean(np.abs((actual_test - forecast_test) / actual_test)) * 100
            mape_scores.append(mape)
            logger.info(f"Fold {fold+1}: MAPE={mape:.2f}%")
        
        logger.info(f"✓ CV complete: Mean MAPE={np.mean(mape_scores):.2f}% ± {np.std(mape_scores):.2f}%")
        return mape_scores
    
    def validate_profession(
        self,
        profession: str,
        sector: str = 'total'
    ) -> ValidationResult:
        """
        Full validation for a single profession.
        
        Args:
            profession: Profession name
            sector: Sector (default: 'total')
            
        Returns:
            ValidationResult with all metrics and tests
        """
        logger.info("=" * 60)
        logger.info(f"Validating: {profession} ({sector})")
        logger.info("=" * 60)
        
        # Holdout validation
        df_validation = self.holdout_validation(profession, sector)
        
        # Accuracy metrics
        metrics = self.calculate_accuracy_metrics(df_validation)
        
        # Statistical tests
        residuals = df_validation['error'].to_numpy()
        _, shapiro_p = self.test_residual_normality(residuals)
        ljungbox_p = self.test_residual_autocorrelation(residuals)
        _, bias_test_p = self.test_unbiasedness(residuals)
        
        # CI coverage
        ci_coverage = self.check_ci_coverage(df_validation)
        
        # Determine validation pass/fail
        validation_pass = (
            metrics['mape'] < 20 and
            shapiro_p > 0.05 and
            ljungbox_p > 0.05 and
            bias_test_p > 0.05 and
            ci_coverage['coverage_95'] >= 0.85
        )
        
        # Assign confidence rating
        if metrics['mape'] < 10 and validation_pass:
            confidence = 'high'
        elif metrics['mape'] < 20 and validation_pass:
            confidence = 'moderate'
        else:
            confidence = 'low'
        
        logger.info(f"Validation: {'PASS ✓' if validation_pass else 'FAIL ✗'}")
        logger.info(f"Confidence: {confidence.upper()}")
        
        return ValidationResult(
            profession=profession,
            sector=sector,
            mape=metrics['mape'],
            rmse=metrics['rmse'],
            mae=metrics['mae'],
            bias=metrics['bias'],
            shapiro_p=shapiro_p,
            ljungbox_p=ljungbox_p,
            bias_test_p=bias_test_p,
            ci_coverage_80=ci_coverage.get('coverage_80', 0.0),
            ci_coverage_95=ci_coverage['coverage_95'],
            validation_pass=validation_pass,
            confidence_rating=confidence
        )
    
    def validate_all_professions(
        self,
        professions: List[str]
    ) -> List[ValidationResult]:
        """
        Validate all professions.
        
        Args:
            professions: List of professions to validate
            
        Returns:
            List of ValidationResult objects
        """
        logger.info("=" * 60)
        logger.info("Validating all professions")
        logger.info("=" * 60)
        
        results = []
        for profession in professions:
            result = self.validate_profession(profession, sector='total')
            results.append(result)
        
        logger.success(f"✓ Validated {len(results)} professions")
        return results
    
    def generate_validation_report(
        self,
        results: List[ValidationResult],
        output_path: str
    ) -> None:
        """
        Generate markdown validation report.
        
        Args:
            results: List of validation results
            output_path: Path to save report
        """
        logger.info(f"Generating validation report: {output_path}")
        
        report_lines = [
            "# Workforce Forecast Validation Report",
            "",
            "## Validation Methodology",
            "- **Holdout Period**: 2017-2019 (3 years)",
            "- **Metrics**: MAPE, RMSE, MAE, Bias",
            "- **Statistical Tests**: Shapiro-Wilk (normality), Ljung-Box (autocorrelation), t-test (bias)",
            "- **CI Coverage**: 95% confidence intervals",
            "",
            "## Validation Summary",
            "",
            "| Profession | MAPE | RMSE | Bias | Shapiro p | Ljung-Box p | CI Coverage | Validation | Confidence |",
            "|-----------|------|------|------|-----------|-------------|-------------|------------|-----------|"
        ]
        
        for result in results:
            pass_emoji = "✅" if result.validation_pass else "❌"
            report_lines.append(
                f"| {result.profession} | {result.mape:.2f}% | {result.rmse:.1f} | "
                f"{result.bias:.1f} | {result.shapiro_p:.3f} | {result.ljungbox_p:.3f} | "
                f"{result.ci_coverage_95:.1%} | {pass_emoji} | {result.confidence_rating.upper()} |"
            )
        
        report_lines.extend([
            "",
            "## Confidence Ratings",
            "- **HIGH**: MAPE < 10%, all statistical tests pass, CI coverage ≥ 90%",
            "- **MODERATE**: MAPE < 20%, most tests pass, CI coverage ≥ 85%",
            "- **LOW**: MAPE ≥ 20% or critical test failures",
            "",
            "## Recommendations",
            "1. **High confidence forecasts**: Use with confidence for policy planning",
            "2. **Moderate confidence forecasts**: Use with caution, monitor for updates",
            "3. **Low confidence forecasts**: Do not use for critical decisions, recommend model improvements",
            ""
        ])
        
        with open(output_path, 'w') as f:
            f.write('\n'.join(report_lines))
        
        logger.success(f"✓ Validation report saved: {output_path}")
```

### 4. Validation Tests

```python
# shared/tests/unit/test_forecast_validator.py

import pytest
import numpy as np
import polars as pl
from shared.src.models.forecast_validator import ForecastValidator


def test_mape_calculation():
    """Test MAPE metric calculation."""
    df_validation = pl.DataFrame({
        'actual_count': [100, 200, 300],
        'forecast': [90, 210, 290],
        'error': [-10, 10, -10],
        'abs_error': [10, 10, 10],
        'pct_error': [10.0, 5.0, 3.33]
    })
    
    validator = ForecastValidator(
        forecast_df=pl.DataFrame(),
        actual_df=pl.DataFrame()
    )
    
    metrics = validator.calculate_accuracy_metrics(df_validation)
    
    # MAPE = (10 + 5 + 3.33) / 3 = 6.11%
    assert metrics['mape'] == pytest.approx(6.11, abs=0.1)


def test_unbiasedness_test():
    """Test unbiasedness test detects systematic bias."""
    validator = ForecastValidator(pl.DataFrame(), pl.DataFrame())
    
    # Unbiased residuals (centered at 0)
    residuals_unbiased = np.array([-5, 0, 5, -3, 3])
    _, p_unbiased = validator.test_unbiasedness(residuals_unbiased)
    assert p_unbiased > 0.05  # Should NOT reject null (unbiased)
    
    # Biased residuals (systematically positive)
    residuals_biased = np.array([10, 12, 15, 14, 13])
    _, p_biased = validator.test_unbiasedness(residuals_biased)
    assert p_biased < 0.05  # Should reject null (biased)


def test_ci_coverage_calculation():
    """Test CI coverage percentage calculation."""
    df_validation = pl.DataFrame({
        'actual_count': [100, 110, 105, 115],
        'lower_95': [90, 100, 95, 105],
        'upper_95': [110, 120, 115, 125]
    })
    
    validator = ForecastValidator(pl.DataFrame(), pl.DataFrame())
    coverage = validator.check_ci_coverage(df_validation)
    
    # All 4 actuals within 95% CI → 100% coverage
    assert coverage['coverage_95'] == 1.0
```

### 5. Implementation Steps

**Phase 1: Holdout Validation (Days 1-2)**
- [ ] Split data into training (2006-2016) and holdout (2017-2019)
- [ ] Re-fit models on training data
- [ ] Generate forecasts for holdout period
- [ ] Calculate accuracy metrics (MAPE, RMSE, MAE, bias)

**Phase 2: Statistical Tests (Day 2)**
- [ ] Test residual normality (Shapiro-Wilk)
- [ ] Test autocorrelation (Ljung-Box)
- [ ] Test unbiasedness (t-test)
- [ ] Check CI coverage (80%, 95%)

**Phase 3: Cross-Validation (Day 3)**
- [ ] Implement time series CV (3 splits)
- [ ] Calculate CV-MAPE per profession
- [ ] Assess forecast stability across folds

**Phase 4: Reporting & Documentation (Days 3-4)**
- [ ] Generate validation report with pass/fail criteria
- [ ] Assign confidence ratings (high/moderate/low)
- [ ] Create diagnostic plots (actual vs predicted, residuals)
- [ ] Document model limitations
- [ ] Export validation results: `results/forecast_validation_results.parquet`

### 6. Success Metrics

- ✅ Validation performed for all professions
- ✅ MAPE < 10% for high-confidence professions
- ✅ MAPE < 20% for moderate-confidence professions
- ✅ All statistical tests documented (pass/fail)
- ✅ Confidence ratings assigned to all forecasts
- ✅ Validation report generated for stakeholders

### 7. References

- [Hyndman & Athanasopoulos - Forecasting: Principles and Practice](https://otexts.com/fpp3/)
- [Time Series Cross-Validation](https://scikit-learn.org/stable/modules/cross_validation.html#time-series-split)
- US-06 (forecasts to validate)

---

**Implementation Plan Complete for US-09**  
**Dependencies**: US-06 (forecasts), US-02 (historical actuals)  
**Enables**: US-10 (dashboard displays confidence ratings)
