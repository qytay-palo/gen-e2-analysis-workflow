# Correlate Expenditure with Utilization & Demographics (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-004-US-05  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Analyst identifying cost drivers**,  
I want **to perform correlation analysis between expenditure growth and utilization patterns (admissions) as well as demographic factors (population aging)**,  
So that **I can quantify which factors drive expenditure increases and inform policy on controllable vs unavoidable cost growth**.

---

## 🎯 Acceptance Criteria

1. **Correlation analysis completed**
   - Correlation matrix: expenditure vs utilization, population, demographics
   - Statistical significance testing: identify meaningful correlations (p < 0.05)
   - Partial correlations: control for confounding variables
   - Time-lagged correlations: test if utilization predicts future expenditure

2. **Driver quantification**
   - Regression models: expenditure ~ utilization + demographics
   - Variance explained: R² for each driver
   - Elasticity estimates: % expenditure change per % utilization change
   - Relative importance: rank drivers by explanatory power

3. **Demographic impact assessed**
   - Aging impact: correlation with elderly population share
   - Population growth impact
   - Dependency ratio correlation

4. **Deliverables**
   - Output: `results/tables/expenditure_correlation_analysis.csv`
   - Regression summary: `results/tables/expenditure_driver_regression.csv`
   - Figures: Scatter plots, correlation heatmap
   - Analysis report: Key drivers identified

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for data; scipy/statsmodels for analysis
- **Visualization**: Matplotlib/Seaborn
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#cost-driver-analysis) - Healthcare cost driver methodologies
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-3) - Driver identification objectives

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scipy>=1.11.0`, `statsmodels>=0.14.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-03 (Expenditure trends - BLOCKING)
- **Data Sources**: `shared/data/3_interim/expenditure_drivers_integrated.parquet`
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### Correlation Analysis
- [ ] Calculate Pearson correlations: expenditure vs all candidate drivers
- [ ] Statistical significance testing for correlations
- [ ] Partial correlations controlling for time trends
- [ ] Time-lagged correlations (expenditure_t ~ utilization_t-1)

### Regression Modeling
- [ ] Simple linear regression: expenditure ~ utilization
- [ ] Multiple regression: expenditure ~ utilization + demographics
- [ ] Extract coefficients, R², p-values
- [ ] Calculate elasticities from log-log models

### Demographic Impact
- [ ] Correlation with elderly population share
- [ ] Correlation with population growth
- [ ] Dependency ratio correlation analysis

### Driver Ranking
- [ ] Rank drivers by correlation strength
- [ ] Rank by R² contribution in multivariate model
- [ ] Identify primary vs secondary drivers

### Visualization
- [ ] Correlation heatmap
- [ ] Scatter plots: expenditure vs key drivers
- [ ] Regression plots with confidence bands
- [ ] Save figures: `reports/figures/expenditure_drivers_*.png/pdf`

### Testing & Documentation
- [ ] Unit tests for correlation calculations
- [ ] Validate regression implementations
- [ ] Docstrings
- [ ] Driver analysis report

---

## 📌 Notes

**Correlation Analysis Example**:
```python
import polars as pl
from scipy.stats import pearsonr
import statsmodels.api as sm

df = pl.read_parquet("shared/data/3_interim/expenditure_drivers_integrated.parquet")

# Calculate correlation
exp = df['total_expenditure'].to_numpy()
util = df['total_admissions'].to_numpy()
corr, p_value = pearsonr(exp, util)

print(f"Correlation: {corr:.3f}, p-value: {p_value:.4f}")

# Regression
X = sm.add_constant(util)
model = sm.OLS(exp, X).fit()
print(model.summary())
```

**Expected Drivers**:
- **Utilization**: Strong positive correlation (more admissions = higher costs)
- **Population aging**: Positive correlation (elderly use more services)
- **Population size**: Positive but weaker (per capita already controls for this)

**Elasticity Interpretation**:
- If elasticity = 1.2: 10% increase in utilization → 12% increase in expenditure

---

## Implementation Plan

### 1. Feature Overview

Quantify statistical relationships between government health expenditure and its candidate drivers (admissions, population, per-capita utilization) using Pearson correlations, OLS regression, and log-log elasticity estimation. Primary role: **Healthcare Financial Analyst**.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | Created by US-02 | **Reuse** |
| `ps-004/src/analysis/expenditure_trend_analyzer.py` | US-03 | **Reference** load pattern |
| `ps-004/src/analysis/expenditure_correlations.py` | Missing | **Create** |
| `results/tables/problem-statement-004/expenditure_correlation_analysis.csv` | Missing | **Create** |
| `results/tables/problem-statement-004/expenditure_driver_regression.csv` | Missing | **Create** |

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/analysis/expenditure_correlations.py`**
  - `ExpenditureCorrelationAnalyzer` class
  - `compute_correlations()` → `pl.DataFrame`
  - `run_ols_regression()` → `dict`
  - `compute_elasticities()` → `dict`
  - `time_lagged_correlations()` → `pl.DataFrame`
  - `save_results()` → `None`

- **[CREATE] `ps-004/tests/unit/test_expenditure_correlations.py`**

- **[CREATE] `ps-004/notebooks/05-correlate-expenditure-utilization.ipynb`**

---

### 5. Data Pipeline

```
shared/data/3_interim/expenditure_drivers_integrated.parquet
  → filter 2006-2018, sort by year
  → identify numeric driver columns: total_admissions, population, per_capita_expenditure

  → Pearson correlations (scipy.stats.pearsonr) vs total_expenditure
      + p-value significance flags
  → time-lagged correlations (expenditure_t vs driver_{t-1})
  → OLS regression: expenditure ~ total_admissions + population (statsmodels)
      extract R², coefficients, p-values
  → log-log regression: ln(expenditure) ~ ln(total_admissions)
      coefficient = expenditure elasticity w.r.t. utilization

  → SAVE: results/tables/problem-statement-004/expenditure_correlation_analysis_YYYYMMDD.csv
  → SAVE: results/tables/problem-statement-004/expenditure_driver_regression_YYYYMMDD.csv
  → SAVE: reports/figures/problem-statement-004/expenditure_correlation_heatmap_*.png
  → SAVE: reports/figures/problem-statement-004/expenditure_scatter_*.png
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `expenditure_correlations.py`

```python
"""
Expenditure Correlation & Regression Analysis
==============================================

Quantifies statistical relationships between Singapore government health
expendhiture and key drivers: utilization volume, population size, demographics.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from datetime import datetime
from pathlib import Path
from typing import Optional

import numpy as np
import polars as pl
from scipy import stats

logger = logging.getLogger(__name__)


class ExpenditureCorrelationAnalyzer:
    """
    Correlation and regression analysis for expenditure drivers.

    Example:
        >>> analyzer = ExpenditureCorrelationAnalyzer()
        >>> df_corr = analyzer.run()
    """

    DRIVER_COLUMNS = ["total_admissions", "population", "per_capita_expenditure",
                       "cost_per_admission", "yoy_expenditure_growth_pct"]

    def __init__(
        self,
        data_path: str = "shared/data/3_interim/expenditure_drivers_integrated.parquet",
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
    ) -> None:
        self.data_path = Path(data_path)
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)

        if not self.data_path.exists():
            raise FileNotFoundError(
                f"Integrated dataset not found: {self.data_path}\n"
                "Run PS-004-US-02 first."
            )
        self.df = (
            pl.read_parquet(str(self.data_path))
            .filter((pl.col("year") >= 2006) & (pl.col("year") <= 2018))
            .sort("year")
        )
        logger.info(f"Loaded {len(self.df)} rows for correlation analysis")

    def compute_correlations(self) -> pl.DataFrame:
        """
        Compute Pearson correlations and p-values between total_expenditure
        and each available driver column.

        Returns:
            DataFrame with columns: driver, pearson_r, p_value, significant_at_05,
            n_observations, interpretation.
        """
        exp = self.df["total_expenditure"].drop_nulls().to_numpy()
        records = []

        drivers = [c for c in self.DRIVER_COLUMNS if c in self.df.columns]
        if not drivers:
            logger.warning("No driver columns found in dataset")
            return pl.DataFrame()

        for driver in drivers:
            driver_series = self.df[driver].drop_nulls()
            exp_aligned = self.df.filter(
                pl.col(driver).is_not_null()
            )["total_expenditure"].to_numpy()
            drv_vals = driver_series.to_numpy()

            if len(drv_vals) < 3:
                logger.warning(f"Insufficient data for {driver} correlation")
                continue

            try:
                r, p = stats.pearsonr(exp_aligned, drv_vals)
                sig = p < 0.05
                interp = (
                    "strong positive" if r > 0.7 else
                    "moderate positive" if r > 0.4 else
                    "weak positive" if r > 0 else
                    "strong negative" if r < -0.7 else
                    "moderate negative" if r < -0.4 else
                    "weak negative"
                )
                records.append({
                    "driver": driver,
                    "pearson_r": round(r, 4),
                    "p_value": round(p, 6),
                    "significant_at_05": sig,
                    "n_observations": len(drv_vals),
                    "interpretation": interp,
                })
                logger.info(
                    f"Correlation {driver}: r={r:.3f}, p={p:.4f}, sig={sig}"
                )
            except Exception as exc:
                logger.warning(f"Could not compute correlation for {driver}: {exc}")

        result = pl.DataFrame(records).sort("pearson_r", descending=True)
        return result

    def time_lagged_correlations(self, lag: int = 1) -> pl.DataFrame:
        """
        Test if utilization at t-lag predicts expenditure at t.

        Args:
            lag: Number of years to lag driver variables.

        Returns:
            DataFrame with lagged correlation results.
        """
        records = []
        drivers = [c for c in self.DRIVER_COLUMNS if c in self.df.columns]

        for driver in drivers:
            df_lag = self.df.with_columns(
                pl.col(driver).shift(lag).alias(f"{driver}_lag{lag}")
            ).drop_nulls(subset=[f"{driver}_lag{lag}", "total_expenditure"])

            if len(df_lag) < 3:
                continue

            try:
                r, p = stats.pearsonr(
                    df_lag["total_expenditure"].to_numpy(),
                    df_lag[f"{driver}_lag{lag}"].to_numpy()
                )
                records.append({
                    "driver": driver,
                    "lag_years": lag,
                    "pearson_r_lagged": round(r, 4),
                    "p_value_lagged": round(p, 6),
                    "significant_at_05": p < 0.05,
                })
            except Exception as exc:
                logger.warning(f"Lagged correlation failed for {driver}: {exc}")

        return pl.DataFrame(records)

    def run_ols_regression(self) -> dict:
        """
        OLS regression: total_expenditure ~ available driver columns.

        Returns:
            Dict with R2, coefficients, p-values, and model summary.
        """
        try:
            import statsmodels.api as sm
        except ImportError:
            logger.warning("statsmodels not installed — skipping OLS regression")
            return {}

        available_drivers = [
            c for c in ["total_admissions", "population"]
            if c in self.df.columns
        ]
        if not available_drivers:
            logger.warning("No OLS driver columns available")
            return {}

        df_ols = self.df.drop_nulls(
            subset=["total_expenditure"] + available_drivers
        )
        if len(df_ols) < len(available_drivers) + 2:
            logger.warning("Insufficient observations for OLS regression")
            return {}

        X = df_ols.select(available_drivers).to_pandas()
        y = df_ols["total_expenditure"].to_numpy()
        X = sm.add_constant(X)
        model = sm.OLS(y, X).fit()

        result = {
            "r_squared": round(model.rsquared, 4),
            "adj_r_squared": round(model.rsquared_adj, 4),
            "n_observations": int(model.nobs),
            "f_pvalue": round(model.f_pvalue, 6),
            "coefficients": {
                k: {"coef": round(v, 6), "p_value": round(model.pvalues[k], 6)}
                for k, v in model.params.items()
            },
        }
        logger.info(
            f"OLS: R²={result['r_squared']}, adj-R²={result['adj_r_squared']}, "
            f"n={result['n_observations']}"
        )
        return result

    def compute_elasticities(self) -> dict:
        """
        Estimate log-log elasticities: ln(expenditure) ~ ln(driver).

        Elasticity = regression coefficient in log-log model.
        E.g., coef=1.2 → 10% increase in driver → 12% increase in expenditure.

        Returns:
            Dict mapping driver_name → elasticity estimate.
        """
        elasticities = {}
        drivers = [c for c in ["total_admissions", "population"] if c in self.df.columns]

        df_pos = self.df.filter(
            (pl.col("total_expenditure") > 0)
        ).drop_nulls(subset=["total_expenditure"] + drivers)

        for driver in drivers:
            df_drv = df_pos.filter(pl.col(driver) > 0)
            if len(df_drv) < 4:
                continue
            log_y = np.log(df_drv["total_expenditure"].to_numpy())
            log_x = np.log(df_drv[driver].to_numpy())
            slope, intercept, r, p, se = stats.linregress(log_x, log_y)
            elasticities[driver] = {
                "elasticity": round(slope, 4),
                "r_squared": round(r ** 2, 4),
                "p_value": round(p, 6),
                "interpretation": (
                    f"10% rise in {driver} → {abs(slope)*10:.1f}% "
                    f"{'rise' if slope > 0 else 'fall'} in expenditure"
                ),
            }
            logger.info(
                f"Elasticity {driver}: {slope:.4f} (p={p:.4f})"
            )

        return elasticities

    def save_results(
        self, df_corr: pl.DataFrame, df_lagged: pl.DataFrame
    ) -> None:
        """Persist correlation and lagged correlation CSVs."""
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        corr_path = self.results_dir / f"expenditure_correlation_analysis_{ts}.csv"
        lag_path = self.results_dir / f"expenditure_lagged_correlations_{ts}.csv"
        df_corr.write_csv(str(corr_path))
        df_lagged.write_csv(str(lag_path))
        logger.info(f"✓ Correlations saved: {corr_path}")
        logger.info(f"✓ Lagged correlations saved: {lag_path}")
        print(f"✓ Results saved: {corr_path}")
        print(f"✓ Results saved: {lag_path}")

    def run(self) -> pl.DataFrame:
        """Full pipeline: correlations → lagged → OLS → elasticities → save."""
        df_corr = self.compute_correlations()
        df_lagged = self.time_lagged_correlations(lag=1)
        ols = self.run_ols_regression()
        elasticities = self.compute_elasticities()
        self.save_results(df_corr, df_lagged)
        logger.info("OLS summary: %s", ols)
        logger.info("Elasticities: %s", elasticities)
        return df_corr
```

#### 6.3 Validation Rules

```python
CORRELATION_VALIDATION_RULES = {
    "min_observations_for_pearson": 3,
    "significance_alpha": 0.05,
    "strong_correlation_threshold": 0.70,
    "min_obs_for_ols": 5,
}
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 scipy>=1.11.0 statsmodels>=0.14.0 matplotlib>=3.8.0 seaborn>=0.13.0
uv pip freeze > requirements.txt
```

---

### 10. Testing Strategy

```python
# ps-004/tests/unit/test_expenditure_correlations.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.expenditure_correlations import (
    ExpenditureCorrelationAnalyzer,
)


@pytest.fixture
def sample_integrated(tmp_path: Path) -> ExpenditureCorrelationAnalyzer:
    df = pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 250 for i in range(13)],
        "total_admissions": [200_000.0 + i * 5_000 for i in range(13)],
        "population": [4_500_000.0 + i * 55_000 for i in range(13)],
    })
    p = tmp_path / "integrated.parquet"
    df.write_parquet(str(p))
    return ExpenditureCorrelationAnalyzer(
        data_path=str(p),
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
    )


def test_compute_correlations_returns_dataframe(sample_integrated: ExpenditureCorrelationAnalyzer) -> None:
    df = sample_integrated.compute_correlations()
    assert isinstance(df, pl.DataFrame)
    assert "pearson_r" in df.columns
    assert "p_value" in df.columns


def test_admissions_correlation_strong_positive(sample_integrated: ExpenditureCorrelationAnalyzer) -> None:
    """Perfectly increasing data → r ≈ 1.0."""
    df = sample_integrated.compute_correlations()
    row = df.filter(pl.col("driver") == "total_admissions")
    assert len(row) > 0
    assert row["pearson_r"][0] > 0.95


def test_lagged_correlations_returns_dataframe(sample_integrated: ExpenditureCorrelationAnalyzer) -> None:
    df = sample_integrated.time_lagged_correlations(lag=1)
    assert isinstance(df, pl.DataFrame)
    assert "lag_years" in df.columns


def test_elasticity_positive_for_admissions(sample_integrated: ExpenditureCorrelationAnalyzer) -> None:
    elasticities = sample_integrated.compute_elasticities()
    assert "total_admissions" in elasticities
    assert elasticities["total_admissions"]["elasticity"] > 0


def test_save_creates_files(sample_integrated: ExpenditureCorrelationAnalyzer) -> None:
    df_corr = sample_integrated.compute_correlations()
    df_lagged = sample_integrated.time_lagged_correlations()
    sample_integrated.save_results(df_corr, df_lagged)
    files = list(Path(sample_integrated.results_dir).glob("*.csv"))
    assert len(files) >= 2
```

---

### 11. Implementation Steps

#### Phase 1: Setup
- [ ] Confirm integrated Parquet has `total_admissions` and/or `population` columns populated
- [ ] List all numeric columns available; log which drivers can be tested

#### Phase 2: Correlation Analysis
- [ ] Implement `compute_correlations()` and run
- [ ] Identify strongest correlates (r > 0.7)
- [ ] Run `time_lagged_correlations(lag=1)` — does prior-year utilization predict current expenditure?

#### Phase 3: Regression & Elasticities
- [ ] Run `run_ols_regression()` — log R² and coefficient p-values
- [ ] Run `compute_elasticities()` — log elasticity for admissions
- [ ] Verify statistical significance (p < 0.05 for key predictors)

#### Phase 4: Visualisation
- [ ] Seaborn correlation heatmap → `expenditure_correlation_heatmap_*.png`
- [ ] Scatter plots: expenditure vs admissions, expenditure vs population → `expenditure_scatter_*.png`
- [ ] Regression lines with 95% confidence bands

#### Phase 5: Testing & Notebook
- [ ] Write unit tests; run `pytest` ≥80% coverage
- [ ] Create `notebooks/05-correlate-expenditure-utilization.ipynb`
- [ ] Save `results/tables/problem-statement-004/expenditure_correlation_analysis_*.csv`
- [ ] Save `results/tables/problem-statement-004/expenditure_driver_regression_*.csv`

---

### 12. Adaptive Implementation Strategy

- **If correlation is low (r < 0.3)** → examine whether confounding time-trend inflates/deflates; apply detrending (first differences) before re-running
- **If OLS p-values > 0.05 for all drivers** → n=13 is small; report with caution; rely on correlation direction and domain knowledge
- **If `statsmodels` not installed** → fall back to `scipy.stats.linregress` for simple OLS; log warning
- **If admissions data absent** → skip volume correlation; proceed with population only; document gap

---

### 21. Version Control

```
Branch: feature/ps-004-us-05-correlate-expenditure
Commits:
  feat(ps-004): add ExpenditureCorrelationAnalyzer with Pearson and OLS
  feat(ps-004): log-log elasticity estimation for expenditure drivers
  test(ps-004): unit tests for correlation and regression analysis
```
