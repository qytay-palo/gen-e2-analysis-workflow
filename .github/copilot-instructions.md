# Package Management & Environment

- **Always use `uv` for package management** (NOT pip/conda)
  ```bash
  uv pip install <package>           # Install new package
  uv pip install -r requirements.txt # Install from file
  uv pip freeze > requirements.txt   # Update after adding packages
  ```
- **Python 3.9+** is required (Databricks runtime compatibility)
- Virtual environment activation: `source .venv/bin/activate` (macOS/Linux)

## Data Processing Stack

### Primary: Polars (MANDATORY)
```python
import polars as pl

# Lazy loading for large files
df = pl.scan_csv("data/1_raw/disease_data.csv").collect()

# Method chaining for transformations
df_clean = (
    df.clone()
    .drop_nulls(subset=['date', 'case_count'])
    .with_columns([
        pl.col('date').str.strptime(pl.Date, '%Y-%m-%d'),
        pl.col('disease').cast(pl.Categorical)
    ])
    .filter(pl.col('year') >= 2012)
)
```
- Use pandas ONLY if Polars lacks functionality (document why in comments)
- Optimize dtypes: `Int32` over `Int64`, `Categorical` for disease names
- Use lazy evaluation (`scan_csv`, `scan_parquet`) for files > 100MB

## MCP Tools Quick Reference

### Filesystem Tools (REQUIRED)
Use for ALL file operations:

```
Read data:     "Use filesystem tools to read shared/data/1_raw/input.csv"
Create dir:    "Use filesystem tools to create directory artifacts/ps-001-{name}/results/"
Write file:    "Use filesystem tools to save results to artifacts/ps-001-{name}/results/tables/output.csv"
List files:    "Use filesystem tools to list files in artifacts/ps-001-{name}/"
```

### Context7 MCP (REQUIRED for Library/Framework Usage)
Use for fetching **current documentation** instead of relying on training data:

**🚨 CRITICAL RULE**: When implementing code that uses libraries, frameworks, or APIs, you MUST use Context7 to fetch current documentation. This ensures API accuracy, correct syntax, and version compatibility.

**WHEN TO USE Context7 (MANDATORY):**

| Scenario | Action | Example |
|----------|--------|----------|
| **Writing data processing code** | Fetch Polars/pandas docs | `resolve-library-id` → "polars" → `query-docs` "lazy evaluation scan_csv" |
| **Implementing forecasting models** | Fetch statsmodels/prophet docs | `resolve-library-id` → "statsmodels" → `query-docs` "ARIMA model fit predict" |
| **Building dashboards** | Fetch Plotly Dash docs | `resolve-library-id` → "plotly" → `query-docs` "Dash app layout callbacks" |
| **Using ML libraries** | Fetch scikit-learn docs | `resolve-library-id` → "scikit-learn" → `query-docs` "train_test_split cross validation" |
| **Databricks integration** | Fetch Databricks docs | `resolve-library-id` → "databricks" → `query-docs` "scheduled jobs DBFS" |
| **API integrations** | Fetch API docs | `resolve-library-id` → "requests" → `query-docs` "session retry timeout" |


## Development Workflow

### 1. Problem-Driven Approach
- Each analysis task starts with a problem statement in `docs/objectives/problem_statements/`
- Reference existing problem statements: `ps-001-seasonal-pattern-forecasting.md`, `ps-002-disease-burden-prioritization.md`, `ps-003-executive-surveillance-dashboard.md`
- Define success criteria and stakeholder requirements upfront

### 2. Data Quality First
- ALWAYS validate data upon loading (check columns, dtypes, ranges)
- Handle missing values explicitly - never ignore them silently
- Generate data quality reports after cleaning steps
- Save validation results to `logs/etl/`

### 3. Modular Code Organization
- Place reusable functions in appropriate `src/` subdirectories:
  - `src/data_processing/`: ETL, cleaning, validation
  - `src/analysis/`: Statistical algorithms, trend detection
  - `src/models/`: Forecasting models, model evaluation
  - `src/visualization/`: Chart generation utilities
  - `src/utils/`: Helpers, config loaders
- Use type hints and docstrings for all functions
- Write unit tests in `tests/unit/` for critical logic

### 4. Logging & Monitoring
- Use `loguru` for logging (NOT `print()` in production code)
- Log to appropriate subdirectories: `logs/etl/`, `logs/errors/`, `logs/audit/`
- Include timestamps and severity levels

### 5. Results Documentation
- Save analysis outputs to `results/` subdirectories:
  - `results/tables/`: Summary statistics, analytical tables
  - `results/metrics/`: KPIs, performance metrics
  - `results/exports/`: Stakeholder-ready CSV/Excel exports
- Generate figures to `reports/figures/` in PNG/PDF for presentations

## Databricks Integration
- Document Databricks runtime and workspace requirements in `docs/project-context/tech-stack.md`
- Workspace paths: `/Workspace/Users/moh-analytics/`
- Data storage: `/dbfs/mnt/moh-data/`
- Use `databricks-connect` for local development with remote Spark clusters
- Enable Spark adaptive query execution in configs

## Testing Strategy

- Unit tests in `tests/unit/` for functions in `src/`
- Integration tests in `tests/integration/` for end-to-end pipelines
- Data validation tests in `tests/data/` for schema compliance
- Run tests with `pytest` before committing code
- Target: >80% code coverage for critical modules

## Quick Reference: Key Files

- [docs/project-context/](docs/project-context/): Project objectives and data source details
- [.github/agents/](.github/agents/): Agent capabilities and mappings

## Agents
- Review the relevant agent specs in `.github/agents/` for workflow-specific guidance.
