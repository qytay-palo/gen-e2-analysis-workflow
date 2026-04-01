# Clean & Prepare Workforce Data (Lifecycle Stage: Data Preparation)

**Story ID**: PS-001-US-02  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4-5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Analyst preparing workforce forecasts**,  
I want **to clean, standardize, and integrate all workforce tables into a unified analytical dataset with consistent sectors and time periods**,  
So that **I can perform cross-sector workforce trend analysis without data quality issues or schema inconsistencies**.

---

## 🎯 Acceptance Criteria

1. **Data cleaning completed**
   - Column names standardized (lowercase, snake_case): `year`, `sector`, `profession`, `count`, `level`
   - Categorical variables converted to Polars `Categorical` dtype (sector, profession)
   - Numeric columns optimized (`Int32` for counts, `Int16` for years)
   - Sector names standardized ("Public Sector" → "public", "Private Sector" → "private")

2. **Integrated workforce dataset created**
   - All 7 workforce tables merged into single dataset
   - Columns: `year`, `profession`, `sector`, `count`, `level` (where applicable)
   - Time alignment handled (nurses start 2008, some allied health start 2014)
   - Total professions covered: doctors, nurses/midwives, pharmacists, dentists, + allied health

3. **Data quality issues documented and resolved**
   - Data quality report: `shared/data/3_interim/workforce_cleaning_report.csv`
   - Issues logged: duplicate records, inconsistent sector naming, missing time periods
   - Cleaning decisions documented in report (e.g., "filled 2006-2007 nurse gaps with null, not imputation")

4. **Data output requirement**
   - Output file: `shared/data/3_interim/workforce_integrated_clean.parquet`
   - Format: Parquet (optimized for analytical queries)
   - Schema documented: Yes - in `shared/data/schemas/workforce_clean_schema.yml`
   - Data dictionary updated: `docs/data_dictionary/workforce-forecast-features.md`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for data processing)
- **Data Quality**: Great Expectations or custom Polars validations
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for cleaning logic

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#calculation-methodologies) - Understanding FTE adjustments and workforce categories
- [Workforce Forecast Features Data Dictionary](../../../../data_dictionary/workforce-forecast-features.md) - Target schema for cleaned data

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data cleaning and transformation
- `pydantic>=2.5.0`: Schema validation
- `loguru>=0.7.0`: Structured logging
- `great-expectations>=0.18.0` (optional): Advanced data quality validation

### Internal Dependencies
- **Upstream**: PS-001-US-01 (Extract workforce data - BLOCKING)
- **Data Sources**: Raw workforce CSVs in `shared/data/1_raw/workforce/`
- **Config Files**: `shared/config/base.yml` (cleaning rules and mappings)

---

## ✅ Implementation Tasks

### Data Cleaning
- [ ] Implement `WorkforceCleaner` class in `shared/src/data_processing/clean_workforce_data.py`
- [ ] Standardize column names across all 7 tables
- [ ] Standardize sector categories (create sector mapping YAML)
- [ ] Convert categorical columns to Polars `Categorical` dtype
- [ ] Optimize numeric dtypes (`Int32` for counts, `Int16` for years)
- [ ] Handle profession name variations (e.g., "Nurses and Midwives" → "nurses_midwives")

### Data Integration
- [ ] Load all 7 workforce tables using Polars
- [ ] Align schemas to common structure (year, profession, sector, count)
- [ ] Concatenate tables using Polars `concat()` with vertical stacking
- [ ] Handle time period variations (nurses start 2008, mark 2006-2007 as null)
- [ ] Add metadata column: `data_source` (table name)
- [ ] Validate integrated dataset has all expected professions

### Data Quality Assessment
- [ ] Generate cleaning report with before/after statistics
- [ ] Document data quality issues found:
   - Duplicate records (if any)
   - Inconsistent sector naming
   - Missing time periods per profession
- [ ] Log cleaning decisions (imputation strategies, outlier handling)
- [ ] Create data quality dashboard plot (counts by profession and year)

### Testing & Validation
- [ ] Unit tests for cleaning functions (column renaming, dtype conversion)
- [ ] Integration test for full cleaning pipeline
- [ ] Validate output schema matches `workforce_clean_schema.yml`
- [ ] Validate no data loss: sum(raw counts) ≈ sum(clean counts)
- [ ] Test edge cases: missing sectors, unexpected profession names

### Documentation
- [ ] Docstrings for all cleaning functions (Google style)
- [ ] Update README: `shared/src/data_processing/README.md`
- [ ] Document cleaning decisions in `logs/etl/workforce_cleaning_YYYYMMDD.log`
- [ ] Update data dictionary: `docs/data_dictionary/workforce-forecast-features.md`

---

## 📌 Notes

**Sector Standardization Mapping** (create in `shared/config/base.yml`):
```yaml
workforce:
  sector_mapping:
    "Public Sector": "public"
    "Public": "public"
    "Private Sector": "private"
    "Private": "private"
    "Not-for-Profit": "non_profit"
```

**Polars Cleaning Example**:
```python
import polars as pl
from loguru import logger

# Load and standardize
df = (
    pl.read_csv("shared/data/1_raw/workforce/number-of-doctors.csv")
    .rename({
        'Year': 'year',
        'Sector': 'sector',
        'Level': 'level',
        'Count': 'count'
    })
    .with_columns([
        pl.col('year').cast(pl.Int16),
        pl.col('count').cast(pl.Int32),
        pl.col('sector').str.to_lowercase().cast(pl.Categorical),
        pl.lit('doctors').alias('profession').cast(pl.Categorical)
    ])
)

logger.info(f"✓ Doctors data cleaned: {len(df)} records")
```

**Time Period Alignment Strategy**:
- **Nurses (2008-2019)**: Mark 2006-2007 as null, do NOT backfill or impute
- **Allied health (2014-2019)**: Mark 2006-2013 as null
- **Rationale**: Preserve data integrity; forecasting models can handle missing historical data for some professions

**Data Quality Checklist**:
- ✓ No duplicate (year, profession, sector, level) combinations
- ✓ All counts are positive integers
- ✓ All years in reasonable range (2006-2019)
- ✓ Sector names consistent across all professions
- ✓ No unexpected null values in non-nullable columns

---

## Implementation Plan

### 1. Feature Overview

This user story implements the data cleaning and integration layer for workforce analytics. The primary goal is to transform raw workforce data from 7 separate CSV tables into a unified, analytically-ready dataset with standardized column names, optimized data types, and consistent sector categorizations.

**Primary User Role**: Data Analyst preparing workforce forecasts

**Key Deliverable**: Integrated, cleaned workforce dataset covering all professions (2006-2019) saved as Parquet with documented schema and quality report.

### 2. Component Analysis & Reuse Strategy

**Existing Components Available for Reuse**:

- **✅ REUSE AS-IS**: Extraction infrastructure from US-01
  - WorkforceExtractor can load raw CSV files
  - SchemaValidator available for output validation
  
**New Components Required**:

- **CREATE**: `shared/src/data_processing/workforce_cleaner.py` - Cleaning and standardization logic
- **CREATE**: `shared/config/cleaning_rules.yml` - Cleaning configuration
- **CREATE**: `shared/data/schemas/workforce_clean_schema.yml` - Cleaned schema definition
- **CREATE**: Unit and integration tests for cleaning pipeline

### 3. Affected Files

**[CREATE] `shared/src/data_processing/workforce_cleaner.py`** - See US-01 Section 6.1 for code patterns
**[CREATE] `shared/config/cleaning_rules.yml`** - Sector mapping, column renames, dtype optimization
**[CREATE] `shared/data/schemas/workforce_clean_schema.yml`** - Integrated schema validation rules

### 4-14. Implementation Details

**For complete implementation specifications including:**
- Component breakdown
- Data pipeline architecture  
- Full code implementations
- Testing strategy
- Quality validation

**See US-01 Implementation Plan sections 4-14** - Apply same patterns with cleaning-specific adaptations:
- Replace extraction logic with cleaning transformations
- Use Polars method chaining for efficient data transformations
- Add sector mapping and dtype optimization steps
- Implement table concatenation for integration

### 15. Key Cleaning Transformations

```python
# Standardize and integrate workforce tables
integrated_df = (
    pl.concat([
        clean_table(df, profession) 
        for profession, df in raw_tables.items()
    ])
    .with_columns([
        pl.col('year').cast(pl.Int16),
        pl.col('count').cast(pl.Int32),
        pl.col('sector').cast(pl.Categorical),
        pl.col('profession').cast(pl.Categorical)
    ])
    .sort(['year', 'profession', 'sector'])
)
```

### 16-24. Testing, Security, Quality

**Apply same standards as US-01**:
- ≥80% test coverage
- No PII/PHI (aggregated data)
- Version control with conventional commits
- All quality metrics verified

---

**Implementation Plan Complete for US-02**  
**Next**: US-03 (Explore Workforce Trends)
