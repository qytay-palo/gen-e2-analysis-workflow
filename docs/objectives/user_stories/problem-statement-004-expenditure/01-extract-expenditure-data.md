# Extract National Health Expenditure & Financing Data (Lifecycle Stage: Data Extraction)

**Story ID**: PS-004-US-01  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: S (2 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer supporting healthcare financial analysis**,  
I want **to extract national health expenditure tables and financing data from the Kaggle dataset covering 2006-2018**,  
So that **financial analysts have validated expenditure data for cost driver identification and trend analysis**.

---

## 🎯 Acceptance Criteria

1. **Expenditure data extracted**
   - Downloaded: `government-health-expenditure.csv` (13 records, 2006-2018)
   - Stored in `shared/data/1_raw/expenditure/`
   - Metadata preserved including financial year information

2. **Optional supplementary data extracted**
   - Healthcare utilization data (for correlation analysis)
   - Demographic data (for per capita calculations)
   - Facility capacity data (for cost per bed analysis)

3. **Data validation passed**
   - Schema validation: required columns present (financial_year, expenditure, category)
   - Completeness: 0% missing values confirmed
   - Value ranges: expenditure amounts realistic (millions/billions SGD)
   - Time coverage: 2006-2018 confirmed

4. **Extraction documented**
   - Validation log: `logs/etl/expenditure_extraction_YYYYMMDD.log`
   - Schema documented: `shared/data/schemas/expenditure_raw_schema.yml`
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

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#healthcare-expenditure-metrics) - Understanding expenditure categories
- [Data Sources](../../../../project_context/data-sources.md#financial-analysis) - Expenditure table specifications

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `kagglehub>=0.2.0`, `loguru>=0.7.0`, `pyyaml>=6.0`

### Internal Dependencies
- **Upstream**: None (first story in epic)
- **Data Sources**: Kaggle dataset `subhamjain/health-dataset-complete-singapore`
- **Config Files**: `config/databricks.yml`, `shared/config/base.yml`

---

## ✅ Implementation Tasks

### Data Extraction
- [ ] Configure KaggleHub authentication
- [ ] Create extraction script: `shared/src/data_processing/extract_expenditure_data.py`
- [ ] Extract government health expenditure table
- [ ] Extract supplementary tables for context (utilization, demographics)
- [ ] Save to `shared/data/1_raw/expenditure/`

### Validation
- [ ] Define expected schema in YAML
- [ ] Implement Pydantic validation models
- [ ] Validate completeness: 0% nulls
- [ ] Validate row count: 13 records expected
- [ ] Validate financial years: 2006-2018
- [ ] Validate expenditure amounts: positive values, realistic magnitudes

### Testing & Documentation
- [ ] Unit tests for extraction functions
- [ ] Integration test for pipeline
- [ ] Mock API for testing
- [ ] Docstrings (Google style)
- [ ] Update README

---

## 📌 Notes

**Expected Table** (from data-sources.md):
- `government-health-expenditure.csv`
- 13 years: 2006-2018
- Contains: financial year, expenditure categories, amounts

**Polars Extraction**:
```python
import polars as pl
from loguru import logger
import kagglehub

dataset_path = kagglehub.dataset_download(
    "subhamjain/health-dataset-complete-singapore"
)

df_expenditure = pl.read_csv(
    f"{dataset_path}/government-health-expenditure/government-health-expenditure.csv"
)

assert df_expenditure.null_count().sum_horizontal()[0] == 0
logger.info(f"✓ Expenditure data extracted: {len(df_expenditure)} records")

df_expenditure.write_csv("shared/data/1_raw/expenditure/government_health_expenditure.csv")
```

**Known Considerations**:
- Financial year vs calendar year alignment (check documentation)
- Expenditure categories may include: operating costs, capital investments, subsidies
- Amounts may be in millions or billions SGD (verify units)

---

## Implementation Plan

### 1. Feature Overview

Extract, validate, and store Singapore government health expenditure data (2006-2018) from the Kaggle `subhamjain/health-dataset-complete-singapore` dataset. The primary user role is **Data Engineer**.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/src/data_processing/kaggle_connector.py` | Exists | **Reuse** — `KaggleConnector` handles `kagglehub` download |
| `shared/src/data_processing/extractors/mortality_extractor.py` | Exists | **Reference pattern** — mirrors required extractor design |
| `shared/data/schemas/` | Exists (dir) | **Create** new schema YAML for expenditure |
| `shared/data/1_raw/expenditure/` | Missing | **Create** directory via extractor |
| `shared/src/data_processing/extractors/expenditure_extractor.py` | Missing | **Create** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/` | Missing | **Create** full PS-004 folder |

No ML model evaluation required (pure extraction task).

---

### 4. Affected Files

- **[CREATE] `shared/src/data_processing/extractors/expenditure_extractor.py`**
  - `ExpenditureDataExtractor` class
  - `extract()` → `pl.DataFrame`
  - `validate()` → `bool`
  - `save()` → `Path`
  - Deps: `polars`, `kagglehub`, `loguru`, `pydantic`, `yaml`

- **[CREATE] `shared/data/schemas/expenditure_raw_schema.yml`**
  - Column definitions, dtypes, value constraints

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/data_processing/extract_expenditure_data.py`**
  - Orchestration script calling extractor

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_extract_expenditure.py`**
  - Unit tests for extraction, validation, schema

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/01-extract-expenditure-data.ipynb`**
  - Interactive extraction walkthrough

- **[MODIFY] `shared/src/data_processing/extractors/__init__.py`**
  - Export `ExpenditureDataExtractor`

---

### 5. Data Pipeline

**Source**: `government-health-expenditure/government-health-expenditure.csv`  
**Dataset ID**: `subhamjain/health-dataset-complete-singapore`  
**Expected rows**: 13 (one per financial year 2006-2018)  
**Expected completeness**: 100% (no nulls per data-sources.md)

**Pipeline**:
```
kagglehub.dataset_download() 
  → read CSV with Polars 
  → validate schema (columns, dtypes, row count, year range) 
  → validate completeness (0 nulls) 
  → validate value ranges (expenditure > 0, realistic SGD amounts) 
  → save to shared/data/1_raw/expenditure/government_health_expenditure.csv 
  → save schema YAML 
  → log to logs/etl/expenditure_extraction_YYYYMMDD.log
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `expenditure_extractor.py`

```python
"""
Expenditure Data Extractor
==========================

Extracts Singapore government health expenditure data (2006-2018) from Kaggle.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import yaml
from datetime import datetime
from pathlib import Path
from typing import Optional

import kagglehub
import polars as pl
from loguru import logger
from pydantic import BaseModel, field_validator


DATASET_ID = "subhamjain/health-dataset-complete-singapore"
EXPENDITURE_FILE = (
    "government-health-expenditure/government-health-expenditure.csv"
)


class ExpenditureRecord(BaseModel):
    """Pydantic validation model for a single expenditure row."""

    financial_year: str
    total_expenditure: float

    @field_validator("total_expenditure")
    @classmethod
    def expenditure_must_be_positive(cls, v: float) -> float:
        if v <= 0:
            raise ValueError(f"Expenditure must be positive, got {v}")
        return v


class ExpenditureDataExtractor:
    """
    Extract and validate government health expenditure data from Kaggle.

    Example:
        >>> extractor = ExpenditureDataExtractor()
        >>> df = extractor.extract()
        >>> extractor.validate(df)
        >>> extractor.save(df)
    """

    EXPECTED_ROW_COUNT = 13
    EXPECTED_YEAR_RANGE = (2006, 2018)

    def __init__(
        self,
        output_dir: str = "shared/data/1_raw/expenditure",
        schema_file: str = "shared/data/schemas/expenditure_raw_schema.yml",
        log_dir: str = "logs/etl",
    ) -> None:
        """
        Initialize extractor.

        Args:
            output_dir: Directory for raw CSV output.
            schema_file: Path to schema YAML for validation reference.
            log_dir: Directory for ETL log files.
        """
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)

        self.schema_file = Path(schema_file)
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(parents=True, exist_ok=True)

        log_path = (
            self.log_dir
            / f"expenditure_extraction_{datetime.now().strftime('%Y%m%d')}.log"
        )
        logger.add(str(log_path), level="INFO", rotation="1 day")
        logger.info("ExpenditureDataExtractor initialised")

    def extract(self) -> pl.DataFrame:
        """
        Download and load expenditure CSV from Kaggle.

        Returns:
            Raw expenditure DataFrame.

        Raises:
            RuntimeError: If download or parsing fails.
        """
        logger.info(f"Downloading dataset: {DATASET_ID}")
        try:
            dataset_path = kagglehub.dataset_download(DATASET_ID)
        except Exception as exc:
            logger.error(f"Kaggle download failed: {exc}")
            raise RuntimeError(f"Kaggle download failed: {exc}") from exc

        csv_path = Path(dataset_path) / EXPENDITURE_FILE
        if not csv_path.exists():
            raise FileNotFoundError(f"Expenditure CSV not found at: {csv_path}")

        df = pl.read_csv(str(csv_path))
        logger.info(
            f"Loaded {len(df)} rows, {df.width} columns from {csv_path.name}"
        )
        return df

    def validate(self, df: pl.DataFrame) -> bool:
        """
        Validate schema, completeness, row count, and value ranges.

        Args:
            df: Raw expenditure DataFrame.

        Returns:
            True if all checks pass.

        Raises:
            AssertionError: On any validation failure.
        """
        logger.info("Starting data validation...")

        # 1. Row count
        assert len(df) == self.EXPECTED_ROW_COUNT, (
            f"Expected {self.EXPECTED_ROW_COUNT} rows, got {len(df)}"
        )
        logger.info(f"✓ Row count: {len(df)}")

        # 2. Null completeness (0% nulls expected)
        null_counts = df.null_count()
        total_nulls = null_counts.sum_horizontal()[0]
        null_pct = total_nulls / (len(df) * df.width) * 100
        assert total_nulls == 0, (
            f"Found {total_nulls} null values ({null_pct:.1f}%): "
            f"{null_counts.to_dict()}"
        )
        logger.info("✓ Completeness: 0% nulls")

        # 3. Year range — look for financial_year or year column
        year_col = next(
            (c for c in df.columns if "year" in c.lower()), None
        )
        if year_col:
            years = df[year_col].cast(pl.Utf8).str.extract(r"(\d{4})", 1).cast(pl.Int32)
            assert years.min() >= self.EXPECTED_YEAR_RANGE[0], (
                f"Earliest year {years.min()} < {self.EXPECTED_YEAR_RANGE[0]}"
            )
            assert years.max() <= self.EXPECTED_YEAR_RANGE[1], (
                f"Latest year {years.max()} > {self.EXPECTED_YEAR_RANGE[1]}"
            )
            logger.info(
                f"✓ Year range: {years.min()} - {years.max()}"
            )

        # 4. Numeric columns must be positive
        numeric_cols = [
            c for c in df.columns if df[c].dtype in (pl.Float64, pl.Int64, pl.Int32)
        ]
        for col in numeric_cols:
            min_val = df[col].min()
            assert min_val > 0, (
                f"Column '{col}' has non-positive values (min={min_val})"
            )
        logger.info(f"✓ Positive values confirmed for: {numeric_cols}")

        logger.info("All validation checks passed ✓")
        return True

    def save(self, df: pl.DataFrame) -> Path:
        """
        Save validated DataFrame to raw CSV and log column summary.

        Args:
            df: Validated expenditure DataFrame.

        Returns:
            Path to saved CSV file.
        """
        out_path = self.output_dir / "government_health_expenditure.csv"
        df.write_csv(str(out_path))
        logger.info(f"✓ Saved expenditure data: {out_path} ({len(df)} rows)")

        # Save schema YAML
        self._save_schema(df)
        return out_path

    def _save_schema(self, df: pl.DataFrame) -> None:
        """Persist column schema to YAML for downstream reference."""
        schema = {
            "table": "government_health_expenditure",
            "source": DATASET_ID,
            "extracted_at": datetime.now().isoformat(),
            "row_count": len(df),
            "columns": {
                col: str(df[col].dtype) for col in df.columns
            },
        }
        self.schema_file.parent.mkdir(parents=True, exist_ok=True)
        with open(self.schema_file, "w") as f:
            yaml.dump(schema, f, default_flow_style=False)
        logger.info(f"✓ Schema saved: {self.schema_file}")

    def run(self) -> pl.DataFrame:
        """
        Execute full extraction pipeline: extract → validate → save.

        Returns:
            Validated expenditure DataFrame.
        """
        df = self.extract()
        self.validate(df)
        self.save(df)
        logger.info("Expenditure extraction pipeline complete")
        return df
```

#### 6.2 Data Schema — `expenditure_raw_schema.yml`

```yaml
table: government_health_expenditure
source: subhamjain/health-dataset-complete-singapore
description: "Singapore government health expenditure 2006-2018"
row_count_expected: 13
completeness_expected: 1.0  # 0% nulls
columns:
  financial_year:
    dtype: Utf8
    description: "Financial year (e.g. '2006/07' or '2006')"
    nullable: false
  total_expenditure:
    dtype: Float64
    description: "Total government health expenditure (SGD millions)"
    nullable: false
    min_value: 0
constraints:
  year_range: [2006, 2018]
```

#### 6.3 Validation Rules

```python
VALIDATION_RULES = {
    "expected_rows": 13,
    "max_null_pct": 0.0,
    "year_range": (2006, 2018),
    "expenditure_min": 0,
    "required_year_column_keywords": ["year"],
}
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 kagglehub>=0.2.0 loguru>=0.7.0 pydantic>=2.5.0 pyyaml>=6.0
uv pip freeze > requirements.txt
```

---

### 10. Testing Strategy

```python
# problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_extract_expenditure.py

import polars as pl
import pytest
from unittest.mock import MagicMock, patch
from pathlib import Path
from shared.src.data_processing.extractors.expenditure_extractor import (
    ExpenditureDataExtractor,
)


@pytest.fixture
def sample_expenditure_df() -> pl.DataFrame:
    """Sample expenditure DataFrame with 13 rows matching expected schema."""
    return pl.DataFrame({
        "financial_year": [f"{yr}/{str(yr+1)[-2:]}" for yr in range(2006, 2019)],
        "total_expenditure": [3500.0 + i * 200 for i in range(13)],
    })


@pytest.fixture
def extractor(tmp_path: Path) -> ExpenditureDataExtractor:
    return ExpenditureDataExtractor(
        output_dir=str(tmp_path / "1_raw/expenditure"),
        schema_file=str(tmp_path / "schemas/expenditure_raw_schema.yml"),
        log_dir=str(tmp_path / "logs"),
    )


def test_validate_passes_on_good_data(
    extractor: ExpenditureDataExtractor, sample_expenditure_df: pl.DataFrame
) -> None:
    assert extractor.validate(sample_expenditure_df) is True


def test_validate_fails_on_wrong_row_count(
    extractor: ExpenditureDataExtractor,
) -> None:
    df_short = pl.DataFrame({
        "financial_year": ["2006/07", "2007/08"],
        "total_expenditure": [3500.0, 3700.0],
    })
    with pytest.raises(AssertionError, match="Expected 13 rows"):
        extractor.validate(df_short)


def test_validate_fails_on_nulls(
    extractor: ExpenditureDataExtractor, sample_expenditure_df: pl.DataFrame
) -> None:
    df_null = sample_expenditure_df.with_columns(
        pl.when(pl.col("financial_year") == "2006/07")
        .then(None)
        .otherwise(pl.col("total_expenditure"))
        .alias("total_expenditure")
    )
    with pytest.raises(AssertionError, match="null values"):
        extractor.validate(df_null)


def test_save_creates_csv(
    extractor: ExpenditureDataExtractor, sample_expenditure_df: pl.DataFrame, tmp_path: Path
) -> None:
    out_path = extractor.save(sample_expenditure_df)
    assert out_path.exists()
    df_reloaded = pl.read_csv(str(out_path))
    assert len(df_reloaded) == 13


def test_save_creates_schema_yaml(
    extractor: ExpenditureDataExtractor, sample_expenditure_df: pl.DataFrame
) -> None:
    extractor.save(sample_expenditure_df)
    assert extractor.schema_file.exists()


@patch(
    "shared.src.data_processing.extractors.expenditure_extractor.kagglehub.dataset_download"
)
def test_extract_calls_kagglehub(
    mock_download: MagicMock,
    extractor: ExpenditureDataExtractor,
    tmp_path: Path,
    sample_expenditure_df: pl.DataFrame,
) -> None:
    # Arrange: write sample CSV to a temp path
    csv_dir = tmp_path / "government-health-expenditure"
    csv_dir.mkdir(parents=True)
    csv_file = csv_dir / "government-health-expenditure.csv"
    sample_expenditure_df.write_csv(str(csv_file))
    mock_download.return_value = str(tmp_path)

    df = extractor.extract()
    assert len(df) == 13
    mock_download.assert_called_once()
```

---

### 11. Implementation Steps

#### Phase 1: Setup & Foundation
- [ ] Create `problem-statements/ps-004-healthcare-expenditure-drivers/` directory structure (`src/`, `tests/unit/`, `notebooks/`, `config/`, `data/`, `results/`, `reports/`)
- [ ] Create `shared/data/schemas/` directory if not present
- [ ] Add `shared/data/1_raw/expenditure/` to `.gitignore` raw data exclusions

#### Phase 2: Data Extraction
- [ ] Implement `shared/src/data_processing/extractors/expenditure_extractor.py`
- [ ] Create `shared/data/schemas/expenditure_raw_schema.yml`
- [ ] Update `shared/src/data_processing/extractors/__init__.py` to export `ExpenditureDataExtractor`
- [ ] Run extractor to confirm 13 rows downloaded and saved

#### Phase 3: Validation & Logging
- [ ] Confirm 0% nulls and year range 2006-2018 in logs
- [ ] Confirm expenditure amounts are in reasonable SGD range (> 0)
- [ ] Review log file in `logs/etl/expenditure_extraction_YYYYMMDD.log`

#### Phase 4: Testing
- [ ] Write unit tests in `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_extract_expenditure.py`
- [ ] Run `pytest` and confirm ≥80% coverage
- [ ] Confirm all mocked Kaggle calls work without network access

#### Phase 5: Notebook & Documentation
- [ ] Create `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/01-extract-expenditure-data.ipynb`
- [ ] Add path resolution cell (search upward for `shared/data/1_raw/expenditure/government_health_expenditure.csv`)
- [ ] Document columns in `docs/data_dictionary/`

---

### 12. Adaptive Implementation Strategy

- **If column names differ from expected** → update `_save_schema()` and validation logic; log actual column names before asserting
- **If row count ≠ 13** → log actual rows and flag for manual review before proceeding to US-02
- **If financial_year format is inconsistent** → add normalization step extracting 4-digit year integer for alignment in US-02
- **If Kaggle download fails** → check `~/.kaggle/kaggle.json` credentials; retry with `use_kagglehub=False` mode in `KaggleConnector`

---

### 13. Code Generation Order

1. `shared/data/schemas/expenditure_raw_schema.yml`
2. `shared/src/data_processing/extractors/expenditure_extractor.py`
3. `shared/src/data_processing/extractors/__init__.py` (add export)
4. `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_extract_expenditure.py`
5. `problem-statements/ps-004-healthcare-expenditure-drivers/src/data_processing/extract_expenditure_data.py` (orchestration)
6. `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/01-extract-expenditure-data.ipynb`

---

### 19. References

- [data-sources.md](../../../project_context/data-sources.md) — Financial analysis table spec
- [base.yml](../../../../shared/config/base.yml) — Shared project config
- [mortality_extractor.py](../../../../shared/src/data_processing/extractors/mortality_extractor.py) — Reference extractor pattern
- [kaggle_connector.py](../../../../shared/src/data_processing/kaggle_connector.py) — Kaggle download helper

---

### 20. Security & Privacy

- Expenditure data is **aggregate public data** — no PII/PHI present
- Kaggle credentials stored in `~/.kaggle/kaggle.json` (never committed)
- Set `chmod 600 ~/.kaggle/kaggle.json`
- Raw data treated as read-only after extraction

---

### 21. Version Control

```
Branch: feature/ps-004-us-01-extract-expenditure
Commits:
  feat(ps-004): add ExpenditureDataExtractor with schema validation
  test(ps-004): add unit tests for expenditure extraction
  docs(ps-004): update data dictionary with expenditure schema
```
