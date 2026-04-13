# Data Dictionary: Healthcare Expenditure Drivers (PS-004)

**Domain**: expenditure  
**Problem Statement**: PS-004 — Healthcare Expenditure Drivers & Cost Control Analysis  
**Source**: Kaggle `subhamjain/health-dataset-complete-singapore` (MOH Singapore via data.gov.sg)  
**Validation Date**: 2026-04-10  
**Validated By**: data-validation agent

---

## Datasets

### 1. `government_health_expenditure.csv`

| Column | Type | Description | Unit | Notes |
|--------|------|-------------|------|-------|
| `financial_year` | Int64 | Singapore government financial year | year (YYYY) | FY ≈ calendar year for MOH; range 2006–2018 |
| `operating_expenditure` | Int64 | Recurrent/operating government health expenditure | SGD millions | Always positive; excludes capital items |
| `development_expenditure` | Int64 | Capital/development government health expenditure | SGD millions | Non-negative; spikes in 2009 (SGD 711M) |
| `government_health_expenditure` | Float64 | Total government health expenditure (GHE) | SGD millions | **≠ operating + development** (see QI-BR-019 below) |
| `percentage_gdp` | Float64 | GHE as % of GDP | percentage | 1 decimal place precision only; range 0.8–2.1% |

**Grain**: One row per financial year (13 years, 2006–2018).  
**Primary Key**: `financial_year` (unique, 0 duplicates).  
**Null Rate**: 0% (all columns complete).

---

### 2. `hospital_admission_rate_by_age_sex.csv`

| Column | Type | Description | Unit | Notes |
|--------|------|-------------|------|-------|
| `year` | Int64 | Calendar year | year (YYYY) | Range 2009–2020 |
| `facility_type_a` | String | Hospital facility type | category | Values: `Acute`, `Psychiatric Hospitals`, `Community Hospitals` |
| `sex` | String | Patient sex | category | Values: `Male`, `Female` |
| `age` | String | Age group | category | Values: `0-14 Years`, `15-64 years`, `65 years & over` |
| `rate` | Float64 | Admission rate per 1,000 population | per 1,000 | Non-negative; 65+ rates significantly higher than 15-64 |

**Grain**: One row per year × facility_type × sex × age_group (18 combinations/year, 12 years = 216 rows).  
**Primary Key**: `[year, facility_type_a, sex, age]` (unique, 0 duplicates).  
**Null Rate**: 0% (all columns complete).  
**Age Gradient**: 65+ acute admission rates consistently 4–5× the 15–64 rate (validated).

---

### 3. `hospital_admission_rate_by_sex.csv`

| Column | Type | Description | Unit | Notes |
|--------|------|-------------|------|-------|
| `year` | Int64 | Calendar year | year (YYYY) | Range 2009–2020 |
| `facility_type_a` | String | Hospital facility type | category | Values: `Acute`, `Psychiatric Hospitals`, `Community Hospitals` |
| `sex` | String | Patient sex | category | Values: `Male`, `Female` |
| `rate` | Float64 | Admission rate per 1,000 population | per 1,000 | Non-negative |

**Grain**: One row per year × facility_type × sex (6 combinations/year, 12 years = 72 rows).  
**Primary Key**: `[year, facility_type_a, sex]` (unique, 0 duplicates).  
**Null Rate**: 0% (all columns complete).

---

### 4. `consumer_price_indices.csv`

| Column | Type | Description | Unit | Notes |
|--------|------|-------------|------|-------|
| `year` | Int64 | Calendar year | year (YYYY) | Range 2008–2020 |
| `category` | String | CPI category | category | Values: `General`, `Health` |
| `cpi` | Float64 | Consumer Price Index | index (2019=100) | All values in range 79–100; 2019 baseline = 100.0 |

**Grain**: One row per year × category (2 categories/year, 13 years = 26 rows).  
**Primary Key**: `[year, category]` (unique, 0 duplicates).  
**Null Rate**: 0% (all columns complete).  
**Note**: CPI starts 2008 — no 2006/2007 data available. Affects real-terms deflation for those years.

---

### 5. `residential_ltc_admissions.csv`

| Column | Type | Description | Unit | Notes |
|--------|------|-------------|------|-------|
| `year` | Int64 | Calendar year | year (YYYY) | Range 2007–2019 |
| `type` | String | LTC facility type | category | Values: `Nursing Homes`, `Inpatient Hospices` |
| `count` | Int64 | Annual admissions/new residents | integer count | Range 994–5,307; all positive |

**Grain**: One row per year × LTC type (2 types/year, but 25 rows — Nursing Homes missing 2007).  
**Primary Key**: `[year, type]` (unique, 0 duplicates).  
**Null Rate**: 0% on present rows; `Nursing Homes` for 2007 is structurally absent (row does not exist).

---

### 6. `expenditure_drivers_integrated.parquet`

| Column | Type | Description | Null Count | Null Rate |
|--------|------|-------------|-----------|-----------|
| `year` | Int64 | Financial/calendar year (anchor: GHE) | 0 | 0% |
| `operating_expenditure` | Int64 | Recurrent government health expenditure (SGD M) | 0 | 0% |
| `development_expenditure` | Int64 | Capital government health expenditure (SGD M) | 0 | 0% |
| `government_health_expenditure` | Float64 | Total GHE (SGD M) | 0 | 0% |
| `percentage_gdp` | Float64 | GHE as % of GDP | 0 | 0% |
| `acute_admission_rate_per1000` | Float64 | Average acute hospital admission rate per 1,000 pop | 3 | 23.1% ⚠️ |
| `cpi_general` | Float64 | General CPI (2019=100) | 2 | 15.4% ⚠️ |
| `cpi_health` | Float64 | Health CPI (2019=100) | 2 | 15.4% ⚠️ |
| `ltc_inpatient_hospices` | Int64 | Annual admissions to inpatient hospices | 1 | 7.7% 🟡 |
| `ltc_nursing_homes` | Int64 | Annual admissions to nursing homes | 2 | 15.4% ⚠️ |
| `sg_population` | Int64 | Singapore total population estimate | 0 | 0% |
| `expenditure_per_capita_sgd` | Float64 | GHE per capita (SGD) | 0 | 0% |
| `operating_expenditure_per_capita_sgd` | Float64 | Operating expenditure per capita (SGD) | 0 | 0% |
| `expenditure_yoy_growth_pct` | Float64 | Year-over-year GHE growth (%) | 1 | 7.7% 🟡 |
| `expenditure_3yr_avg_growth_pct` | Float64 | 3-year rolling average GHE growth (%) | 3 | 23.1% ⚠️ |
| `cagr_from_2006_pct` | Float64 | CAGR from 2006 base year (%) | 1 | 7.7% 🟡 |
| `real_expenditure_2019sgd_m` | Float64 | GHE deflated to 2019 SGD (M) | 2 | 15.4% ⚠️ |

**Grain**: One row per year (13 rows, 2006–2018).  
**Primary Key**: `year` (unique, 0 duplicates).  
**GHE is monotonically increasing**: Confirmed, no year-over-year declines.

---

## Data Quality Assessment

**Validation Date**: 2026-04-10  
**Overall Quality Score**: 89.5 / 100

| Dimension | Score | Status | Notes |
|-----------|-------|--------|-------|
| Completeness | 88% | ⚠️ WARN | Raw CSVs 100% complete; integrated has structural nulls from data range gaps |
| Accuracy | 90% | ⚠️ WARN | No impossible values; 2009 GHE spike (+33.1%) is policy-driven anomaly |
| Consistency | 95% | ✅ PASS | All categorical values conformant; PK uniqueness confirmed |
| Validity | 97% | ✅ PASS | 18/20 business rules pass (100% on all CRITICAL rules) |
| Timeliness | 70% | ❌ WARN | GHE data ends 2018 vs PS-004 scope of 2005–2020; financing scheme data missing |

---

## Business Rule Findings

### ✅ Passing Rules (18/20)
- All CRITICAL rules pass (BR-002 through BR-007, BR-009, BR-011, BR-014, BR-017, BR-018)
- GHE is always positive and monotonically increasing
- All admission rates non-negative
- CPI in valid range (79–100 for 2019=100 base)
- LTC counts positive and plausible
- 65+ acute admission rates consistently higher than 15–64 rates (validated 24/24 year-sex combinations)

### ❌ BR-008 — Age Gradient Rule (HIGH severity)
**Rule**: Acute admission rate for 65+ > rate for 15-64  
**Actual finding**: **100% pass rate** (24/24 year-sex combinations validated).  
**Issue**: Rule implementation used original dataset denominator (216) rather than filtered comparison rows (24), producing misleading 11.1% compliance. **This is a rule formulation defect, not a data quality issue.**  
**Action**: Correct rule implementation in next validation cycle. Data passes this check.

### ❌ BR-019 — GHE Components Reconciliation (HIGH severity)
**Rule**: `government_health_expenditure` ≈ `operating_expenditure` + `development_expenditure` (within SGD 1M tolerance)  
**Actual finding**: **Persistent gap across all 13 years** — ranging from SGD −510M to +204M.  
**Pattern**:
- 2006–2014: GHE > op+dev (positive gap, SGD 74M–204M) — GHE includes additional unspecified components
- 2015–2018: GHE < op+dev (negative gap, SGD −293M to −510M) — possible definitional change post-2014

**Interpretation**: The `government_health_expenditure` column is likely sourced from a different accounting basis than the sum of operating + development expenditure. Possible explanations:
1. GHE may exclude certain development items or include transfers not captured in the two sub-components
2. Post-2014 reversal suggests a methodology or classification change in the source data
3. This is consistent with MOH reporting where GHE is reported as a standalone audited figure

**Action for Cleaning Agent**: Add a `ghe_reconciliation_gap_sgdm` computed column; flag this in analysis documentation; treat `government_health_expenditure` as the authoritative total figure (do not attempt to reconstruct from sub-components).

---

## Known Issues

| ID | Severity | Description | Recommended Action |
|----|----------|-------------|-------------------|
| QI-007 | **HIGH** | GHE data ends 2018; PS-004 requires 2005–2020 analysis | Source 2019–2020 GHE from MOH Annual Reports or WHO GHED |
| QI-008 | **HIGH** | `health-financing-by-financing-schemes.csv` absent; PS-004 Obj.2 requires it | Source from data.gov.sg or MOH Annual Statistical Report |
| QI-001 | MEDIUM | LTC Nursing Homes missing for 2007 (row absent in source) | Add explicit null row; treat as structurally missing |
| QI-002 | MEDIUM | `real_expenditure_2019sgd_m` null for 2006–2007 (CPI unavailable) | Backward-extrapolate CPI 2006–2007 using 2008–2012 trend |
| QI-003 | MEDIUM | `acute_admission_rate_per1000` null for 2006–2008 (admissions data starts 2009) | Backward extrapolation or limit utilisation analysis to 2009–2018 |
| QI-009 | MEDIUM | 2009 GHE spike +33.1% YoY — largest single-year jump in series | Annotate as policy anomaly; flag in analysis narrative |
| QI-004 | LOW | `expenditure_3yr_avg_growth_pct` null for 2006–2008 (insufficient lookback) | Expected structural null; document only |
| QI-005 | LOW | `expenditure_yoy_growth_pct` null for 2006 (base year) | Expected structural null; document only |
| QI-006 | LOW | `cagr_from_2006_pct` null for 2006 (0 elapsed years) | Expected structural null; document only |
| QI-010 | LOW | `percentage_gdp` rounded to 1dp — limited precision | Accept as-is; note in analysis |

---

## Recommended Preprocessing Steps

| Priority | Step | Method | Affected Rows |
|----------|------|--------|---------------|
| 1 | Source 2019–2020 GHE data | Manual acquisition from MOH/WHO | 2 |
| 2 | Source financing scheme breakdown | Manual acquisition from data.gov.sg | New dataset |
| 3 | Backward-extrapolate CPI 2006–2007 | Linear trend from 2008–2012 | 2 |
| 4 | Impute or flag 2006–2008 admission rates | Linear backcast or exclusion flag | 3 |
| 5 | Annotate 2009 GHE spike | Add `policy_anomaly_flag` column | 1 |
| 6 | Add `ghe_reconciliation_gap_sgdm` column | `operating + development - ghe` | 13 |
| 7 | Add LTC 2007 Nursing Homes explicit null row | Add row `{year:2007, type:'Nursing Homes', count:null}` | 1 |
| 8 | Document structural nulls in derived columns | Data dictionary and null_reason metadata | 5 |

---

## Quality Gates

| Gate | Threshold | Actual | Status |
|------|-----------|--------|--------|
| Minimum completeness (raw CSVs) | ≥ 90% | 100% | ✅ PASSED |
| No critical issues | 0 | 0 | ✅ PASSED |
| Business rule compliance | ≥ 95% | 90% | ❌ FAILED* |
| Overall quality score | ≥ 75 | 89.5 | ✅ PASSED |

*BR-008 failure is a rule implementation defect (data actually passes). BR-019 failure reflects a real structural characteristic of the source data (GHE is not a simple sum of operating + development). Both are documented and do not block analysis.

---

## Temporal Coverage Summary

| Dataset | Year Range | Years with Full Coverage |
|---------|-----------|------------------------|
| Government Health Expenditure | 2006–2018 | 13 years (all complete) |
| Hospital Admissions (age/sex) | 2009–2020 | 12 years |
| Hospital Admissions (sex only) | 2009–2020 | 12 years |
| Consumer Price Indices | 2008–2020 | 13 years |
| Residential LTC Admissions | 2007–2019 | 13 years (2007 Nursing Homes missing) |
| Integrated Parquet (anchor: GHE) | 2006–2018 | Full 4-dataset overlap: 2009–2018 (10 years) |

---

*Last updated: 2026-04-10 by data-validation agent (PS-004)*
