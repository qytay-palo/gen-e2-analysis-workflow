# User Story: 2 - Exploratory Time Series Analysis and Pattern Identification

**As a** forecasting analyst,  
**I want** to conduct comprehensive exploratory time series analysis on mortality trends,  
**so that** I can understand temporal patterns, identify appropriate forecasting methods, and establish baseline expectations.

## 1. 🎯 Acceptance Criteria

1. **Stationarity Assessment Complete**
   - Perform Augmented Dickey-Fuller (ADF) test for each disease time series
   - Perform KPSS test (Kwiatkowski-Phillips-Schmidt-Shin) for confirmation
   - Document stationarity status (stationary/non-stationary) with p-values
   - Determine differencing requirements (d parameter for ARIMA)
   - Generate stationarity test summary report

2. **Trend Analysis Conducted**
   - Decompose time series into trend, seasonal (if applicable), and residual components
   - Calculate Annual Percent Change (APC) for each disease 1990-2019
   - Identify trend direction and magnitude (increasing/decreasing/stable)
   - Detect trend inflection points using statistical methods
   - Visualize trend evolution with 95% confidence bands

3. **Autocorrelation Structure Analyzed**
   - Generate ACF (Autocorrelation Function) plots for each disease
   - Generate PACF (Partial Autocorrelation Function) plots
   - Identify significant lags (AR order estimation)
   - Determine MA order from ACF cutoff pattern
   - Document recommended ARIMA parameters (p, d, q) based on ACF/PACF

4. **Temporal Pattern Characterization**
   - Calculate coefficient of variation (temporal volatility) by disease
   - Identify long-term trends vs short-term fluctuations
   - Detect potential structural breaks (e.g., policy changes, medical advances)
   - Quantify forecast difficulty (high/medium/low based on pattern complexity)
   - Compare temporal patterns across 3 diseases

5. **Exploratory Findings Documented**
   - EDA report generated with key findings and implications for forecasting
   - Preliminary forecasting method recommendations by disease
   - Visualization gallery: trends, ACF/PACF, decomposition charts
   - Saved to: `reports/figures/problem-statement-006/exploratory_analysis_{timestamp}.pdf`

## 2. 🔒 Technical Constraints

- **Analysis library**: `statsmodels` for time series decomposition and tests
- **Visualization**: `matplotlib` + `seaborn` for static charts, `plotly` for interactive exploration
- **Data input**: Cleaned data from User Story 1 (`shared/data/3_interim/forecasting/mortality_time_series_clean.parquet`)
- **Output storage**: Figures to `reports/figures/problem-statement-006/`, analysis results to `results/tables/problem-statement-006/`
- **Notebook**: Exploratory work in `problem-statements/ps-006-forecasting/notebooks/01-exploratory-time-series-analysis.ipynb`

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Stationarity, autocorrelation, trend analysis
- [Disease Burden Feature Engineering Guide](../../../domain_knowledge/disease-burden-feature-engineering-guide.md) - Annual Percent Change calculations, trend methodologies

**Key Statistical Concepts**:
- **Stationarity**: Time series with constant mean, variance, autocorrelation over time (required for ARIMA)
- **ADF Test**: Null hypothesis = series has unit root (non-stationary); reject if p < 0.05
- **ACF/PACF**: Identify AR and MA orders for ARIMA model specification
- **Trend Decomposition**: Additive vs multiplicative models

**Decision Rules**:
- If ADF p-value > 0.05 → Series is non-stationary, requires differencing
- ACF decays slowly → Non-stationary or strong AR component
- PACF cuts off after lag p → AR(p) model suggested
- ACF cuts off after lag q → MA(q) model suggested

## 4. 📦 Dependencies

**External Packages**:
- `statsmodels>=0.14.0` - Time series analysis, stationarity tests, decomposition
- `scipy>=1.10.0` - Statistical functions
- `matplotlib>=3.7.0` - Static visualizations
- `seaborn>=0.12.0` - Enhanced statistical plots
- `plotly>=5.14.0` - Interactive exploration (optional)
- `polars>=0.20.0` - Data manipulation

**Internal Dependencies**:
- Input data: `shared/data/3_interim/forecasting/mortality_time_series_clean.parquet` (from User Story 1)
- Utility functions: `shared/src/analysis/time_series_utils.py` (to be created)

## 5. ✅ Implementation Tasks

### Setup and Configuration
- ⬜ **Create Jupyter notebook** `01-exploratory-time-series-analysis.ipynb`
- ⬜ **Setup output directories** for figures and results
- ⬜ **Load cleaned mortality data** from interim storage
- ⬜ **Import time series analysis libraries** (statsmodels, scipy)

### Stationarity Testing
- ⬜ **Implement ADF test function**
  - Test each disease time series separately
  - Report test statistic, p-value, critical values
  - Interpretation: stationary if p < 0.05
- ⬜ **Implement KPSS test function** (confirmation)
  - Null hypothesis: series is stationary (opposite of ADF)
  - Report test statistic and critical values
- ⬜ **Create stationarity summary table**
  - Columns: disease, ADF_statistic, ADF_pvalue, KPSS_statistic, is_stationary
  - Save to: `results/tables/problem-statement-006/stationarity_tests_{timestamp}.csv`
- ⬜ **Determine differencing requirements**
  - If non-stationary, test first-difference (d=1)
  - Document recommended d parameter for ARIMA

### Trend Analysis
- ⬜ **Perform time series decomposition** using STL (Seasonal-Trend decomposition using LOESS)
  - Decompose into: trend, seasonal (if exists), residual
  - Visual inspection for trend patterns
- ⬜ **Calculate Annual Percent Change (APC)**
  - Year-over-year % change for each disease
  - Formula: APC = [(Rate_t / Rate_t-1) - 1] × 100
  - Visualize APC time series
- ⬜ **Fit linear trend model** for each disease
  - Simple linear regression: ASMR ~ year
  - Extract slope coefficient (trend magnitude)
  - Test statistical significance of trend
- ⬜ **Detect inflection points** using piecewise regression
  - Identify years where trend slope changes
  - Document inflection points for each disease
- ⬜ **Visualize trend evolution**
  - Line charts with trend lines overlaid
  - Confidence bands (95%) around trend
  - Save figures to `reports/figures/problem-statement-006/trends/`

### Autocorrelation Analysis
- ⬜ **Generate ACF plots** for each disease
  - Lags: 0 to 10 years
  - Identify significant lags (outside confidence interval)
  - Visual pattern recognition (decay, cutoff, oscillation)
- ⬜ **Generate PACF plots** for each disease
  - Identify direct autocorrelation after removing indirect effects
  - Determine AR order (p) from PACF cutoff lag
- ⬜ **Calculate Ljung-Box test statistic**
  - Test for presence of autocorrelation in residuals
  - Null hypothesis: no autocorrelation exists
- ⬜ **Recommend ARIMA parameters (p, d, q)**
  - p: from PACF cutoff lag or decay pattern
  - d: from stationarity tests
  - q: from ACF cutoff lag
  - Document recommendations in markdown table

### Temporal Pattern Characterization
- ⬜ **Calculate coefficient of variation** (CV) for each disease
  - CV = std / mean × 100
  - Higher CV = greater volatility = harder to forecast
- ⬜ **Compare long-term vs short-term variability**
  - Rolling 5-year std vs overall std
  - Identify periods of increased volatility
- ⬜ **Detect potential structural breaks** using Chow test
  - Test for parameter stability over time
  - Identify candidate break dates (e.g., 2005, 2010, 2015)
- ⬜ **Assess forecast difficulty** by disease
  - Combine: CV, stationarity, trend strength, autocorrelation strength
  - Classify: easy/medium/hard

### Cross-Disease Comparison
- ⬜ **Create comparative summary table**
  - Rows: cancer, stroke, heart disease
  - Columns: Mean ASMR, CV, Trend (slope), APC, Stationarity, Recommended ARIMA order
- ⬜ **Visualize all 3 diseases on same chart**
  - Normalized ASMR (index 1990 = 100) for comparability
  - Identify diverging vs converging trends
- ⬜ **Correlation analysis** between diseases
  - Check if mortality trends move together
  - Implication: univariate vs multivariate forecasting

### Deliverables and Documentation
- ⬜ **Generate comprehensive EDA report**
  - Executive summary: key findings in 2-3 paragraphs
  - Disease-by-disease analysis sections
  - Forecasting implications and method recommendations
  - Save as: `reports/figures/problem-statement-006/exploratory_analysis_report_{timestamp}.pdf`
- ⬜ **Create visualization gallery**
  - Trend evolution charts (3 diseases)
  - ACF/PACF plots (3 diseases)
  - Decomposition plots (3 diseases)
  - Comparative trends chart
  - Stationarity test summary visual
- ⬜ **Save analysis results to CSV/JSON**
  - Stationarity tests: `results/tables/problem-statement-006/stationarity_tests.csv`
  - APC calculations: `results/tables/problem-statement-006/annual_percent_change.csv`
  - ARIMA recommendations: `results/tables/problem-statement-006/arima_parameter_recommendations.json`
- ⬜ **Document findings in notebook** with markdown cells explaining insights

### Logger Setup
- ⬜ **Setup loguru logger** for analysis workflow
- ⬜ **Log analysis milestones** (stationarity tests completed, decomposition done, etc.)
- ⬜ **Log key findings** (e.g., "Cancer mortality is stationary after first-differencing")

## 6. Notes

**Critical Success Factors**:
- Stationarity clearly established (or differencing strategy determined)
- ARIMA parameter recommendations data-driven (from ACF/PACF patterns)
- Clear forecasting difficulty assessment per disease guides method selection

**Expected Findings** (based on epidemiological literature):
- Cancer mortality: Likely stationary or weakly trending down (improved treatments)
- Stroke mortality: Strong downward trend (better prevention and acute care)
- Heart disease: Downward trend (lifestyle interventions, medications)
- Moderate autocorrelation expected (disease burden changes gradually)

**Forecasting Method Implications**:
- If stationary + weak autocorrelation → Simple methods (SES, naïve) may suffice
- If strong trend + high autocorrelation → ARIMA(p,d,q) or Holt's method recommended
- If structural breaks detected → Prophet (changepoint detection) may be better than ARIMA

**Risk**:
- If all 3 diseases show very different temporal patterns → separate model strategies needed, complicates ensemble approach

**Next Steps**:
- Use ARIMA parameter recommendations for User Story 4: Baseline Model Development
- Stationarity findings guide preprocessing in User Story 3: Feature Engineering
- Trend insights inform scenario modeling in User Story 7

---

## Implementation Plan

### 1. Feature Overview

Conduct comprehensive exploratory time series analysis on mortality trends to understand temporal patterns, identify appropriate forecasting methods, and establish baseline expectations. Primary user: forecasting analyst exploring data characteristics before model development.

### 2. Component Analysis & Reuse Strategy

**Existing Components to Reuse**:
- Input data from User Story 01: `shared/data/3_interim/forecasting/mortality_time_series_clean.parquet`
- Utility functions: `shared/src/utils/data_profiling.py` (for summary stats)
- Configuration: `shared/config/forecasting.yml`

**Components to Create**:
- `shared/src/analysis/time_series_analysis.py` - NEW: Stationarity tests, ACF/PACF, decomposition
- `shared/src/analysis/trend_detection.py` - NEW: Trend fitting, inflection point detection
- `shared/src/visualization/ts_plots.py` - NEW: Time series visualization utilities
- `problem-statements/ps-006-forecasting/notebooks/02_exploratory_time_series_analysis.ipynb` - NEW: EDA notebook

**Justification**:
- Time series analysis functions are reusable across forecasting projects → `shared/src/analysis/`
- Visualization utilities can be used for other problem statements → `shared/src/visualization/`
- Problem-specific exploration and findings → notebook in `problem-statements/ps-006-forecasting/`

### 3. ML Model Evaluation & Selection

Not applicable - this user story focuses on exploratory analysis to inform model selection in User Story 4.

### 4. Affected Files

**[CREATE] `shared/src/analysis/time_series_analysis.py`**
- Function: `test_stationarity(series: pl.Series, method: str = 'adf') -> dict`
- Function: `decompose_time_series(df: pl.DataFrame, value_col: str, period: int) -> dict`
- Function: `compute_acf_pacf(series: pl.Series, lags: int = 10) -> tuple[np.ndarray, np.ndarray]`
- Function: `recommend_arima_order(series: pl.Series, max_p: int = 5, max_q: int = 5) -> tuple[int, int, int]`
- Dependencies: `polars`, `statsmodels`, `scipy`, `numpy`, `loguru`

**[CREATE] `shared/src/analysis/trend_detection.py`**
- Function: `calculate_annual_percent_change(df: pl.DataFrame, value_col: str) -> pl.DataFrame`
- Function: `fit_linear_trend(df: pl.DataFrame, x_col: str, y_col: str) -> dict`
- Function: `detect_structural_breaks(series: pl.Series, breakpoints: int = 2) -> list[int]`
- Function: `calculate_coefficient_of_variation(series: pl.Series) -> float`
- Dependencies: `polars`, `scipy`, `sklearn`, `numpy`

**[CREATE] `shared/src/visualization/ts_plots.py`**
- Function: `plot_time_series(df: pl.DataFrame, x_col: str, y_col: str, **kwargs) -> plt.Figure`
- Function: `plot_acf_pacf(series: pl.Series, lags: int, **kwargs) -> plt.Figure`
- Function: `plot_decomposition(decomposition: dict, **kwargs) -> plt.Figure`
- Function: `plot_trend_with_ci(df:pl.DataFrame, x_col: str, y_col: str, ci: float = 0.95) -> plt.Figure`
- Dependencies:`matplotlib`, `seaborn`, `polars`

**[CREATE] `problem-statements/ps-006-forecasting/notebooks/02_exploratory_time_series_analysis.ipynb`**
- Interactive notebook conducting full EDA workflow
- Visualization gallery creation
- ARIMA parameter recommendations documentation

**[CREATE] `shared/tests/unit/test_time_series_analysis.py`**
- Unit tests for stationarity tests, ACF/PACF, decomposition

**[CREATE] `shared/tests/unit/test_trend_detection.py`**
- Unit tests for APC, trend fitting, structural breaks

### 5. Data Pipeline

**Data Sources**:
- Input: `shared/data/3_interim/forecasting/mortality_time_series_clean.parquet` (from User Story 01)
- 90 rows (30 years × 3 diseases), 3 columns (year, asmr, disease)

**Transformation Steps**:
1. **Load Clean Data**: Read Parquet file using Polars
2. **Split by Disease**: Separate time series for cancer, stroke, heart_disease
3. **Stationarity Testing**: Run ADF and KPSS tests on each series
4. **Decomposition**: STL decomposition (trend, residual) - no seasonality for annual data  
5. **Trend Analysis**: Fit linear trends, calculate APC, detect inflection points
6. **Autocorrelation**: Compute ACF/PACF up to lag 10
7. **Pattern Characterization**: CV, volatility assessment, forecast difficulty rating

**Analysis Outputs**:
- Stationarity test results (p-values, recommendations)
- Trend coefficients and significance tests
- ACF/PACF values and plots
- ARIMA parameter recommendations (p, d, q) per disease
- Forecast difficulty ratings

**Target Consumption**:
- ARIMA parameters → User Story 04 (model training)
- Stationarity assessment → User Story 03 (differencing decisions)
- Reports → `results/tables/problem-statement-006/`
- Figures → `reports/figures/problem-statement-006/`

**Orchestration**:
- Single notebook execution (no pipeline dependencies after US-01)
- Error handling: Validate input data schema, handle edge cases in tests

### 6. Code Generation Specifications

#### 6.1 Function Signatures & Complete Implementations

```python
# shared/src/analysis/time_series_analysis.py

import polars as pl
import numpy as np
from statsmodels.tsa.stattools import adfuller, kpss, acf, pacf
from statsmodels.tsa.seasonal import STL
from loguru import logger
from typing import Literal, Dict, Tuple


def test_stationarity(
    series: pl.Series,
    method: Literal['adf', 'kpss'] = 'adf',
    significance_level: float = 0.05
) -> Dict:
    """Test time series for stationarity.
    
    Args:
        series: Time series data
        method: Test method ('adf' or 'kpss')
        significance_level: Significance level for test (default: 0.05)
        
    Returns:
        Dict with test results
    """
    logger.info(f"Testing stationarity using {method.upper()} test")
    
    # Convert to numpy for statsmodels
    data = series.to_numpy()
    
    if method == 'adf':
        # Augmented Dickey-Fuller test
        # H0: Series has a unit root (non-stationary)
        adf_result = adfuller(data, autolag='AIC')
        
        result = {
            'test': 'ADF',
            'test_statistic': float(adf_result[0]),
            'p_value': float(adf_result[1]),
            'lags_used': int(adf_result[2]),
            'n_obs': int(adf_result[3]),
            'critical_values': {k: float(v) for k, v in adf_result[4].items()},
            'is_stationary': adf_result[1] < significance_level
        }
        
        if result['is_stationary']:
            logger.info(f"✅ ADF test: Series is stationary (p={result['p_value']:.4f})")
        else:
            logger.warning(f"⚠️ ADF test: Series is non-stationary (p={result['p_value']:.4f})")
            
    elif method == 'kpss':
        # KPSS test
        # H0: Series is stationary
        kpss_result = kpss(data, regression='c', nlags='auto')
        
        result = {
            'test': 'KPSS',
            'test_statistic': float(kpss_result[0]),
            'p_value': float(kpss_result[1]),
            'lags_used': int(kpss_result[2]),
            'critical_values': {k: float(v) for k, v in kpss_result[3].items()},
            'is_stationary': kpss_result[1] > significance_level  # Note: opposite interpretation
        }
        
        if result['is_stationary']:
            logger.info(f"✅ KPSS test: Series is stationary (p={result['p_value']:.4f})")
        else:
            logger.warning(f"⚠️ KPSS test: Series is non-stationary (p={result['p_value']:.4f})")
    
    else:
        raise ValueError(f"Unknown method: {method}. Use 'adf' or 'kpss'")
    
    return result


def decompose_time_series(
    df: pl.DataFrame,
    time_col: str,
    value_col: str,
    period: int = 1,
    model: Literal['additive', 'multiplicative'] = 'additive'
) -> Dict:
    """Decompose time series into trend and residual components.
    
    Note: For annual data, period=1 (no seasonality).
    
    Args:
        df: DataFrame with time series
        time_col: Time column name
        value_col: Value column name
        period: Seasonal period (1 for annual, no seasonality)
        model: Decomposition model
        
    Returns:
        Dict with trend, seasonal, residual arrays
    """
    logger.info(f"Decomposing time series: {value_col}")
    
    # Sort by time
    df_sorted = df.sort(time_col)
    
    # Convert to numpy
    values = df_sorted[value_col].to_numpy()
    
    if period == 1:
        # No seasonality - simple trend extraction using moving average
        from scipy.ndimage import uniform_filter1d
        
        window = min(7, len(values) // 3)  # Use 1/3 of data or 7, whichever smaller
        trend = uniform_filter1d(values, size=window, mode='nearest')
        residual = values - trend
        seasonal = np.zeros_like(values)
        
        logger.info("Decomposition complete (no seasonality, period=1)")
        
    else:
        # Use STL for seasonal data
        stl = STL(values, seasonal=period, trend=period*2+1)
        result = stl.fit()
        
        trend = result.trend
        seasonal = result.seasonal
        residual = result.resid
        
        logger.info(f"STL decomposition complete (period={period})")
    
    return {
        'trend': trend,
        'seasonal': seasonal,
        'residual': residual,
        'original': values
    }


def compute_acf_pacf(
    series: pl.Series,
    lags: int = 10,
    alpha: float = 0.05
) -> Tuple[np.ndarray, np.ndarray, Tuple[float, float]]:
    """Compute ACF and PACF for time series.
    
    Args:
        series: Time series data
        lags: Number of lags
        alpha: Significance level for confidence intervals
        
    Returns:
        Tuple of (acf_values, pacf_values, confidence_interval)
    """
    logger.info(f"Computing ACF/PACF for {lags} lags")
    
    data = series.to_numpy()
    
    # Compute ACF
    acf_values = acf(data, nlags=lags, alpha=alpha, fft=False)
    
    # Compute PACF
    pacf_values = pacf(data, nlags=lags, alpha=alpha, method='ywm')
    
    # Confidence interval (approximate 95% CI for white noise)
    n = len(data)
    ci = 1.96 / np.sqrt(n)
    
    logger.info(f"ACF/PACF computed: max ACF={np.max(np.abs(acf_values[0][1:])):.3f}")
    
    return acf_values[0], pacf_values[0], (-ci, ci)


def recommend_arima_order(
    series: pl.Series,
    max_p: int = 5,
    max_q: int = 5
) -> Tuple[int, int, int]:
    """Recommend ARIMA(p,d,q) order based on ACF/PACF patterns.
    
    Args:
        series: Time series data
        max_p: Maximum AR order to consider
        max_q: Maximum MA order to consider
        
    Returns:
        Recommended (p, d, q) order
    """
    logger.info("Recommending ARIMA order")
    
    # Test stationarity
    adf_result = test_stationarity(series, method='adf')
    d = 0 if adf_result['is_stationary'] else 1
    
    # If non-stationary, difference once and retest
    if d == 1:
        data = series.to_numpy()
        data_diff = np.diff(data)
        series_diff = pl.Series(data_diff)
        
        # Verify differencing made it stationary
        adf_diff = test_stationarity(series_diff, method='adf')
        if not adf_diff['is_stationary']:
            logger.warning("First differencing did not achieve stationarity, trying d=2")
            d = 2
            data_diff = np.diff(data_diff)
            series_diff = pl.Series(data_diff)
    else:
        series_diff = series
    
    # Compute ACF and PACF
    acf_vals, pacf_vals, ci = compute_acf_pacf(series_diff, lags=max(max_p, max_q))
    
    # Determine p from PACF cutoff
    p = 0
    for lag in range(1, min(len(pacf_vals), max_p + 1)):
        if abs(pacf_vals[lag]) > abs(ci[1]):
            p = lag
        else:
            break
    
    # Determine q from ACF cutoff
    q = 0
    for lag in range(1, min(len(acf_vals), max_q + 1)):
        if abs(acf_vals[lag]) > abs(ci[1]):
            q = lag
        else:
            break
    
    # Default to (1,d,1) if no clear pattern
    if p == 0 and q == 0:
        logger.warning("No clear ACF/PACF pattern, defaulting to (1,d,1)")
        p, q = 1, 1
    
    logger.info(f"Recommended ARIMA order: ({p},{d},{q})")
    
    return (p, d, q)
```

```python
# shared/src/analysis/trend_detection.py

import polars as pl
import numpy as np
from scipy import stats
from sklearn.linear_model import LinearRegression
from loguru import logger
from typing import Dict, List


def calculate_annual_percent_change(
    df: pl.DataFrame,
    time_col: str = 'year',
    value_col: str = 'asmr',
    group_col: str = None
) -> pl.DataFrame:
    """Calculate year-over-year percent change.
    
    Formula: APC = [(Value_t / Value_t-1) - 1] × 100
    
    Args:
        df: DataFrame with time series
        time_col: Time column
        value_col: Value column
        group_col: Optional grouping column (e.g., 'disease')
        
    Returns:
        DataFrame with APC column added
    """
    logger.info(f"Calculating Annual Percent Change for {value_col}")
    
    df_sorted = df.sort(time_col if not group_col else [group_col, time_col])
    
    if group_col:
        # Calculate APC within each group
        df_with_apc = df_sorted.with_columns([
            ((pl.col(value_col) / pl.col(value_col).shift(1) - 1) * 100)
            .over(group_col)
            .alias('apc')
        ])
    else:
        # Calculate APC for entire series
        df_with_apc = df_sorted.with_columns([
            ((pl.col(value_col) / pl.col(value_col).shift(1) - 1) * 100)
            .alias('apc')
        ])
    
    logger.info("APC calculation complete")
    return df_with_apc


def fit_linear_trend(
    df: pl.DataFrame,
    x_col: str,
    y_col: str
) -> Dict:
    """Fit linear trend model and test significance.
    
    Model: y = β0 + β1*x + ε
    
    Args:
        df: DataFrame
        x_col: Independent variable (e.g., 'year')
        y_col: Dependent variable (e.g., 'asmr')
        
    Returns:
        Dict with trend statistics
    """
    logger.info(f"Fitting linear trend: {y_col} ~ {x_col}")
    
    X = df[x_col].to_numpy().reshape(-1, 1)
    y = df[y_col].to_numpy()
    
    # Fit linear regression
    model = LinearRegression()
    model.fit(X, y)
    
    # Predict and calculate residuals
    y_pred = model.predict(X)
    residuals = y - y_pred
    
    # Calculate R-squared
    ss_res = np.sum(residuals ** 2)
    ss_tot = np.sum((y - np.mean(y)) ** 2)
    r_squared = 1 - (ss_res / ss_tot)
    
    # T-test for slope significance
    n = len(y)
    se = np.sqrt(ss_res / (n - 2)) / np.sqrt(np.sum((X.flatten() - np.mean(X)) ** 2))
    t_stat = model.coef_[0] / se
    p_value = 2 * (1 - stats.t.cdf(abs(t_stat), df=n-2))
    
    result = {
        'intercept': float(model.intercept_),
        'slope': float(model.coef_[0]),
        'r_squared': float(r_squared),
        't_statistic': float(t_stat),
        'p_value': float(p_value),
        'is_significant': p_value < 0.05,
        'trend_direction': 'increasing' if model.coef_[0] > 0 else 'decreasing'
    }
    
    logger.info(f"Trend: slope={result['slope']:.3f}, R²={result['r_squared']:.3f}, p={result['p_value']:.4f}")
    
    return result


def detect_structural_breaks(
    series: pl.Series,
    min_segment_size: int = 10
) -> List[int]:
    """Detect structural breaks in time series using simple change point detection.
    
    Args:
        series: Time series data
        min_segment_size: Minimum observations per segment
        
    Returns:
        List of breakpoint indices
    """
    logger.info("Detecting structural breaks")
    
    data = series.to_numpy()
    n = len(data)
    
    if n < min_segment_size * 2:
        logger.warning("Series too short for break detection")
        return []
    
    # Simple cumulative sum method (CUSUM)
    mean = np.mean(data)
    cusum = np.cumsum(data - mean)
    
    # Find points with maximum deviation
    # This is a simplified approach - production code would use more sophisticated methods
    abs_cusum = np.abs(cusum)
    
    # Find local maxima as candidate breakpoints
    candidates = []
    for i in range(min_segment_size, n - min_segment_size):
        if abs_cusum[i] > abs_cusum[i-1] and abs_cusum[i] > abs_cusum[i+1]:
            if abs_cusum[i] > np.std(cusum):  # Threshold: 1 std
                candidates.append(i)
    
    logger.info(f"Detected {len(candidates)} potential structural breaks")
    
    return candidates


def calculate_coefficient_of_variation(series: pl.Series) -> float:
    """Calculate coefficient of variation (CV).
    
    CV = (std / mean) × 100
    
    Args:
        series: Data series
        
    Returns:
        CV percentage
    """
    mean = series.mean()
    std = series.std()
    
    if mean == 0:
        logger.warning("Mean is zero, CV undefined")
        return np.inf
    
    cv = (std / mean) * 100
    
    logger.info(f"Coefficient of Variation: {cv:.2f}%")
    
    return float(cv)
```

```python
# shared/src/visualization/ts_plots.py

import matplotlib.pyplot as plt
import seaborn as sns
import polars as pl
import numpy as np
from typing import Optional
from loguru import logger

sns.set_style('whitegrid')


def plot_time_series(
    df: pl.DataFrame,
    x_col: str,
    y_col: str,
    group_col: Optional[str] = None,
    title: str = "Time Series",
    ylabel: str = "Value",
    figsize: tuple = (12, 6)
) -> plt.Figure:
    """Plot time series line chart.
    
    Args:
        df: DataFrame
        x_col: X-axis column (time)
        y_col: Y-axis column (value)
        group_col: Optional grouping column for multiple lines
        title: Plot title
        ylabel: Y-axis label
        figsize: Figure size
        
    Returns:
        Matplotlib figure
    """
    logger.info(f"Plotting time series: {y_col} vs {x_col}")
    
    fig, ax = plt.subplots(figsize=figsize)
    
    if group_col:
        groups = df[group_col].unique().to_list()
        for group in groups:
            df_group = df.filter(pl.col(group_col) == group).sort(x_col)
            ax.plot(df_group[x_col], df_group[y_col], marker='o', label=group, linewidth=2)
        ax.legend()
    else:
        df_sorted = df.sort(x_col)
        ax.plot(df_sorted[x_col], df_sorted[y_col], marker='o', linewidth=2, color='steelblue')
    
    ax.set_xlabel(x_col.capitalize(), fontsize=12)
    ax.set_ylabel(ylabel, fontsize=12)
    ax.set_title(title, fontsize=14, fontweight='bold')
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    
    return fig


def plot_acf_pacf(
    acf_values: np.ndarray,
    pacf_values: np.ndarray,
    lags: int,
    ci: tuple,
    title_prefix: str = "",
    figsize: tuple = (14, 5)
) -> plt.Figure:
    """Plot ACF and PACF with confidence intervals.
    
    Args:
        acf_values: ACF values
        pacf_values: PACF values
        lags: Number of lags
        ci: Confidence interval tuple (lower, upper)
        title_prefix: Prefix for plot titles
        figsize: Figure size
        
    Returns:
        Matplotlib figure
    """
    logger.info("Plotting ACF and PACF")
    
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=figsize)
    
    # ACF plot
    lags_array = np.arange(len(acf_values))
    ax1.stem(lags_array, acf_values, basefmt=' ')
    ax1.axhline(y=0, linestyle='--', color='black', linewidth=0.8)
    ax1.axhline(y=ci[1], linestyle='--', color='red', linewidth=0.8)
    ax1.axhline(y=ci[0], linestyle='--', color='red', linewidth=0.8)
    ax1.set_xlabel('Lag', fontsize=11)
    ax1.set_ylabel('ACF', fontsize=11)
    ax1.set_title(f'{title_prefix} Autocorrelation Function', fontsize=12, fontweight='bold')
    ax1.grid(True, alpha=0.3)
    
    # PACF plot
    lags_array_pacf = np.arange(len(pacf_values))
    ax2.stem(lags_array_pacf, pacf_values, basefmt=' ')
    ax2.axhline(y=0, linestyle='--', color='black', linewidth=0.8)
    ax2.axhline(y=ci[1], linestyle='--', color='red', linewidth=0.8)
    ax2.axhline(y=ci[0], linestyle='--', color='red', linewidth=0.8)
    ax2.set_xlabel('Lag', fontsize=11)
    ax2.set_ylabel('PACF', fontsize=11)
    ax2.set_title(f'{title_prefix} Partial Autocorrelation Function', fontsize=12, fontweight='bold')
    ax2.grid(True, alpha=0.3)
    
    plt.tight_layout()
    
    return fig


def plot_decomposition(
    decomposition: dict,
    title_prefix: str = "",
    figsize: tuple = (14, 8)
) -> plt.Figure:
    """Plot time series decomposition.
    
    Args:
        decomposition: Dict with 'original', 'trend', 'seasonal', 'residual'
        title_prefix: Prefix for plot title
        figsize: Figure size
        
    Returns:
        Matplotlib figure
    """
    logger.info("Plotting time series decomposition")
    
    fig, axes = plt.subplots(4, 1, figsize=figsize, sharex=True)
    
    time_index = np.arange(len(decomposition['original']))
    
    # Original
    axes[0].plot(time_index, decomposition['original'], color='steelblue', linewidth=2)
    axes[0].set_ylabel('Original', fontsize=11)
    axes[0].set_title(f'{title_prefix} Time Series Decomposition', fontsize=13, fontweight='bold')
    axes[0].grid(True, alpha=0.3)
    
    # Trend
    axes[1].plot(time_index, decomposition['trend'], color='orange', linewidth=2)
    axes[1].set_ylabel('Trend', fontsize=11)
    axes[1].grid(True, alpha=0.3)
    
    # Seasonal
    if np.any(decomposition['seasonal'] != 0):
        axes[2].plot(time_index, decomposition['seasonal'], color='green', linewidth=2)
    else:
        axes[2].axhline(y=0, color='gray', linestyle='--')
        axes[2].text(len(time_index)/2, 0, 'No Seasonality (Annual Data)', ha='center', va='center')
    axes[2].set_ylabel('Seasonal', fontsize=11)
    axes[2].grid(True, alpha=0.3)
    
    # Residual
    axes[3].plot(time_index, decomposition['residual'], color='red', linewidth=1, alpha=0.7)
    axes[3].axhline(y=0, linestyle='--', color='black', linewidth=0.8)
    axes[3].set_ylabel('Residual', fontsize=11)
    axes[3].set_xlabel('Time Index', fontsize=11)
    axes[3].grid(True, alpha=0.3)
    
    plt.tight_layout()
    
    return fig


def plot_trend_with_ci(
    df: pl.DataFrame,
    x_col: str,
    y_col: str,
    trend_result: dict,
    ci: float = 0.95,
    title: str = "Trend Analysis",
    figsize: tuple = (12, 6)
) -> plt.Figure:
    """Plot time series with fitted trend line and confidence interval.
    
    Args:
        df: DataFrame
        x_col: X column
        y_col: Y column
        trend_result: Dict from fit_linear_trend()
        ci: Confidence level
        title: Plot title
        figsize: Figure size
        
    Returns:
        Matplotlib figure
    """
    logger.info("Plotting trend with confidence interval")
    
    fig, ax = plt.subplots(figsize=figsize)
    
    df_sorted = df.sort(x_col)
    x = df_sorted[x_col].to_numpy()
    y = df_sorted[y_col].to_numpy()
    
    # Fitted values
    y_fit = trend_result['intercept'] + trend_result['slope'] * x
    
    # Plot actual data
    ax.scatter(x, y, color='steelblue', s=50, alpha=0.6, label='Actual')
    
    # Plot trend line
    ax.plot(x, y_fit, color='red', linewidth=2, label=f"Trend (slope={trend_result['slope']:.2f})")
    
    # Calculate CI (simplified)
    from scipy import stats
    n = len(y)
    residuals = y - y_fit
    se = np.sqrt(np.sum(residuals**2) / (n-2))
    t_val = stats.t.ppf((1 + ci) / 2, n - 2)
    ci_band = t_val * se
    
    ax.fill_between(x, y_fit - ci_band, y_fit + ci_band, color='red', alpha=0.2, label=f'{int(ci*100)}% CI')
    
    ax.set_xlabel(x_col.capitalize(), fontsize=12)
    ax.set_ylabel(y_col.upper(), fontsize=12)
    ax.set_title(title, fontsize=14, fontweight='bold')
    ax.legend()
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    
    return fig
```

#### 6.2 Data Schemas

Not needed for EDA - analysis outputs are saved as CSV/JSON with documented structure.

#### 6.3 Data Validation Rules

```python
# Input validation
REQUIRED_COLUMNS = ['year', 'asmr', 'disease']
MIN_OBSERVATIONS = 20  # Need at least 20 years for robust analysis
DISEASES_EXPECTED = ['cancer', 'stroke', 'heart_disease']

# Analysis parameters
MAX_LAGS = 10  # For ACF/PACF
MAX_AR_ORDER = 5
MAX_MA_ORDER = 5
SIGNIFICANCE_LEVEL = 0.05
```

#### 6.4 Library-Specific Patterns

```python
# Statsmodels pattern
from statsmodels.tsa.stattools import adfuller

result = adfuller(data, autolag='AIC')
is_stationary = result[1] < 0.05

# Matplotlib saving pattern
import matplotlib.pyplot as plt
from datetime import datetime
from pathlib import Path

timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
fig_dir = Path('reports/figures/problem-statement-006/trends')
fig_dir.mkdir(parents=True, exist_ok=True)

fig.savefig(fig_dir / f'trend_analysis_{timestamp}.png', dpi=300, bbox_inches='tight')
logger.info(f"✅ Saved: {fig_dir / f'trend_analysis_{timestamp}.png'}")
plt.close(fig)
```

#### 6.5 Test Specifications (See Section 10)

#### 6.6 Package Management

```bash
uv pip install statsmodels>=0.14.0
uv pip install scipy>=1.10.0
uv pip install matplotlib>=3.7.0
uv pip install seaborn>=0.12.0
uv pip install scikit-learn>=1.3.0
uv pip freeze > requirements.txt
```

### 7. Domain-Driven Feature Engineering

Not applicable - this user story focuses on analysis, not feature creation. Feature engineering covered in User Story 3.

**Domain Knowledge Used**:
- **Stationarity**: Required assumption for ARIMA models - test and document
- **Autocorrelation**: Indicates temporal dependence - drives AR/MA order selection
- **Trend**: Long-term direction - must be modeled or removed
- **Coefficient of Variation**: Measures forecast difficulty - higher CV = harder to predict

### 8. API Endpoints & Data Contracts

Not applicable - exploratory analysis only.

### 9. Styling & Visualization

**Visualization Requirements**:
1. **Time Series Plots**: Line charts with markers, grid, clear labels
2. **ACF/PACF**: Stem plots with confidence bands (red dashed lines)
3. **Decomposition**: 4-panel stacked charts (original, trend, seasonal, residual)
4. **Trend with CI**: Scatter + fitted line + shaded confidence band

**Style Standards**:
- Color palette: seaborn 'whitegrid' style
- Font sizes: Title=14pt bold, axis labels=12pt, ticks=10pt
- DPI: 300 for publication quality
- Format: PNG for reports, SVG for presentations

### 10. Testing Strategy

**Unit Tests** (`shared/tests/unit/test_time_series_analysis.py`):

```python
import pytest
import polars as pl
import numpy as np
from shared.src.analysis.time_series_analysis import (
    test_stationarity,
    decompose_time_series,
    compute_acf_pacf,
    recommend_arima_order
)


@pytest.fixture
def stationary_series():
    """Stationary white noise series."""
    np.random.seed(42)
    return pl.Series(np.random.randn(100))


@pytest.fixture
def non_stationary_series():
    """Non-stationary random walk."""
    np.random.seed(42)
    return pl.Series(np.cumsum(np.random.randn(100)))


@pytest.fixture
def mortality_data():
    """Realistic mortality time series."""
    years = list(range(1990, 2020))
    # Declining trend with noise
    asmr = [150 - i*2 + np.random.randn()*5 for i in range(30)]
    return pl.DataFrame({'year': years, 'asmr': asmr, 'disease': ['cancer']*30})


def test_adf_stationary(stationary_series):
    """Test ADF identifies stationary series."""
    result = test_stationarity(stationary_series, method='adf')
    
    assert result['test'] == 'ADF'
    assert result['is_stationary'] == True
    assert result['p_value'] < 0.05


def test_adf_non_stationary(non_stationary_series):
    """Test ADF identifies non-stationary series."""
    result = test_stationarity(non_stationary_series, method='adf')
    
    assert result['is_stationary'] == False
    assert result['p_value'] > 0.05


def test_kpss_stationary(stationary_series):
    """Test KPSS identifies stationary series."""
    result = test_stationarity(stationary_series, method='kpss')
    
    assert result['test'] == 'KPSS'
    assert result['is_stationary'] == True


def test_decompose_returns_components(mortality_data):
    """Test decomposition returns all components."""
    result = decompose_time_series(
        mortality_data,
        time_col='year',
        value_col='asmr',
        period=1
    )
    
    assert 'trend' in result
    assert 'seasonal' in result
    assert 'residual' in result
    assert 'original' in result
    assert len(result['trend']) == 30


def test_compute_acf_pacf_shape(stationary_series):
    """Test ACF/PACF computation returns correct shape."""
    acf_vals, pacf_vals, ci = compute_acf_pacf(stationary_series, lags=10)
    
    assert len(acf_vals) == 11  # lags 0 to 10
    assert len(pacf_vals) == 11
    assert acf_vals[0] == pytest.approx(1.0)  # Lag 0 always 1.0
    assert isinstance(ci, tuple)
    assert len(ci) == 2


def test_recommend_arima_non_stationary(non_stationary_series):
    """Test ARIMA recommendation includes differencing for non-stationary data."""
    p, d, q = recommend_arima_order(non_stationary_series)
    
    assert d >= 1  # Should recommend differencing
    assert isinstance(p, int) and p >= 0
    assert isinstance(q, int) and q >= 0


def test_recommend_arima_stationary(stationary_series):
    """Test ARIMA recommendation for stationary data."""
    p, d, q = recommend_arima_order(stationary_series)
    
    assert d == 0  # Stationary, no differencing needed
    assert p + q > 0  # Should have some AR or MA component
```

**Unit Tests** (`shared/tests/unit/test_trend_detection.py`):

```python
import pytest
import polars as pl
import numpy as np
from shared.src.analysis.trend_detection import (
    calculate_annual_percent_change,
    fit_linear_trend,
    calculate_coefficient_of_variation
)


@pytest.fixture
def trending_data():
    """Data with clear linear trend."""
    years = list(range(1990, 2020))
    values = [100 + 2*i for i in range(30)]  # Upward trend
    return pl.DataFrame({'year': years, 'value': values})


def test_calculate_apc():
    """Test APC calculation."""
    df = pl.DataFrame({
        'year': [1990, 1991, 1992],
        'asmr': [100.0, 105.0, 110.25],
        'disease': ['cancer', 'cancer', 'cancer']
    })
    
    result = calculate_annual_percent_change(df, 'year', 'asmr')
    
    assert 'apc' in result.columns
    assert result.filter(pl.col('year') == 1991)['apc'][0] == pytest.approx(5.0, rel=0.01)
    assert result.filter(pl.col('year') == 1992)['apc'][0] == pytest.approx(5.0, rel=0.01)


def test_fit_linear_trend_upward(trending_data):
    """Test trend fitting with upward trend."""
    result = fit_linear_trend(trending_data, 'year', 'value')
    
    assert result['slope'] > 0  # Upward trend
    assert result['slope'] == pytest.approx(2.0, rel=0.01)
    assert result['r_squared'] > 0.99  # Perfect linear fit
    assert result['is_significant'] == True
    assert result['trend_direction'] == 'increasing'


def test_fit_linear_trend_flat():
    """Test trend fitting with flat data."""
    df = pl.DataFrame({
        'year': list(range(1990, 2020)),
        'value': [100.0] * 30
    })
    
    result = fit_linear_trend(df, 'year', 'value')
    
    assert result['slope'] == pytest.approx(0.0, abs=0.01)
    assert result['is_significant'] == False


def test_coefficient_of_variation():
    """Test CV calculation."""
    series = pl.Series([10, 12, 8, 9, 11])  # Mean=10, Std≈1.58
    
    cv = calculate_coefficient_of_variation(series)
    
    assert cv == pytest.approx(15.8, rel=0.1)  # CV ≈ 15.8%
```

### 11. Implementation Steps

**Phase 1: Setup**
- [ ] Create output directories: `reports/figures/problem-statement-006/{trends, acf_pacf, decomposition}`
- [ ] Create `problem-statements/ps-006-forecasting/notebooks/02_exploratory_time_series_analysis.ipynb`
- [ ] Install required packages: statsmodels, scipy, matplotlib, seaborn

**Phase 2: Analysis Functions**
- [ ] Create `shared/src/analysis/time_series_analysis.py`
- [ ] Implement `test_stationarity()` (ADF and KPSS tests)
- [ ] Implement `decompose_time_series()` (STL decomposition)
- [ ] Implement `compute_acf_pacf()`
- [ ] Implement `recommend_arima_order()`

**Phase 3: Trend Detection**
- [ ] Create `shared/src/analysis/trend_detection.py`
- [ ] Implement `calculate_annual_percent_change()`
- [ ] Implement `fit_linear_trend()`
- [ ] Implement `detect_structural_breaks()`
- [ ] Implement `calculate_coefficient_of_variation()`

**Phase 4: Visualization Functions**
- [ ] Create `shared/src/visualization/ts_plots.py`
- [ ] Implement `plot_time_series()`
- [ ] Implement `plot_acf_pacf()`
- [ ] Implement `plot_decomposition()`
- [ ] Implement `plot_trend_with_ci()`

**Phase 5: Exploratory Analysis Execution**
- [ ] Load cleaned data from User Story 01
- [ ] Run stationarity tests (ADF, KPSS) for each disease
- [ ] Perform time series decomposition for each disease
- [ ] Calculate APC and fit linear trends
- [ ] Generate ACF/PACF plots and recommend ARIMA orders
- [ ] Calculate CV and assess forecast difficulty

**Phase 6: Cross-Disease Comparison**
- [ ] Create comparative summary table (all diseases)
- [ ] Visualize normalized trends (index 1990=100)
- [ ] Analyze correlations between diseases

**Phase 7: Results Documentation**
- [ ] Save stationarity results: `results/tables/problem-statement-006/stationarity_tests_{timestamp}.csv`
- [ ] Save APC: `results/tables/problem-statement-006/annual_percent_change_{timestamp}.csv`
- [ ] Save ARIMA recommendations: `results/tables/problem-statement-006/arima_recommendations_{timestamp}.json`
- [ ] Generate visualization gallery (all plots)
- [ ] Create EDA report markdown with key findings

**Phase 8: Testing**
- [ ] Create `shared/tests/unit/test_time_series_analysis.py`
- [ ] Create `shared/tests/unit/test_trend_detection.py`
- [ ] Run all tests with `pytest`
- [ ] Verify >80% coverage

### 12. Adaptive Implementation Strategy

**Mandatory Output Reviews**:
- After stationarity tests: If all series non-stationary → Plan differencing in US-03
- After ACF/PACF: If no clear patterns → Consider alternative models (Prophet) in US-04
- After trend detection: If structural breaks detected → Document for scenario modeling in US-07

**Automatic Plan Updates Required When**:
- Unexpected stationarity results → Investigate data quality, consider transformations
- Very high CV (>30%) → Flag as "difficult to forecast", lower accuracy expectations
- Strong correlations between diseases → Consider multivariate models (VAR) in US-04

### 13. Code Generation Order

1. Config: `shared/config/forecasting.yml` (add analysis parameters)
2. Analysis: `shared/src/analysis/time_series_analysis.py`
3. Trends: `shared/src/analysis/trend_detection.py`
4. Visualization: `shared/src/visualization/ts_plots.py`
5. Tests: Unit tests for analysis modules
6. Notebook: `02_exploratory_time_series_analysis.ipynb`
7. Documentation: EDA report, methodology notes

### 14. Data Quality & Validation

**Pre-Analysis Validation**:
- Verify input data from US-01 is complete and clean
- Check for minimum 20 observations per disease (30 expected)
- Validate no nulls, correct dtypes

**Analysis Validation**:
- Stationarity: Cross-validate ADF and KPSS (should agree)
- ACF/PACF: Verify lag 0 = 1.0 (autocorrelation with self)
- Decomposition: Verify original = trend + seasonal + residual (approximately)

### 15-24. (Other Sections)

See User Story 01 implementation plan for reference - many sections (Security, Version Control, etc.) are similar.

**Success Metrics**:
- Stationarity determined for all 3 diseases
- ARIMA parameters recommended with statistical justification
- Forecast difficulty assessed (all diseases classified as easy/medium/hard)
- Comprehensive visualization gallery generated
- EDA report completed with actionable findings

✅ **PLAN READY FOR CODE GENERATION**
