# Clean & Integrate Capacity and Utilization Datasets (Lifecycle Stage: Data Preparation)

**Story ID**: PS-003-US-02  
**Epic**: Healthcare System Capacity & Utilization Optimization  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer preparing capacity analysis datasets**,  
I want **to clean, standardize, and integrate facility capacity and utilization data into a unified schema with consistent facility categorizations**,  
So that **healthcare planners can perform integrated gap analysis comparing capacity supply against demand across all facility types**.

---

## 🎯 Acceptance Criteria

1. **Schema standardization completed**
   - Unified schema: `year`, `facility_category`, `sector`, `capacity_metric`, `utilization_metric`, `capacity_type`
   - Facility categorizations standardized: `acute_care`, `community_hospital`, `long_term_care`, `primary_care`
   - Sector names consistent: `public`, `private`, `not_for_profit`
   - Data types enforced: year→Int32, capacity→Int32, utilization_rate→Float64

2. **Data integration achieved**
   - Capacity and utilization joined on year and facility type
   - Missing utilization data flagged (some facility types may lack admission rates)
   - Sector-level aggregations created for macro-level analysis
   - Integrated dataset spans common time period (2009-2020 where overlapping)

3. **Data quality issues resolved**
   - Facility type naming variations standardized
   - Rate bases confirmed: admissions per 100k population
   - Duplicate records identified and removed
   - Completeness validated: expected records present for each year

4. **Output deliverables**
   - Integrated dataset: `shared/data/3_interim/capacity_utilization_integrated.parquet`
   - Schema documentation: `shared/data/schemas/capacity_integrated_schema.yml`
   - Cleaning report: `shared/data/3_interim/capacity_cleaning_report.csv`
   - Test coverage: ≥80%

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Schema Validation**: Pydantic 2.5+
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#healthcare-capacity-metrics) - Understanding bed types and capacity metrics
- [Data Sources](../../../../project_context/data-sources.md#capacity-management) - Table schemas

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `loguru>=0.7.0`, `pyyaml>=6.0`

### Internal Dependencies
- **Upstream**: PS-003-US-01 (Extract capacity data - BLOCKING)
- **Data Sources**: `shared/data/1_raw/capacity/*.csv`, `shared/data/1_raw/utilization/*.csv`
- **Config Files**: `shared/config/base.yml`

---

## ✅ Implementation Tasks

### Schema Standardization
- [ ] Define unified schema in YAML
- [ ] Map source facility types to standard categories
- [ ] Standardize sector names (public/private/not-for-profit)
- [ ] Rename columns to consistent naming convention
- [ ] Cast data types appropriately

### Data Cleaning
- [ ] Handle facility type naming variations
- [ ] Validate capacity metrics: non-negative bed counts
- [ ] Validate utilization rates: realistic ranges
- [ ] Remove duplicates if any
- [ ] Flag missing utilization data for certain facility types

### Data Integration  
- [ ] Join capacity and utilization on year and facility category
- [ ] Create sector-level aggregations
- [ ] Calculate derived metrics: capacity per capita, utilization vs capacity ratio
- [ ] Handle non-overlapping years appropriately

### Output Generation
- [ ] Save integrated Parquet dataset
- [ ] Generate cleaning report (issues found, corrections made)
- [ ] Document schema in YAML
- [ ] Log processing summary

### Testing & Documentation
- [ ] Unit tests for cleaning functions
- [ ] Integration tests for full pipeline
- [ ] Validation tests for schema compliance
- [ ] Docstrings (Google style)

---

## 📌 Notes

**Polars Integration Example**:
```python
import polars as pl
from loguru import logger

# Load capacity data
df_capacity = pl.read_csv("shared/data/1_raw/capacity/inpatient_facilities.csv")

# Standardize facility types
facility_mapping = {
    'Acute Hospital Beds': 'acute_care',
    'Community Hospital Beds': 'community_hospital',
    # ... additional mappings
}

df_capacity = df_capacity.with_columns([
    pl.col('facility_type').map_dict(facility_mapping).alias('facility_category')
])

# Load utilization data
df_utilization = pl.read_csv("shared/data/1_raw/utilization/hospital_admissions.csv")

# Aggregate to facility level (sum across age/sex)
df_util_agg = df_utilization.group_by(['year', 'facility_category']).agg([
    pl.col('admission_rate').sum().alias('total_admission_rate')
])

# Join capacity and utilization
df_integrated = df_capacity.join(
    df_util_agg, 
    on=['year', 'facility_category'], 
    how='left'
)

logger.info(f"✓ Integrated {len(df_integrated)} capacity-utilization records")
```

**Facility Category Mapping**:
- **Acute Care**: General hospitals, specialist hospitals
- **Community Hospital**: Step-down care, rehab facilities
- **Long-Term Care**: Nursing homes, chronic sick facilities
- **Primary Care**: Polyclinics, GP clinics

**Known Challenges**:
- Admission rates may not map cleanly to all facility types
- Some facility types lack utilization metrics
- Sector breakdowns may be incomplete for certain years

---

## Implementation Plan

### 1. Feature Overview

**Objective**: Clean, standardize, and integrate facility capacity (inpatient beds, primary care, long-term care) and utilization (hospital admissions) data into a single unified Parquet dataset. Facility categories and sector names are normalized to consistent enumerations so downstream gap analysis can join on stable keys.

**Primary User Role**: Data Engineer preparing capacity analysis datasets

**Key Success Metric**: `shared/data/3_interim/capacity_utilization_integrated.parquet` exists with unified `facility_category` ∈ {`acute_care`, `community_hospital`, `long_term_care`, `primary_care`}, `sector` ∈ {`public`, `private`, `not_for_profit`}, and zero null values in required columns covering 2009-2020.

---

### 2. Component Analysis & Reuse Strategy

| Component | Path | Status | Justification |
|-----------|------|--------|---------------|
| `KaggleConnector` | `shared/src/data_processing/kaggle_connector.py` | ✅ Reuse as-is | Auth and dataset download already implemented |
| `workforce_cleaner.py` patterns | `shared/src/data_processing/workforce_cleaner.py` | 🔧 Pattern reference | `map_dict()` + `cast(pl.Categorical)` pattern for category standardization can be replicated |
| `schema_validator.py` | `shared/src/utils/schema_validator.py` | 🔧 Extend | Add `validate_capacity_integrated_schema()` for the unified dataset |
| `capacity_extractor.py` | `shared/src/data_processing/capacity_extractor.py` | ✅ Upstream output | Raw CSVs from US-01 are the input |

**New Components Required**:
- `shared/src/data_processing/capacity_cleaner.py` — schema standardization, integration join, derived metric calculation
- `shared/config/capacity_mappings.yml` — facility type and sector lookup tables (keeps business logic out of code)
- `shared/data/schemas/capacity_integrated_schema.yml` — expected columns, dtypes, and value constraints for integrated dataset
- `shared/tests/unit/test_capacity_cleaner.py` — unit tests for all cleaning functions

---

### 3. Affected Files

```
- [CREATE] shared/src/data_processing/capacity_cleaner.py
  - CapacityCleaner class
  - standardize_facility_types(df: pl.DataFrame, mapping: dict) -> pl.DataFrame
  - standardize_sectors(df: pl.DataFrame, mapping: dict) -> pl.DataFrame
  - enforce_dtypes(df: pl.DataFrame) -> pl.DataFrame
  - aggregate_utilization(df: pl.DataFrame) -> pl.DataFrame
  - join_capacity_utilization(cap: pl.DataFrame, util: pl.DataFrame) -> pl.DataFrame
  - calculate_derived_metrics(df: pl.DataFrame, population: int) -> pl.DataFrame
  - generate_cleaning_report(issues: list[dict]) -> pl.DataFrame
  Dependencies: polars, yaml, loguru, pathlib
  Config: shared/config/capacity_mappings.yml
  Logging: logs/etl/capacity_cleaning_{timestamp}.log

- [CREATE] shared/config/capacity_mappings.yml
  - facility_categories: mapping dict
  - sector_names: mapping dict
  - required_columns: list by table name
  - dtype_map: column -> polars dtype string

- [CREATE] shared/data/schemas/capacity_integrated_schema.yml
  - integrated dataset schema with constraints

- [MODIFY] shared/src/utils/schema_validator.py
  - Add: validate_capacity_integrated_schema(df: pl.DataFrame) -> tuple[bool, list[str]]

- [CREATE] shared/tests/unit/test_capacity_cleaner.py
  - test_standardize_facility_types()
  - test_standardize_sectors()
  - test_enforce_dtypes()
  - test_join_no_capacity_rows_lost()
  - test_null_utilization_flagged()
  - test_derived_metrics_calculation()
```

---

### 4. Component Breakdown

#### `CapacityCleaner` class

**Location**: `shared/src/data_processing/capacity_cleaner.py`

**Responsibility**: Accept raw DataFrames from US-01, apply standardization, join, and write integrated Parquet.

**Technical Constraints**:
- Memory budget: < 500 MB (all tables combined < 1 MB raw, safe to use eager loading)
- Target execution time: < 60 seconds end-to-end
- Data volume: trivially small — use `pl.read_csv()` not `pl.scan_csv()`
- Write format: Parquet with `zstd` compression for downstream query performance
- No row deletion without logging reason and count

---

### 5. Data Pipeline

```
Raw CSVs (US-01 output)
  shared/data/1_raw/capacity/inpatient_facilities.csv    (180 rows)
  shared/data/1_raw/capacity/primary_care_facilities.csv (96 rows)
  shared/data/1_raw/capacity/ltc_admissions.csv          (25 rows)
  shared/data/1_raw/utilization/hospital_admissions.csv  (216 rows)
        │
        ▼
  Load raw CSVs with pl.read_csv()
        │
        ▼
  Standardize facility_type → facility_category  (map_dict from YAML)
  Standardize sector names                        (map_dict from YAML)
  Enforce dtypes: year→Int32, beds→Int32, rate→Float64
  Flag unmapped values (warn, do NOT silently drop)
        │
        ▼
  Aggregate utilization by [year] — sum admission_rate across age/sex groups
        │
        ▼
  Left-join capacity on [year] with utilization aggregate
  (Left join preserves all capacity years; missing utilization flagged)
        │
        ▼
  Calculate derived metrics
    - capacity_per_capita = total_beds / population * 10_000
    - utilization_capacity_ratio = total_admission_rate / total_beds
        │
        ▼
  Validate integrated schema
  Generate cleaning report CSV
        │
        ▼
  Write shared/data/3_interim/capacity_utilization_integrated.parquet
  Write shared/data/3_interim/capacity_cleaning_report.csv
```

**Data Sources**: Output of PS-003-US-01 (BLOCKING dependency)

---

### 6. Code Generation Specifications

#### 6.1 Function Signatures & Complete Implementations

```python
# shared/src/data_processing/capacity_cleaner.py
import polars as pl
import yaml
from pathlib import Path
from loguru import logger
from datetime import datetime


_MAPPINGS_FILE = Path("shared/config/capacity_mappings.yml")
_OUTPUT_PARQUET = Path("shared/data/3_interim/capacity_utilization_integrated.parquet")
_CLEANING_REPORT = Path("shared/data/3_interim/capacity_cleaning_report.csv")
SINGAPORE_POPULATION = 5_686_000  # 2023 estimate


def _load_mappings(mappings_file: Path = _MAPPINGS_FILE) -> dict:
    """Load facility and sector mapping dictionaries from YAML.

    Args:
        mappings_file: Path to YAML mappings file.

    Returns:
        dict with keys 'facility_categories' and 'sector_names'.

    Raises:
        FileNotFoundError: If mappings file does not exist.
    """
    if not mappings_file.exists():
        raise FileNotFoundError(
            f"Mappings file not found: {mappings_file}. "
            "Run ps-003-us-01 extraction first."
        )
    with open(mappings_file) as fh:
        return yaml.safe_load(fh)


def standardize_facility_types(
    df: pl.DataFrame,
    facility_map: dict[str, str],
    source_col: str = "facility_type",
    target_col: str = "facility_category",
) -> tuple[pl.DataFrame, list[dict]]:
    """Map raw facility_type strings to standard category enumeration.

    Args:
        df: Input DataFrame with a facility_type column.
        facility_map: Dict of {raw_value: standard_category}.
        source_col: Column containing raw facility types.
        target_col: Output column name for standardized category.

    Returns:
        Tuple of (transformed DataFrame, list of issue dicts).
    """
    issues: list[dict] = []

    unmapped = (
        df.filter(~pl.col(source_col).is_in(list(facility_map.keys())))
        [source_col]
        .unique()
        .to_list()
    )
    if unmapped:
        issues.append({"type": "unmapped_facility_types", "values": unmapped})
        logger.warning(f"Unmapped facility types: {unmapped}")

    df_out = df.with_columns(
        pl.col(source_col)
        .replace(facility_map)
        .cast(pl.Categorical)
        .alias(target_col)
    )
    logger.info(
        f"standardize_facility_types: {df_out[target_col].n_unique()} "
        f"distinct categories created"
    )
    return df_out, issues


def standardize_sectors(
    df: pl.DataFrame,
    sector_map: dict[str, str],
    source_col: str = "sector",
) -> tuple[pl.DataFrame, list[dict]]:
    """Standardize sector names to {public, private, not_for_profit}.

    Args:
        df: Input DataFrame.
        sector_map: Dict mapping raw sector values to standard names.
        source_col: Column name containing sector labels.

    Returns:
        Tuple of (transformed DataFrame, issue list).
    """
    issues: list[dict] = []
    unmapped = (
        df.filter(~pl.col(source_col).is_in(list(sector_map.keys())))
        [source_col]
        .unique()
        .to_list()
    )
    if unmapped:
        issues.append({"type": "unmapped_sectors", "values": unmapped})
        logger.warning(f"Unmapped sector values: {unmapped}")

    df_out = df.with_columns(
        pl.col(source_col)
        .replace(sector_map)
        .cast(pl.Categorical)
        .alias(source_col)
    )
    logger.info(f"standardize_sectors: sectors normalised → {df_out[source_col].unique().to_list()}")
    return df_out, issues


def enforce_dtypes(df: pl.DataFrame) -> pl.DataFrame:
    """Cast columns to expected dtypes as defined in capacity_mappings.yml.

    Enforced casts:
      year → Int32
      total_beds / facility_count / ltc_admissions → Int32
      admission_rate / capacity_per_capita → Float64

    Args:
        df: Input DataFrame.

    Returns:
        DataFrame with enforced dtypes.
    """
    int32_cols = [c for c in ["year", "total_beds", "facility_count", "ltc_admissions"]
                  if c in df.columns]
    float64_cols = [c for c in ["admission_rate", "capacity_per_capita", "utilization_capacity_ratio"]
                    if c in df.columns]

    df_out = df.with_columns(
        [pl.col(c).cast(pl.Int32) for c in int32_cols] +
        [pl.col(c).cast(pl.Float64) for c in float64_cols]
    )
    logger.info(f"enforce_dtypes: applied to {len(int32_cols)+len(float64_cols)} columns")
    return df_out


def aggregate_utilization(df_util: pl.DataFrame) -> pl.DataFrame:
    """Aggregate hospital admissions to annual total across all age/sex groups.

    The raw utilization table has one row per (year, age_group, sex).
    Downstream join needs one row per year.

    Args:
        df_util: Raw hospital admissions DataFrame.

    Returns:
        DataFrame with columns [year, total_admission_rate, n_demographic_rows].
    """
    df_agg = (
        df_util
        .group_by("year")
        .agg([
            pl.col("admission_rate").sum().alias("total_admission_rate"),
            pl.len().alias("n_demographic_rows"),
        ])
        .sort("year")
    )
    logger.info(f"aggregate_utilization: {df_agg.height} annual records")
    return df_agg


def join_capacity_utilization(
    cap: pl.DataFrame,
    util_agg: pl.DataFrame,
) -> pl.DataFrame:
    """Left-join capacity data with aggregated utilization on year.

    Left join preserves all capacity rows; missing utilization years
    result in null admission_rate (flagged in cleaning report).

    Args:
        cap: Capacity DataFrame (must have 'year' column).
        util_agg: Annual utilization aggregate from aggregate_utilization().

    Returns:
        Integrated DataFrame.
    """
    integrated = cap.join(util_agg, on="year", how="left")
    missing_util = integrated.filter(pl.col("total_admission_rate").is_null()).height
    if missing_util > 0:
        logger.warning(f"{missing_util} capacity rows have no utilization match")
    logger.info(f"join_capacity_utilization: {integrated.height} integrated rows")
    return integrated


def calculate_derived_metrics(
    df: pl.DataFrame,
    population: int = SINGAPORE_POPULATION,
) -> pl.DataFrame:
    """Add derived capacity and utilization ratio columns.

    Derived columns:
      capacity_per_10k_pop = total_beds / population * 10_000
      utilization_capacity_ratio = total_admission_rate / total_beds
        (null when total_beds == 0 to avoid division by zero)

    Args:
        df: Integrated DataFrame with total_beds and total_admission_rate.
        population: National population denominator.

    Returns:
        DataFrame with additional derived columns.
    """
    df_out = df.with_columns([
        (pl.col("total_beds") / population * 10_000)
        .alias("capacity_per_10k_pop"),

        pl.when(pl.col("total_beds") > 0)
        .then(pl.col("total_admission_rate") / pl.col("total_beds"))
        .otherwise(None)
        .cast(pl.Float64)
        .alias("utilization_capacity_ratio"),
    ])
    logger.info("calculate_derived_metrics: added capacity_per_10k_pop, utilization_capacity_ratio")
    return df_out


def generate_cleaning_report(issues: list[dict]) -> pl.DataFrame:
    """Serialise cleaning issue log to a DataFrame for CSV export.

    Args:
        issues: List of issue dicts with keys 'type' and 'values'.

    Returns:
        DataFrame with columns [issue_type, value, timestamp].
    """
    timestamp = datetime.now().isoformat()
    rows = [
        {"issue_type": iss["type"], "value": str(v), "detected_at": timestamp}
        for iss in issues
        for v in (iss["values"] if isinstance(iss["values"], list) else [iss["values"]])
    ]
    if not rows:
        rows = [{"issue_type": "none", "value": "no issues detected", "detected_at": timestamp}]
    df_report = pl.DataFrame(rows)
    logger.info(f"generate_cleaning_report: {len(rows)} issue records")
    return df_report


def run_cleaning_pipeline(
    inpatient_path: str = "shared/data/1_raw/capacity/inpatient_facilities.csv",
    primary_care_path: str = "shared/data/1_raw/capacity/primary_care_facilities.csv",
    ltc_path: str = "shared/data/1_raw/capacity/ltc_admissions.csv",
    utilization_path: str = "shared/data/1_raw/utilization/hospital_admissions.csv",
    mappings_file: str = str(_MAPPINGS_FILE),
    output_parquet: str = str(_OUTPUT_PARQUET),
    output_report: str = str(_CLEANING_REPORT),
) -> pl.DataFrame:
    """Orchestrate the full cleaning and integration pipeline.

    Args:
        inpatient_path: Path to raw inpatient facilities CSV.
        primary_care_path: Path to raw primary care CSV.
        ltc_path: Path to raw LTC admissions CSV.
        utilization_path: Path to raw hospital admissions CSV.
        mappings_file: Path to YAML mappings.
        output_parquet: Destination for integrated Parquet.
        output_report: Destination for cleaning report CSV.

    Returns:
        Final integrated DataFrame.

    Raises:
        FileNotFoundError: If any input file is missing.
        ValueError: If integrated dataset fails schema validation.
    """
    mappings = _load_mappings(Path(mappings_file))
    fac_map = mappings["facility_categories"]
    sec_map = mappings["sector_names"]
    all_issues: list[dict] = []

    # Load raw tables
    df_inpatient = pl.read_csv(inpatient_path)
    df_primary = pl.read_csv(primary_care_path)
    df_ltc = pl.read_csv(ltc_path)
    df_util = pl.read_csv(utilization_path)
    logger.info(
        f"Loaded raw tables — inpatient:{df_inpatient.height}, "
        f"primary_care:{df_primary.height}, ltc:{df_ltc.height}, "
        f"utilization:{df_util.height}"
    )

    # Combine capacity tables
    # Add source_table tag before concat
    df_inpatient = df_inpatient.with_columns(pl.lit("inpatient").alias("source_table"))
    df_primary = df_primary.with_columns(pl.lit("primary_care").alias("source_table"))
    df_ltc = df_ltc.with_columns(pl.lit("ltc").alias("source_table"))

    common_cols = list(set(df_inpatient.columns) & set(df_primary.columns) & set(df_ltc.columns))
    df_capacity = pl.concat([
        df_inpatient.select(common_cols),
        df_primary.select(common_cols),
        df_ltc.select(common_cols),
    ], how="diagonal")
    logger.info(f"Combined capacity: {df_capacity.height} rows")

    # Standardize
    df_capacity, iss1 = standardize_facility_types(df_capacity, fac_map)
    df_capacity, iss2 = standardize_sectors(df_capacity, sec_map)
    all_issues.extend(iss1 + iss2)

    # Enforce dtypes
    df_capacity = enforce_dtypes(df_capacity)

    # Aggregate & join utilization
    util_agg = aggregate_utilization(df_util.with_columns(
        pl.col("year").cast(pl.Int32)
    ))
    df_integrated = join_capacity_utilization(df_capacity, util_agg)
    df_integrated = calculate_derived_metrics(df_integrated)

    # Remove exact duplicates
    before = df_integrated.height
    df_integrated = df_integrated.unique()
    removed = before - df_integrated.height
    if removed:
        all_issues.append({"type": "duplicates_removed", "values": [removed]})
        logger.warning(f"Removed {removed} duplicate rows")

    # Write outputs
    output_p = Path(output_parquet)
    output_p.parent.mkdir(parents=True, exist_ok=True)
    df_integrated.write_parquet(output_p, compression="zstd")
    logger.info(f"✅ Integrated dataset saved: {output_p} ({df_integrated.height} rows)")

    report_df = generate_cleaning_report(all_issues)
    report_p = Path(output_report)
    report_p.parent.mkdir(parents=True, exist_ok=True)
    report_df.write_csv(report_p)
    logger.info(f"✅ Cleaning report saved: {report_p}")

    return df_integrated
```

#### 6.2 Configuration File

```yaml
# shared/config/capacity_mappings.yml

facility_categories:
  # Inpatient facility types
  "Acute Hospital Beds": "acute_care"
  "Community Hospital Beds": "community_hospital"
  "Nursing Homes": "long_term_care"
  "Chronic Sick Hospital Beds": "long_term_care"
  "Convalescent Hospital Beds": "community_hospital"
  # Primary care
  "Polyclinics": "primary_care"
  "GP Clinics": "primary_care"
  "Dental Clinics": "primary_care"
  "Pharmacies": "primary_care"
  # LTC
  "Nursing Home Admissions": "long_term_care"
  "Rehabilitation Centre": "community_hospital"

sector_names:
  "Public": "public"
  "Public Sector": "public"
  "Private": "private"
  "Private Sector": "private"
  "Not-for-profit": "not_for_profit"
  "Voluntary Welfare Organisations": "not_for_profit"
  "VWO": "not_for_profit"
  "Non-profit": "not_for_profit"

required_columns_by_table:
  inpatient_facilities: [year, facility_type, sector, total_beds]
  primary_care_facilities: [year, facility_type, total_facilities]
  ltc_admissions: [year, facility_type, total_admissions]
  hospital_admissions: [year, age_group, sex, admission_rate]

int32_columns: [year, total_beds, facility_count, ltc_admissions]
float64_columns: [admission_rate, capacity_per_10k_pop, utilization_capacity_ratio]
```

#### 6.3 Data Validation Rules

```python
def validate_integrated_dataset(df: pl.DataFrame) -> tuple[bool, list[str]]:
    """Validate the integrated capacity-utilization dataset.

    Args:
        df: Integrated DataFrame to validate.

    Returns:
        Tuple of (passed: bool, errors: list[str]).
    """
    errors: list[str] = []
    required_cols = [
        "year", "facility_category", "sector",
        "total_beds", "source_table", "capacity_per_10k_pop"
    ]
    missing = [c for c in required_cols if c not in df.columns]
    if missing:
        errors.append(f"Missing required columns: {missing}")

    # Year range
    yr_min, yr_max = df["year"].min(), df["year"].max()
    if yr_min < 2009 or yr_max > 2020:
        errors.append(f"Year out of range: {yr_min}-{yr_max} (expected 2009-2020)")

    # Non-negative beds
    if "total_beds" in df.columns:
        neg_beds = df.filter(pl.col("total_beds") < 0).height
        if neg_beds:
            errors.append(f"{neg_beds} rows have negative total_beds")

    # Null check on key columns
    for col in ["year", "facility_category", "sector"]:
        if col in df.columns:
            null_pct = df[col].null_count() / df.height * 100
            if null_pct > 0:
                errors.append(f"Column '{col}' has {null_pct:.1f}% nulls (expected 0%)")

    passed = len(errors) == 0
    return passed, errors
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 pydantic>=2.5.0 pyyaml>=6.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

**Derived metrics validated against data availability**:

| Feature | Formula | Required Fields | Expected Range | Validation |
|---------|---------|---------|---------|---------|
| `capacity_per_10k_pop` | `total_beds / population × 10,000` | `total_beds`, static population | 0.5 – 50 | Warn if outside range |
| `utilization_capacity_ratio` | `total_admission_rate / total_beds` | Both columns | 0 – 500 | Null-safe division |

**Explicitly rejected features** (data not available in US-01 outputs):
- Actual bed occupancy rate — requires discharge counts (not available)
- Average length of stay — not in Kaggle dataset
- Cost per bed per year — no financial data in schema

---

### 8. Testing Strategy

```python
# shared/tests/unit/test_capacity_cleaner.py
import polars as pl
import pytest
from shared.src.data_processing.capacity_cleaner import (
    standardize_facility_types,
    standardize_sectors,
    enforce_dtypes,
    aggregate_utilization,
    join_capacity_utilization,
    calculate_derived_metrics,
)


FAC_MAP = {"Acute Hospital Beds": "acute_care", "Nursing Homes": "long_term_care"}
SEC_MAP = {"Public": "public", "Private": "private"}


def test_standardize_facility_types_happy_path():
    df = pl.DataFrame({"facility_type": ["Acute Hospital Beds", "Nursing Homes"]})
    result, issues = standardize_facility_types(df, FAC_MAP)
    assert result["facility_category"].to_list() == ["acute_care", "long_term_care"]
    assert len(issues) == 0


def test_standardize_facility_types_unmapped():
    df = pl.DataFrame({"facility_type": ["Unknown Type"]})
    result, issues = standardize_facility_types(df, FAC_MAP)
    assert any(i["type"] == "unmapped_facility_types" for i in issues)


def test_standardize_sectors():
    df = pl.DataFrame({"sector": ["Public", "Private"]})
    result, issues = standardize_sectors(df, SEC_MAP)
    assert result["sector"].to_list() == ["public", "private"]
    assert len(issues) == 0


def test_enforce_dtypes():
    df = pl.DataFrame({"year": [2015, 2016], "total_beds": [1000, 1200]})
    result = enforce_dtypes(df)
    assert result["year"].dtype == pl.Int32
    assert result["total_beds"].dtype == pl.Int32


def test_aggregate_utilization():
    df = pl.DataFrame({
        "year": [2015, 2015, 2016],
        "age_group": ["0-14", "15-64", "0-14"],
        "sex": ["M", "M", "F"],
        "admission_rate": [100.0, 200.0, 150.0],
    })
    result = aggregate_utilization(df.with_columns(pl.col("year").cast(pl.Int32)))
    assert result.height == 2
    assert result.filter(pl.col("year") == 2015)["total_admission_rate"][0] == pytest.approx(300.0)


def test_join_no_capacity_rows_lost():
    cap = pl.DataFrame({"year": pl.Series([2015, 2016, 2017], dtype=pl.Int32), "total_beds": [1000, 1100, 1200]})
    util = pl.DataFrame({"year": pl.Series([2015, 2016], dtype=pl.Int32), "total_admission_rate": [500.0, 600.0]})
    result = join_capacity_utilization(cap, util)
    assert result.height == 3  # all capacity rows preserved
    assert result.filter(pl.col("year") == 2017)["total_admission_rate"].is_null()[0]


def test_derived_metrics_not_null_for_valid_rows():
    df = pl.DataFrame({
        "total_beds": pl.Series([1000], dtype=pl.Int32),
        "total_admission_rate": [500.0],
    })
    result = calculate_derived_metrics(df, population=5_686_000)
    assert result["capacity_per_10k_pop"][0] == pytest.approx(1000 / 5_686_000 * 10_000, rel=1e-4)
    assert result["utilization_capacity_ratio"][0] == pytest.approx(0.5, rel=1e-4)
```

---

### 9. Implementation Steps

#### Phase 1: Configuration & Schema
- [ ] Create `shared/config/capacity_mappings.yml` with all facility and sector mappings
- [ ] Create `shared/data/schemas/capacity_integrated_schema.yml`
- [ ] Create output dirs: `shared/data/3_interim/`

#### Phase 2: Core Cleaning Logic
- [ ] Create `shared/src/data_processing/capacity_cleaner.py` with all functions above
- [ ] Implement `_load_mappings()` with error handling
- [ ] Implement `standardize_facility_types()` with unmapped-value logging
- [ ] Implement `standardize_sectors()` with unmapped-value logging
- [ ] Implement `enforce_dtypes()` for Int32/Float64 coercions
- [ ] Implement `aggregate_utilization()` — sum admission_rate across age/sex
- [ ] Implement `join_capacity_utilization()` — left join on year
- [ ] Implement `calculate_derived_metrics()` — per-capita and ratio columns
- [ ] Implement `generate_cleaning_report()` and `run_cleaning_pipeline()`

#### Phase 3: Schema Validation Extension
- [ ] Add `validate_integrated_dataset()` to `shared/src/utils/schema_validator.py`
- [ ] Integrate validation into `run_cleaning_pipeline()`

#### Phase 4: Tests
- [ ] Create `shared/tests/unit/test_capacity_cleaner.py` with all test functions above
- [ ] Run: `pytest shared/tests/unit/test_capacity_cleaner.py -v --cov`
- [ ] Verify ≥ 80% coverage

#### Phase 5: Pipeline Execution & Verification
- [ ] Verify US-01 raw files exist before running
- [ ] Execute `run_cleaning_pipeline()` with default paths
- [ ] Confirm `capacity_utilization_integrated.parquet` written successfully
- [ ] Confirm `capacity_cleaning_report.csv` shows acceptable issue count
- [ ] Spot-check 5 sample rows from integrated dataset

---

### 10. Code Generation Order

1. `shared/config/capacity_mappings.yml`
2. `shared/data/schemas/capacity_integrated_schema.yml`
3. `shared/tests/unit/test_capacity_cleaner.py` (fixtures first)
4. `shared/src/data_processing/capacity_cleaner.py`
5. extension to `shared/src/utils/schema_validator.py`
6. integration test: `shared/tests/integration/test_cleaning_pipeline.py`

---

### 11. Data Quality & Validation

**Pre-pipeline checks**:
- All 4 raw CSVs present (FileNotFoundError if missing)
- Row counts match US-01 expectations (180, 96, 25, 216) — warn if they don't

**Post-pipeline checks**:
- `facility_category` has exactly 4 distinct values
- `sector` has exactly 3 distinct values
- `year` range is 2009–2020
- Null % in `total_beds` = 0%
- `capacity_per_10k_pop` range: 0.5 to 50 (flag extremes)

**Outlier detection** (IQR on `total_beds` per facility_category, visualize boxplot):
```python
q1 = df["total_beds"].quantile(0.25)
q3 = df["total_beds"].quantile(0.75)
iqr = q3 - q1
outliers = df.filter(
    (pl.col("total_beds") < q1 - 1.5 * iqr) | (pl.col("total_beds") > q3 + 1.5 * iqr)
)
logger.info(f"Outlier beds: {outliers.height} rows ({outliers.height/df.height*100:.1f}%)")
```

---

### 12. Adaptive Implementation Strategy

After running the pipeline, pause and inspect:
- If unmapped facility types > 0 → update `capacity_mappings.yml` before proceeding
- If >5% rows have null `total_admission_rate` → investigate year coverage mismatch, may need to extend utilization join key
- If `capacity_per_10k_pop` values seem implausible → verify population constant

---

### 13. Security & Privacy

No PII or individual-level data. All tables are aggregate statistics published by MOH. No anonymization required. Raw CSVs must remain in `shared/data/1_raw/` (read-only), and no credentials are needed beyond Kaggle auth already configured in US-01.

---

### 14. Version Control

Branch: `feature/ps-003-us-02-clean-integrate`
Commits: `feat(ps-003): add capacity_cleaner and mappings config`, `test(ps-003): unit tests for capacity cleaning pipeline`

---

**Implementation Plan Checklist**
- [x] Feature overview defined
- [x] Component reuse strategy specified
- [x] All affected files listed with CREATE/MODIFY indicators
- [x] Complete executable function signatures provided
- [x] Configuration YAML defined
- [x] Validation rules as executable code
- [x] Testing strategy with specific assertions
- [x] Implementation steps ordered by phase
- [x] Code generation order specified
- [x] Data quality checks documented
