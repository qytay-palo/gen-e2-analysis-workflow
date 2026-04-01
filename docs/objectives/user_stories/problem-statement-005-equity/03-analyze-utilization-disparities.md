# Analyze Healthcare Utilization Disparities (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-005-US-03  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Population Health Strategist assessing healthcare access equity**,  
I want **to quantify utilization disparities across demographic groups (age, sex) to identify underserved and overserved populations**,  
So that **I can recommend targeted interventions to reduce access barriers and improve healthcare equity**.

---

## 🎯 Acceptance Criteria

1. **Disparities quantified**
   - Rate ratios calculated: demographic group rate / reference rate
   - Absolute disparities: rate differences
   - Disparity magnitude rankings
   - Statistical significance testing

2. **Demographic patterns identified**
   - High-utilization demographics
   - Low-utilization (underserved) demographics
   - Age-sex interaction effects
   - Temporal trends in disparities

3. **Concentration metrics**
   - Lorenz curves: utilization concentration
   - Gini coefficient: inequality measure
   - Theil index: disparity decomposition

4. **Deliverables**
   - Output: `results/tables/utilization_disparity_analysis.csv`
   - Figures: Disparity charts, Lorenz curves
   - Report: Utilization equity assessment

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Statistical Analysis**: scipy, statsmodels
- **Visualization**: Matplotlib/Seaborn
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#health-equity-disparity-metrics)
- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objective-1)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scipy>=1.11.0`, `matplotlib>=3.8.0`, `seaborn>=0.13.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-005-US-02 (Equity data prep - BLOCKING)
- **Data Sources**: `shared/data/3_interim/equity_analysis_integrated.parquet`

---

## ✅ Implementation Tasks

### Disparity Calculation
- [ ] Calculate rate ratios for each demographic group
- [ ] Calculate absolute disparities (rate differences)
- [ ] Rank demographics by disparity magnitude
- [ ] Statistical significance tests (chi-square, t-tests)

### Pattern Identification
- [ ] Identify high-utilization groups (rate ratio >1.5)
- [ ] Identify low-utilization groups (rate ratio <0.67)
- [ ] Analyze age-sex interactions
- [ ] Temporal trend analysis: disparities widening or narrowing?

### Concentration Analysis
- [ ] Calculate Lorenz curve coordinates
- [ ] Compute Gini coefficient
- [ ] Calculate Theil index
- [ ] Decompose inequality by demographic dimensions

### Visualization
- [ ] Rate ratio charts
- [ ] Lorenz curves
- [ ] Disparity heatmaps
- [ ] Save figures

### Testing & Documentation
- [ ] Unit tests
- [ ] Docstrings
- [ ] Equity assessment report

---

## 📌 Notes

**Rate Ratio Interpretation**:
- Rate ratio = 1.0: No disparity (equal utilization)
- Rate ratio > 1.5: High utilization (potential overuse or higher need)
- Rate ratio < 0.67: Low utilization (potential access barriers)

**Gini Coefficient**:
- 0 = Perfect equality
- 1 = Perfect inequality
- Healthcare typically Gini ~0.2-0.4
