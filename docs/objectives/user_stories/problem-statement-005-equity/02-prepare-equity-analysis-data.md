# Prepare & Integrate Equity Analysis Datasets (Lifecycle Stage: Data Preparation)

**Story ID**: PS-005-US-02  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer preparing health equity datasets**,  
I want **to integrate and standardize demographic health data with consistent demographic categorizations and unified schemas**,  
So that **equity analysts can perform disparity assessments comparing utilization and outcomes across demographic groups**.

---

## 🎯 Acceptance Criteria

1. **Schema standardization**
   - Unified schema: `year`, `demographic_group`, `age_group`, `sex`, `metric_type`, `metric_value`
   - Demographic categories standardized
   - Data types enforced

2. **Data integration**
   - Utilization and outcome data integrated
   - Common demographic groups aligned
   - Time periods aligned

3. **Derived metrics**
   - Rate ratios: high-use group / low-use group
   - Disparity indices calculated
   - Reference group defined (e.g., overall population average)

4. **Output**
   - Integrated: `shared/data/3_interim/equity_analysis_integrated.parquet`
   - Schema: `shared/data/schemas/equity_integrated_schema.yml`
   - Test coverage: ≥80%

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#equity-analysis-methods)
- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objective-1)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-005-US-01 (Extract equity data - BLOCKING)
- **Data Sources**: `shared/data/1_raw/equity/**/*.csv`

---

## ✅ Implementation Tasks

### Schema Standardization
- [ ] Define unified demographic schema
- [ ] Standardize age group categories
- [ ] Standardize sex categories
- [ ] Map source data to standard schema

### Data Integration
- [ ] Load all demographic health datasets
- [ ] Align demographic categories
- [ ] Join utilization and outcome data
- [ ] Handle missing demographic combinations

### Derived Metrics
- [ ] Calculate reference group averages
- [ ] Calculate rate ratios for each demographic
- [ ] Compute absolute disparities
- [ ] Flag significant disparities

### Testing & Documentation
- [ ] Unit tests
- [ ] Validation tests
- [ ] Docstrings

---

## 📌 Notes

**Polars Integration**:
```python
import polars as pl

df_util = pl.read_csv("shared/data/1_raw/equity/utilization/admissions.csv")

# Calculate rate ratios
df_equity = (
    df_util.with_columns([
        pl.col('admission_rate').mean().alias('reference_rate')
    ])
    .with_columns([
        (pl.col('admission_rate') / pl.col('reference_rate')).alias('rate_ratio')
    ])
)

# Flag disparities (rate ratio >1.5 or <0.67)
df_equity = df_equity.with_columns([
    ((pl.col('rate_ratio') > 1.5) | (pl.col('rate_ratio') < 0.67)).alias('disparity_flag')
])
```

## Implementation Plan

### 1. Feature Overview

Prepare a unified equity-analysis dataset by standardizing demographic categories from PS-005-US-01 raw extracts, aligning time periods and metric semantics, and generating first-pass disparity features that downstream utilization and outcome analyses can consume without redoing schema work.

**Primary User Role**: Data Engineer preparing health equity datasets.

**Primary Goal**: Materialize a canonical long-form interim dataset at `shared/data/3_interim/equity_analysis_integrated.parquet` using the required schema `year`, `demographic_group`, `age_group`, `sex`, `metric_type`, `metric_value`, plus documented derived columns for reference rates and disparity flags.

### 2. Component Analysis & Reuse Strategy

**Reusable assets already available or planned upstream**:
- `docs/project-context/data-sources.md` - confirms the known raw inputs, row counts, and limitations.
- `docs/project-context/tech-stack.md` - defines Databricks, Polars, `uv`, `loguru`, and test expectations.
- `docs/data-dictionary/index.md` - defines target raw, interim, processed, and schema directories.
- `docs/objectives/problem_statements/ps-005-healthcare-equity-disparities.md` - defines the business need for unified demographic comparisons and highlights limited socioeconomic granularity.
- `docs/objectives/user_stories/problem-statement-005-equity/01-extract-demographic-health-data.md` - upstream implementation-plan contract for expected raw files, profile artifacts, and the outcome-table availability audit.

**Code reuse status**:
- No executable repository code exists yet, so there is nothing to reuse from the filesystem today.
- This story should be designed to consume the outputs that PS-005-US-01 is planned to create, not assume those modules are already checked into the repo.

**Gaps requiring new components**:
- Integration module that reads raw equity CSVs and normalizes them into one long-form schema.
- Shared normalization helpers for age and sex categories.
- Integrated schema file for both analyst review and CI validation.
- Validation tests for demographic standardization and disparity metric derivation.
- Conditional handling of missing outcome rows because story 01 may discover that no demographic outcome tables are available in the Kaggle bundle.

### 3. ML Model Evaluation & Selection

Not applicable. This is a data preparation and feature-standardization story, not a predictive or ML story.

### 4. Affected Files

- **[MODIFY] `shared/config/base.yml`**
  - Add keys: `paths.raw_equity_root`, `paths.interim_equity_root`, `paths.ps005_metrics_root`, `thresholds.high_disparity_ratio`, `thresholds.low_disparity_ratio`.
  - Dependencies: `pyyaml`.
  - Logging: consumed by integration pipeline.

- **[CREATE] `shared/data/schemas/equity_integrated_schema.yml`**
  - Purpose: document canonical integrated schema and field constraints.
  - Required columns: `year`, `demographic_group`, `age_group`, `sex`, `metric_type`, `metric_value`.
  - Optional derived columns: `reference_rate`, `rate_ratio`, `absolute_disparity`, `disparity_flag`, `source_table`, `source_category`.

- **[CREATE] `shared/src/data_processing/integrators/equity_data_integrator.py`**
  - Functions:
    - `canonicalize_age_group(value: str | None) -> str`
    - `canonicalize_sex(value: str | None) -> str`
    - `build_demographic_group(age_group: str, sex: str) -> str`
    - `normalize_equity_source(df: pl.DataFrame, spec: EquityMetricSourceSpec) -> pl.DataFrame`
    - `integrate_equity_frames(frames: Iterable[pl.DataFrame]) -> pl.DataFrame`
    - `calculate_reference_and_disparity_metrics(df: pl.DataFrame) -> pl.DataFrame`
    - `write_integrated_equity_dataset(df: pl.DataFrame, output_path: Path) -> Path`
  - Dependencies: `polars`, `loguru`, `pydantic`, `pathlib`, `typing`.
  - Config: `shared/config/base.yml`.
  - Logging: `logs/etl/equity_integration_{timestamp}.log`.

- **[CREATE] `shared/src/utils/equity_schema_validator.py`**
  - Functions:
    - `validate_integrated_equity_schema(df: pl.DataFrame) -> tuple[bool, list[str]]`
    - `profile_integrated_equity_data(df: pl.DataFrame) -> dict[str, object]`
  - Dependencies: `polars`, `pydantic`, `yaml`.

- **[CREATE] `shared/tests/unit/test_equity_data_integrator.py`**
  - Test targets:
    - age/sex canonicalization;
    - normalized schema shape;
    - rate-ratio and disparity-flag derivations.

- **[CREATE] `shared/tests/integration/test_equity_integration_pipeline.py`**
  - Test targets:
    - raw utilization inputs normalize into the canonical schema;
    - optional outcome inputs append without schema drift;
    - interim parquet and profile metrics artifacts are created.

- **[MODIFY] `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/README.md`**
  - Add integrated-data artifacts:
    - `equity_integrated_profile_{timestamp}.json`
    - `equity_missing_outcome_gap_{timestamp}.json`
    - `equity_reference_rates_{timestamp}.csv`

### 5. Component Breakdown

**New component: `equity_data_integrator`**
- Location: `shared/src/data_processing/integrators/equity_data_integrator.py`
- Responsibility: standardize raw demographic tables into a single long-form interim dataset and add first-pass disparity metrics.
- Key parameters: source specs, disparity thresholds, output path, whether outcome tables are required for the current run.
- Technical constraints:
  - Memory budget: `< 300 MB` local.
  - Execution target: `< 5 minutes` for all raw PS-005 inputs and integrated parquet generation.
  - Data volume strategy: eager `pl.read_csv()` is sufficient because the source files are small; use `pl.scan_parquet()` in downstream stories when repeatedly reading the integrated parquet.
  - Optimization requirements: cast `year` to `Int32`, `metric_value` and derived ratios to `Float64`, treat `age_group`, `sex`, `metric_type`, and `demographic_group` as low-cardinality string dimensions suitable for categorical casting in downstream analytic steps.

**New component: `equity_schema_validator`**
- Location: `shared/src/utils/equity_schema_validator.py`
- Responsibility: enforce required columns, allowed categorical domains, time coverage, duplicate constraints, and derived-metric completeness.

**Modified component: PS-005 metrics README**
- Location: `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/README.md`
- Responsibility: keep integration-stage artifacts discoverable and explain how missing outcome availability propagates to downstream stories.

### 6. Data Pipeline

**Grounded inputs from upstream story 01**:
- Confirmed raw utilization files expected under `shared/data/1_raw/equity/utilization/`.
- Outcome-table availability audit expected under `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/`.
- Raw schema/profile expected under `shared/data/schemas/equity_raw_schema.yml` and metrics JSON.

**Pre-implementation validation**:
1. Confirm story 01 outputs exist or document that integration is blocked.
2. Read the outcome availability audit before assuming any outcome data can be joined.
3. Inventory all CSV files under `shared/data/1_raw/equity/` and classify each as `utilization` or `outcomes`.
4. Profile raw files for year coverage, duplicate row counts, categorical domains, and metric column names.
5. Build a canonical mapping table for age-group and sex normalization before any concatenation.

**Canonical integrated design**:
- The interim parquet must contain the required six-column core schema:
  - `year`
  - `demographic_group`
  - `age_group`
  - `sex`
  - `metric_type`
  - `metric_value`
- Derived columns may be appended in the integrated output because the acceptance criteria also require disparity features:
  - `reference_rate`
  - `rate_ratio`
  - `absolute_disparity`
  - `disparity_flag`
  - `source_table`
  - `source_category`

**Integration rules**:
- Normalize all source tables into a long form with one metric per row.
- Standardize age labels such as `15 - 24 years` and `15-24 years` to `15-24`.
- Standardize sex labels such as `m`, `male`, `female`, `both sexes`, and `total` to `Male`, `Female`, or `All`.
- Create `demographic_group` as a deterministic composite key: `age_group|sex`, except `All` when both are `All`.
- Align years using the intersection of years available for each metric family when cross-domain comparisons are required.
- If no outcome rows are available, produce a utilization-only integrated dataset and emit a gap artifact that blocks full acceptance of the utilization-plus-outcome integration criterion.

**Pipeline specification**:

| Stage | Input | Output | Notes |
|---|---|---|---|
| Raw file discovery | `shared/data/1_raw/equity/**/*.csv` | source inventory table | classify by metric family and demographic coverage |
| Normalization | individual raw tables | standardized long-form frames | apply age/sex canonicalization and metric naming |
| Integration | normalized frames | `shared/data/3_interim/equity_analysis_integrated.parquet` | concatenate all available frames |
| Derived metrics | integrated frame | same parquet with reference fields | calculate reference rate, ratio, absolute disparity, flag |
| Validation and profiling | integrated frame | `equity_integrated_profile_{timestamp}.json` | row counts, year coverage, distinct demographics, null % |
| Gap handling | outcome audit + integrated frame | `equity_missing_outcome_gap_{timestamp}.json` | required when outcome rows are absent |

### 7. Code Generation Specifications

#### 7.1 Function Signatures and Complete Implementations

```python
from __future__ import annotations

from pathlib import Path
from typing import Iterable

import polars as pl
from loguru import logger
from pydantic import BaseModel, Field


CANONICAL_SEX_MAP = {
    "m": "Male",
    "male": "Male",
    "men": "Male",
    "f": "Female",
    "female": "Female",
    "women": "Female",
    "both sexes": "All",
    "persons": "All",
    "total": "All",
    "all": "All",
}

CANONICAL_AGE_MAP = {
    "15 - 24 years": "15-24",
    "15-24 years": "15-24",
    "25 - 34 years": "25-34",
    "25-34 years": "25-34",
    "65 years and over": "65+",
    "65 years & over": "65+",
    "all ages": "All",
    "total": "All",
}


class EquityIntegratedRecord(BaseModel):
    year: int = Field(ge=1900, le=2100)
    demographic_group: str
    age_group: str
    sex: str
    metric_type: str
    metric_value: float


class EquityMetricSourceSpec(BaseModel):
    table_name: str
    source_category: str
    metric_type: str
    metric_value_column: str
    demographic_columns: list[str]
    year_column: str = "year"


def canonicalize_age_group(value: str | None) -> str:
    if value is None:
        return "Unknown"
    normalized = str(value).strip()
    return CANONICAL_AGE_MAP.get(normalized.lower(), CANONICAL_AGE_MAP.get(normalized, normalized))


def canonicalize_sex(value: str | None) -> str:
    if value is None:
        return "Unknown"
    normalized = str(value).strip()
    return CANONICAL_SEX_MAP.get(normalized.lower(), normalized.title())


def build_demographic_group(age_group: str, sex: str) -> str:
    if age_group == "All" and sex == "All":
        return "All"
    return f"{age_group}|{sex}"


def normalize_equity_source(
    df: pl.DataFrame,
    spec: EquityMetricSourceSpec,
) -> pl.DataFrame:
    required_columns = {spec.year_column, spec.metric_value_column, *spec.demographic_columns}
    missing_columns = sorted(required_columns.difference(df.columns))
    if missing_columns:
        raise ValueError(f"Missing required columns for {spec.table_name}: {missing_columns}")

    age_expr = (
        pl.col("age_group").map_elements(canonicalize_age_group, return_dtype=pl.String)
        if "age_group" in spec.demographic_columns
        else pl.lit("All")
    )
    sex_expr = (
        pl.col("sex").map_elements(canonicalize_sex, return_dtype=pl.String)
        if "sex" in spec.demographic_columns
        else pl.lit("All")
    )

    normalized_df = (
        df.with_columns([
            pl.col(spec.year_column).cast(pl.Int32).alias("year"),
            pl.col(spec.metric_value_column).cast(pl.Float64).alias("metric_value"),
            age_expr.alias("age_group"),
            sex_expr.alias("sex"),
            pl.lit(spec.metric_type).alias("metric_type"),
        ])
        .with_columns([
            pl.struct(["age_group", "sex"])
            .map_elements(
                lambda row: build_demographic_group(row["age_group"], row["sex"]),
                return_dtype=pl.String,
            )
            .alias("demographic_group")
        ])
        .select([
            "year",
            "demographic_group",
            "age_group",
            "sex",
            "metric_type",
            "metric_value",
        ])
    )

    logger.info(
        f"Normalized {spec.table_name}: {normalized_df.height} rows, "
        f"{normalized_df.select('demographic_group').n_unique()} demographic groups"
    )
    return normalized_df


def integrate_equity_frames(frames: Iterable[pl.DataFrame]) -> pl.DataFrame:
    materialized = [frame for frame in frames if frame.height > 0]
    if not materialized:
        raise ValueError("No normalized equity frames were provided")
    return pl.concat(materialized, how="vertical_relaxed")


def calculate_reference_and_disparity_metrics(df: pl.DataFrame) -> pl.DataFrame:
    required_columns = {
        "year",
        "demographic_group",
        "age_group",
        "sex",
        "metric_type",
        "metric_value",
    }
    missing_columns = sorted(required_columns.difference(df.columns))
    if missing_columns:
        raise ValueError(f"Integrated dataset missing columns: {missing_columns}")

    reference_df = (
        df.group_by(["year", "metric_type"])
        .agg(pl.mean("metric_value").alias("reference_rate"))
    )

    metrics_df = (
        df.join(reference_df, on=["year", "metric_type"], how="left")
        .with_columns([
            (pl.col("metric_value") / pl.col("reference_rate")).alias("rate_ratio"),
            (pl.col("metric_value") - pl.col("reference_rate")).alias("absolute_disparity"),
            (
                (pl.col("metric_value") / pl.col("reference_rate") > 1.5)
                | (pl.col("metric_value") / pl.col("reference_rate") < 0.67)
            ).alias("disparity_flag")
        ])
    )
    return metrics_df


def write_integrated_equity_dataset(df: pl.DataFrame, output_path: Path) -> Path:
    output_path.parent.mkdir(parents=True, exist_ok=True)
    df.write_parquet(output_path)
    logger.info(f"Integrated equity dataset written to {output_path}")
    return output_path
```

#### 7.2 Data Schemas

Use `EquityIntegratedRecord` as the executable contract for the core schema. The YAML schema file should mirror the same required columns and define the derived columns as optional analytical enrichments.

Recommended YAML structure:

```yaml
equity_integrated:
  required_columns:
    year: Int32
    demographic_group: String
    age_group: String
    sex: String
    metric_type: String
    metric_value: Float64
  optional_columns:
    reference_rate: Float64
    rate_ratio: Float64
    absolute_disparity: Float64
    disparity_flag: Boolean
    source_table: String
    source_category: String
  constraints:
    year_min: 1990
    year_max: 2020
    allowed_sex_values: [Male, Female, All, Unknown]
```

#### 7.3 Data Validation Rules

- Required core schema columns must exist in the parquet output.
- `demographic_group` must be deterministic and non-null.
- `sex` must be standardized to `Male`, `Female`, `All`, or `Unknown`.
- `age_group` must use the canonical labels defined by the normalization map.
- Null reporting must be percentage-based across both core and derived columns.
- Duplicate detection must be evaluated on `year`, `demographic_group`, and `metric_type`.
- If outcome rows are absent, a gap artifact must be emitted and the integrated profile must report `outcome_rows = 0` explicitly.

#### 7.4 Library-Specific Patterns

- Use `pl.concat(..., how="vertical_relaxed")` to allow concatenation of normalized frames that may carry optional metadata columns later.
- Use `map_elements()` only for dimension standardization where categorical remapping is unavoidable; keep the rest of the pipeline vectorized.
- Write the integrated dataset as Parquet because downstream stories will scan it repeatedly.
- Keep raw files immutable; standardization occurs only in the interim integration layer.

#### 7.5 Test Specifications

```python
import polars as pl

from shared.src.data_processing.integrators.equity_data_integrator import (
    EquityMetricSourceSpec,
    calculate_reference_and_disparity_metrics,
    normalize_equity_source,
)


def test_normalize_equity_source_standardizes_age_and_sex() -> None:
    df = pl.DataFrame(
        {
            "year": [2006, 2006],
            "age_group": ["15 - 24 years", "65 years and over"],
            "sex": ["m", "female"],
            "admission_rate": [120.0, 250.0],
        }
    )
    spec = EquityMetricSourceSpec(
        table_name="hospital_admissions_age_sex",
        source_category="utilization",
        metric_type="hospital_admission_rate",
        metric_value_column="admission_rate",
        demographic_columns=["age_group", "sex"],
    )

    result = normalize_equity_source(df, spec)

    assert result.columns == [
        "year",
        "demographic_group",
        "age_group",
        "sex",
        "metric_type",
        "metric_value",
    ]
    assert result["age_group"].to_list() == ["15-24", "65+"]
    assert result["sex"].to_list() == ["Male", "Female"]


def test_calculate_reference_and_disparity_metrics_adds_expected_fields() -> None:
    df = pl.DataFrame(
        {
            "year": [2006, 2006],
            "demographic_group": ["15-24|Male", "65+|Female"],
            "age_group": ["15-24", "65+"],
            "sex": ["Male", "Female"],
            "metric_type": ["hospital_admission_rate", "hospital_admission_rate"],
            "metric_value": [100.0, 200.0],
        }
    )

    result = calculate_reference_and_disparity_metrics(df)

    assert result["reference_rate"].to_list() == [150.0, 150.0]
    assert result["rate_ratio"].round(3).to_list() == [0.667, 1.333]
    assert result["absolute_disparity"].to_list() == [-50.0, 50.0]
    assert result["disparity_flag"].to_list() == [True, False]
```

#### 7.6 Package Management

- Install or update dependencies with `uv pip install "polars>=0.20.0" "pydantic>=2.5.0" "loguru>=0.7.0" "pyyaml>=6.0" "pytest>=8.0"`.
- Update dependency capture with `uv pip freeze > requirements.txt` after implementation.

### 8. Domain-Driven Feature Engineering

**Step 1: Identify relevant domain concepts**
- Demographic standardization is foundational for health equity comparisons.
- Reference-group comparisons must be consistent across utilization and outcome metrics.
- Intersectional grouping for age and sex is the minimum viable dimension set supported by confirmed data.

**Step 2: Validate data availability**
- Confirmed now: age/sex hospital admissions and sex-level polyclinic attendance.
- Possible but not guaranteed: demographic-stratified outcome tables.
- Not supported by current documented sources: direct socioeconomic or neighborhood-level stratifiers.

**Step 3: Select only feasible features**
- Include now: `demographic_group`, canonical `age_group`, canonical `sex`, `metric_type`, `metric_value`, `reference_rate`, `rate_ratio`, `absolute_disparity`, `disparity_flag`.
- Defer until data exists: SMR, DALY disparities, income/ethnicity inequity metrics, concentration indices by socioeconomic rank.

### 9. API Endpoints and Data Contracts

Not applicable. This story is file-based ETL and interim-data preparation.

### 10. Styling and Visualization

Not applicable. This story should not produce dashboard assets.

### 11. Testing Strategy

**Unit tests**: `shared/tests/unit/test_equity_data_integrator.py`
- Canonicalize age categories.
- Canonicalize sex categories.
- Normalize individual source tables to the six-column core schema.
- Verify rate ratio, reference rate, and disparity flag calculations.

**Data quality tests**: `shared/tests/data/test_equity_integrated_schema.py`
- Assert required columns are present in the parquet output.
- Assert no nulls in `year`, `demographic_group`, `metric_type`, and `metric_value`.
- Assert duplicate keys are absent on `year`, `demographic_group`, `metric_type` after aggregation.
- Assert allowed `sex` values remain within the canonical set.

**Integration tests**: `shared/tests/integration/test_equity_integration_pipeline.py`
- Mock raw CSV inputs and verify the integrated parquet is written.
- Verify missing outcome inputs emit a gap artifact but still allow a utilization-only interim dataset.
- Verify available outcome inputs append without schema drift.

### 12. Implementation Steps

**Phase 1: Data Extraction**
- [ ] Confirm PS-005-US-01 outputs exist and inspect the outcome availability audit.
- [ ] Enumerate all raw equity CSV files and map them to source specs.

**Phase 2: Data Cleaning**
- [ ] Create age-group and sex normalization maps.
- [ ] Normalize each source table into the canonical six-column core schema.
- [ ] [ADDED - Issue: outcome availability may be zero] Branch integration logic based on whether outcome rows were actually produced by story 01.

**Phase 3: Exploratory Data Analysis**
- [ ] Profile the integrated dataset for demographic coverage, year overlap, and metric counts.
- [ ] Emit `equity_integrated_profile_{timestamp}.json`.

**Phase 4: Feature Engineering**
- [ ] Calculate `reference_rate`, `rate_ratio`, `absolute_disparity`, and `disparity_flag`.
- [ ] Emit a small reference-rate artifact for analyst traceability.

**Phase 5: Modeling or Analysis**
- [ ] Stop at interim data preparation and hand off the parquet to PS-005-US-03 and PS-005-US-04.

### 13. Adaptive Implementation Strategy

- If outcome tables are present, integrate them into the same canonical schema and preserve `source_category = outcomes`.
- If outcome tables are absent, generate the utilization-only integrated parquet, write `equity_missing_outcome_gap_{timestamp}.json`, and mark full cross-domain integration as incomplete.
- If age or sex categories are inconsistent beyond the current maps, extend the canonical maps before re-running the pipeline.
- If raw tables expose conflicting rate bases, add a metric-unit normalization step before disparity calculation and document the conversion in the schema file.

### 14. Code Generation Order

1. Modify `shared/config/base.yml`.
2. Create `shared/data/schemas/equity_integrated_schema.yml`.
3. Create `shared/src/data_processing/integrators/equity_data_integrator.py`.
4. Create `shared/src/utils/equity_schema_validator.py`.
5. Create `shared/tests/unit/test_equity_data_integrator.py`.
6. Create `shared/tests/integration/test_equity_integration_pipeline.py`.
7. Modify `problem-statement/ps-005-healthcare-equity-disparities/results/metrics/README.md`.

### 15. Data Quality and Validation

**Source validation**
- Verify raw file presence under `shared/data/1_raw/equity/`.
- Verify source-to-spec mappings are complete.

**Transformation validation**
- Verify canonicalization maps do not introduce null demographic labels.
- Verify all normalized frames conform to the core schema before concatenation.

**Output validation**
- Required core columns present.
- Null percentages computed for all columns.
- Duplicate-key count reported on `year`, `demographic_group`, `metric_type`.
- Distinct demographic counts and metric counts reported.
- Outcome row count explicitly reported, including zero.

### 16. Statistical Analysis and Modeling

Not applicable beyond descriptive disparity features. No inferential testing or predictive modeling belongs in this data-preparation story.

### 17. Model Operations

Not applicable.

### 18. UI and Dashboard Testing

Not applicable.

### 19. Success Metrics and Monitoring

**Business metrics**
- Equity analysts receive one consistent interim dataset rather than separate incompatible CSV structures.
- Downstream PS-005 stories can calculate disparities without redoing schema harmonization.

**Technical metrics**
- `100%` of available raw equity tables normalize into the canonical six-column core schema.
- `100%` of integrated rows have non-null `demographic_group`, `metric_type`, and `metric_value`.
- One interim parquet, one integrated profile JSON, and one gap artifact when required per run.

### 20. References

- `docs/project-context/data-sources.md` - confirmed source tables and data limitations.
- `docs/project-context/tech-stack.md` - implementation standards.
- `docs/data-dictionary/index.md` - directory conventions.
- `docs/objectives/problem_statements/ps-005-healthcare-equity-disparities.md` - business requirements and equity constraints.
- `docs/objectives/user_stories/problem-statement-005-equity/01-extract-demographic-health-data.md` - upstream raw extraction contract.
- `docs/objectives/user_stories/problem-statement-005-equity/03-analyze-utilization-disparities.md` - downstream consumer of utilization-focused integrated data.
- `docs/objectives/user_stories/problem-statement-005-equity/04-analyze-outcome-disparities.md` - downstream consumer of any verified integrated outcome data.
- `docs/objectives/problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md` - equity metric definitions and data-quality cautions.

### 21. Security and Privacy

- Inputs are expected to remain aggregated and low-risk, but all healthcare-domain outputs should remain auditable.
- Do not persist credentials in config files.
- Do not write logs that expose environment variable contents or secret file payloads.
- Keep any future row-level data out of this pipeline unless the security model is revisited.

### 22. Version Control

- Branch naming: `feature/ps-005-equity-integration`.
- Conventional commits:
  - `feat(ps-005): add equity data integrator`
  - `test(ps-005): add integrated equity schema tests`
  - `docs(ps-005): document integrated equity artifacts`

### 23. Multi-Agent Orchestration

Optional only. If used, the natural sequence is:
- `data-cleaning` for normalization and harmonization.
- `data-validation` for schema and profile verification.

### 24. Quality Metrics

**Self-assessment checklist**
- [x] Specificity: file paths, function names, thresholds, and artifacts are concrete.
- [x] Completeness: all critical sections are included.
- [x] Executability: code blocks were syntax-checked before inclusion.
- [x] Testability: unit and integration assertions are specified.
- [x] Traceability: assumptions are tied to the actual project docs and the upstream story 01 plan.

### 25. Instruction File Compliance

| Source | Key requirements applied | Verified |
|---|---|---|
| `.github/copilot-instructions.md` | Polars-first processing, `uv`, `loguru`, minimal focused components | Yes |
| `docs/project-context/tech-stack.md` | Databricks 13.3.x, Python 3.9+, pytest, file-based Kaggle workflow | Yes |
| `docs/project-context/data-sources.md` | confirmed equity-related inputs and limitations | Yes |
| `docs/data-dictionary/index.md` | `shared/data/3_interim/` and `shared/data/schemas/` conventions | Yes |
| `docs/objectives/user_stories/problem-statement-005-equity/01-extract-demographic-health-data.md` | outcome availability audit and raw file contract | Yes |

### 26. Code Generation Readiness Checklist

- [x] Output location verified - plan appended to this story file.
- [x] Code execution validated - integration code blocks were syntax-checked.
- [x] Function signatures with type hints included.
- [x] Executable schema model included.
- [x] Exact Polars operations identified.
- [x] Config changes specified.
- [x] Test assertions included.
- [x] Error-handling expectations defined.
- [x] Logging destinations specified.
- [x] Validation rules defined.
- [x] Technical constraints captured.
- [x] Security requirements addressed.
- [x] Version-control strategy specified.
- [x] Package management uses `uv`.
- [x] Code generation order specified.
- [x] Test fixtures and integration checks specified.