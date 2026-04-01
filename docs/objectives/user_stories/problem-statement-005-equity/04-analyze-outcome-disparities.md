# Analyze Health Outcome Disparities (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-005-US-04  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Population Health Strategist evaluating outcome equity**,  
I want **to assess health outcome disparities (mortality, disease burden) across demographic groups**,  
So that **I can identify populations with worse health outcomes and link outcome inequities to potential access or care quality disparities**.

---

## 🎯 Acceptance Criteria

1. **Outcome disparities quantified**
   - Mortality rate ratios by demographics
   - Disease burden (DALY) disparities if data available
   - Standardized mortality ratios (SMR)
   - Statistical significance testing

2. **Demographic outcome patterns**
   - High-mortality demographics identified
   - Outcome gaps vs reference groups
   - Age-sex-specific outcome disparities
   - Disease-specific outcome equity

3. **Utilization-outcome linkage**
   - Correlation: utilization disparities vs outcome disparities
   - Paradoxes identified: high utilization but poor outcomes (quality issue?)
   - Access gaps: low utilization AND poor outcomes (access barrier)

4. **Deliverables**
   - Output: `results/tables/outcome_disparity_analysis.csv`
   - Figures: Outcome disparity charts
   - Report: Health outcome equity assessment

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Statistical Analysis**: scipy
- **Visualization**: Matplotlib
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#outcome-disparity-metrics)
- [Disease Burden Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#standardized-mortality-ratio-smr)
- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objective-2)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scipy>=1.11.0`, `matplotlib>=3.8.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-005-US-03 (Utilization disparities - parallel OK)
- **Data Sources**: `shared/data/3_interim/equity_analysis_integrated.parquet`

---

## ✅ Implementation Tasks

### Outcome Disparity Calculation
- [ ] Calculate mortality rate ratios
- [ ] Calculate SMRs (observed/expected deaths)
- [ ] Disease burden disparities (if DALY data available)
- [ ] Statistical significance tests

### Pattern Analysis
- [ ] Identify high-mortality demographics
- [ ] Quantify outcome gaps vs reference groups
- [ ] Age-sex-disease interaction analysis

### Utilization-Outcome Linkage
- [ ] Correlate utilization disparities with outcome disparities
- [ ] Identify quality paradoxes (high use, poor outcomes)
- [ ] Identify access gaps (low use, poor outcomes)

### Visualization
- [ ] Outcome disparity charts
- [ ] 2×2 matrix: utilization vs outcomes
- [ ] Save figures

### Testing & Documentation
- [ ] Unit tests
- [ ] Docstrings
- [ ] Outcome equity report

---

## 📌 Notes

**SMR Calculation**:
```python
SMR = (Observed deaths / Expected deaths) × 100
```

**Utilization-Outcome Matrix**:
- High utilization + Good outcomes: Effective care
- High utilization + Poor outcomes: Quality concerns
- Low utilization + Good outcomes: Healthy population
- Low utilization + Poor outcomes: **Access barrier** (priority intervention)
