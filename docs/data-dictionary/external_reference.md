# External Reference Data Dictionary

## Fields
- region: Region name
- population: Population count
- benchmark_rate: Reference rate

## Data Quality
- Validate population and benchmark_rate ranges

## Lineage
- Source: External agencies
- Transformations: ETL scripts in src/data_processing/
- Final Table: data/4_processed/

## Business Owner
- MOH Singapore

## Refresh Frequency
- Monthly

## Sample Data
| region     | population | benchmark_rate |
|------------|------------|---------------|
| Central    | 1200000    | 0.02          |
| East       | 800000     | 0.015         |
