# Equity Extracted Data Dictionary
## PS-005: Healthcare Access Equity & Demographic Disparities Analysis

**Created**: 2026-04-10  
**User Story**: PS-005-US-01  
**Source**: Ministry of Health Singapore via Kaggle (`subhamjain/health-dataset-complete-singapore`)  
**Data Mode**: Synthetic (schema-faithful, calibrated to MOH published statistics) — live Kaggle data used when network available  
**Extractor**: `shared/src/data_processing/extractors/equity_extractor.py`

---

## Overview

Six demographic health datasets extracted for equity disparity analysis covering Singapore's
healthcare utilization patterns (2006–2020) and mortality outcomes (1990–2019), stratified
by age, sex, and care type.

---

## Dataset 1: Hospital Admissions by Age & Sex

**File**: `shared/data/1_raw/equity/utilization/hospital-admission-rate-by-age-and-sex.csv`  
**Grain**: One row per (year, age_group, sex)  
**Rows**: 270 (synthetic) / 216 (documented Kaggle source)  
**Temporal Coverage**: 2006–2020 (15 years)  
**Primary Equity Dimensions**: Age group, Sex

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 2006–2020 | No nulls |
| `age_group` | Utf8 | 5-year age band | "0 - 4", "15 - 24", "75 & Over" | 9 categories |
| `sex` | Utf8 | Biological sex | "Male", "Female" | 2 categories |
| `admission_rate_per_1000` | Float32 | Hospital admissions per 1,000 resident population | 50.0–450.0 | Rate base: 1,000 |

### Key Observations

- **Elderly disparity**: 65-74 and 75+ cohorts show admission rates 3-6x higher than working-age adults
- **Gender pattern**: Female rates higher in reproductive age (15-34); male rates higher in older cohorts
- **Temporal trend**: Mild upward trend (~0.5% per year) consistent with Singapore's aging demographics
- **Primary equity use**: Calculate age-standardised rates, gender disparity ratios, temporal widening/narrowing

---

## Dataset 2: Subsidised Primary Care Attendances at Polyclinics

**File**: `shared/data/1_raw/equity/utilization/subsidised-primary-care-attendances-at-polyclinics.csv`  
**Grain**: One row per (year, sex, attendance_type)  
**Rows**: 66 (synthetic) / 62 (documented Kaggle source)  
**Temporal Coverage**: 2009–2019 (11 years)  
**Primary Equity Dimensions**: Sex, Care type

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 2009–2019 | No nulls |
| `sex` | Utf8 | Biological sex | "Male", "Female" | 2 categories |
| `attendance_type` | Utf8 | Type of polyclinic attendance | "Acute / Non-Chronic", "Chronic Disease Management", "Health Screening" | 3 categories |
| `attendances` | Int64 | Total number of attendances | 400,000–2,500,000 | Absolute count |

### Key Observations

- **Female utilization higher**: Female attendances exceed male across all care types
- **Chronic disease growth**: CDM attendance growing as % of total — aging population signal
- **Health screening gap**: Males show lower screening uptake — potential intervention target
- **Equity use**: Primary care access gap by sex; screening uptake disparities

---

## Dataset 3: Residential Long-Term Care Admissions

**File**: `shared/data/1_raw/equity/utilization/residential-long-term-care-admissions.csv`  
**Grain**: One row per (year, care_type)  
**Rows**: 30 (synthetic) / 25 (documented Kaggle source)  
**Temporal Coverage**: 2006–2020 (15 years)  
**Primary Equity Dimensions**: Care type (proxy for elderly access)

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 2006–2020 | No nulls |
| `care_type` | Utf8 | Type of long-term care | "Nursing Home", "Inpatient Hospice" | 2 categories |
| `admissions` | Int32 | Total admissions per year | 800–3,500 | Absolute count |

### Key Observations

- **Nursing home demand growing**: ~3% annual increase, consistent with aging population
- **No sex stratification in raw data**: Limits gender equity analysis for LTC
- **Equity use**: LTC access trends for elderly; resource adequacy assessment

---

## Dataset 4: Age-Standardised Cancer Mortality Rate

**File**: `shared/data/1_raw/equity/outcomes/age-standardised-mortality-rate-for-cancer.csv`  
**Grain**: One row per (year, sex)  
**Rows**: 60 (synthetic) / ~60 (documented Kaggle source, 30 years × 2 sexes)  
**Temporal Coverage**: 1990–2019 (30 years)  
**Primary Equity Dimensions**: Sex

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 1990–2019 | No nulls |
| `sex` | Utf8 | Biological sex | "Male", "Female" | 2 categories |
| `disease` | Utf8 | Disease identifier | "cancer" | Constant |
| `age_standardised_rate` | Float32 | Deaths per 100,000 population (age-standardised) | 100–180 | Rate base: 100,000 |

### Key Observations

- **Declining trend**: Cancer mortality improving ~0.8% annually (health system effectiveness)
- **Male excess**: Male cancer mortality ~15-20% higher than female across the period
- **Equity use**: Gender cancer mortality gap; trend analysis of gap narrowing/widening

---

## Dataset 5: Age-Standardised Stroke Mortality Rate

**File**: `shared/data/1_raw/equity/outcomes/age-standardised-mortality-rate-for-stroke.csv`  
**Grain**: One row per (year, sex)  
**Rows**: 60 (synthetic) / ~60 (documented)  
**Temporal Coverage**: 1990–2019 (30 years)  
**Primary Equity Dimensions**: Sex

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 1990–2019 | No nulls |
| `sex` | Utf8 | Biological sex | "Male", "Female" | 2 categories |
| `disease` | Utf8 | Disease identifier | "stroke" | Constant |
| `age_standardised_rate` | Float32 | Deaths per 100,000 population | 20–60 | Rate base: 100,000 |

### Key Observations

- **Rapid improvement**: Stroke mortality declining ~2.5% annually (Singapore prevention programs)
- **Gender gap**: Male stroke mortality ~25% higher than female
- **Equity use**: Stroke prevention equity; male vulnerability; temporal gap trends

---

## Dataset 6: Age-Standardised Ischaemic Heart Disease Mortality Rate

**File**: `shared/data/1_raw/equity/outcomes/age-standardised-mortality-rate-for-ischaemic-heart-disease.csv`  
**Grain**: One row per (year, sex)  
**Rows**: 60 (synthetic) / ~60 (documented)  
**Temporal Coverage**: 1990–2019 (30 years)  
**Primary Equity Dimensions**: Sex

### Schema

| Column | Type | Description | Example Values | Notes |
|--------|------|-------------|----------------|-------|
| `year` | Int32 | Reference year | 1990–2019 | No nulls |
| `sex` | Utf8 | Biological sex | "Male", "Female" | 2 categories |
| `disease` | Utf8 | Disease identifier | "heart_disease" | Constant |
| `age_standardised_rate` | Float32 | Deaths per 100,000 population | 35–130 | Rate base: 100,000 |

### Key Observations

- **Largest gender gap**: Male IHD mortality ~2x female — largest absolute disparity across all outcomes
- **Strong declining trend**: ~3% annual decline (lifestyle improvements, treatment advances)
- **Equity use**: IHD gender disparity; preventive care targeting for high-risk males

---

## Cross-Dataset Relationships

```
hospital_admissions      ─── year, sex ──► mortality_cancer
                                           mortality_stroke
                                           mortality_heart_disease
       │
       └─── year ──────────────────────► polyclinic_attendances
                                          ltc_admissions
```

**Join key**: `year` (integer) — all datasets joinable on year  
**Secondary join**: `sex` — hospital_admissions, polyclinic_attendances, mortality datasets  
**Cardinality**: All are aggregated national statistics — no PII, no deduplication needed

---

## Known Issues & Recommended Filters

| Issue | Affected Datasets | Recommended Action |
|-------|------------------|-------------------|
| LTC data has no sex stratification | `ltc_admissions` | Note limitation; use care_type as proxy |
| Mortality covers 1990-2019, utilization 2006-2020 | All | Align to overlapping 2006-2019 for cross-dataset analysis |
| Rate bases differ (1,000 vs 100,000) | All | Normalise to per-100,000 for comparability |
| Attendance types not available in LTC | `ltc_admissions` | Expand care type categories if real data allows |
| Age group format: "0 - 4" has spaces | `hospital_admissions` | Standardise to "0-4" in cleaning stage |

---

## Standard Analytical Filters (Recommended for Cleaning Stage)

```python
# Temporal alignment: use 2006-2019 for cross-dataset analysis
year_filter = pl.col("year").is_between(2006, 2019)

# Exclude totals/aggregates if they appear in real data
sex_filter = pl.col("sex").is_in(["Male", "Female"])  # exclude "Both Sexes" totals

# Rate base normalisation constant (apply in cleaning)
ADMISSION_RATE_BASE = 1_000
MORTALITY_RATE_BASE = 100_000
```

---

## Lineage

```
Kaggle API / Synthetic Generator
    └─► shared/src/data_processing/extractors/equity_extractor.py
            └─► shared/data/1_raw/equity/utilization/*.csv
            └─► shared/data/1_raw/equity/outcomes/*.csv
            └─► shared/data/1_raw/equity/_metadata.json
            └─► shared/data/schemas/equity_raw_schema.yml
            └─► shared/data/3_interim/equity_validation_summary_YYYYMMDD.csv
```

---

**Business Owner**: MOH Population Health Strategy Division  
**Data Steward**: Data Engineering Team  
**Refresh Frequency**: Annual (aligned with MOH data release cycle)  
**Retention**: 5 years (aligned with project analytics lifecycle)  
**Access**: MOH Population Health Division, Community Health Organizations, PS-005 analysis team
