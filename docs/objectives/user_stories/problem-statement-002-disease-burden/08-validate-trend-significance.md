# Validate Statistical Significance of Trends (Lifecycle Stage: Validation)

**Story ID**: PS-002-US-08  
**Epic**: National Disease Burden Temporal Trends Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: S (3 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Scientist ensuring analytical rigor**,  
I want **to validate the statistical significance of mortality trends using hypothesis testing, confidence intervals, and robustness checks**,  
So that **policy makers can confidently rely on trend findings without risk of spurious patterns or chance fluctuations**.

---

## 🎯 Acceptance Criteria

1. **Trend significance testing completed**
   - Null hypothesis tests: are trends significantly different from zero? (p < 0.05)
   - Linear regression significance: slope coefficients tested for each disease
   - Confidence intervals: 95% CI calculated for all annual percentage changes
   - Multiple testing correction: Bonferroni or FDR adjustment applied

2. **Robustness validation performed**
   - Outlier sensitivity: trends recalculated excluding potential outlier years
   - Alternative time periods: trends tested for 1990-2009 vs 2010-2019 subsets
   - Different trend models: compare linear vs exponential vs polynomial trends
   - Bootstrap validation: resampling to assess trend stability

3. **Assumption testing conducted**
   - Normality of residuals: Shapiro-Wilk or Q-Q plots
   - Homoscedasticity: constant variance of residuals over time
   - Autocorrelation: Durbin-Watson test for serial correlation
   - Linearity: residual plots to check linear model appropriateness

4. **Validation report generated**
   - Output file: `results/tables/mortality_trend_validation_report.csv`
   - Statistical test results: p-values, confidence intervals, test statistics
   - Robustness summary: sensitivity to outliers and model specifications
   - Recommendations: which trends are robust vs require cautious interpretation

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for data processing
- **Statistical Testing**: scipy, statsmodels for hypothesis tests
- **Visualization**: Matplotlib for diagnostic plots
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for validation functions

---

## 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#temporal-trend-features) - Statistical methods for trend analysis
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md) - Ensuring robust trend identification

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data manipulation
- `scipy>=1.11.0`: Statistical tests (t-tests, normality tests)
- `statsmodels>=0.14.0`: Regression diagnostics, Durbin-Watson test
- `matplotlib>=3.8.0`: Diagnostic plots (residuals, Q-Q plots)
- `numpy>=1.24.0`: Bootstrap resampling
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: PS-002-US-03, PS-002-US-04 (Trend analyses - BLOCKING)
- **Data Sources**: `shared/data/3_interim/mortality_trends_integrated_clean.parquet`
- **Config Files**: `config/analysis.yml` (significance thresholds, alpha levels)

---

## ✅ Implementation Tasks

### Trend Significance Testing
- [ ] Linear regression for each disease: mortality_rate ~ year
- [ ] Extract slope coefficients and standard errors
- [ ] Calculate t-statistics and p-values for slopes
- [ ] Test null hypothesis: H₀: slope = 0 (no trend)
- [ ] 95% confidence intervals for slopes
- [ ] Bonferroni correction for multiple comparisons (3 diseases)

### Robustness Checks
- [ ] Outlier detection: identify years with residuals > 2 std dev
- [ ] Sensitivity analysis: recalculate trends excluding each potential outlier
- [ ] Subset analysis: trends for 1990-2009 vs 2010-2019
- [ ] Model comparison: linear vs exponential vs segmented regression (AIC/BIC)
- [ ] Document robust trends: significant across multiple specifications

### Assumption Validation
- [ ] Normality test: Shapiro-Wilk test on residuals (p > 0.05 = normal)
- [ ] Q-Q plots: visual assessment of residual normality
- [ ] Homoscedasticity: Breusch-Pagan test for constant variance
- [ ] Autocorrelation: Durbin-Watson statistic (value ~2 = no autocorrelation)
- [ ] Linearity check: residual plots (should show no pattern)

### Bootstrap Validation
- [ ] Implement bootstrap resampling: 1,000 bootstrap samples
- [ ] Recalculate trend slopes for each bootstrap sample
- [ ] Generate bootstrap confidence intervals (percentile method)
- [ ] Compare bootstrap CI with standard regression CI
- [ ] Assess trend stability: coefficient of variation across bootstrap samples

### Diagnostic Visualization
- [ ] Residual plots: residuals vs fitted values (check homoscedasticity)
- [ ] Q-Q plots: assess normality of residuals
- [ ] Time series of residuals: check for autocorrelation patterns
- [ ] Save diagnostic plots: `reports/figures/trend_validation_diagnostics_*.png`

### Validation Reporting
- [ ] Compile test results: all p-values, confidence intervals, test statistics
- [ ] Robustness summary: trends unchanged vs sensitive to outliers
- [ ] Recommendation flags: robust (green), caution (yellow), unreliable (red)
- [ ] Generate validation report CSV
- [ ] Update analytical report with validation section

### Testing & Documentation
- [ ] Unit tests for statistical test functions
- [ ] Validate bootstrap implementation: test with known distributions
- [ ] Test assumption checks with simulated data
- [ ] Docstrings for all validation functions
- [ ] Methodology documentation: explain statistical tests used

---

## 📌 Notes

**Polars + statsmodels Validation Example**:
```python
import polars as pl
import statsmodels.api as sm
from scipy import stats
from loguru import logger

# Load mortality data
df = pl.read_parquet("shared/data/3_interim/mortality_trends_integrated_clean.parquet")

# Linear regression for cancer
df_cancer = df.filter(pl.col('disease_category') == 'cancer')
X = df_cancer['year'].to_numpy()
y = df_cancer['mortality_rate'].to_numpy()
X_with_const = sm.add_constant(X)

# Fit model
model = sm.OLS(y, X_with_const).fit()

# Extract results
slope = model.params[1]
p_value = model.pvalues[1]
ci_lower, ci_upper = model.conf_int()[1]

logger.info(f"Cancer trend: slope={slope:.4f}, p={p_value:.4f}, 95% CI=[{ci_lower:.4f}, {ci_upper:.4f}]")

# Test significance
if p_value < 0.05:
    logger.info("✓ Trend is statistically significant (p < 0.05)")
else:
    logger.warning("⚠ Trend not statistically significant")
```

**Bootstrap Resampling Implementation**:
```python
import numpy as np

def bootstrap_trend(X, y, n_bootstrap=1000):
    """Bootstrap confidence intervals for trend slope."""
    n = len(X)
    slopes = []
    
    for _ in range(n_bootstrap):
        # Resample with replacement
        indices = np.random.choice(n, size=n, replace=True)
        X_boot = X[indices]
        y_boot = y[indices]
        
        # Fit model
        X_boot_const = sm.add_constant(X_boot)
        model = sm.OLS(y_boot, X_boot_const).fit()
        slopes.append(model.params[1])
    
    # 95% confidence interval (percentile method)
    ci_lower = np.percentile(slopes, 2.5)
    ci_upper = np.percentile(slopes, 97.5)
    
    return np.mean(slopes), ci_lower, ci_upper

mean_slope, boot_ci_lower, boot_ci_upper = bootstrap_trend(X, y)
logger.info(f"Bootstrap 95% CI: [{boot_ci_lower:.4f}, {boot_ci_upper:.4f}]")
```

**Statistical Significance Thresholds**:
- **p-value < 0.05**: Statistically significant trend (standard threshold)
- **p-value < 0.01**: Highly significant (stronger evidence)
- **p-value < 0.001**: Very highly significant (very strong evidence)

**Multiple Testing Correction**:
- Testing 3 diseases → Bonferroni correction: α = 0.05/3 = 0.017
- Adjust significance threshold to 0.017 to control family-wise error rate

**Interpretation Guidelines**:
- **Robust trend**: Significant (p < 0.05) AND survives outlier exclusion AND bootstrap CI excludes 0
- **Potentially robust**: Significant but sensitive to outliers or model specification
- **Unreliable trend**: Non-significant (p ≥ 0.05) OR highly sensitive to outliers

**Durbin-Watson Interpretation**:
- DW < 1.5: Positive autocorrelation (concern for time series)
- 1.5 ≤ DW ≤ 2.5: No autocorrelation (good)
- DW > 2.5: Negative autocorrelation

**Expected Validation Results**:
- Cancer, stroke, IHD trends likely robust (30 years data, consistent trends)
- Autocorrelation possible due to time series nature (may need ARIMA models)
- Linear trends may oversimplify (Joinpoint may better capture reality)

**Output Schema**:
```yaml
# results/tables/mortality_trend_validation_report.csv
columns:
  disease_category: categorical
  slope: float  # regression slope
  p_value: float  # significance test
  ci_95_lower: float
  ci_95_upper: float
  shapiro_p: float  # normality test
  durbin_watson: float  # autocorrelation
  robust_to_outliers: boolean
  validation_status: string  # robust, caution, unreliable
```

---

## Implementation Plan

### 1. Feature Overview & Component Analysis

**Purpose**: Rigorously validate statistical significance of mortality trends using hypothesis tests, confidence intervals, and robustness checks to ensure policy decisions rest on solid evidence.

**Primary User Role**: Data Scientist ensuring analytical rigor

**Reuse Strategy**:
- ✅ **US-03 outputs**: Trend analysis results from `results/tables/mortality_trend_analysis.csv`
- ✅ **`shared/data/3_interim/mortality_trends_integrated_clean.parquet`** - Raw data for regression
- 🆕 **`shared/src/analysis/statistical_validation.py`** - Validation functions module
- 🆕 **`results/tables/mortality_trend_validation_report.csv`** - Validation results
- 🆕 **`reports/figures/trend_validation_diagnostics_*.png`** - Diagnostic plots

### 3. ML Model Evaluation & Selection

**N/A** - Statistical hypothesis testing and validation, not ML.

### 4-6. Code Specifications

**Validation Functions** (`shared/src/analysis/statistical_validation.py`):

```python
import polars as pl
import numpy as np
import statsmodels.api as sm
from scipy import stats
from loguru import logger
from typing import Dict, Tuple, List
from pathlib import Path

def test_trend_significance(
    df: pl.DataFrame,
    disease: str,
    alpha: float = 0.05
) -> Dict[str, float]:
    """Test statistical significance of linear trend.
    
    Args:
        df: Mortality data with [year, disease_category, mortality_rate]
        disease: Disease category to test
        alpha: Significance level
        
    Returns:
        Dictionary with slope, p-value, confidence intervals, R²
    """
    df_disease = df.filter(pl.col('disease_category') == disease)
    
    X = df_disease['year'].to_numpy()
    y = df_disease['mortality_rate'].to_numpy()
    X_with_const = sm.add_constant(X)
    
    # Fit OLS model
    model = sm.OLS(y, X_with_const).fit()
    
    # Extract statistics
    slope = model.params[1]
    p_value = model.pvalues[1]
    ci_lower, ci_upper = model.conf_int(alpha=alpha)[1]
    r_squared = model.rsquared
    
    results = {
        'disease_category': disease,
        'slope': slope,
        'p_value': p_value,
        'ci_95_lower': ci_lower,
        'ci_95_upper': ci_upper,
        'r_squared': r_squared,
        'significant': p_value < alpha
    }
    
    logger.info(f"{disease}: slope={slope:.4f}, p={p_value:.4f}, significant={p_value < alpha}")
    return results


def bonferroni_correction(
    p_values: List[float],
    alpha: float = 0.05
) -> Tuple[List[bool], float]:
    """Apply Bonferroni correction for multiple testing.
    
    Args:
        p_values: List of p-values from multiple tests
        alpha: Family-wise error rate
        
    Returns:
        Tuple of (list of significance booleans, adjusted alpha)
    """
    n_tests = len(p_values)
    adjusted_alpha = alpha / n_tests
    
    significant = [p < adjusted_alpha for p in p_values]
    
    logger.info(f"Bonferroni correction: {n_tests} tests, adjusted α = {adjusted_alpha:.4f}")
    logger.info(f"Significant after correction: {sum(significant)}/{n_tests}")
    
    return significant, adjusted_alpha


def test_normality(residuals: np.ndarray) -> Tuple[float, float]:
    """Test normality of residuals using Shapiro-Wilk test.
    
    Args:
        residuals: Model residuals
        
    Returns:
        Tuple of (test statistic, p-value)
    """
    statistic, p_value = stats.shapiro(residuals)
    
    if p_value > 0.05:
        logger.info(f"✓ Residuals appear normal (Shapiro-Wilk p={p_value:.4f})")
    else:
        logger.warning(f"⚠ Residuals may not be normal (Shapiro-Wilk p={p_value:.4f})")
    
    return statistic, p_value


def test_autocorrelation(residuals: np.ndarray) -> float:
    """Test for autocorrelation using Durbin-Watson statistic.
    
    Args:
        residuals: Model residuals
        
    Returns:
        Durbin-Watson statistic (0-4 scale, ~2 = no autocorrelation)
    """
    from statsmodels.stats.stattools import durbin_watson
    
    dw_stat = durbin_watson(residuals)
    
    if 1.5 <= dw_stat <= 2.5:
        logger.info(f"✓ No significant autocorrelation (DW={dw_stat:.2f})")
    else:
        logger.warning(f"⚠ Possible autocorrelation (DW={dw_stat:.2f})")
    
    return dw_stat


def bootstrap_confidence_intervals(
    X: np.ndarray,
    y: np.ndarray,
    n_bootstrap: int = 1000,
    alpha: float = 0.05
) -> Tuple[float, float, float]:
    """Calculate bootstrap confidence intervals for slope.
    
    Args:
        X: Independent variable (year)
        y: Dependent variable (mortality rate)
        n_bootstrap: Number of bootstrap samples
        alpha: Significance level for CI
        
    Returns:
        Tuple of (mean slope, CI lower, CI upper)
    """
    n = len(X)
    slopes = []
    
    for _ in range(n_bootstrap):
        # Resample with replacement
        indices = np.random.choice(n, size=n, replace=True)
        X_boot = X[indices]
        y_boot = y[indices]
        
        # Fit model
        X_boot_const = sm.add_constant(X_boot)
        try:
            model = sm.OLS(y_boot, X_boot_const).fit()
            slopes.append(model.params[1])
        except:
            continue  # Skip failed fits
    
    slopes = np.array(slopes)
    mean_slope = np.mean(slopes)
    ci_lower = np.percentile(slopes, (alpha/2) * 100)
    ci_upper = np.percentile(slopes, (1 - alpha/2) * 100)
    
    logger.info(f"Bootstrap ({n_bootstrap} samples): mean slope={mean_slope:.4f}, 95% CI=[{ci_lower:.4f}, {ci_upper:.4f}]")
    
    return mean_slope, ci_lower, ci_upper


def outlier_sensitivity_analysis(
    df: pl.DataFrame,
    disease: str,
    threshold_std: float = 2.0
) -> Dict[str, any]:
    """Test trend robustness to outliers.
    
    Args:
        df: Mortality data
        disease: Disease to analyze
        threshold_std: Standard deviations for outlier detection
        
    Returns:
        Dictionary with full vs outlier-excluded results
    """
    df_disease = df.filter(pl.col('disease_category') == disease)
    
    X = df_disease['year'].to_numpy()
    y = df_disease['mortality_rate'].to_numpy()
    X_const = sm.add_constant(X)
    
    # Full model
    model_full = sm.OLS(y, X_const).fit()
    slope_full = model_full.params[1]
    
    # Detect outliers
    residuals = model_full.resid
    std_resid = np.std(residuals)
    outlier_mask = np.abs(residuals) > threshold_std * std_resid
    outlier_years = X[outlier_mask]
    
    # Model without outliers
    X_clean = X[~outlier_mask]
    y_clean = y[~outlier_mask]
    X_clean_const = sm.add_constant(X_clean)
    
    model_clean = sm.OLS(y_clean, X_clean_const).fit()
    slope_clean = model_clean.params[1]
    
    # Calculate sensitivity
    slope_change_pct = abs((slope_clean - slope_full) / slope_full) * 100 if slope_full != 0 else 0
    
    robust = slope_change_pct < 10  # Less than 10% change = robust
    
    results = {
        'disease_category': disease,
        'slope_full_model': slope_full,
        'slope_no_outliers': slope_clean,
        'outlier_years': outlier_years.tolist() if len(outlier_years) > 0 else [],
        'slope_change_pct': slope_change_pct,
        'robust_to_outliers': robust
    }
    
    if robust:
        logger.info(f"✓ {disease} trend robust to outliers (change: {slope_change_pct:.1f}%)")
    else:
        logger.warning(f"⚠ {disease} trend sensitive to outliers (change: {slope_change_pct:.1f}%)")
    
    return results


def comprehensive_validation(
    df: pl.DataFrame,
    disease: str,
    alpha: float = 0.05
) -> Dict[str, any]:
    """Run comprehensive statistical validation for a disease trend.
    
    Args:
        df: Mortality data
        disease: Disease category
        alpha: Significance level
        
    Returns:
        Dictionary with all validation metrics
    """
    logger.info(f"=== Comprehensive Validation: {disease} ===")
    
    # 1. Trend significance
    trend_results = test_trend_significance(df, disease, alpha)
    
    # Extract data for regression
    df_disease = df.filter(pl.col('disease_category') == disease)
    X = df_disease['year'].to_numpy()
    y = df_disease['mortality_rate'].to_numpy()
    X_const = sm.add_constant(X)
    model = sm.OLS(y, X_const).fit()
    
    # 2. Normality test
    shapiro_stat, shapiro_p = test_normality(model.resid)
    
    # 3. Autocorrelation test
    dw_stat = test_autocorrelation(model.resid)
    
    # 4. Bootstrap CI
    boot_mean, boot_ci_lower, boot_ci_upper = bootstrap_confidence_intervals(X, y, 1000, alpha)
    
    # 5. Outlier sensitivity
    outlier_results = outlier_sensitivity_analysis(df, disease, 2.0)
    
    # Determine validation status
    if trend_results['significant'] and outlier_results['robust_to_outliers'] and shapiro_p > 0.05:
        validation_status = 'robust'
    elif trend_results['significant']:
        validation_status = 'caution'
    else:
        validation_status = 'unreliable'
    
    # Compile results
    results = {
        **trend_results,
        'shapiro_statistic': shapiro_stat,
        'shapiro_p': shapiro_p,
        'durbin_watson': dw_stat,
        'boot_ci_lower': boot_ci_lower,
        'boot_ci_upper': boot_ci_upper,
        'robust_to_outliers': outlier_results['robust_to_outliers'],
        'outlier_years': outlier_results['outlier_years'],
        'validation_status': validation_status
    }
    
    logger.info(f"{disease} validation status: {validation_status.upper()}")
    
    return results
```

### 10. Testing Strategy

```python
# shared/tests/unit/test_statistical_validation.py

import pytest
import numpy as np
import polars as pl

def test_trend_significance_with_known_data():
    """Test trend significance with data that has known slope."""
    # Create perfect linear trend
    years = np.arange(1990, 2020)
    rates = 150 - 2 * (years - 1990)  # Declining by 2 per year
    
    df = pl.DataFrame({
        'year': years,
        'disease_category': ['cancer'] * len(years),
        'mortality_rate': rates
    })
    
    result = test_trend_significance(df, 'cancer', 0.05)
    
    assert result['slope'] == pytest.approx(-2.0, abs=0.01)
    assert result['p_value'] < 0.001  # Should be highly significant
    assert result['significant'] == True


def test_bonferroni_correction():
    """Test Bonferroni correction for multiple testing."""
    p_values = [0.01, 0.04, 0.10]
    
    significant, adj_alpha = bonferroni_correction(p_values, alpha=0.05)
    
    assert adj_alpha == pytest.approx(0.0167, abs=0.001)  # 0.05/3
    assert significant == [True, False, False]  # Only first p<0.0167


def test_outlier_sensitivity():
    """Test outlier sensitivity analysis."""
    # Create data with one outlier
    years = np.arange(1990, 2020)
    rates = 150 - 2 * (years - 1990)
    rates[15] = 200  # Outlier in year 2005
    
    df = pl.DataFrame({
        'year': years,
        'disease_category': ['cancer'] * len(years),
        'mortality_rate': rates
    })
    
    result = outlier_sensitivity_analysis(df, 'cancer', 2.0)
    
    assert 2005 in result['outlier_years']
    assert 'robust_to_outliers' in result
```

### 11. Implementation Steps

**Phase 1: Trend Significance Testing**
- [ ] Implement `test_trend_significance()` for linear regression and p-values
- [ ] Calculate 95% confidence intervals for slopes
- [ ] Apply Bonferroni correction for multiple diseases
- [ ] Unit test significance testing functions

**Phase 2: Assumption Testing**
- [ ] Implement `test_normality()` using Shapiro-Wilk
- [ ] Implement `test_autocorrelation()` using Durbin-Watson
- [ ] Generate diagnostic plots (Q-Q plots, residual plots)
- [ ] Unit test assumption tests

**Phase 3: Robustness Checks**
- [ ] Implement `bootstrap_confidence_intervals()`
- [ ] Implement `outlier_sensitivity_analysis()`
- [ ] Compare bootstrap CI vs standard CI
- [ ] Test trend stability with subset periods (1990-2009 vs 2010-2019)

**Phase 4: Comprehensive Validation**
- [ ] Implement `comprehensive_validation()` combining all tests
- [ ] Run validation for all diseases
- [ ] Generate validation report CSV
- [ ] Create diagnostic visualizations

**Phase 5: Reporting**
- [ ] Compile validation results into summary table
- [ ] Flag trends as robust/caution/unreliable
- [ ] Document methodology and interpretation guidelines
- [ ] Generate validation report section for analytical documentation

### 14. Data Quality & Validation

**Pre-Validation Checks**:
- Verify continuous time series (no gaps 1990-2019)
- Check for data entry errors (implausible values)
- Confirm data loaded correctly from US-03

**Validation Criteria**:
- **Robust**: p < 0.05, passes normality, bootstrap CI excludes 0, outlier-insensitive
- **Caution**: p < 0.05 but violates some assumptions or outlier-sensitive
- **Unreliable**: p ≥ 0.05 or highly unstable

### 18. Success Metrics

- ✅ Trend significance tested for all 3 diseases
- ✅ Bonferroni correction applied
- ✅ Bootstrap CIs calculated (n=1000 samples)
- ✅ Outlier sensitivity assessed
- ✅ Validation report generated with status flags
- ✅ All tests passing (≥80% coverage)

---

✅ **US-08 Implementation Plan Complete** - Statistical validation framework defined

# Add constant for intercept
X_with_const = sm.add_constant(X)

# Fit OLS regression
model = sm.OLS(y, X_with_const).fit()

# Extract statistics
slope = model.params[1]
p_value = model.pvalues[1]
ci_lower, ci_upper = model.conf_int()[1]

logger.info(f"Cancer trend: slope={slope:.4f}, p={p_value:.4f}, 95% CI=[{ci_lower:.4f}, {ci_upper:.4f}]")

# Test assumptions
residuals = model.resid
shapiro_stat, shapiro_p = stats.shapiro(residuals)
dw_statistic = sm.stats.stattools.durbin_watson(residuals)

logger.info(f"Normality (Shapiro-Wilk): p={shapiro_p:.4f}")
logger.info(f"Autocorrelation (Durbin-Watson): {dw_statistic:.4f}")

# Robustness: exclude outliers
outliers = np.abs(residuals) > 2 * np.std(residuals)
if outliers.any():
    model_robust = sm.OLS(y[~outliers], X_with_const[~outliers]).fit()
    logger.info(f"Robust trend (excl. outliers): slope={model_robust.params[1]:.4f}")
```

**Bootstrap Validation Example**:
```python
import numpy as np

def bootstrap_trend(X, y, n_bootstrap=1000):
    """Calculate bootstrap confidence interval for trend slope"""
    n = len(X)
    slopes = []
    
    for _ in range(n_bootstrap):
        # Resample with replacement
        indices = np.random.choice(n, size=n, replace=True)
        X_boot = X[indices]
        y_boot = y[indices]
        
        # Fit model
        X_boot_const = sm.add_constant(X_boot)
        model_boot = sm.OLS(y_boot, X_boot_const).fit()
        slopes.append(model_boot.params[1])
    
    # 95% confidence interval (percentile method)
    ci_lower = np.percentile(slopes, 2.5)
    ci_upper = np.percentile(slopes, 97.5)
    
    return np.mean(slopes), ci_lower, ci_upper

mean_slope, boot_ci_lower, boot_ci_upper = bootstrap_trend(X, y)
logger.info(f"Bootstrap 95% CI: [{boot_ci_lower:.4f}, {boot_ci_upper:.4f}]")
```

**Statistical Significance Thresholds**:
- **p-value < 0.05**: Statistically significant trend (standard threshold)
- **p-value < 0.01**: Highly significant (stronger evidence)
- **p-value < 0.001**: Very highly significant (very strong evidence)

**Multiple Testing Correction**:
- Testing 3 diseases → Bonferroni correction: α = 0.05/3 = 0.017
- Adjust significance threshold to 0.017 to control family-wise error rate

**Interpretation Guidelines**:
- **Robust trend**: Significant (p < 0.05) AND survives outlier exclusion AND bootstrap CI excludes 0
- **Potentially robust**: Significant but sensitive to outliers or model specification
- **Unreliable trend**: Non-significant (p ≥ 0.05) OR highly sensitive to outliers

**Durbin-Watson Interpretation**:
- DW < 1.5: Positive autocorrelation (concern for time series)
- 1.5 ≤ DW ≤ 2.5: No autocorrelation (good)
- DW > 2.5: Negative autocorrelation

**Expected Validation Results**:
- Cancer, stroke, IHD trends likely robust (30 years data, consistent trends)
- Autocorrelation possible due to time series nature (may need ARIMA models)
- Linear trends may oversimplify (Joinpoint may better capture reality)

**Output Schema**:
```yaml
# results/tables/mortality_trend_validation_report.csv
columns:
  disease_category: categorical
  slope: float  # regression slope
  p_value: float  # significance test
  ci_95_lower: float
  ci_95_upper: float
  shapiro_p: float  # normality test
  durbin_watson: float  # autocorrelation
  robust_to_outliers: boolean
  validation_status: string  # robust, caution, unreliable
```
