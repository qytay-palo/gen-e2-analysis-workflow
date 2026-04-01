# Prepare & Integrate Equity Analysis Datasets (Lifecycle Stage: Data Preparation)

**Story ID**: PS-005-US-02  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer preparing health equity datasets**,  
I want **to integrate and standardize demographic health data with consistent demographic categorizations and unified schemas**,  
So that **equity analysts can perform disparity assessments comparing utilization and outcomes across demographic groups**.

---

## 🎯 Acceptance Criteria

1. **Schema standardization**
   - Unified schema: `year`, `demographic_group`, `age_group`, `sex`, `metric_type`, `metric_value`
   - Demographic categories standardized
   - Data types enforced

2. **Data integration**
   - Utilization and outcome data integrated
   - Common demographic groups aligned
   - Time periods aligned

3. **Derived metrics**
   - Rate ratios: high-use group / low-use group
   - Disparity indices calculated
   - Reference group defined (e.g., overall population average)

4. **Output**
   - Integrated: `shared/data/3_interim/equity_analysis_integrated.parquet`
   - Schema: `shared/data/schemas/equity_integrated_schema.yml`
   - Test coverage: ≥80%

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#equity-analysis-methods)
- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objective-1)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-005-US-01 (Extract equity data - BLOCKING)
- **Data Sources**: `shared/data/1_raw/equity/**/*.csv`

---

## ✅ Implementation Tasks

### Schema Standardization
- [ ] Define unified demographic schema
- [ ] Standardize age group categories
- [ ] Standardize sex categories
- [ ] Map source data to standard schema

### Data Integration
- [ ] Load all demographic health datasets
- [ ] Align demographic categories
- [ ] Join utilization and outcome data
- [ ] Handle missing demographic combinations

### Derived Metrics
- [ ] Calculate reference group averages
- [ ] Calculate rate ratios for each demographic
- [ ] Compute absolute disparities
- [ ] Flag significant disparities

### Testing & Documentation
- [ ] Unit tests
- [ ] Validation tests
- [ ] Docstrings

---

## 📌 Notes

**Polars Integration**:
```python
import polars as pl

df_util = pl.read_csv("shared/data/1_raw/equity/utilization/admissions.csv")

# Calculate rate ratios
df_equity = (
    df_util.with_columns([
        pl.col('admission_rate').mean().alias('reference_rate')
    ])
    .with_columns([
        (pl.col('admission_rate') / pl.col('reference_rate')).alias('rate_ratio')
    ])
)

# Flag disparities (rate ratio >1.5 or <0.67)
df_equity = df_equity.with_columns([
    ((pl.col('rate_ratio') > 1.5) | (pl.col('rate_ratio') < 0.67)).alias('disparity_flag')
])
```
