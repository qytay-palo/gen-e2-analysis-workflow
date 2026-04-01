# Integrate Expenditure with Utilization & Demographics (Lifecycle Stage: Data Preparation)

**Story ID**: PS-004-US-02  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Data Engineer preparing cost analysis datasets**,  
I want **to integrate expenditure data with utilization metrics (admissions) and demographic data (population, age structure) into a unified analytical dataset**,  
So that **financial analysts can perform driver analysis correlating costs with utilization patterns and population characteristics**.

---

## 🎯 Acceptance Criteria

1. **Data integration completed**
   - Expenditure joined with utilization data on year/financial_year
   - Population data integrated for per capita calculations
   - Demographic age structure data joined for aging impact analysis
   - Common time period established (2006-2018 overlap)

2. **Derived metrics calculated**
   - Per capita expenditure: total expenditure / population
   - Cost per admission: expenditure / total admissions
   - Expenditure growth rate: YoY % change
   - Age-adjusted expenditure: controlling for demographic changes

3. **Data quality ensured**
   - Year/financial year alignment validated
   - Missing data identified and documented
   - Unit consistency: all monetary values in same currency (SGD)
   - Duplicates removed

4. **Output deliverables**
   - Integrated dataset: `shared/data/3_interim/expenditure_drivers_integrated.parquet`
   - Schema documentation: `shared/data/schemas/expenditure_integrated_schema.yml`
   - Integration report: documented joins and derived metrics
   - Test coverage: ≥80%

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY)
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#expenditure-analysis-methods) - Cost driver analysis frameworks
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-2) - Integration objectives

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `pydantic>=2.5.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-01 (Extract expenditure - BLOCKING)
- **Data Sources**: 
  - `shared/data/1_raw/expenditure/*.csv`
  - `shared/data/1_raw/utilization/*.csv` (from PS-003)
  - Population data (may need external source or derive from admission rate denominators)
- **Config Files**: `shared/config/base.yml`

---

## ✅ Implementation Tasks

### Data Integration
- [ ] Load expenditure, utilization, and demographic datasets
- [ ] Align financial_year with calendar year (resolve any offset)
- [ ] Join expenditure with utilization on year
- [ ] Join with population/demographic data
- [ ] Filter to common time period: 2006-2018

### Derived Metrics
- [ ] Calculate per capita expenditure
- [ ] Calculate cost per admission
- [ ] Calculate YoY expenditure growth rate
- [ ] Age-adjust expenditure (if age structure data available)
- [ ] Calculate expenditure per bed (using capacity data)

### Data Quality
- [ ] Validate year alignment across sources
- [ ] Check for missing data and document
- [ ] Ensure unit consistency (all SGD, same year basis)
- [ ] Remove any duplicates

### Output & Documentation
- [ ] Save integrated Parquet dataset
- [ ] Document schema in YAML
- [ ] Generate integration report: joins performed, metrics derived
- [ ] Log integration summary

### Testing
- [ ] Unit tests for join logic
- [ ] Validate derived metrics with sample calculations
- [ ] Integration test for full pipeline
- [ ] Docstrings

---

## 📌 Notes

**Polars Integration Example**:
```python
import polars as pl

df_exp = pl.read_csv("shared/data/1_raw/expenditure/government_health_expenditure.csv")
df_util = pl.read_csv("shared/data/1_raw/utilization/hospital_admissions.csv")

# Aggregate utilization to annual level
df_util_agg = df_util.group_by('year').agg([
    pl.col('admission_rate').sum().alias('total_admissions')
])

# Join expenditure and utilization
df_integrated = df_exp.join(df_util_agg, 
    left_on='financial_year', right_on='year', how='inner')

# Calculate per capita expenditure (assuming population column exists)
df_integrated = df_integrated.with_columns([
    (pl.col('total_expenditure') / pl.col('population')).alias('per_capita_expenditure')
])

# Calculate cost per admission
df_integrated = df_integrated.with_columns([
    (pl.col('total_expenditure') / pl.col('total_admissions')).alias('cost_per_admission')
])
```

**Known Challenges**:
- Financial year may differ from calendar year (e.g., Apr-Mar vs Jan-Dec)
- Population data may need to be sourced externally or derived
- Admission rates vs absolute admissions conversion needed

---

## Implementation Plan

### 1. Feature Overview

Join government health expenditure (2006-2018) with hospital utilization data and demographic population structures into a single analytical Parquet file. Primary user role: **Data Engineer**. Blocking upstream: PS-004-US-01.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/src/data_processing/kaggle_connector.py` | Exists | **Reuse** for loading raw CSVs |
| `shared/data/1_raw/expenditure/` | Created by US-01 | **Reuse** |
| `shared/data/1_raw/utilization/` | May exist from PS-003 | **Check & reuse** |
| `shared/src/analysis/trend_analysis.py` | Exists (workforce)  | **Reference pattern** |
| `shared/src/data_processing/expenditure_integrator.py` | Missing | **Create** |
| `shared/data/schemas/expenditure_integrated_schema.yml` | Missing | **Create** |
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | Missing | **Create** |

No ML model evaluation required (data preparation task).

---

### 4. Affected Files

- **[CREATE] `shared/src/data_processing/expenditure_integrator.py`**
  - `ExpenditureIntegrator` class
  - `load_expenditure()` → `pl.DataFrame`
  - `load_utilization()` → `pl.DataFrame`
  - `load_population()` → `pl.DataFrame | None`
  - `align_years()` → `pl.DataFrame`
  - `join_datasets()` → `pl.DataFrame`
  - `calculate_derived_metrics()` → `pl.DataFrame`
  - `validate_integrated()` → `bool`
  - `save()` → `Path`
  - `run()` → `pl.DataFrame`

- **[CREATE] `shared/data/schemas/expenditure_integrated_schema.yml`**
  - Integrated schema with derived column definitions

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/data_processing/integrate_expenditure_drivers.py`**
  - CLI orchestration script

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_integrate_expenditure.py`**
  - Unit tests for join logic and derived metrics

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/02-integrate-expenditure-drivers.ipynb`**
  - Interactive integration walkthrough

---

### 5. Data Pipeline

```
shared/data/1_raw/expenditure/government_health_expenditure.csv
  → normalize financial_year → integer year (2006-2018)

shared/data/1_raw/utilization/hospital-admission-rate-by-age-and-sex.csv  (Kaggle)
  → aggregate to annual total_admissions per year

[Optional] population from Kaggle demographic tables
  → annual population by year

LEFT JOIN expenditure ← utilization ON year
LEFT JOIN ← population ON year (if available)

WITH_COLUMNS:
  per_capita_expenditure = total_expenditure / population
  cost_per_admission     = total_expenditure / total_admissions
  yoy_expenditure_growth = (exp_t - exp_t-1) / exp_t-1 * 100

FILTER → 2006-2018 overlap
VALIDATE → nulls, unit consistency, duplicates
SAVE → shared/data/3_interim/expenditure_drivers_integrated.parquet
SAVE → shared/data/schemas/expenditure_integrated_schema.yml
LOG  → logs/etl/expenditure_integration_YYYYMMDD.log
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `expenditure_integrator.py`

```python
"""
Expenditure Drivers Integrator
================================

Joins government health expenditure with utilization and demographic data.
Produces the analytical base dataset for PS-004 cost driver analysis.

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


DATASET_ID = "subhamjain/health-dataset-complete-singapore"
ANALYSIS_YEAR_MIN = 2006
ANALYSIS_YEAR_MAX = 2018


def _extract_year_int(series: pl.Series) -> pl.Series:
    """
    Parse financial year strings like '2006/07' or plain '2006' → Int32.

    Args:
        series: Column containing year strings.

    Returns:
        Int32 Series of 4-digit calendar years.
    """
    return series.cast(pl.Utf8).str.extract(r"(\d{4})", 1).cast(pl.Int32)


class ExpenditureIntegrator:
    """
    Integrate expenditure, utilization, and demographic data for PS-004.

    Example:
        >>> integrator = ExpenditureIntegrator()
        >>> df = integrator.run()
    """

    def __init__(
        self,
        expenditure_path: str = "shared/data/1_raw/expenditure/government_health_expenditure.csv",
        output_path: str = "shared/data/3_interim/expenditure_drivers_integrated.parquet",
        schema_file: str = "shared/data/schemas/expenditure_integrated_schema.yml",
        log_dir: str = "logs/etl",
    ) -> None:
        """
        Initialise integrator with configurable paths.

        Args:
            expenditure_path: Path to raw expenditure CSV (output of US-01).
            output_path: Target path for integrated Parquet file.
            schema_file: Path to store integrated schema YAML.
            log_dir: Directory for ETL log files.
        """
        self.expenditure_path = Path(expenditure_path)
        self.output_path = Path(output_path)
        self.schema_file = Path(schema_file)
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(parents=True, exist_ok=True)

        log_file = (
            self.log_dir
            / f"expenditure_integration_{datetime.now().strftime('%Y%m%d')}.log"
        )
        logger.add(str(log_file), level="INFO", rotation="1 day")
        logger.info("ExpenditureIntegrator initialised")

    # ------------------------------------------------------------------
    # Data loaders
    # ------------------------------------------------------------------

    def load_expenditure(self) -> pl.DataFrame:
        """
        Load expenditure CSV and normalise financial_year to integer year.

        Returns:
            DataFrame with columns: year (Int32), total_expenditure (Float64), ...

        Raises:
            FileNotFoundError: If US-01 extraction has not been run.
        """
        if not self.expenditure_path.exists():
            raise FileNotFoundError(
                f"Expenditure data not found: {self.expenditure_path}\n"
                "Please run PS-004-US-01 extraction first."
            )
        df = pl.read_csv(str(self.expenditure_path))

        # Detect year column
        year_col = next(
            (c for c in df.columns if "year" in c.lower()), None
        )
        if year_col is None:
            raise ValueError(f"No year column found in expenditure CSV. Columns: {df.columns}")

        df = df.with_columns(
            _extract_year_int(df[year_col]).alias("year")
        )
        if year_col != "year":
            df = df.drop(year_col)

        # Detect expenditure column
        exp_cols = [c for c in df.columns if "expenditure" in c.lower() or "exp" in c.lower()]
        if exp_cols and "total_expenditure" not in df.columns:
            df = df.rename({exp_cols[0]: "total_expenditure"})

        df = df.with_columns(pl.col("total_expenditure").cast(pl.Float64))
        logger.info(f"Loaded expenditure: {len(df)} rows, years {df['year'].min()}-{df['year'].max()}")
        return df

    def load_utilization(self) -> pl.DataFrame:
        """
        Load hospital admission rate data from Kaggle and aggregate to annual totals.

        Returns:
            DataFrame with columns: year (Int32), total_admissions (Float64).
        """
        logger.info("Downloading utilization data from Kaggle...")
        try:
            dataset_path = kagglehub.dataset_download(DATASET_ID)
        except Exception as exc:
            logger.warning(f"Kaggle download failed: {exc} — returning empty utilization")
            return pl.DataFrame({"year": pl.Series([], dtype=pl.Int32),
                                  "total_admissions": pl.Series([], dtype=pl.Float64)})

        util_file = (
            Path(dataset_path)
            / "hospital-admission-rate-by-age-and-sex"
            / "hospital-admission-rate-by-age-and-sex.csv"
        )
        if not util_file.exists():
            logger.warning(f"Utilization file not found: {util_file}")
            return pl.DataFrame({"year": pl.Series([], dtype=pl.Int32),
                                  "total_admissions": pl.Series([], dtype=pl.Float64)})

        df_util = pl.read_csv(str(util_file))
        year_col = next((c for c in df_util.columns if "year" in c.lower()), None)
        rate_col = next((c for c in df_util.columns if "rate" in c.lower() or "admission" in c.lower()), None)

        if year_col is None or rate_col is None:
            logger.warning(f"Unexpected utilization columns: {df_util.columns}")
            return pl.DataFrame({"year": pl.Series([], dtype=pl.Int32),
                                  "total_admissions": pl.Series([], dtype=pl.Float64)})

        df_agg = (
            df_util
            .with_columns(_extract_year_int(df_util[year_col]).alias("year"))
            .group_by("year")
            .agg(pl.col(rate_col).sum().alias("total_admissions"))
            .sort("year")
        )
        logger.info(f"Loaded utilization: {len(df_agg)} annual rows")

        # Persist raw utilization for downstream reference
        util_out = Path("shared/data/1_raw/utilization")
        util_out.mkdir(parents=True, exist_ok=True)
        df_util.write_csv(str(util_out / "hospital_admissions.csv"))
        return df_agg

    def load_population(self) -> Optional[pl.DataFrame]:
        """
        Attempt to load population data from available Kaggle tables.

        Returns:
            DataFrame with columns: year (Int32), population (Float64), or None if unavailable.
        """
        logger.info("Attempting to load population data...")
        try:
            dataset_path = kagglehub.dataset_download(DATASET_ID)
        except Exception:
            logger.warning("Could not download dataset for population lookup")
            return None

        # Try resident population table if it exists
        candidates = [
            "total-population/total-population.csv",
            "resident-population/resident-population.csv",
        ]
        for candidate in candidates:
            pop_file = Path(dataset_path) / candidate
            if pop_file.exists():
                df_pop = pl.read_csv(str(pop_file))
                year_col = next((c for c in df_pop.columns if "year" in c.lower()), None)
                pop_col = next((c for c in df_pop.columns if "population" in c.lower() or "total" in c.lower()), None)
                if year_col and pop_col:
                    df_pop = (
                        df_pop
                        .with_columns(_extract_year_int(df_pop[year_col]).alias("year"))
                        .rename({pop_col: "population"})
                        .select(["year", "population"])
                        .with_columns(pl.col("population").cast(pl.Float64))
                    )
                    logger.info(f"Population loaded from {candidate}: {len(df_pop)} rows")
                    return df_pop
        logger.warning("No population table found — per_capita_expenditure will be null")
        return None

    # ------------------------------------------------------------------
    # Integration
    # ------------------------------------------------------------------

    def join_datasets(
        self,
        df_exp: pl.DataFrame,
        df_util: pl.DataFrame,
        df_pop: Optional[pl.DataFrame],
    ) -> pl.DataFrame:
        """
        Left-join expenditure with utilization and optionally population.

        Args:
            df_exp: Expenditure DataFrame with 'year' column.
            df_util: Utilization DataFrame with 'year' column.
            df_pop: Optional population DataFrame with 'year' column.

        Returns:
            Joined DataFrame filtered to ANALYSIS_YEAR_MIN - ANALYSIS_YEAR_MAX.
        """
        df = df_exp.join(df_util, on="year", how="left")
        if df_pop is not None:
            df = df.join(df_pop, on="year", how="left")

        df = df.filter(
            (pl.col("year") >= ANALYSIS_YEAR_MIN)
            & (pl.col("year") <= ANALYSIS_YEAR_MAX)
        ).sort("year")

        logger.info(
            f"Joined dataset: {len(df)} rows, {df.width} columns, "
            f"years {df['year'].min()}-{df['year'].max()}"
        )
        return df

    def calculate_derived_metrics(self, df: pl.DataFrame) -> pl.DataFrame:
        """
        Add per_capita_expenditure, cost_per_admission, and yoy_expenditure_growth.

        Args:
            df: Joined DataFrame sorted by year.

        Returns:
            DataFrame with additional derived metric columns.
        """
        exprs = []

        if "population" in df.columns:
            exprs.append(
                (pl.col("total_expenditure") / pl.col("population"))
                .alias("per_capita_expenditure")
            )
        else:
            exprs.append(pl.lit(None).cast(pl.Float64).alias("per_capita_expenditure"))

        if "total_admissions" in df.columns:
            exprs.append(
                (pl.col("total_expenditure") / pl.col("total_admissions"))
                .alias("cost_per_admission")
            )
        else:
            exprs.append(pl.lit(None).cast(pl.Float64).alias("cost_per_admission"))

        # YoY growth requires sorted data
        exprs.append(
            (
                (pl.col("total_expenditure") - pl.col("total_expenditure").shift(1))
                / pl.col("total_expenditure").shift(1)
                * 100
            ).alias("yoy_expenditure_growth_pct")
        )

        df = df.with_columns(exprs)
        logger.info("Derived metrics calculated: per_capita_expenditure, cost_per_admission, yoy_expenditure_growth_pct")
        return df

    def validate_integrated(self, df: pl.DataFrame) -> bool:
        """
        Validate integrated dataset: required columns, year range, no duplicate years.

        Args:
            df: Integrated DataFrame.

        Returns:
            True if all checks pass.

        Raises:
            AssertionError: On validation failure.
        """
        required_cols = ["year", "total_expenditure", "yoy_expenditure_growth_pct"]
        for col in required_cols:
            assert col in df.columns, f"Missing required column: '{col}'"

        assert df["year"].min() >= ANALYSIS_YEAR_MIN
        assert df["year"].max() <= ANALYSIS_YEAR_MAX

        duplicate_years = df.filter(df["year"].is_duplicated())
        assert len(duplicate_years) == 0, (
            f"Duplicate years found: {duplicate_years['year'].to_list()}"
        )

        null_counts = df.null_count()
        for col in required_cols:
            null_pct = null_counts[col][0] / len(df) * 100
            if null_pct > 0:
                logger.warning(f"Column '{col}' has {null_pct:.1f}% nulls")

        logger.info("✓ Integrated dataset validation passed")
        return True

    def save(self, df: pl.DataFrame) -> Path:
        """
        Persist integrated dataset as Parquet and write schema YAML.

        Args:
            df: Validated integrated DataFrame.

        Returns:
            Path to saved Parquet file.
        """
        self.output_path.parent.mkdir(parents=True, exist_ok=True)
        df.write_parquet(str(self.output_path))
        logger.info(f"✓ Integrated dataset saved: {self.output_path} ({len(df)} rows)")

        schema = {
            "table": "expenditure_drivers_integrated",
            "created_at": datetime.now().isoformat(),
            "row_count": len(df),
            "year_range": [int(df["year"].min()), int(df["year"].max())],
            "columns": {col: str(df[col].dtype) for col in df.columns},
            "derived_metrics": [
                "per_capita_expenditure",
                "cost_per_admission",
                "yoy_expenditure_growth_pct",
            ],
        }
        self.schema_file.parent.mkdir(parents=True, exist_ok=True)
        with open(self.schema_file, "w") as f:
            yaml.dump(schema, f, default_flow_style=False)
        logger.info(f"✓ Schema saved: {self.schema_file}")
        return self.output_path

    def run(self) -> pl.DataFrame:
        """
        Full pipeline: load → join → derive → validate → save.

        Returns:
            Integrated expenditure drivers DataFrame.
        """
        df_exp = self.load_expenditure()
        df_util = self.load_utilization()
        df_pop = self.load_population()
        df = self.join_datasets(df_exp, df_util, df_pop)
        df = self.calculate_derived_metrics(df)
        self.validate_integrated(df)
        self.save(df)
        logger.info("Expenditure integration pipeline complete")
        return df
```

#### 6.2 Data Schema — `expenditure_integrated_schema.yml`

```yaml
table: expenditure_drivers_integrated
description: "Integrated expenditure, utilization, and demographic data 2006-2018"
row_count_expected: 13
columns:
  year:
    dtype: Int32
    description: "Calendar year (derived from financial_year)"
    nullable: false
  total_expenditure:
    dtype: Float64
    description: "Government health expenditure (SGD millions)"
    nullable: false
  total_admissions:
    dtype: Float64
    description: "Aggregated annual hospital admissions or admission rate sum"
    nullable: true     # Available only if utilization table found
  population:
    dtype: Float64
    description: "Resident population (thousands)"
    nullable: true     # Available only if population table found
  per_capita_expenditure:
    dtype: Float64
    description: "total_expenditure / population"
    nullable: true
  cost_per_admission:
    dtype: Float64
    description: "total_expenditure / total_admissions"
    nullable: true
  yoy_expenditure_growth_pct:
    dtype: Float64
    description: "Year-on-year % growth in total_expenditure"
    nullable: true     # Null for first year (2006)
```

#### 6.3 Validation Rules

```python
INTEGRATION_VALIDATION_RULES = {
    "required_columns": ["year", "total_expenditure", "yoy_expenditure_growth_pct"],
    "year_range": (2006, 2018),
    "no_duplicate_years": True,
    "monetary_unit": "SGD_millions",
    "warn_null_pct_threshold": 0.10,  # Warn if >10% null for optional cols
}
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 kagglehub>=0.2.0 loguru>=0.7.0 pydantic>=2.5.0 pyyaml>=6.0
uv pip freeze > requirements.txt
```

---

### 7. Domain-Driven Feature Engineering

**Step 1 — Relevant Domain Knowledge**:  
SGD monetary amounts; cost-per-admission as efficiency proxy; per-capita as equity/burden indicator; YoY growth for trend direction.

**Step 2 — Data Availability**:

| Feature | Formula | Required Fields | Available |
|---------|---------|----------------|-----------|
| `per_capita_expenditure` | `total_expenditure / population` | population | Conditional (if table found) |
| `cost_per_admission` | `total_expenditure / total_admissions` | total_admissions | Conditional |
| `yoy_expenditure_growth_pct` | `(exp_t - exp_t-1)/exp_t-1 * 100` | total_expenditure | ✅ Always |

**Step 3 — Selected Features** (validated against available data):

```python
import polars as pl

def calculate_derived_metrics_standalone(df: pl.DataFrame) -> pl.DataFrame:
    """
    Standalone derived metric calculation for testing.

    Args:
        df: Joined DataFrame sorted by year with at least 'year' and
            'total_expenditure' columns.

    Returns:
        DataFrame enriched with derived columns.
    """
    exprs = [
        (
            (pl.col("total_expenditure") - pl.col("total_expenditure").shift(1))
            / pl.col("total_expenditure").shift(1)
            * 100
        ).alias("yoy_expenditure_growth_pct"),
    ]
    if "population" in df.columns:
        exprs.append(
            (pl.col("total_expenditure") / pl.col("population"))
            .alias("per_capita_expenditure")
        )
    if "total_admissions" in df.columns:
        exprs.append(
            (pl.col("total_expenditure") / pl.col("total_admissions"))
            .alias("cost_per_admission")
        )
    return df.with_columns(exprs)
```

---

### 10. Testing Strategy

```python
# problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_integrate_expenditure.py

import polars as pl
import pytest
from pathlib import Path
from shared.src.data_processing.expenditure_integrator import (
    ExpenditureIntegrator,
    _extract_year_int,
    calculate_derived_metrics_standalone,
)


@pytest.fixture
def sample_exp() -> pl.DataFrame:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 200 for i in range(13)],
    })


@pytest.fixture
def sample_util() -> pl.DataFrame:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_admissions": [200000.0 + i * 5000 for i in range(13)],
    })


@pytest.fixture
def sample_pop() -> pl.DataFrame:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "population": [4500000.0 + i * 50000 for i in range(13)],
    })


@pytest.fixture
def integrator(tmp_path: Path) -> ExpenditureIntegrator:
    exp_path = tmp_path / "expenditure" / "government_health_expenditure.csv"
    exp_path.parent.mkdir(parents=True)
    pl.DataFrame({
        "financial_year": [f"{yr}/{str(yr+1)[-2:]}" for yr in range(2006, 2019)],
        "total_expenditure": [3500.0 + i * 200 for i in range(13)],
    }).write_csv(str(exp_path))
    return ExpenditureIntegrator(
        expenditure_path=str(exp_path),
        output_path=str(tmp_path / "integrated.parquet"),
        schema_file=str(tmp_path / "schema.yml"),
        log_dir=str(tmp_path / "logs"),
    )


def test_extract_year_int_financial_year() -> None:
    s = pl.Series(["2006/07", "2007/08", "2018/19"])
    result = _extract_year_int(s)
    assert result.to_list() == [2006, 2007, 2018]


def test_extract_year_int_plain_year() -> None:
    s = pl.Series(["2006", "2010", "2018"])
    result = _extract_year_int(s)
    assert result.to_list() == [2006, 2010, 2018]


def test_join_datasets_correct_row_count(
    integrator: ExpenditureIntegrator,
    sample_exp: pl.DataFrame,
    sample_util: pl.DataFrame,
) -> None:
    df = integrator.join_datasets(sample_exp, sample_util, None)
    assert len(df) == 13


def test_join_datasets_includes_population(
    integrator: ExpenditureIntegrator,
    sample_exp: pl.DataFrame,
    sample_util: pl.DataFrame,
    sample_pop: pl.DataFrame,
) -> None:
    df = integrator.join_datasets(sample_exp, sample_util, sample_pop)
    assert "population" in df.columns


def test_derived_metrics_yoy_growth(sample_exp: pl.DataFrame) -> None:
    df = calculate_derived_metrics_standalone(sample_exp)
    assert "yoy_expenditure_growth_pct" in df.columns
    # First row should be null (no previous year)
    assert df["yoy_expenditure_growth_pct"][0] is None
    # Growth for year two: (3700-3500)/3500*100 ≈ 5.71
    assert abs(df["yoy_expenditure_growth_pct"][1] - 5.714) < 0.01


def test_derived_metrics_per_capita(
    sample_exp: pl.DataFrame, sample_pop: pl.DataFrame
) -> None:
    df = sample_exp.join(sample_pop, on="year")
    df = calculate_derived_metrics_standalone(df)
    assert "per_capita_expenditure" in df.columns
    # 3500 / 4500000 ≈ 0.000778
    assert abs(df["per_capita_expenditure"][0] - 3500 / 4500000) < 1e-6


def test_validate_integrated_passes(
    integrator: ExpenditureIntegrator, sample_exp: pl.DataFrame
) -> None:
    df = sample_exp.with_columns(
        pl.lit(None).cast(pl.Float64).alias("yoy_expenditure_growth_pct")
    )
    assert integrator.validate_integrated(df) is True


def test_save_creates_parquet(
    integrator: ExpenditureIntegrator, sample_exp: pl.DataFrame
) -> None:
    df = sample_exp.with_columns(
        pl.lit(None).cast(pl.Float64).alias("yoy_expenditure_growth_pct")
    )
    out = integrator.save(df)
    assert out.exists()
    df_reloaded = pl.read_parquet(str(out))
    assert len(df_reloaded) == 13
```

---

### 11. Implementation Steps

#### Phase 1: Setup
- [ ] Confirm US-01 extraction ran successfully and `shared/data/1_raw/expenditure/government_health_expenditure.csv` exists
- [ ] Create `shared/data/3_interim/` directory if not present
- [ ] Create `shared/data/1_raw/utilization/` directory

#### Phase 2: Data Integration
- [ ] Implement `shared/src/data_processing/expenditure_integrator.py`
- [ ] Run `load_expenditure()` and log column names and dtypes
- [ ] Run `load_utilization()` and confirm annual aggregation
- [ ] Run `load_population()` — log whether population table was found
- [ ] Run `join_datasets()` and confirm 13-row result

#### Phase 3: Derived Metrics & Validation
- [ ] Run `calculate_derived_metrics()` and review YoY growth series
- [ ] Log null percentages for optional columns (`per_capita_expenditure`, `cost_per_admission`)
- [ ] Run `validate_integrated()` and confirm all assertions pass
- [ ] Save `expenditure_drivers_integrated.parquet`

#### Phase 4: Testing
- [ ] Write unit tests in `tests/unit/test_integrate_expenditure.py`
- [ ] Confirm `_extract_year_int` handles both `2006/07` and `2006` formats
- [ ] Run `pytest` and confirm ≥80% coverage

#### Phase 5: Documentation & Notebook
- [ ] Create `notebooks/02-integrate-expenditure-drivers.ipynb`
- [ ] Document schema in `shared/data/schemas/expenditure_integrated_schema.yml`
- [ ] Log integration summary: rows, date range, null rates per column

---

### 12. Adaptive Implementation Strategy

- **If utilization file not found** → proceed without `total_admissions`; mark `cost_per_admission` as null; document gap
- **If financial_year format is not `2006/07` or `2006`** → inspect actual format and update `_extract_year_int` regex before joining
- **If no year overlap between expenditure and utilization** → log full year sets from both and investigate alignment
- **If population table absent** → derive population from admission rate denominators in the utilization CSV if a `per_1000` column is present

---

### 13. Code Generation Order

1. `shared/src/data_processing/expenditure_integrator.py`
2. `shared/data/schemas/expenditure_integrated_schema.yml`
3. `problem-statements/ps-004-healthcare-expenditure-drivers/tests/unit/test_integrate_expenditure.py`
4. `problem-statements/ps-004-healthcare-expenditure-drivers/src/data_processing/integrate_expenditure_drivers.py` (CLI)
5. `problem-statements/ps-004-healthcare-expenditure-drivers/notebooks/02-integrate-expenditure-drivers.ipynb`

---

### 19. References

- [data-sources.md — Financial Analysis](../../../project_context/data-sources.md) — Source table specs
- [expenditure_raw_schema.yml](../../../../shared/data/schemas/expenditure_raw_schema.yml) — Raw schema from US-01
- [trend_analysis.py](../../../../shared/src/analysis/trend_analysis.py) — YoY calculation reference

---

### 20. Security & Privacy

- All data is aggregate, anonymised government public data — no PII
- Kaggle credentials in `~/.kaggle/kaggle.json` (never committed); set `chmod 600`
- Parquet output tagged with creation timestamp in schema YAML for auditability

---

### 21. Version Control

```
Branch: feature/ps-004-us-02-integrate-expenditure
Commits:
  feat(ps-004): add ExpenditureIntegrator with join and derived metrics
  test(ps-004): unit tests for integration logic and year normalisation
  docs(ps-004): integrated schema YAML for expenditure drivers
```
