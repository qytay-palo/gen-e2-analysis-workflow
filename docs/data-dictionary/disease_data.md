# Disease Data Dictionary

## Fields
- date: Date of case report (YYYY-MM-DD)
- disease: Disease name (categorical)
- case_count: Number of cases (integer)

## Data Quality
- Validate date format and case_count range
- Handle missing values explicitly

## Lineage
- Source: MOH Singapore
- Transformations: ETL scripts in src/data_processing/
- Final Table: data/4_processed/

## Business Owner
- MOH Singapore

## Refresh Frequency
- Weekly

## Sample Data
| date       | disease   | case_count |
|------------|-----------|------------|
| 2025-01-01 | Dengue    | 12         |
| 2025-01-01 | HFMD      | 5          |
| 2025-01-01 | Influenza | 8          |
