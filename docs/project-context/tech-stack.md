# Tech Stack Preferences

This project uses the following analytics platforms and tools. When providing code examples, recommendations, or technical guidance, prioritize these technologies:

## Current Project Configuration (March 2026)

**Selected Platform**: HEALIX/Databricks (Large-scale data processing >1GB)

**Primary Data Sources**: Kaggle datasets + Singapore health data

**Project Focus**: Singapore health trends analysis for policy-making decisions

### Analytics Platforms

**HEALIX (GCC Cloud Environment)** ✅ *Selected for this project*
- Platform: Databricks
- Languages: Python (primary), R (secondary)
- Available tools: STATA for statistical analysis
- Scale: Optimized for large datasets (>1GB)
- Deployment: Cloud-native workflows

**MCDR (On-Premise Compute Cluster)** *(Available as fallback)*
- Platform: Cloudera Data Science Workbench (CDSW)
  - Languages: R and Python
  - Analytics Engine: Apache Spark
  - File System: Hadoop Distributed File System (HDFS)
- Platform: HUE for SQL-based data retrieval
- Available tools: STATA for statistical analysis

### Tool Specifications

- **Databricks**: Preferred for new analytics workflows in the cloud environment
- **CDSW**: Primary platform for on-premise Spark-based analytics with R/Python
- **HUE**: SQL interface for ad-hoc queries and data exploration
- **STATA**: Statistical analysis tool, especially for economics and econometrics research
- **Spark**: Main distributed computing engine for large-scale data processing
- **Hadoop/HDFS**: Underlying distributed storage system

## Python Data Processing Stack

### Primary Libraries (MANDATORY)

**Polars** - High-performance DataFrame library
- **Use Case**: All data processing, ETL, transformations, and feature engineering
- **Why**: 5-10x faster than pandas, better memory efficiency, lazy evaluation support
- **When to use**: Default choice for all data manipulation tasks
```python
import polars as pl
df = pl.scan_csv("data.csv").collect()  # Lazy loading for large files
```

**Why Polars over Pandas**:
- Better performance on large datasets (>1GB)
- Memory efficient with optimized dtypes
- Lazy evaluation for query optimization
- Better type safety and error messages
- Native support for multi-threading

### Secondary Libraries (Use When Justified)

**Pandas** - Only when Polars lacks specific functionality
- Document reason in code comments when using pandas
- Example valid use cases: Specific ML library integrations, specialized time series functions

### Data Acquisition

**Kaggle API** - Primary external data source
```python
from kaggle.api.kaggle_api_extended import KaggleApi
api = KaggleApi()
api.authenticate()
```

### Logging & Monitoring

**Loguru** - Structured logging (NOT print() statements)
```python
from loguru import logger
logger.info("Processing started")
```

### Configuration Management

**PyYAML** - Configuration files
```python
import yaml
with open('config/analysis.yml') as f:
    config = yaml.safe_load(f)
```

### Testing Framework

**Pytest** - Unit and integration testing
- Target: >80% code coverage for critical modules
- Location: `shared/tests/` and `problem-statement/ps-*/tests/`

### Development Tools

**uv** - Package management (NOT pip/conda)
```bash
uv pip install <package>
uv pip freeze > requirements.txt
```

### Guidance for Code Generation

When generating code or providing technical solutions for this project:
1. **ALWAYS use Polars** for data processing (NOT pandas unless justified)
2. **Use Python 3.9+** as primary language (Databricks runtime compatible)
3. **Use Kaggle API** for external dataset acquisition
4. **Use `uv`** for package management, NOT pip or conda
5. **Use Loguru** for logging, never print() in production code
6. **Use type hints** and docstrings for all functions
7. **Write unit tests** with pytest for critical logic
8. **Save intermediate results** with timestamps in `data/3_interim/`
9. **Never modify raw data** - keep `data/1_raw/` immutable
10. **Use lazy evaluation** (`scan_csv`, `scan_parquet`) for large files (>100MB)

### Code Quality Standards

- Type hints required for all function signatures
- Docstrings required for all public functions
- Maximum line length: 88 characters (Black formatter default)
- Use f-strings for string formatting
- Avoid silent failures - always log data quality issues
- Configuration-driven parameters (use `config/*.yml`)

### Platform-Specific Guidance

**For HEALIX/Databricks**:
- Cluster config: Runtime 13.3.x (Python 3.9)
- Workspace paths: `/Workspace/Users/moh-analytics/`
- Data storage: `/dbfs/mnt/moh-data/`
- Use `databricks-connect` for local development
- Enable Spark adaptive query execution

**For MCDR/CDSW** (if needed):
- Use HDFS paths for distributed storage
- Leverage Spark for distributed processing
- Use HUE for SQL-based extraction

### Reference Documentation

- Python best practices: `.github/instructions/python-best-practices.instructions.md`
- Data analysis lifecycle: `.github/instructions/data-analysis-best-practices.instructions.md`
- Folder structure: `.github/instructions/data-analysis-folder-structure.instructions.md`
- Multi-agent system: `.agents/config.yml` and `.agents/registry.yml`