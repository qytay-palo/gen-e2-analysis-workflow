# Data Dictionary Index

> **Complete reference for all datasets used in Singapore Health Trends Analysis**

This section provides comprehensive documentation for each data domain used in the project. All datasets are stored in `shared/data/` following our hybrid structure.

---

## 📊 Core Data Domains

### 1. Singapore Health Surveillance Data
- **[Disease Data](disease_data.md)** - Notifiable disease surveillance and epidemiological data
  - Source: Ministry of Health Singapore
  - Update Frequency: Weekly
  - Key Fields: disease type, case counts, demographics, temporal patterns
  - Location: `shared/data/1_raw/disease_surveillance/`

- **[Healthcare Workforce Data](workforce-forecast-features.md)** - Healthcare workforce planning and forecasting
  - Source: Ministry of Health Singapore
  - Update Frequency: Quarterly
  - Key Fields: workforce categories, headcount, projections
  - Location: `shared/data/1_raw/workforce/`

### 2. Reference & External Data
- **[External Reference Data](external_reference.md)** - Demographics, benchmarks, and contextual datasets
  - Source: Singapore Department of Statistics, WHO
  - Update Frequency: Annually
  - Key Fields: population statistics, geographic boundaries, health benchmarks
  - Location: `shared/data/2_external/`

### 3. Public Health Datasets
- **Kaggle Health Datasets** - *To be added as datasets are acquired*
  - Examples: Global disease burden, climate health impacts
  - Location: `shared/data/2_external/kaggle/`

---

## 📋 Data Dictionary Standards

Each data dictionary file must include:

### Required Sections
1. **Overview**
   - Dataset name and description
   - Business purpose and use cases
   - Source system and owner

2. **Schema Definition**
   - Field names and descriptions
   - Data types (following our preferred types: Date, Categorical, Int32, Float32)
   - Value ranges and constraints
   - Nullable/Required indicators

3. **Data Quality**
   - Known limitations and caveats
   - Null percentage by field
   - Data quality score (if available)
   - Validation rules

4. **Lineage & Provenance**
   - Source system → extraction → transformation → final table
   - Transformation logic applied
   - Dependencies on other datasets

5. **Governance**
   - Data owner / Subject Matter Expert
   - Refresh frequency and schedule
   - Retention period
   - Access restrictions (if applicable)

6. **Sample Data**
   - Example records (anonymized if sensitive)
   - Common query patterns
   - Edge cases and special values

---

## 🔄 Data Update Procedures

### Extraction Schedule
- **Disease Surveillance**: Weekly (Monday 9:00 AM SGT)
- **Workforce Data**: Quarterly (1st of Jan/Apr/Jul/Oct)
- **Reference Data**: Annually (January)

### Quality Checks
All data must pass validation before being available for analysis:
- Schema compliance check
- Null/missing value assessment
- Outlier detection
- Referential integrity validation

**Validation logs**: `logs/etl/data_quality/`

---

## 📁 File Locations

| Data Type | Location | Git Tracked |
|-----------|----------|-------------|
| Raw data (large files) | `shared/data/1_raw/` | ❌ No |
| Raw data (< 1MB, samples) | `shared/data/1_raw/samples/` | ✅ Yes |
| External reference | `shared/data/2_external/` | ✅ Yes (if < 10MB) |
| Data schemas | `shared/data/schemas/` | ✅ Yes |
| Interim processing | `shared/data/3_interim/` | ❌ No |
| Processed datasets | `shared/data/4_processed/` | ❌ No |

---

## 🆕 Adding New Datasets

When adding a new dataset to the project:

1. **Create Data Dictionary**
   - Copy template: `docs/data_dictionary/_template.md`
   - Fill in all required sections
   - Save as: `{dataset_name}.md`

2. **Update This Index**
   - Add entry under appropriate domain
   - Link to new data dictionary file
   - Specify location in `shared/data/`

3. **Create Schema File**
   - Define schema in `shared/data/schemas/{dataset_name}.yml`
   - Include field types, constraints, validation rules

4. **Document Extraction**
   - Create extraction script in `shared/src/data_processing/extractors/`
   - Document connection parameters in script header
   - Add to extraction schedule (if recurring)

5. **Add Sample Data**
   - Place small sample in `shared/data/1_raw/samples/`
   - Ensure it's Git-tracked for reference
   - Anonymize if contains sensitive information

---

## 🔍 Quick Reference

### Common Data Types
- **Date**: `Date` (Polars) - all temporal fields
- **Disease**: `Categorical` - disease names, categories
- **Counts**: `Int32` - case counts, headcounts
- **Rates**: `Float32` - percentages, rates per capita
- **Location**: `Categorical` - regions, hospitals, clinics

### Naming Conventions
- Files: `lowercase_with_underscores.csv`
- Columns: `lowercase_with_underscores`
- Dates: `YYYY-MM-DD` format
- Categories: PascalCase or Sentence Case

---

**Last Updated**: 2026-03-11  
**Maintained By**: Data Engineering Team  
**Contact**: [Project Team Leads]
