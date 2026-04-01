# Extract Demographic Health Utilization & Outcome Data (Lifecycle Stage: Data Extraction)

**Story ID**: PS-005-US-01  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: S (2 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer supporting health equity analysis**,  
I want **to extract utilization and outcome data stratified by demographics (age, sex, ethnicity if available) from the Kaggle dataset**,  
So that **population health strategists have verified demographic health data for disparity analysis and equity assessments**.

---

## 🎯 Acceptance Criteria

1. **Demographic utilization data extracted**
   - Downloaded: `hospital-admission-rate-by-age-and-sex.csv` (216 records)
   - Demographic stratifications preserved: age groups, sex
   - Stored in `shared/data/1_raw/equity/utilization/`

2. **Health outcome data extracted**
   - Mortality data by demographics (if available with demographic breakdowns)
   - Disease burden data by demographics
   - Stored in `shared/data/1_raw/equity/outcomes/`

3. **Data validation passed**
   - Schema validation: demographic columns present
   - Completeness: 0% missing values
   - Demographic categories validated: age groups, sex categories consistent
   - Time coverage: 2006-2020

4. **Extraction documented**
   - Validation log: `logs/etl/equity_extraction_YYYYMMDD.log`
   - Schema: `shared/data/schemas/equity_raw_schema.yml`
   - Test coverage: ≥80%

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Extraction Method**: KaggleHub API
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#health-equity-metrics) - Equity and disparity metrics
- [Data Sources](../../../../project_context/data-sources.md#healthcare-utilization) - Demographic health data specifications

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `kagglehub>=0.2.0`, `loguru>=0.7.0`, `pyyaml>=6.0`

### Internal Dependencies
- **Upstream**: None (first story in epic)
- **Data Sources**: Kaggle dataset
- **Config Files**: `config/databricks.yml`, `shared/config/base.yml`

---

## ✅ Implementation Tasks

### Data Extraction
- [ ] Configure KaggleHub authentication
- [ ] Create extraction script: `shared/src/data_processing/extract_equity_data.py`
- [ ] Extract hospital admission rate by age and sex
- [ ] Extract mortality data with demographic breakdowns (if available)
- [ ] Save to appropriate directories

### Validation
- [ ] Define schemas for demographic health data
- [ ] Validate demographic stratifications present
- [ ] Validate completeness
- [ ] Validate value ranges

### Testing & Documentation
- [ ] Unit tests
- [ ] Integration tests
- [ ] Docstrings
- [ ] Update README

---

## 📌 Notes

**Expected Tables**:
- `hospital-admission-rate-by-age-and-sex.csv` (216 records, 2006-2020)
- Potentially: disease-specific mortality by demographics

**Polars Extraction**:
```python
import polars as pl
from loguru import logger
import kagglehub

dataset_path = kagglehub.dataset_download(
    "subhamjain/health-dataset-complete-singapore"
)

df_admissions = pl.read_csv(
    f"{dataset_path}/hospital-admission-rate-by-age-and-sex/hospital-admission-rate-by-age-and-sex.csv"
)

assert df_admissions.null_count().sum_horizontal()[0] == 0
logger.info(f"✓ Demographic admissions extracted: {len(df_admissions)} records")
```
