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

## Implementation Plan

### 1. Feature Overview

Extract and validate demographic healthcare utilization tables from the Kaggle Singapore health dataset, and explicitly audit whether any demographic-stratified outcome tables are actually present before downstream equity analysis assumes they exist.

**Primary User Role**: Data Engineer supporting MOH population health strategists.

**Primary Goal**: Produce a trusted raw-data foundation for PS-005 with preserved demographic stratifications, extraction evidence, and a machine-readable availability audit for outcome tables.

### 2. Component Analysis & Reuse Strategy

**Existing reusable assets in the workspace**:
- `docs/project-context/data-sources.md` - authoritative source for confirmed Kaggle tables, record counts, quality expectations, and access method.
- `docs/project-context/tech-stack.md` - authoritative source for Databricks, Polars, `uv`, `loguru`, and test conventions.
- `docs/data-dictionary/index.md` - authoritative source for target directory layout (`shared/data/1_raw/`, `shared/data/schemas/`, `shared/data/3_interim/`, `shared/data/4_processed/`).
- `README.md` - defines the intended `shared/` and `problem-statement/` structure for generated implementation assets.
- `docs/objectives/user_stories/problem-statement-003-capacity/01-extract-capacity-utilization-data.md` - design reference only for extracting `hospital-admission-rate-by-age-and-sex.csv`; no executable component exists yet.

**Reusable code components**:
- None currently exist. There is no `shared/src/`, `shared/config/`, `config/`, or `tests/` tree in the repository yet, so this story should be treated as greenfield implementation work.

**Gaps requiring new components**:
- Kaggle dataset download and file-discovery utility for equity extraction.
- Schema and profiling logic for demographic raw tables.
- Test fixtures and extraction tests.
- Outcome-table availability audit artifact because the documented data source confirms utilization tables but does not clearly confirm demographic-stratified outcome tables.

### 3. ML Model Evaluation & Selection

Not applicable for this story. This is a data extraction and validation story, so no predictive, NLP, classification, regression, or forecasting model should be selected here.

### 4. Affected Files

- **[CREATE] `shared/config/base.yml`**
  - Purpose: Central paths and dataset identifiers for shared extraction utilities.
  - Keys: `kaggle.dataset_id`, `paths.raw_root`, `paths.schema_root`, `paths.log_root`, `paths.metrics_root`.
  - Dependencies: `pyyaml`.
  - Logging: consumed by extraction modules to route ETL logs to `logs/etl/`.

- **[CREATE] `config/databricks.yml`**
  - Purpose: Databricks runtime metadata and environment variable names for Kaggle authentication.
  - Keys: `runtime`, `workspace_root`, `dbfs_root`, `env.KAGGLE_USERNAME`, `env.KAGGLE_KEY`.
  - Dependencies: `pyyaml`.
  - Logging: none directly; referenced by bootstrap scripts.

- **[CREATE] `shared/data/schemas/equity_raw_schema.yml`**
  - Purpose: Human-readable schema mirror for demographic utilization tables and any verified outcome tables.
  - Tables: `hospital_admissions_age_sex`, `polyclinic_attendances_sex`, `residential_ltc_admissions`, and `demographic_outcome_candidates`.
  - Logging: validation results written to `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/`.

- **[CREATE] `shared/src/data_processing/extractors/equity_demographic_extractor.py`**
  - Functions:
   - `get_equity_table_specs() -> list[EquityTableSpec]`
   - `catalog_demographic_outcome_candidates(dataset_root: Path) -> list[str]`
   - `validate_extracted_table(df: pl.DataFrame, spec: EquityTableSpec) -> RawEquityTableProfile`
   - `extract_demographic_health_data(dataset_id: str, output_root: Path, report_path: Path, specs: list[EquityTableSpec] | None = None) -> dict[str, pl.DataFrame]`
  - Dependencies: `polars`, `kagglehub`, `loguru`, `pydantic`, `pathlib`, `json`, `shutil`.
  - Config: `shared/config/base.yml`, `config/databricks.yml`.
  - Logging: `logs/etl/equity_extraction_{timestamp}.log`.

- **[CREATE] `shared/tests/conftest.py`**
  - Fixtures:
   - `sample_hospital_admissions_df() -> pl.DataFrame`
   - `sample_polyclinic_attendance_df() -> pl.DataFrame`
   - `mock_kaggle_dataset_tree(tmp_path: Path) -> Path`
  - Dependencies: `pytest`, `polars`, `pathlib`.

- **[CREATE] `shared/tests/unit/test_equity_demographic_extractor.py`**
  - Test targets:
   - `validate_extracted_table()`
   - `catalog_demographic_outcome_candidates()`
   - `get_equity_table_specs()`
  - Dependencies: `pytest`, `polars`.

- **[CREATE] `shared/tests/integration/test_equity_raw_extraction.py`**
  - Test targets:
   - full extraction flow with mocked Kaggle download path;
   - output copies written to `shared/data/1_raw/equity/`;
   - JSON profile emitted to `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/`.
  - Dependencies: `pytest`, `polars`, `unittest.mock`, `pathlib`.

- **[CREATE] `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/README.md`**
  - Purpose: describe runtime-generated artifacts such as `raw_equity_extraction_profile_{timestamp}.json` and `outcome_table_availability_{timestamp}.json`.
  - Dependencies: none.

### 5. Component Breakdown

**New component: `equity_demographic_extractor`**
- Location: `shared/src/data_processing/extractors/equity_demographic_extractor.py`
- Responsibility: download the Kaggle dataset once, extract confirmed demographic utilization tables, scan for possible demographic outcome tables, validate schemas, and copy raw files without mutating their contents.
- Key parameters: `dataset_id`, `output_root`, `report_path`, `specs`.
- Technical constraints:
  - Memory budget: `< 200 MB` local, because the documented Kaggle dataset is ~3.5 MB.
  - Execution target: `< 2 minutes` including download from cache and validation.
  - Data volume strategy: use `pl.read_csv()` rather than lazy scanning because each source file is small; reserve `pl.scan_csv()` for later stories operating on larger joined outputs.
  - Optimization requirements: cast `year` to `Int32` only in validation/profile objects, preserve raw CSV copies unchanged, treat `age_group`, `sex`, and `ethnicity` as candidate categorical fields in profiling output.

**New component: `equity_raw_schema`**
- Location: `shared/data/schemas/equity_raw_schema.yml`
- Responsibility: encode required columns, expected year ranges, and categorical expectations for raw equity tables.
- Key parameters: `required_columns`, `expected_year_min`, `expected_year_max`, `expected_row_count`, `required_demographic_dimensions`.
- Technical constraints: schema is a mirror of executable runtime validation, not a replacement for it.

**New component: extraction tests**
- Locations: `shared/tests/unit/test_equity_demographic_extractor.py`, `shared/tests/integration/test_equity_raw_extraction.py`
- Responsibility: prove the extractor rejects missing demographic columns, reports null percentages, preserves file layout, and surfaces missing outcome tables as an auditable data gap rather than silently fabricating data.

### 6. Data Pipeline

**Grounded source inputs**:
- Confirmed demographic utilization table: `hospital-admission-rate-by-age-and-sex.csv` with 216 records and 2006-2020 coverage.
- Additional confirmed utilization-support tables relevant to equity context: `subsidised-primary-care-attendances-at-polyclinics.csv` and `residential-long-term-care-admissions.csv`.
- Outcome tables with demographic breakdowns are **not explicitly confirmed** by `docs/project-context/data-sources.md`; they must be discovered from the downloaded Kaggle directory and classified before extraction is considered complete.

**Pre-implementation validation**:
1. Confirm there is no existing `shared/data/1_raw/equity/` directory to reuse.
2. Confirm the Kaggle dataset ID from `docs/project-context/data-sources.md` remains `subhamjain/health-dataset-complete-singapore`.
3. Download the dataset once and inventory all CSV files.
4. Audit file names for demographic tokens (`age`, `sex`, `gender`, `ethnicity`, `race`) crossed with outcome tokens (`mortality`, `death`, `deaths`, `disease`, `prevalence`).
5. Profile confirmed extracted tables for row count, null percentages, duplicate rows, unique counts, and year coverage.
6. Record any missing demographic outcome tables as a blocking limitation for PS-005-US-04 rather than masking the gap.

**Pipeline specification**:

| Stage | Input | Output | Notes |
|---|---|---|---|
| Authenticate & download | Kaggle credentials from environment or `~/.kaggle/kaggle.json` | Local Kaggle cache path | Use `kagglehub.dataset_download()` |
| Confirmed table extraction | Kaggle CSVs | `shared/data/1_raw/equity/utilization/*.csv` | Copy raw files unchanged |
| Outcome-candidate audit | Downloaded dataset tree | `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/outcome_table_availability_{timestamp}.json` | Required because source docs do not confirm demographic outcome tables |
| Schema/profile validation | Extracted DataFrames | `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/raw_equity_extraction_profile_{timestamp}.json` | Include null %, duplicates, unique counts, year coverage |
| Logging | Extraction events and failures | `logs/etl/equity_extraction_{timestamp}.log` | Use `loguru` with timestamps and severity |

**Confirmed raw table targets**:

| Planned table name | Source relative path in Kaggle dataset | Raw destination | Expected demographic fields | Expected years |
|---|---|---|---|---|
| `hospital_admissions_age_sex` | `hospital-admission-rate-by-age-and-sex/hospital-admission-rate-by-age-and-sex.csv` | `shared/data/1_raw/equity/utilization/hospital_admission_rate_by_age_and_sex.csv` | `age_group`, `sex` | 2006-2020 |
| `polyclinic_attendances_sex` | `subsidised-primary-care-attendances-at-polyclinics/subsidised-primary-care-attendances-at-polyclinics.csv` | `shared/data/1_raw/equity/utilization/subsidised_primary_care_attendances.csv` | `sex` | verify on extraction |
| `residential_ltc_admissions` | `residential-long-term-care-admissions/residential-long-term-care-admissions.csv` | `shared/data/1_raw/equity/utilization/residential_long_term_care_admissions.csv` | no guaranteed demographic split in docs; treat as supporting utilization context | verify on extraction |

**Orchestration and execution order**:
1. Load config.
2. Download dataset.
3. Audit dataset tree for outcome candidates.
4. Extract confirmed utilization tables.
5. Validate and profile extracted tables.
6. Persist audit and profile artifacts.
7. If zero demographic outcome candidates are found, stop with a documented data-gap handoff for PS-005-US-02 and PS-005-US-04.

### 7. Code Generation Specifications

#### 7.1 Function Signatures and Complete Implementations

```python
from __future__ import annotations

from pathlib import Path
from typing import Any
import json
import shutil

import kagglehub
import polars as pl
from loguru import logger
from pydantic import BaseModel, Field, field_validator


class EquityTableSpec(BaseModel):
   table_name: str
   source_relative_path: str
   output_relative_path: str
   required_columns: list[str]
   expected_year_min: int
   expected_year_max: int
   category: str = Field(pattern="^(utilization|outcomes)$")

   @field_validator("required_columns")
   @classmethod
   def validate_required_columns(cls, value: list[str]) -> list[str]:
      if not value:
         raise ValueError("required_columns must not be empty")
      return value


class RawEquityTableProfile(BaseModel):
   table_name: str
   row_count: int = Field(ge=0)
   duplicate_rows: int = Field(ge=0)
   null_percentages: dict[str, float]
   unique_counts: dict[str, int]
   category_distributions: dict[str, dict[str, int]]
   min_year: int | None = None
   max_year: int | None = None


def get_equity_table_specs() -> list[EquityTableSpec]:
   return [
      EquityTableSpec(
         table_name="hospital_admissions_age_sex",
         source_relative_path=(
            "hospital-admission-rate-by-age-and-sex/"
            "hospital-admission-rate-by-age-and-sex.csv"
         ),
         output_relative_path="equity/utilization/hospital_admission_rate_by_age_and_sex.csv",
         required_columns=["year", "age_group", "sex"],
         expected_year_min=2006,
         expected_year_max=2020,
         category="utilization",
      ),
      EquityTableSpec(
         table_name="polyclinic_attendances_sex",
         source_relative_path=(
            "subsidised-primary-care-attendances-at-polyclinics/"
            "subsidised-primary-care-attendances-at-polyclinics.csv"
         ),
         output_relative_path="equity/utilization/subsidised_primary_care_attendances.csv",
         required_columns=["year", "sex"],
         expected_year_min=2006,
         expected_year_max=2020,
         category="utilization",
      ),
      EquityTableSpec(
         table_name="residential_ltc_admissions",
         source_relative_path=(
            "residential-long-term-care-admissions/"
            "residential-long-term-care-admissions.csv"
         ),
         output_relative_path="equity/utilization/residential_long_term_care_admissions.csv",
         required_columns=["year"],
         expected_year_min=2006,
         expected_year_max=2020,
         category="utilization",
      ),
   ]


def catalog_demographic_outcome_candidates(dataset_root: Path) -> list[str]:
   demographic_tokens = ("age", "sex", "gender", "ethnicity", "race")
   outcome_tokens = ("mortality", "death", "deaths", "disease", "prevalence")
   candidates: list[str] = []

   for csv_path in dataset_root.rglob("*.csv"):
      normalized = csv_path.as_posix().lower()
      if any(token in normalized for token in demographic_tokens) and any(
         token in normalized for token in outcome_tokens
      ):
         candidates.append(str(csv_path.relative_to(dataset_root)))

   return sorted(candidates)


def validate_extracted_table(
   df: pl.DataFrame,
   spec: EquityTableSpec,
) -> RawEquityTableProfile:
   missing_columns = [column for column in spec.required_columns if column not in df.columns]
   if missing_columns:
      raise ValueError(f"Missing required columns for {spec.table_name}: {missing_columns}")

   row_count = df.height
   if row_count == 0:
      raise ValueError(f"Extracted table is empty: {spec.table_name}")

   null_percentages = {
      column: round(float(df.get_column(column).is_null().sum()) / row_count * 100, 2)
      for column in df.columns
   }
   duplicate_rows = int(df.is_duplicated().sum())

   min_year: int | None = None
   max_year: int | None = None
   if "year" in df.columns:
      year_series = df.get_column("year").cast(pl.Int32, strict=False)
      min_year = year_series.min()
      max_year = year_series.max()
      if min_year is None or max_year is None:
         raise ValueError(f"Year column could not be parsed for {spec.table_name}")
      if min_year < spec.expected_year_min or max_year > spec.expected_year_max:
         raise ValueError(
            f"Unexpected year range for {spec.table_name}: {min_year}-{max_year}"
         )

   category_columns = [column for column in ("age_group", "sex", "ethnicity") if column in df.columns]
   category_distributions = {
      column: {
         str(row[column]): int(row["count"])
         for row in df.group_by(column).len(name="count").sort(column).to_dicts()
      }
      for column in category_columns
   }

   return RawEquityTableProfile(
      table_name=spec.table_name,
      row_count=row_count,
      duplicate_rows=duplicate_rows,
      null_percentages=null_percentages,
      unique_counts={column: df.get_column(column).n_unique() for column in df.columns},
      category_distributions=category_distributions,
      min_year=min_year,
      max_year=max_year,
   )


def extract_demographic_health_data(
   dataset_id: str,
   output_root: Path,
   report_path: Path,
   specs: list[EquityTableSpec] | None = None,
) -> dict[str, pl.DataFrame]:
   extraction_specs = specs or get_equity_table_specs()
   try:
      dataset_root = Path(kagglehub.dataset_download(dataset_id))
   except Exception as exc:
      logger.exception("Failed to download Kaggle dataset")
      raise RuntimeError("Kaggle dataset download failed") from exc

   output_root.mkdir(parents=True, exist_ok=True)
   report_path.parent.mkdir(parents=True, exist_ok=True)

   extracted_tables: dict[str, pl.DataFrame] = {}
   extraction_report: dict[str, Any] = {
      "dataset_id": dataset_id,
      "demographic_outcome_candidates": catalog_demographic_outcome_candidates(dataset_root),
      "tables": {},
   }

   for spec in extraction_specs:
      source_path = dataset_root / spec.source_relative_path
      if not source_path.exists():
         raise FileNotFoundError(f"Source file not found: {source_path}")

      try:
         df = pl.read_csv(source_path)
      except Exception as exc:
         logger.exception(f"Failed to load {source_path}")
         raise RuntimeError(f"Failed to read {source_path}") from exc

      profile = validate_extracted_table(df, spec)
      destination = output_root / spec.output_relative_path
      destination.parent.mkdir(parents=True, exist_ok=True)
      shutil.copy2(source_path, destination)

      extracted_tables[spec.table_name] = df
      extraction_report["tables"][spec.table_name] = profile.model_dump()
      logger.info(
         f"Extracted {spec.table_name} with {profile.row_count} rows to {destination}"
      )

   report_path.write_text(json.dumps(extraction_report, indent=2), encoding="utf-8")
   return extracted_tables
```

#### 7.2 Data Schemas

Use the executable Pydantic models above as the runtime source of truth:
- `EquityTableSpec` defines the contract for each source table.
- `RawEquityTableProfile` defines the persisted validation profile for each extracted table.
- `shared/data/schemas/equity_raw_schema.yml` should mirror these rules for analyst review and CI checks, but runtime validation must remain in Python.

#### 7.3 Data Validation Rules

Validation logic for this story must enforce all of the following:
- Required demographic columns are present before the table is copied into `shared/data/1_raw/equity/`.
- Null reporting is percentage-based for every extracted column.
- Duplicate detection reports row impact, even if duplicates are allowed temporarily in raw copies.
- Categorical distributions are recorded for `age_group`, `sex`, and `ethnicity` when present.
- Year coverage is validated against the documented minimum and maximum range for each table.
- Outcome-table availability is audited separately from table validation so the absence of demographic outcome data becomes an explicit decision point.

#### 7.4 Library-Specific Patterns

- Use `pl.read_csv()` for raw extraction because the source files are small and copying them unchanged is part of the requirement.
- Use `Path` objects for all file locations; do not concatenate raw strings for filesystem paths.
- Use `logger.exception()` when wrapping external failures such as Kaggle download or CSV loading.
- Keep raw files immutable; any string normalization, schema harmonization, or demographic relabeling belongs in PS-005-US-02.

#### 7.5 Test Specifications

```python
import polars as pl
import pytest

from shared.src.data_processing.extractors.equity_demographic_extractor import (
   EquityTableSpec,
   validate_extracted_table,
)


def test_validate_extracted_table_rejects_missing_columns() -> None:
   spec = EquityTableSpec(
      table_name="hospital_admissions_age_sex",
      source_relative_path="unused.csv",
      output_relative_path="equity/utilization/example.csv",
      required_columns=["year", "age_group", "sex"],
      expected_year_min=2006,
      expected_year_max=2020,
      category="utilization",
   )
   df = pl.DataFrame({"year": [2006], "age_group": ["15-24"]})

   with pytest.raises(ValueError, match="Missing required columns"):
      validate_extracted_table(df, spec)


def test_validate_extracted_table_profiles_expected_shape() -> None:
   spec = EquityTableSpec(
      table_name="hospital_admissions_age_sex",
      source_relative_path="unused.csv",
      output_relative_path="equity/utilization/example.csv",
      required_columns=["year", "age_group", "sex"],
      expected_year_min=2006,
      expected_year_max=2020,
      category="utilization",
   )
   df = pl.DataFrame(
      {
         "year": [2006, 2007],
         "age_group": ["15-24", "25-34"],
         "sex": ["Male", "Female"],
         "admission_rate": [12.4, 14.1],
      }
   )

   profile = validate_extracted_table(df, spec)

   assert profile.row_count == 2
   assert profile.duplicate_rows == 0
   assert profile.null_percentages["sex"] == 0.0
   assert profile.min_year == 2006
   assert profile.max_year == 2007
```

#### 7.6 Package Management

- Install dependencies with `uv pip install "polars>=0.20.0" "pydantic>=2.5.0" "kagglehub>=0.2.0" "loguru>=0.7.0" "pyyaml>=6.0" "pytest>=8.0"`.
- Refresh dependency capture with `uv pip freeze > requirements.txt` after the extractor and tests are in place.

### 8. Domain-Driven Feature Engineering

This story should only select domain dimensions that are verified from available data.

**Step 1: Relevant domain knowledge**
- Equity analysis relies on demographic stratifiers, reference-group comparisons, and service-type access patterns.
- Immediate extraction-relevant dimensions: `year`, `age_group`, `sex`, `service_type`, `admission_rate`, and any explicit outcome-rate column discovered in the Kaggle files.

**Step 2: Validate data availability**
- Confirmed: age/sex stratification in hospital admissions.
- Partially confirmed: sex stratification in subsidised polyclinic attendances.
- Not confirmed in source docs: ethnicity, income, education, or demographic outcome tables.

**Step 3: Select only feasible dimensions for this story**
- Include now: `year`, `age_group`, `sex`, `admission_rate`, source table name, and file lineage.
- Defer to later stories: harmonized demographic grouping, disparity ratios, reference-group flags.
- Explicitly reject until verified: ethnicity-specific equity metrics, socioeconomic inequity metrics, disease-specific mortality-by-age/sex extractions.

### 9. API Endpoints and Data Contracts

Not applicable. This story uses file-based Kaggle extraction and local artifact generation, not service endpoints.

### 10. Styling and Visualization

Not applicable. No dashboard or UI should be produced in this extraction story.

### 11. Testing Strategy

**Unit tests**: `shared/tests/unit/test_equity_demographic_extractor.py`
- Happy path: required columns present, expected year range, null percentages reported.
- Edge cases: empty table, missing demographic columns, unparseable year values.
- Output assertions: categorical distributions include expected keys for `age_group` and `sex` when present.

**Data quality tests**: `shared/tests/data/test_equity_raw_schema.py`
- Assert confirmed tables have 0% nulls if the source documentation remains accurate.
- Assert `hospital_admissions_age_sex` spans 2006-2020.
- Assert the row count for `hospital-admission-rate-by-age-and-sex.csv` is 216 unless the upstream dataset version changes.

**Integration tests**: `shared/tests/integration/test_equity_raw_extraction.py`
- Mock Kaggle download root and verify raw CSVs are copied to `shared/data/1_raw/equity/utilization/`.
- Verify profile JSON and outcome availability audit are written to the PS-005 results metrics directory.
- Verify missing demographic outcome candidates do not cause silent success.

**Fixture strategy**
- Build fixtures from tiny Polars DataFrames representing age-sex admissions and sex-only polyclinic utilization.
- Use a temporary directory tree to mimic the Kaggle dataset layout.

### 12. Implementation Steps

**Phase 1: Data Extraction**
- [ ] Create `shared/config/base.yml` and `config/databricks.yml`.
- [ ] Implement `shared/src/data_processing/extractors/equity_demographic_extractor.py`.
- [ ] Download the Kaggle dataset and inventory all CSV files.
- [ ] Extract confirmed utilization tables into `shared/data/1_raw/equity/utilization/`.

**Phase 2: Data Cleaning**
- [ ] Do not transform raw CSV contents.
- [ ] Generate only profiling metadata and schema validation outputs for raw tables.
- [ ] [ADDED - Issue: demographic outcome tables unconfirmed] Create an explicit outcome-table availability audit before any downstream cleaning assumptions are made.

**Phase 3: Exploratory Data Analysis**
- [ ] Produce a lightweight extraction profile JSON with row counts, null percentages, unique demographic counts, and year coverage.
- [ ] Review demographic distributions for obviously malformed categories that should be handled in PS-005-US-02.

**Phase 4: Feature Engineering**
- [ ] None in this story beyond source-table metadata and profile artifacts.

**Phase 5: Modeling or Analysis**
- [ ] None in this story; hand off extracted assets and documented gaps to PS-005-US-02 and PS-005-US-04.

### 13. Adaptive Implementation Strategy

This implementation plan is intentionally conditional on actual extraction outputs.

- If the Kaggle dataset contains demographic outcome tables, extend `shared/data/schemas/equity_raw_schema.yml` and copy those tables into `shared/data/1_raw/equity/outcomes/` during the same run.
- If outcome tables are absent, insert a blocker note into the results metrics artifact and update downstream story assumptions before moving to disparity analysis.
- If nulls are discovered despite the source documentation claiming 100% completeness, add raw anomaly logging and quarantine affected tables in `shared/data/3_interim/equity/` before any integration work.
- If schemas differ from the documented conventions, update the runtime specs first, then mirror the change in the YAML schema file and PS-005-US-02 plan.

### 14. Code Generation Order

1. Create `shared/config/base.yml`.
2. Create `config/databricks.yml`.
3. Create `shared/data/schemas/equity_raw_schema.yml`.
4. Create `shared/src/data_processing/extractors/equity_demographic_extractor.py`.
5. Create `shared/tests/conftest.py`.
6. Create `shared/tests/unit/test_equity_demographic_extractor.py`.
7. Create `shared/tests/integration/test_equity_raw_extraction.py`.
8. Create `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/README.md`.

### 15. Data Quality and Validation

**Source validation**
- Verify the source dataset ID and dataset cache path.
- Verify file existence for each confirmed utilization table.

**Transformation validation**
- Confirm raw files are copied byte-for-byte and not rewritten through a transformed export path.
- Confirm profile JSON matches the DataFrame-derived counts and null percentages.

**Output validation**
- Required fields must have 0% nulls unless a newly discovered upstream issue is documented.
- Duplicate row counts must be reported even if zero.
- Demographic category distributions must be present for every detected categorical field.
- Year coverage must be within the expected bounds per table.

**Healthcare data quality notes**
- Data is aggregated and low sensitivity, but still belongs to a healthcare domain and should be handled with auditability.
- No PHI or direct identifiers are expected, but logs should avoid echoing credentials or full secret values.

### 16. Statistical Analysis and Modeling

Not applicable for this extraction story. Only descriptive profiling statistics are required here.

### 17. Model Operations

Not applicable for this extraction story.

### 18. UI and Dashboard Testing

Not applicable for this extraction story.

### 19. Success Metrics and Monitoring

**Business metrics**
- MOH analysts can locate all confirmed demographic raw tables in one predictable folder tree.
- Downstream PS-005 stories inherit a documented answer to the question, "Do demographic outcome tables exist in the source dataset?"

**Technical monitoring**
- Extraction success rate: `100%` for all confirmed utilization tables.
- Schema pass rate: `100%` for required columns and year coverage.
- Audit completeness: one extraction log, one raw profile JSON, one outcome availability JSON per run.

**Alerting**
- Raise an error for missing confirmed source files.
- Raise a warning for unconfirmed outcome tables not found.
- Raise an error for empty extracts or unexpected year ranges.

### 20. References

- `docs/project-context/data-sources.md` - source dataset ID, confirmed utilization tables, record counts, and quality expectations.
- `docs/project-context/tech-stack.md` - Databricks runtime, Polars, `uv`, `loguru`, and testing guidance.
- `docs/objectives/problem_statements/ps-005-healthcare-equity-disparities.md` - business objective, stakeholder context, and known equity-data limitations.
- `docs/objectives/user_stories/problem-statement-003-capacity/01-extract-capacity-utilization-data.md` - design reference for shared extraction of hospital admission data.
- `docs/data-dictionary/index.md` - raw/interim/processed directory conventions and schema-file expectations.
- `README.md` - intended repository structure for `shared/` and `problem-statement/` assets.

### 21. Security and Privacy

**PII and PHI handling**
- Expected extracted data is aggregated and should not contain direct identifiers.
- Treat demographic categories as sensitive analytical context, not personal data.
- Do not add any row-level or subject-level datasets to `shared/data/1_raw/equity/` without revisiting this plan.

**Access controls**
- Raw extracts should remain in shared project storage with team-only access.
- Result metrics artifacts should be readable by analysts but not expose secrets.
- Kaggle credentials must come from environment variables or `~/.kaggle/kaggle.json`, never committed to the repository.

**Retention and disposal**
- Retain raw Kaggle copies only as needed for reproducibility.
- Rotate ETL logs according to Databricks or workspace policy.

### 22. Version Control

- Branch from `main` using `feature/ps-005-equity-raw-extraction`.
- Prefer conventional commits such as `feat(ps-005): add equity demographic extractor` and `test(ps-005): add extraction validation tests`.
- Require passing unit and integration tests before merge.

### 23. Multi-Agent Orchestration

Optional, not required. If agent orchestration is used, the appropriate sequence is:
- `data-extractor` to implement and run the extractor.
- `data-validation` to verify raw schema, completeness, and demographic coverage.

This story is simple enough to execute without a multi-agent pipeline.

### 24. Quality Metrics

**Self-assessment checklist**
- [x] Specificity: concrete file paths, function names, and output artifacts defined.
- [x] Completeness: all critical sections included; conditional sections explicitly evaluated.
- [x] Executability: Python code blocks include imports, type hints, and full implementations.
- [x] Testability: unit and integration test assertions specified.
- [x] Traceability: all source assumptions tied back to documented project files.

### 25. Instruction File Compliance

The repository references some instruction-file names that are not present as files in the current workspace. This plan is therefore aligned against the effective source documents that do exist.

| Source | Key requirements applied | Verified |
|---|---|---|
| `.github/copilot-instructions.md` | Polars-first processing, `uv`, `loguru`, shared/problem-statement structure | Yes |
| `docs/project-context/tech-stack.md` | Databricks 13.3.x, Kaggle download, Python 3.9+, pytest | Yes |
| `docs/project-context/data-sources.md` | dataset ID, confirmed tables, completeness expectations | Yes |
| `docs/data-dictionary/index.md` | raw/schema/interim/processed folder conventions | Yes |
| `README.md` | workflow and target repository structure | Yes |

### 26. Code Generation Readiness Checklist

- [x] Output location verified - plan appended to this user story file.
- [x] Code execution validated - Python example blocks were syntax-checked before inclusion.
- [x] Function signatures with complete type hints included.
- [x] Data schemas defined with executable Pydantic models.
- [x] Specific library methods identified (`kagglehub.dataset_download`, `pl.read_csv`, `logger.exception`).
- [x] Config file structure specified.
- [x] Test assertions with expected values included.
- [x] Import statements included for all Python blocks.
- [x] Error handling patterns included.
- [x] Logging destinations specified.
- [x] Validation rules defined.
- [x] Example input and output paths documented.
- [x] Technical constraints captured.
- [x] Security requirements addressed.
- [x] Version control strategy specified.
- [x] Package management uses `uv`.
- [x] Code generation order specified.
- [x] Test fixtures specified.
- [x] Performance benchmarks specified.