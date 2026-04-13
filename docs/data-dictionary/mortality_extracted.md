# Mortality Data Dictionary — Extracted

**Generated**: 2026-04-06T07:09:04.864786+00:00
**Source**: Kaggle `subhamjain/health-dataset-complete-singapore`
**Problem Statement**: PS-002 National Disease Burden Temporal Trends Analysis

## Grain
One row per **year** (and sex where applicable) per disease table.

## Temporal Coverage
**1990-2019** — annual data, 30-year continuous time series.

## Tables

### Cancer
- **File**: `shared/data/1_raw/mortality/age-standardised-mortality-rate-for-cancer.csv`
- **Rows**: 30
- **Columns**: 2

| Column | Type | Null% | Unique | Min | Max |
|--------|------|-------|--------|-----|-----|
| year | Int64 | 0.0% | 30 | 1990 | 2019 |
| cancer | Float64 | 0.0% | 29 | 122.2 | 244.5 |

### Stroke
- **File**: `shared/data/1_raw/mortality/age-standardised-mortality-rate-for-stroke.csv`
- **Rows**: 30
- **Columns**: 2

| Column | Type | Null% | Unique | Min | Max |
|--------|------|-------|--------|-----|-----|
| year | Int64 | 0.0% | 30 | 1990 | 2019 |
| stroke | Float64 | 0.0% | 30 | 16.4 | 95.8 |

### Ischaemic Heart Disease
- **File**: `shared/data/1_raw/mortality/age-standardised-mortality-rate-for-ischaemic-heart-disease.csv`
- **Rows**: 30
- **Columns**: 2

| Column | Type | Null% | Unique | Min | Max |
|--------|------|-------|--------|-----|-----|
| year | Int64 | 0.0% | 30 | 1990 | 2019 |
| ihd | Float64 | 0.0% | 30 | 61.0 | 178.9 |

## Known Issues
- None identified during extraction.

## Recommended Cleaning Steps
- Standardise column names to snake_case
- Cast year column to Int32
- Cast mortality rate columns to Float32
- Add `disease` label column
- Pivot wide → long if multiple sex columns exist