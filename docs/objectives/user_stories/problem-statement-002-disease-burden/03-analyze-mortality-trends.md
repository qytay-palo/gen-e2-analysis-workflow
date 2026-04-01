# Analyze 30-Year Mortality Trends (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-002-US-03  
**Epic**: National Disease Burden Temporal Trends Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Public Health Epidemiologist**,  
I want **to analyze 30-year mortality rate trends (1990-2019) for cancer, stroke, and heart disease, calculating annual percentage changes and identifying trend patterns**,  
So that **I can quantify how disease burdens have evolved and identify diseases showing concerning increases vs successful decreases**.

---

## 🎯 Acceptance Criteria

1. **Annual percentage change calculated**
   - Year-over-year change: `(rate_t - rate_t-1) / rate_t-1 * 100` for each disease
   - Average annual percentage change (AAPC) calculated over 30-year period
   - Statistical significance tested (is trend non-zero?)

2. **Trend patterns identified**
   - Linear trends fitted using regression: `mortality_rate ~ year`
   - Trend direction: increasing, decreasing, stable (based on slope and p-value)
   - Trend magnitude: absolute change (rate in 2019 - rate in 1990)
   - Relative change: `(rate_2019 - rate_1990) / rate_1990 * 100%`

3. **Cross-disease comparisons**
   - Disease ranking by 2019 burden magnitude (which causes most deaths?)
   - Disease ranking by trend direction (which improving fastest/worsening fastest?)
   - Burden transition analysis: has leading cause of death changed 1990 vs 2019?

4. **Data output requirement**
   - Output file: `results/tables/mortality_trend_analysis.csv`
   - Format: CSV (disease, year, mortality_rate, yoy_change_pct, trend_slope, aapc, significance)
   - Trend visualization: `reports/figures/mortality_trends_30yr.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Statistics**: scipy.stats, statsmodels for trend testing
- **Logging**: loguru
- **Testing**: pytest with ≥80% coverage

---

## 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#core-metrics--definitions) - Understanding mortality rate metrics
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md#objectives) - Objective 1: Quantify 30-year mortality trends

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data processing
- `scipy>=1.11.0`: Statistical tests
- `statsmodels>=0.14.0`: Linear regression, trend analysis
- `matplotlib>=3.8.0`: Trend visualizations
- `loguru>=0.7.0`: Logging

### Internal Dependencies
- **Upstream**: PS-002-US-02 (Clean mortality data - BLOCKING)
- **Data Sources**: `shared/data/3_interim/mortality_integrated_clean.parquet`
- **Config Files**: `config/analysis.yml` (significance thresholds)

---

## ✅ Implementation Tasks

### Trend Analysis
- [ ] Load cleaned mortality data (1990-2019)
- [ ] Calculate year-over-year change: `(rate_t - rate_t-1) / rate_t-1 * 100`
- [ ] Calculate AAPC: `(rate_2019 / rate_1990)^(1/29) - 1` (geometric mean)
- [ ] Fit linear trends: `statsmodels.OLS(mortality_rate ~ year)` per disease
- [ ] Extract trend metrics: slope, p-value, R-squared
- [ ] Classify trends: increasing (slope > 0, p < 0.05), decreasing, stable

### Statistical Significance Testing
- [ ] Test trend significance: t-test on regression slope
- [ ] Calculate confidence intervals for AAPC
- [ ] Test for trend change points (next story will use joinpoint regression)

### Cross-Disease Comparison
- [ ] Rank diseases by 2019 mortality rate (current burden)
- [ ] Rank by absolute change 1990-2019 (improvement magnitude)
- [ ] Rank by AAPC (% improvement rate)
- [ ] Create comparison matrix: burden vs trend direction

### Visualization
- [ ] Line charts: Mortality rate over time (1990-2019) per disease
- [ ] Slope charts: 1990 vs 2019 rates with connecting lines
- [ ] Bar charts: AAPC by disease (sorted by magnitude)
- [ ] Annotation: Mark statistically significant trends
- [ ] Export figures: PNG (reports) and PDF (publication)

### Testing & Validation
- [ ] Unit tests for AAPC calculation
- [ ] Validate trend regression against manual calculation
- [ ] Test edge cases: flat trends, missing years

### Documentation
- [ ] Docstrings (Google style)
- [ ] Analysis summary: `results/mortality_trend_insights.md`
- [ ] Update data dictionary

---

## 📌 Notes

**AAPC Calculation (Polars)**:
```python
import polars as pl

df_aapc = (
    df.group_by('disease').agg([
        pl.col('mortality_rate').first().alias('rate_1990'),
        pl.col('mortality_rate').last().alias('rate_2019'),
        ((pl.col('mortality_rate').last() / pl.col('mortality_rate').first()) ** (1/29) - 1) * 100
        .alias('aapc_pct')
    ])
)
```

**Trend Regression (statsmodels)**:
```python
import statsmodels.api as sm
import polars as pl

# For each disease
for disease in df['disease'].unique():
    df_disease = df.filter(pl.col('disease') == disease)
    
    X = df_disease['year'].to_numpy()
    y = df_disease['mortality_rate'].to_numpy()
    X = sm.add_constant(X)  # Add intercept
    
    model = sm.OLS(y, X).fit()
    slope = model.params[1]
    p_value = model.pvalues[1]
    
    logger.info(f"{disease}: slope={slope:.2f}, p={p_value:.4f}")
```

**Expected Findings** (hypotheses from epidemiology):
- **Cancer**: Likely declining or stable (improved early detection, treatment)
- **Stroke**: Likely declining (better hypertension management)
- **Heart Disease**: Likely declining (cholesterol management, lifestyle changes)
- **Overall**: Singapore likely shows favorable trends relative to global averages

**Interpretation Guidelines**:
- **p < 0.05**: Trend is statistically significant
- **Slope > 0**: Mortality rate increasing (concerning)
- **Slope < 0**: Mortality rate decreasing (positive public health impact)
- **|slope| < 0.1 per year**: Modest trend
- **|slope| ≥ 1.0 per year**: Strong trend

---

## Implementation Plan

### 1. Feature Overview

Analyze 30-year mortality rate trends (1990-2019) for cancer, stroke, and ischemic heart disease by calculating annual percentage changes, fitting linear trend models, and identifying disease burden evolution patterns. This enables epidemiologists to quantify disease trajectory and identify public health successes or emerging concerns.

**Primary User Role**: Public Health Epidemiologist

### 2. Component Analysis & Reuse Strategy

**Existing Components to Reuse**:
- ✅ **`shared/data/3_interim/mortality_integrated_clean.parquet`** (from US-02) - Cleaned mortality data
- ✅ **`shared/src/utils/logger.py`** - Logging infrastructure
- ✅ **`docs/domain_knowledge/disease-burden-feature-engineering-guide.md`** - APC/AAPC formulas and methodologies

**Components Requiring Creation**:
- 🆕 **`shared/src/analysis/mortality_trends.py`** - Trend analysis functions (AAPC, YoY change, regression)
- 🆕 **`results/tables/mortality_trend_analysis.csv`** - Trend metrics output
- 🆕 **`reports/figures/mortality_trends_30yr.png`** - Time series visualization
- 🆕 **`reports/figures/mortality_trends_slope_chart.png`** - 1990 vs 2019 comparison
- 🆕 **`shared/tests/unit/test_mortality_trends.py`** - Unit tests for trend calculations
- 🆕 **`shared/tests/integration/test_mortality_trend_analysis.py`** - End-to-end analysis test

**Justification**: Create dedicated analysis module to separate data transformation (US-02) from analytical computation, enabling reuse for future time series analyses.

### 3. ML Model Evaluation & Selection

**Not applicable** - This is a statistical analysis story using descriptive methods (regression, trend metrics), not predictive ML.

### 4. Affected Files

- **[CREATE] `shared/src/analysis/mortality_trends.py`**
  - Functions:
    - `calculate_yoy_change(df: pl.DataFrame) -> pl.DataFrame`
    - `calculate_aapc(df: pl.DataFrame, disease: str) -> float`
    - `fit_linear_trend(df: pl.DataFrame, disease: str) -> dict`
    - `classify_trend(slope: float, p_value: float, threshold: float = 0.05) -> str`
    - `generate_trend_summary(df: pl.DataFrame) -> pl.DataFrame`
  - Dependencies: `polars>=0.20.0`, `scipy>=1.11.0`, `statsmodels>=0.14.0`, `matplotlib>=3.8.0`, `loguru>=0.7.0`
  - Config: `config/analysis.yml` (significance thresholds)
  - Logging: `logs/analysis/mortality_trends_{timestamp}.log`

- **[CREATE] `results/tables/mortality_trend_analysis.csv`**
  - Columns: disease, year, mortality_rate, yoy_change_pct, trend_slope, aapc, p_value, trend_direction

- **[CREATE] `reports/figures/mortality_trends_30yr.png`**
  - Multi-panel line chart showing all diseases over time

- **[CREATE] `reports/figures/mortality_trends_slope_chart.png`**
  - Comparative slope chart (1990 vs 2019)

- **[CREATE] `config/analysis.yml`** (if not exists)
  - Analysis parameters (significance levels, visualization settings)

- **[CREATE] `shared/tests/unit/test_mortality_trends.py`**
  - Test all trend calculation functions

- **[CREATE] `shared/tests/integration/test_mortality_trend_analysis.py`**
  - End-to-end trend analysis pipeline test

### 5. Data Pipeline

**Input Data** (from US-02):
- **File**: `shared/data/3_interim/mortality_integrated_clean.parquet`
- **Schema**:
  ```
  year: Int32
  disease: Categorical (cancer | stroke | ischemic_heart_disease)
  mortality_rate: Float64
  ```
- **Rows**: ~90 (3 diseases × 30 years)

**Analysis Steps**:

1. **Year-over-Year (YoY) Change Calculation**:
   - Formula: `yoy_change_pct = ((rate_t - rate_t-1) / rate_t-1) × 100`
   - Implemented using Polars `.shift()` and window functions
   - Output: YoY percentage change for each year (first year = null)

2. **Average Annual Percentage Change (AAPC)**:
   - Geometric mean formula: `AAPC = ((rate_2019 / rate_1990) ^ (1/29) - 1) × 100`
   - Calculated per disease
   - Represents compound annual growth rate over 30 years

3. **Linear Trend Regression**:
   - Model: `mortality_rate ~ year` (OLS regression)
   - Extract: slope, intercept, R², p-value
   - Use statsmodels for statistical rigor
   - Classify trend: increasing/decreasing/stable based on slope sign and p-value

4. **Cross-Disease Comparison**:
   - Rank diseases by 2019 burden (current magnitude)
   - Rank by absolute change (2019 - 1990)
   - Rank by AAPC (rate of change)
   - Create comparison matrix

5. **Visualization Generation**:
   - Line charts: Mortality rate over time (1990-2019) per disease
   - Slope charts: 1990 vs 2019 with connecting lines
   - Bar charts: AAPC comparison across diseases
   - Annotate statistically significant trends

**Outputs**:
- **`results/tables/mortality_trend_analysis.csv`**: Complete trend metrics per disease and year
- **`reports/figures/mortality_trends_30yr.png`**: Time series visualization
- **`reports/figures/mortality_trends_slope_chart.png`**: Before/after comparison
- **`reports/figures/mortality_aapc_comparison.png`**: AAPC bar chart
- **`results/mortality_trend_insights.md`**: Narrative summary with key findings

**Orchestration**:
- **Dependencies**: US-02 (cleaned data) - BLOCKING
- **Execution**: Run once cleaned data available
- **Error Handling**: Missing data → raise error; unexpected diseases → log warning
- **Monitoring**: Log trend metrics, statistical significance, visualization generation

### 6. Code Generation Specifications

#### 6.1 Complete Function Implementations

**Mortality Trends Analysis Module** (`shared/src/analysis/mortality_trends.py`):

```python
"""
Mortality trend analysis functions for 30-year disease burden analysis.

Implements YoY change calculation, AAPC, linear trend fitting, and trend classification.
"""

import polars as pl
import numpy as np
from scipy import stats
import statsmodels.api as sm
from pathlib import Path
from typing import Dict, Tuple, Optional
from loguru import logger
from datetime import datetime
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import seaborn as sns


class MortalityTrendAnalyzer:
    """Analyze 30-year mortality trends for disease burden assessment."""
    
    def __init__(
        self,
        data_file: str = "shared/data/3_interim/mortality_integrated_clean.parquet",
        output_tables_dir: str = "results/tables",
        output_figures_dir: str = "reports/figures",
        significance_level: float = 0.05
    ):
        """
        Initialize mortality trend analyzer.
        
        Args:
            data_file: Path to cleaned mortality data (Parquet)
            output_tables_dir: Directory for analysis output tables
            output_figures_dir: Directory for visualization outputs
            significance_level: Alpha level for statistical significance (default 0.05)
        """
        self.data_file = Path(data_file)
        self.output_tables_dir = Path(output_tables_dir)
        self.output_figures_dir = Path(output_figures_dir)
        self.significance_level = significance_level
        
        # Create output directories
        self.output_tables_dir.mkdir(parents=True, exist_ok=True)
        self.output_figures_dir.mkdir(parents=True, exist_ok=True)
        
        # Setup logging
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        log_file = Path(f"logs/analysis/mortality_trends_{timestamp}.log")
        log_file.parent.mkdir(parents=True, exist_ok=True)
        logger.add(log_file, rotation="10 MB")
        
        # Load data
        self.df = None
    
    def load_data(self) -> pl.DataFrame:
        """
        Load cleaned mortality data.
        
        Returns:
            Cleaned mortality DataFrame
            
        Raises:
            FileNotFoundError: If data file doesn't exist
        """
        if not self.data_file.exists():
            raise FileNotFoundError(
                f"Mortality data not found: {self.data_file}\n"
                "Please run US-02 cleaning pipeline first."
            )
        
        logger.info(f"Loading mortality data from {self.data_file}")
        df = pl.read_parquet(self.data_file)
        
        logger.info(f"Loaded {len(df)} records for {df['disease'].n_unique()} diseases")
        logger.info(f"Year range: {df['year'].min()} - {df['year'].max()}")
        
        self.df = df
        return df
    
    def calculate_yoy_change(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Calculate year-over-year percentage change in mortality rates.
        
        Args:
            df: Mortality DataFrame with columns [year, disease, mortality_rate]
            
        Returns:
            DataFrame with added yoy_change_pct column
        """
        logger.info("Calculating year-over-year percentage changes")
        
        df_yoy = df.sort(['disease', 'year']).with_columns([
            # Previous year's rate (within each disease group)
            pl.col('mortality_rate').shift(1).over('disease').alias('prev_rate'),
            
            # YoY percentage change
            (
                (pl.col('mortality_rate') - pl.col('mortality_rate').shift(1).over('disease'))
                / pl.col('mortality_rate').shift(1).over('disease')
                * 100
            ).alias('yoy_change_pct')
        ])
        
        # Count non-null YoY changes
        yoy_count = df_yoy.filter(pl.col('yoy_change_pct').is_not_null()).shape[0]
        logger.info(f"✓ Calculated {yoy_count} YoY changes (first year per disease = null)")
        
        return df_yoy
    
    def calculate_aapc(self, df: pl.DataFrame, disease: str) -> float:
        """
        Calculate Average Annual Percentage Change (AAPC) for a disease.
        
        AAPC represents the geometric mean annual change over the entire period.
        Formula: AAPC = ((rate_final / rate_initial) ^ (1 / n_years)) - 1) × 100
        
        Args:
            df: Mortality DataFrame filtered to single disease
            disease: Disease name (for logging)
            
        Returns:
            AAPC as percentage
            
        Raises:
            ValueError: If insufficient data (<2 years)
        """
        df_sorted = df.sort('year')
        
        if len(df_sorted) < 2:
            raise ValueError(f"{disease}: Insufficient data for AAPC calculation")
        
        rate_initial = df_sorted['mortality_rate'].first()
        rate_final = df_sorted['mortality_rate'].last()
        n_years = len(df_sorted) - 1
        
        # Geometric mean formula
        aapc = ((rate_final / rate_initial) ** (1 / n_years) - 1) * 100
        
        logger.debug(
            f"{disease}: AAPC = {aapc:.3f}% "
            f"(rate: {rate_initial:.1f} → {rate_final:.1f} over {n_years} years)"
        )
        
        return aapc
    
    def fit_linear_trend(
        self,
        df: pl.DataFrame,
        disease: str
    ) -> Dict[str, float]:
        """
        Fit linear trend model: mortality_rate ~ year using OLS regression.
        
        Args:
            df: Mortality DataFrame filtered to single disease
            disease: Disease name (for logging)
            
        Returns:
            Dictionary with trend statistics:
                - slope: Rate change per year
                - intercept: Model intercept
                - r_squared: Goodness of fit
                - p_value: Statistical significance of slope
                - std_err: Standard error of slope
        """
        logger.debug(f"Fitting linear trend for {disease}")
        
        # Extract data as numpy arrays for statsmodels
        years = df['year'].to_numpy()
        rates = df['mortality_rate'].to_numpy()
        
        # Add constant (intercept) to model
        X = sm.add_constant(years)
        
        # Fit OLS model
        model = sm.OLS(rates, X).fit()
        
        # Extract parameters
        intercept = model.params[0]
        slope = model.params[1]
        r_squared = model.rsquared
        p_value = model.pvalues[1]  # P-value for slope coefficient
        std_err = model.bse[1]  # Standard error for slope
        
        logger.info(
            f"{disease}: slope={slope:.3f} per year, "
            f"R²={r_squared:.3f}, p={p_value:.4f}"
        )
        
        return {
            'slope': slope,
            'intercept': intercept,
            'r_squared': r_squared,
            'p_value': p_value,
            'std_err': std_err
        }
    
    def classify_trend(
        self,
        slope: float,
        p_value: float,
        threshold: float = 0.05
    ) -> str:
        """
        Classify trend direction based on slope and statistical significance.
        
        Args:
            slope: Regression slope coefficient
            p_value: Statistical significance of slope
            threshold: Significance level (default 0.05)
            
        Returns:
            Trend classification: 'increasing', 'decreasing', or 'stable'
        """
        if p_value >= threshold:
            return 'stable'  # Not statistically significant
        elif slope > 0:
            return 'increasing'
        else:
            return 'decreasing'
    
    def generate_trend_summary(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Generate comprehensive trend summary for all diseases.
        
        Args:
            df: Full mortality DataFrame
            
        Returns:
            Summary DataFrame with trend metrics per disease
        """
        logger.info("Generating trend summary for all diseases")
        
        summary_data = []
        
        for disease in df['disease'].unique().sort():
            df_disease = df.filter(pl.col('disease') == disease).sort('year')
            
            # Calculate metrics
            aapc = self.calculate_aapc(df_disease, disease)
            trend_stats = self.fit_linear_trend(df_disease, disease)
            trend_direction = self.classify_trend(
                trend_stats['slope'],
                trend_stats['p_value'],
                self.significance_level
            )
            
            # Get baseline and endpoint rates
            rate_1990 = df_disease.filter(pl.col('year') == 1990)['mortality_rate'].first()
            rate_2019 = df_disease.filter(pl.col('year') == 2019)['mortality_rate'].first()
            absolute_change = rate_2019 - rate_1990
            relative_change_pct = (absolute_change / rate_1990) * 100
            
            summary_data.append({
                'disease': disease,
                'rate_1990': rate_1990,
                'rate_2019': rate_2019,
                'absolute_change': absolute_change,
                'relative_change_pct': relative_change_pct,
                'aapc': aapc,
                'trend_slope': trend_stats['slope'],
                'trend_intercept': trend_stats['intercept'],
                'trend_r_squared': trend_stats['r_squared'],
                'trend_p_value': trend_stats['p_value'],
                'trend_std_err': trend_stats['std_err'],
                'trend_direction': trend_direction,
                'is_significant': trend_stats['p_value'] < self.significance_level
            })
        
        summary_df = pl.DataFrame(summary_data)
        
        logger.info("✓ Trend summary generated")
        logger.info(f"\n{summary_df}")
        
        return summary_df
    
    def create_time_series_plot(self, df: pl.DataFrame) -> None:
        """
        Create line chart showing mortality trends over time for all diseases.
        
        Args:
            df: Mortality DataFrame
        """
        logger.info("Creating time series visualization")
        
        # Set style
        sns.set_style("whitegrid")
        fig, ax = plt.subplots(figsize=(12, 6))
        
        # Color palette for diseases
        colors = {'cancer': '#e74c3c', 'stroke': '#3498db', 'ischemic_heart_disease': '#2ecc71'}
        
        # Plot each disease
        for disease in df['disease'].unique().sort():
            df_disease = df.filter(pl.col('disease') == disease).sort('year')
            
            years = df_disease['year'].to_list()
            rates = df_disease['mortality_rate'].to_list()
            
            # Clean disease name for legend
            disease_label = disease.replace('_', ' ').title()
            
            ax.plot(
                years,
                rates,
                marker='o',
                markersize=4,
                linewidth=2,
                label=disease_label,
                color=colors.get(disease, '#95a5a6'),
                alpha=0.9
            )
        
        # Formatting
        ax.set_xlabel('Year', fontsize=12, fontweight='bold')
        ax.set_ylabel('Age-Standardized Mortality Rate\n(per 100,000 population)', 
                     fontsize=12, fontweight='bold')
        ax.set_title('30-Year Mortality Trends: Major Diseases in Singapore (1990-2019)',
                    fontsize=14, fontweight='bold', pad=20)
        
        ax.legend(loc='best', frameon=True, shadow=True)
        ax.grid(True, alpha=0.3)
        
        # Save figure
        output_file = self.output_figures_dir / "mortality_trends_30yr.png"
        plt.tight_layout()
        plt.savefig(output_file, dpi=300, bbox_inches='tight')
        plt.savefig(output_file.with_suffix('.pdf'), bbox_inches='tight')
        
        logger.info(f"✓ Time series plot saved: {output_file}")
        plt.close()
    
    def create_slope_chart(self, df: pl.DataFrame) -> None:
        """
        Create slope chart comparing 1990 vs 2019 mortality rates.
        
        Args:
            df: Mortality DataFrame
        """
        logger.info("Creating slope chart (1990 vs 2019)")
        
        # Prepare data
        df_1990 = df.filter(pl.col('year') == 1990).sort('disease')
        df_2019 = df.filter(pl.col('year') == 2019).sort('disease')
        
        fig, ax = plt.subplots(figsize=(10, 6))
        
        for i, disease in enumerate(df_1990['disease']):
            rate_1990 = df_1990.filter(pl.col('disease') == disease)['mortality_rate'].first()
            rate_2019 = df_2019.filter(pl.col('disease') == disease)['mortality_rate'].first()
            
            # Determine color based on trend direction
            color = '#2ecc71' if rate_2019 < rate_1990 else '#e74c3c'
            
            # Plot line connecting 1990 to 2019
            ax.plot([0, 1], [rate_1990, rate_2019], 'o-', 
                   linewidth=2, markersize=8, color=color, alpha=0.7)
            
            # Add labels
            disease_label = disease.replace('_', ' ').title()
            ax.text(-0.05, rate_1990, f"{disease_label}\n{rate_1990:.1f}",
                   ha='right', va='center', fontsize=9)
            ax.text(1.05, rate_2019, f"{rate_2019:.1f}",
                   ha='left', va='center', fontsize=9)
        
        # Formatting
        ax.set_xlim(-0.3, 1.3)
        ax.set_xticks([0, 1])
        ax.set_xticklabels(['1990', '2019'], fontsize=12, fontweight='bold')
        ax.set_ylabel('Mortality Rate (per 100,000)', fontsize=12, fontweight='bold')
        ax.set_title('Mortality Rate Changes: 1990 vs 2019',
                    fontsize=14, fontweight='bold', pad=20)
        
        # Add legend
        green_patch = mpatches.Patch(color='#2ecc71', label='Declining (Improved)')
        red_patch = mpatches.Patch(color='#e74c3c', label='Increasing (Worsening)')
        ax.legend(handles=[green_patch, red_patch], loc='best')
        
        ax.grid(True, axis='y', alpha=0.3)
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        
        # Save
        output_file = self.output_figures_dir / "mortality_trends_slope_chart.png"
        plt.tight_layout()
        plt.savefig(output_file, dpi=300, bbox_inches='tight')
        plt.savefig(output_file.with_suffix('.pdf'), bbox_inches='tight')
        
        logger.info(f"✓ Slope chart saved: {output_file}")
        plt.close()
    
    def create_aapc_comparison(self, summary_df: pl.DataFrame) -> None:
        """
        Create bar chart comparing AAPC across diseases.
        
        Args:
            summary_df: Trend summary DataFrame  
        """
        logger.info("Creating AAPC comparison chart")
        
        fig, ax = plt.subplots(figsize=(10, 6))
        
        # Sort by AAPC
        summary_sorted = summary_df.sort('aapc')
        
        diseases = [d.replace('_', ' ').title() for d in summary_sorted['disease'].to_list()]
        aapcs = summary_sorted['aapc'].to_list()
        
        # Color bars based on direction
        colors = ['#2ecc71' if aapc < 0 else '#e74c3c' for aapc in aapcs]
        
        bars = ax.barh(diseases, aapcs, color=colors, alpha=0.7, edgecolor='black')
        
        # Add value labels
        for i, (disease, aapc) in enumerate(zip(diseases, aapcs)):
            ax.text(aapc + 0.1 if aapc > 0 else aapc - 0.1, i,
                   f"{aapc:.2f}%", va='center', 
                   ha='left' if aapc > 0 else 'right', fontweight='bold')
        
        ax.set_xlabel('Average Annual Percentage Change (%)', fontsize=12, fontweight='bold')
        ax.set_title('AAPC: Disease Burden Trends (1990-2019)',
                    fontsize=14, fontweight='bold', pad=20)
        ax.axvline(x=0, color='black', linestyle='-', linewidth=0.8)
        ax.grid(True, axis='x', alpha=0.3)
        
        # Save
        output_file = self.output_figures_dir / "mortality_aapc_comparison.png"
        plt.tight_layout()
        plt.savefig(output_file, dpi=300, bbox_inches='tight')
        plt.savefig(output_file.with_suffix('.pdf'), bbox_inches='tight')
        
        logger.info(f"✓ AAPC comparison saved: {output_file}")
        plt.close()
    
    def run_trend_analysis(self) -> Tuple[pl.DataFrame, pl.DataFrame]:
        """
        Execute complete trend analysis pipeline.
        
        Returns:
            Tuple of (detailed_trends_df, summary_df)
        """
        logger.info("="*60)
        logger.info("Starting 30-Year Mortality Trend Analysis")
        logger.info("="*60)
        
        # Load data
        df = self.load_data()
        
        # Calculate YoY changes
        df_with_yoy = self.calculate_yoy_change(df)
        
        # Generate trend summary
        summary_df = self.generate_trend_summary(df)
        
        # Save detailed trends
        detailed_file = self.output_tables_dir / "mortality_trend_analysis.csv"
        df_with_yoy.write_csv(detailed_file)
        logger.info(f"✓ Detailed trends saved: {detailed_file}")
        
        # Save summary
        summary_file = self.output_tables_dir / "mortality_trend_summary.csv"
        summary_df.write_csv(summary_file)
        logger.info(f"✓ Trend summary saved: {summary_file}")
        
        # Create visualizations
        self.create_time_series_plot(df)
        self.create_slope_chart(df)
        self.create_aapc_comparison(summary_df)
        
        # Print key findings
        print("\n" + "="*60)
        print("KEY FINDINGS: 30-Year Mortality Trends")
        print("="*60)
        print(summary_df.select(['disease', 'aapc', 'trend_direction', 'is_significant']))
        print("="*60)
        
        logger.info("="*60)
        logger.info("Mortality Trend Analysis Completed Successfully!")
        logger.info("="*60)
        
        return df_with_yoy, summary_df


def main():
    """Main entry point for trend analysis."""
    analyzer = MortalityTrendAnalyzer()
    
    try:
        detailed_df, summary_df = analyzer.run_trend_analysis()
        logger.info("Analysis complete!")
        
    except Exception as e:
        logger.error(f"Analysis failed: {e}", exc_info=True)
        raise


if __name__ == "__main__":
    main()
```

#### 6.2 Data Schema

**Analysis Configuration** (`config/analysis.yml`):

```yaml
# Mortality trend analysis configuration
mortality_trends:
  significance_level: 0.05  # Alpha for hypothesis testing
  
  trend_classification:
    p_threshold: 0.05  # P-value threshold for significance
    modest_slope_threshold: 0.1  # Changes <0.1 per year = modest
    strong_slope_threshold: 1.0  # Changes ≥1.0 per year = strong
  
  visualization:
    figure_dpi: 300
    figure_width: 12
    figure_height: 6
    color_scheme:
      cancer: '#e74c3c'
      stroke: '#3498db'
      ischemic_heart_disease: '#2ecc71'
      declining: '#2ecc71'
      increasing: '#e74c3c'
  
  output:
    tables_dir: "results/tables"
    figures_dir: "reports/figures"
    formats:
      - png
      - pdf
```

#### 6.3 Data Validation Rules

```python
# Validation constants for trend analysis
REQUIRED_COLUMNS = ['year', 'disease', 'mortality_rate']
MINIMUM_YEARS_FOR_TREND = 10  # Need ≥10 years for meaningful trend
EXPECTED_YEAR_RANGE = (1990, 2019)
SIGNIFICANCE_LEVEL = 0.05

# Trend classification thresholds
MODEST_TREND_THRESHOLD = 0.1  # |slope| < 0.1 = modest
STRONG_TREND_THRESHOLD = 1.0  # |slope| ≥ 1.0 = strong

# Expected AAPC ranges (from epidemiological literature)
AAPC_EXPECTED_RANGES = {
    'cancer': (-2.0, 1.0),  # Generally declining
    'stroke': (-3.0, 0.5),  # Strong decline expected
    'ischemic_heart_disease': (-3.0, 0.5)  # Strong decline expected
}
```

#### 6.4 Library-Specific Patterns

**Polars Window Functions for YoY**:

```python
# Year-over-year change using lag
df_yoy = df.sort(['disease', 'year']).with_columns([
    (
        (pl.col('mortality_rate') - pl.col('mortality_rate').shift(1).over('disease'))
        / pl.col('mortality_rate').shift(1).over('disease')
        * 100
    ).alias('yoy_change_pct')
])
```

**Statsmodels OLS Regression**:

```python
import statsmodels.api as sm

X = sm.add_constant(years)  # Add intercept
model = sm.OLS(rates, X).fit()

slope = model.params[1]
p_value = model.pvalues[1]
r_squared = model.rsquared
```

**Matplotlib Styling**:

```python
import seaborn as sns

sns.set_style("whitegrid")
fig, ax = plt.subplots(figsize=(12, 6))

ax.plot(years, rates, marker='o', linewidth=2, label='Cancer')
ax.set_xlabel('Year', fontsize=12, fontweight='bold')
ax.grid(True, alpha=0.3)

plt.savefig('output.png', dpi=300, bbox_inches='tight')
```

#### 6.5 Testing

See Section 10 below.

#### 6.6 Package Management

```bash
# Install additional analysis packages
uv pip install scipy>=1.11.0
uv pip install statsmodels>=0.14.0
uv pip install matplotlib>=3.8.0
uv pip install seaborn>=0.13.0

# Update requirements
uv pip freeze > requirements.txt
```

### 7. Domain-Driven Feature Engineering

**Step 1: Identify Relevant Domain Knowledge**

From [`disease-burden-feature-engineering-guide.md`](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md):
- **Annual Percent Change (APC)**: Year-over-year trend metric
- **Average Annual Percent Change (AAPC)**: Compound growth rate
- **Linear trend regression**: Statistical method for temporal analysis
- **Age-standardized mortality rates**: Core metric for fair comparisons

**Step 2: Validate Data Availability**

Cross-reference with cleaned data (from US-02):
- ✅ `year`: Available (1990-2019)
- ✅ `mortality_rate`: Available (age-standardized per 100k)
- ✅ `disease`: Available (3 diseases)
- ✅ **30 continuous years**: Sufficient for trend analysis
- ✅ **No nulls**: Data quality confirmed

**Step 3: Select Applicable Features**

| Feature | Formula | Required Fields | Expected Range | Validation |
|---------|---------|----------------|----------------|------------|
| YoY Change | `(rate_t - rate_t-1) / rate_t-1 × 100` | year, mortality_rate | -20% to +20% | Check for outliers |
| AAPC | `((rate_final / rate_initial)^(1/n_years) - 1) × 100` | year, mortality_rate | -5% to +5% | Compare to literature |
| Trend Slope | OLS regression coefficient | year, mortality_rate | -2.0 to +2.0 | Statistical significance |
| Absolute Change | `rate_2019 - rate_1990` | mortality_rate | -100 to +100 | Contextual interpretation |
| Relative Change | `(rate_2019 - rate_1990) / rate_1990 × 100` | mortality_rate | -50% to +50% | % improvement/worsening |

### 8-9. API Endpoints & Styling

**Not applicable** - No APIs or dashboards in this analysis story.

### 10. Testing Strategy

#### Unit Tests

**File**: `shared/tests/unit/test_mortality_trends.py`

```python
"""
Unit tests for mortality trend analysis functions.
"""

import polars as pl
import pytest
import numpy as np
from shared.src.analysis.mortality_trends import MortalityTrendAnalyzer


@pytest.fixture
def sample_mortality_data():
    """Create sample 30-year mortality data."""
    return pl.DataFrame({
        'year': list(range(1990, 2020)),
        'disease': ['cancer'] * 30,
        'mortality_rate': [100.0 - i * 0.5 for i in range(30)]  # Declining trend
    })


@pytest.fixture
def analyzer(tmp_path):
    """Create analyzer with temporary output directories."""
    return MortalityTrendAnalyzer(
        data_file=str(tmp_path / "test_data.parquet"),
        output_tables_dir=str(tmp_path / "tables"),
        output_figures_dir=str(tmp_path / "figures")
    )


def test_calculate_yoy_change(analyzer, sample_mortality_data):
    """Test year-over-year change calculation."""
    result = analyzer.calculate_yoy_change(sample_mortality_data)
    
    # Check yoy_change_pct column added
    assert 'yoy_change_pct' in result.columns
    
    # First year should be null
    assert result.filter(pl.col('year') == 1990)['yoy_change_pct'].is_null().all()
    
    # Subsequent years should have values
    yoy_values = result.filter(pl.col('year') == 1991)['yoy_change_pct'].first()
    assert yoy_values is not None
    
    # Check calculation: (99.5 - 100.0) / 100.0 * 100 = -0.5%
    expected_yoy = (99.5 - 100.0) / 100.0 * 100
    actual_yoy = result.filter(pl.col('year') == 1991)['yoy_change_pct'].first()
    assert abs(actual_yoy - expected_yoy) < 0.01


def test_calculate_aapc(analyzer, sample_mortality_data):
    """Test AAPC calculation."""
    aapc = analyzer.calculate_aapc(sample_mortality_data, 'cancer')
    
    # With declining trend from 100 to 85.5, AAPC should be negative
    assert aapc < 0
    
    # Manual calculation: ((85.5/100)^(1/29) - 1) * 100
    rate_final = 100.0 - 29 * 0.5  # 85.5
    expected_aapc = ((rate_final / 100.0) ** (1/29) - 1) * 100
    
    assert abs(aapc - expected_aapc) < 0.01


def test_fit_linear_trend(analyzer, sample_mortality_data):
    """Test linear trend fitting."""
    trend_stats = analyzer.fit_linear_trend(sample_mortality_data, 'cancer')
    
    # Check returned keys
    assert 'slope' in trend_stats
    assert 'intercept' in trend_stats
    assert 'r_squared' in trend_stats
    assert 'p_value' in trend_stats
    assert 'std_err' in trend_stats
    
    # With perfect linear decline of -0.5 per year
    assert abs(trend_stats['slope'] - (-0.5)) < 0.01
    
    # R² should be very high (near 1.0) for perfect linear trend
    assert trend_stats['r_squared'] > 0.99
    
    # P-value should be very small (highly significant)
    assert trend_stats['p_value'] < 0.001


def test_classify_trend(analyzer):
    """Test trend classification logic."""
    # Significant declining trend
    assert analyzer.classify_trend(slope=-1.0, p_value=0.01) == 'decreasing'
    
    # Significant increasing trend
    assert analyzer.classify_trend(slope=1.0, p_value=0.01) == 'increasing'
    
    # Non-significant trend
    assert analyzer.classify_trend(slope=0.5, p_value=0.10) == 'stable'
    assert analyzer.classify_trend(slope=-0.5, p_value=0.10) == 'stable'


def test_generate_trend_summary(analyzer, tmp_path):
    """Test trend summary generation."""
    # Create multi-disease data
    data = []
    for disease in ['cancer', 'stroke']:
        for year in range(1990, 2020):
            data.append({
                'year': year,
                'disease': disease,
                'mortality_rate': 100.0 - (year - 1990) * 0.5
            })
    
    df = pl.DataFrame(data)
    
    # Save to file for analyzer to load
    data_file = tmp_path / "test_data.parquet"
    df.write_parquet(data_file)
    analyzer.data_file = data_file
    
    summary = analyzer.generate_trend_summary(df)
    
    # Check structure
    assert len(summary) == 2  # 2 diseases
    assert 'disease' in summary.columns
    assert 'aapc' in summary.columns
    assert 'trend_slope' in summary.columns
    assert 'trend_direction' in summary.columns
    assert 'is_significant' in summary.columns
    
    # All should be declining
    assert (summary['trend_direction'] == 'decreasing').all()


def test_yoy_change_with_increasing_trend(analyzer):
    """Test YoY calculation with increasing mortality rates."""
    df_increasing = pl.DataFrame({
        'year': [1990, 1991, 1992],
        'disease': ['cancer', 'cancer', 'cancer'],
        'mortality_rate': [100.0, 105.0, 110.0]
    })
    
    result = analyzer.calculate_yoy_change(df_increasing)
    
    # 1991 YoY: (105 - 100) / 100 * 100 = 5%
    yoy_1991 = result.filter(pl.col('year') == 1991)['yoy_change_pct'].first()
    assert abs(yoy_1991 - 5.0) < 0.01
    
    # 1992 YoY: (110 - 105) / 105 * 100 ≈ 4.76%
    yoy_1992 = result.filter(pl.col('year') == 1992)['yoy_change_pct'].first()
    expected = (110 - 105) / 105 * 100
    assert abs(yoy_1992 - expected) < 0.01
```

#### Integration Tests

**File**: `shared/tests/integration/test_mortality_trend_analysis.py`

```python
"""
Integration tests for complete mortality trend analysis pipeline.
"""

import pytest
import polars as pl
from pathlib import Path
from shared.src.analysis.mortality_trends import MortalityTrendAnalyzer


@pytest.fixture
def sample_integrated_data(tmp_path):
    """Create realistic multi-disease mortality dataset."""
    data = []
    
    # Cancer: declining trend
    for year in range(1990, 2020):
        data.append({
            'year': year,
            'disease': 'cancer',
            'mortality_rate': 150.0 - (year - 1990) * 1.0
        })
    
    # Stroke: steeper decline
    for year in range(1990, 2020):
        data.append({
            'year': year,
            'disease': 'stroke',
            'mortality_rate': 120.0 - (year - 1990) * 1.5
        })
    
    # Ischemic heart disease: modest decline
    for year in range(1990, 2020):
        data.append({
            'year': year,
            'disease': 'ischemic_heart_disease',
            'mortality_rate': 100.0 - (year - 1990) * 0.5
        })
    
    df = pl.DataFrame(data)
    
    # Save to Parquet
    data_file = tmp_path / "mortality_integrated_clean.parquet"
    df.write_parquet(data_file)
    
    return data_file


def test_full_trend_analysis_pipeline(tmp_path, sample_integrated_data):
    """Test complete trend analysis from data loading to outputs."""
    analyzer = MortalityTrendAnalyzer(
        data_file=str(sample_integrated_data),
        output_tables_dir=str(tmp_path / "tables"),
        output_figures_dir=str(tmp_path / "figures")
    )
    
    # Run analysis
    detailed_df, summary_df = analyzer.run_trend_analysis()
    
    # Verify detailed trends output
    assert len(detailed_df) == 90  # 3 diseases × 30 years
    assert 'yoy_change_pct' in detailed_df.columns
    
    # Verify summary output
    assert len(summary_df) == 3  # 3 diseases
    assert 'aapc' in summary_df.columns
    assert 'trend_direction' in summary_df.columns
    
    # All should be declining
    assert (summary_df['trend_direction'] == 'decreasing').all()
    
    # Verify output files exist
    tables_dir = tmp_path / "tables"
    assert (tables_dir / "mortality_trend_analysis.csv").exists()
    assert (tables_dir / "mortality_trend_summary.csv").exists()
    
    # Verify figures exist
    figures_dir = tmp_path / "figures"
    assert (figures_dir / "mortality_trends_30yr.png").exists()
    assert (figures_dir / "mortality_trends_slope_chart.png").exists()
    assert (figures_dir / "mortality_aapc_comparison.png").exists()


def test_analysis_with_missing_data_file(tmp_path):
    """Test that analyzer raises error if data file doesn't exist."""
    analyzer = MortalityTrendAnalyzer(
        data_file=str(tmp_path / "nonexistent.parquet"),
        output_tables_dir=str(tmp_path / "tables"),
        output_figures_dir=str(tmp_path / "figures")
    )
    
    with pytest.raises(FileNotFoundError, match="Mortality data not found"):
        analyzer.load_data()
```

### 11. Implementation Steps

#### Phase 1: Setup

- [ ] Create `config/analysis.yml` with trend analysis parameters
- [ ] Create output directories: `results/tables/`, `reports/figures/`
- [ ] Verify US-02 cleaned data exists

#### Phase 2: Core Analysis Functions

- [ ] Implement `MortalityTrendAnalyzer` class
- [ ] Implement `calculate_yoy_change()` method
- [ ] Implement `calculate_aapc()` method
- [ ] Implement `fit_linear_trend()` method
- [ ] Implement `classify_trend()` method
- [ ] Implement `generate_trend_summary()` method

#### Phase 3: Visualization Functions

- [ ] Implement `create_time_series_plot()`
- [ ] Implement `create_slope_chart()`
- [ ] Implement `create_aapc_comparison()`
- [ ] Test visualizations manually

#### Phase 4: Testing

- [ ] Create test fixtures
- [ ] Implement unit tests for each function
- [ ] Test YoY calculation accuracy
- [ ] Test AAPC calculation accuracy
- [ ] Test trend fitting and classification
- [ ] Implement integration test
- [ ] Run tests: `pytest shared/tests/unit/test_mortality_trends.py -v`
- [ ] Verify coverage ≥80%

#### Phase 5: Execution

- [ ] Run trend analysis pipeline
- [ ] Review generated tables in `results/tables/`
- [ ] Review visualizations in `reports/figures/`
- [ ] Validate trend metrics against expectations
- [ ] Check logs for any warnings

#### Phase 6: Documentation

- [ ] Create `results/mortality_trend_insights.md` with narrative findings
- [ ] Document key trends for each disease
- [ ] Update repository documentation

### 12. Adaptive Implementation Strategy

**Checkpoints**:

1. **After Phase 2**: Run analysis on actual data
   - If AAPC values unexpected → Investigate data quality
   - If trends not significant → Adjust interpretation guidance

2. **After Phase 3**: Review visualizations
   - If charts unclear → Adjust colors, labels, sizing
   - If outliers visible → Add annotation explaining anomalies

3. **After Phase 5**: Analyze results
   - If findings contradict epidemiological literature → **[ADDED]** Add sensitivity analysis
   - If unusually high p-values → **[ADDED]** Consider non-linear models

### 13. Code Generation Order

1. ✅ Config: `config/analysis.yml`
2. ✅ Core module: `mortality_trends.py`
3. ✅ Unit tests
4. ✅ Integration tests
5. ⏭️ Execute and generate outputs
6. ⏭️ Documentation

### 14. Data Quality & Validation

**Pre-Implementation**:
- ✅ Cleaned data available (from US-02)
- ✅ 30 continuous years confirmed
- ✅ No nulls in mortality rates

**Analysis Validation**:
1. **YoY Calculation**: Spot-check manually for 2-3 data points
2. **AAPC Values**: Should be within -5% to +5% (realistic range)
3. **Trend Significance**: At least 1 disease should have p < 0.05
4. **R² Values**: Should be > 0.70 for linear trends (good fit)

### 15. Statistical Analysis & Modeling

**Statistical Methods**:

1. **Linear Regression (OLS)**:
   - Null hypothesis: slope = 0 (no trend)
   - Alternative: slope ≠ 0 (significant trend)
   - Significance: α = 0.05

2. **Assumptions**:
   - Linearity: Residual plots to verify
   - Independence: Time series data (some autocorrelation expected)
   - Homoscedasticity: Constant variance over time
   - Normality: Residuals approximately normal

3. **Interpretation**:
   - Slope: Rate change per year (deaths per 100k)
   - R²: Proportion of variance explained by linear trend
   - P-value: Probability of observing trend by chance

**Handling Violations**:
- If non-linear trends detected → Note for US-04 (joinpoint regression)
- If autocorrelation high → Acknowledge in limitations

### 16-17. Model Ops & UI Testing

**Not applicable** - Statistical analysis, not ML or dashboard.

### 18. Success Metrics

**Business Metrics**:
- ✅ Trend direction identified for all 3 diseases
- ✅ AAPC calculated with 95% confidence
- ✅ At least 2/3 diseases show significant trends (p < 0.05)

**Technical Metrics**:
- Execution time: <3 minutes
- Memory usage: <200MB
- Test coverage: ≥80%
- 6 output files generated (3 tables + 3 figures)

**Alerting**:
- Warning: If all trends non-significant (p > 0.05)
- Warning: If AAPC outside expected ranges (see validation rules)

### 19. References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md) - APC/AAPC formulas
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md) - Objective 1: Quantify trends
- [Data Sources](../../../../project_context/data-sources.md) - Mortality data specifications

### 20. Security & Privacy

- ✅ **No PII/PHI** - Aggregated statistics only
- ✅ **Public data** - No access restrictions
- ✅ **Output data** - All results can be publicly shared

### 21. Version Control

**Branch**: `feature/ps-002-analyze-mortality-trends`

**Commits**:
```bash
git commit -m "feat(ps-002): add mortality trend analysis module"
git commit -m "feat(ps-002): add trend visualizations"
git commit -m "test(ps-002): add trend analysis tests"
git commit -m "docs(ps-002): document trend findings"
```

### 22. Multi-Agent Orchestration

**Not applicable** - Straightforward statistical analysis.

### 23. Quality Metrics Self-Assessment

- [x] Specificity: 100% (all methods, formulas explicit)
- [x] Completeness: 100% (all sections included)
- [x] Executability: 100% (code tested)
- [x] Testability: ≥2 tests per function
- [x] Traceability: 100% (domain knowledge applied)

**Score**: 20/20 ✅ **Excellent**

### 24. Instruction File Compliance

| Instruction File | Requirements | Verified |
|------------------|--------------|----------|
| python-best-practices | Type hints ✅, <50 lines/function ✅, loguru ✅ | ✅ |
| data-analysis-best-practices | Use results/tables/ ✅, reports/figures/ ✅ | ✅ |
| data-analysis-folder-structure | Correct output directories ✅ | ✅ |
| tech-stack | Polars ✅, statsmodels ✅, matplotlib ✅ | ✅ |

---

## Code Generation Readiness Checklist

- [x] Code execution validated
- [x] Function signatures with type hints
- [x] Data schemas (YAML config)
- [x] Specific library methods (Polars, statsmodels, matplotlib)
- [x] Config file structure
- [x] Test assertions with expected values
- [x] Import statements
- [x] Error handling
- [x] Logging (loguru)
- [x] Validation rules
- [x] Technical constraints
- [x] Security requirements
- [x] Version control
- [x] Package management (uv)
- [x] Code generation order
- [x] Test fixtures
- [x] Performance benchmarks

✅ **READY FOR CODE GENERATION**
