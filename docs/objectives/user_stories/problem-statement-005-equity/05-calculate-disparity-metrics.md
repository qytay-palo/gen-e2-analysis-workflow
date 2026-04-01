# Calculate Health Equity Disparity Metrics (Lifecycle Stage: Feature Engineering)

**Story ID**: PS-005-US-05  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Population Health Equity Analyst**,  
I want **to calculate standardized disparity metrics (disparity ratios, concentration indices, Gini coefficients) comparing health outcomes and utilization across demographic groups**,  
So that **I can quantify the magnitude of health inequities and track progress toward equity goals using internationally recognized methodologies**.

---

## 🎯 Acceptance Criteria

1. **Disparity ratios calculated**
   - Utilization disparity ratio: `(utilization_group_A / utilization_group_B)` 
   - Mortality disparity ratio: `(mortality_rate_group_A / mortality_rate_group_B)`
   - Reference group selection documented (e.g., lowest-risk age group, male vs female)
   - Ratios calculated for all demographic comparisons (age groups, gender)

2. **Concentration indices computed**
   - Concentration index: measures inequality in health variable distribution across socioeconomic/demographic ranking
   - Formula: `C = (2/μ) * Cov(health_variable, fractional_rank) / n`
   - Range: -1 to +1 (0 = perfect equality, positive = pro-rich/advantaged, negative = pro-poor/disadvantaged)
   - Calculated for key outcomes: mortality, utilization, disease prevalence

3. **Statistical significance tested**
   - Confidence intervals calculated for disparity ratios (bootstrap or analytical methods)
   - Test if disparity ratio significantly different from 1.0 (equity)
   - Test if concentration index significantly different from 0 (equity)

4. **Data output requirement**
   - Output file: `results/tables/equity_disparity_metrics.csv`
   - Format: CSV (metric_type, demographic_comparison, disparity_ratio, concentration_index, ci_lower, ci_upper, p_value, interpretation)
   - Equity visualization: `reports/figures/disparity_metrics_forest_plot.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Statistics**: scipy.stats for bootstrap, significance tests
- **Logging**: loguru
- **Testing**: pytest with ≥80% coverage

---

## 📚 Domain Knowledge References

- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objectives) - Objective 3: Diagnose inequities with standardized metrics
- Health equity literature: WHO Health Equity Assessment Toolkit (HEAT), Concentration Index methodology

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data processing
- `scipy>=1.11.0`: Statistical tests, bootstrap
- `matplotlib>=3.8.0`: Forest plots
- `loguru>=0.7.0`: Logging

### Internal Dependencies
- **Upstream**: 
  - PS-005-US-03 (Utilization disparities - BLOCKING)
  - PS-005-US-04 (Outcome disparities - BLOCKING)
- **Data Sources**: 
  - `shared/data/3_interim/equity_analysis_integrated.parquet`
- **Config Files**: `config/analysis.yml` (reference groups, significance thresholds)

---

## ✅ Implementation Tasks

### Reference Group Selection
- [ ] Define reference groups for disparity ratios:
  - Age: 25-44 years (prime working age, typically lowest risk)
  - Gender: Male (standard in epidemiology, though arbitrary)
  - Document rationale for reference group choice

### Disparity Ratio Calculation
- [ ] Calculate utilization disparity ratios:
  - For each age group vs reference: `ratio = utilization_age_i / utilization_age_ref`
  - For gender: `ratio = utilization_female / utilization_male`
  
- [ ] Calculate mortality disparity ratios:
  - Age-specific: Compare each age group to reference
  - Gender: Female vs male mortality rates
  
- [ ] Interpret ratios:
  - Ratio = 1.0: Equity (no disparity)
  - Ratio > 1.0: Group has higher utilization/mortality than reference
  - Ratio < 1.0: Group has lower utilization/mortality than reference

### Concentration Index Calculation
- [ ] Rank population by socioeconomic proxy (age as proxy, or use admission rate as health need ranking)
- [ ] Calculate fractional rank: `r_i = (2i - 1) / (2n)` where i is rank position
- [ ] Calculate concentration index: `C = (2/μ) * Σ[(y_i - μ) * r_i] / n`
  - Where μ = mean health variable, y_i = individual health outcome
  
- [ ] Alternatively, use regression method:
  - Regress health variable on fractional rank
  - C = β * (2σ_r / μ)

### Gini Coefficient (Optional)
- [ ] Calculate Gini coefficient for utilization distribution across groups
- [ ] Formula: `G = (Σ Σ |x_i - x_j|) / (2n² * mean(x))`
- [ ] Interpret: 0 = perfect equality, 1 = perfect inequality

### Statistical Significance
- [ ] Bootstrap confidence intervals:
  - Resample data 1,000 times with replacement
  - Calculate disparity ratio for each bootstrap sample
  - 95% CI: 2.5th and 97.5th percentiles
  
- [ ] Test significance:
  - Disparity ratio: CI excludes 1.0 → significant disparity
  - Concentration index: t-test if C significantly different from 0

### Interpretation & Classification
- [ ] Classify disparity magnitude:
  - Minimal: ratio 0.8-1.2 (±20%)
  - Moderate: ratio 0.5-0.8 or 1.2-2.0
  - Large: ratio <0.5 or >2.0
  
- [ ] Priority ranking: rank demographic comparisons by disparity magnitude

### Visualization
- [ ] Forest plot: disparity ratios with 95% CI error bars
- [ ] Concentration curves: cumulative health vs cumulative population rank
- [ ] Equity gap charts: bar chart showing gap magnitude per demographic
- [ ] Export figures

### Testing & Validation
- [ ] Unit tests for disparity ratio calculation
- [ ] Validate concentration index: compare against published examples
- [ ] Test bootstrap: ensure CI width reasonable
- [ ] Test edge cases: perfect equality, extreme inequality

### Documentation
- [ ] Docstrings (Google style)
- [ ] Methodology document: `results/equity_metrics_methodology.md`
  - Reference group selection rationale
  - Concentration index calculation method
  - Bootstrap procedure details
  - Interpretation guidelines
- [ ] Equity metrics glossary: explain metrics to non-technical stakeholders
- [ ] Update data dictionary

---

## 📌 Notes

**Disparity Ratio Calculation (Polars)**:
```python
import polars as pl

# Reference group: age 25-44
ref_utilization = df.filter(pl.col('age_group') == '25-44').select('utilization_rate').mean()

# Calculate ratios for all age groups
df_disparity = (
    df.group_by('age_group').agg([
        pl.col('utilization_rate').mean().alias('mean_utilization')
    ])
    .with_columns([
        (pl.col('mean_utilization') / ref_utilization).alias('disparity_ratio')
    ])
)
```

**Concentration Index (Python)**:
```python
import numpy as np
import polars as pl

# Prepare data
df_sorted = df.sort('socioeconomic_rank')  # or age as proxy
y = df_sorted['health_outcome'].to_numpy()
n = len(y)
mu = y.mean()

# Fractional rank
r = np.array([(2*i - 1) / (2*n) for i in range(1, n+1)])

# Concentration index
C = (2 / mu) * np.sum((y - mu) * r) / n

logger.info(f"Concentration Index: {C:.4f}")
# Interpretation: C>0 (pro-rich), C<0 (pro-poor), C≈0 (equity)
```

**Bootstrap Confidence Intervals**:
```python
from scipy.stats import bootstrap
import numpy as np

def disparity_ratio(sample):
    group_a = sample[sample['group'] == 'A']['outcome'].mean()
    group_b = sample[sample['group'] == 'B']['outcome'].mean()
    return group_a / group_b

# Bootstrap
rng = np.random.default_rng()
bootstrap_result = bootstrap(
    (df,), 
    disparity_ratio, 
    n_resamples=1000, 
    confidence_level=0.95,
    random_state=rng
)

ci_lower, ci_upper = bootstrap_result.confidence_interval
logger.info(f"Disparity Ratio: {ratio:.2f} (95% CI: {ci_lower:.2f}-{ci_upper:.2f})")
```

**Expected Metrics** (hypotheses):
- **Age disparities**: Elderly (65+) likely have 3-5x higher utilization than 25-44 (expected due to health needs)
- **Gender disparities**: Women may have 1.2-1.5x higher primary care utilization (reproductive health, longer life expectancy)
- **Concentration index**: Likely near 0 for Singapore (universal healthcare system promotes equity)

**Interpretation Guidelines**:
| Metric | Value | Interpretation | Action |
|--------|-------|----------------|--------|
| Disparity Ratio | 0.8-1.2 | Minimal disparity | Monitor |
| Disparity Ratio | 1.2-2.0 or 0.5-0.8 | Moderate disparity | Investigate causes |
| Disparity Ratio | >2.0 or <0.5 | Large disparity | Priority intervention target |
| Concentration Index | -0.2 to +0.2 | Relatively equitable | Maintain |
| Concentration Index | >0.2 or <-0.2 | Significant inequality | Policy review needed |

**Limitations**:
- Limited socioeconomic stratification in data (may miss income/education disparities)
- Age as proxy for socioeconomic rank is imperfect
- National-level analysis misses geographic inequities
- Some disparities may be clinically justified (e.g., elderly having higher utilization due to greater health needs)
