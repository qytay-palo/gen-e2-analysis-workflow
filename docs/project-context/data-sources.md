# Data Sources: MOH Polyclinic Data Analysis

**Last Updated:** 30 January 2026  
**Status:** Active, Fully Documented

---

## Primary Data Source: Kaggle Health Dataset (Singapore)

### Overview

**Source**: Kaggle Public Dataset  
**Dataset ID**: `subhamjain/health-dataset-complete-singapore`  
**URL**: https://www.kaggle.com/datasets/subhamjain/health-dataset-complete-singapore  
**Original Source**: Ministry of Health Singapore (via data.gov.sg)  
**Last Dataset Update**: 2020-04-20  
**Data Domain**: Comprehensive Singapore Healthcare Data  
**Access Method**: Kaggle Hub API  
**Format**: CSV files (35 data tables)  
**Total Size**: ~3.5 MB  
**Data Quality**: 100% completeness (no missing values)

### Dataset Composition

**Total Files**: 70
- **Data Tables (CSV)**: 35 files
- **Metadata Files (TXT)**: 28 files
- **Documentation (PDF)**: 1 file (National Nutrition Survey 2010)
- **Other**: 6 files

### Data Categories & Coverage

| Category | Tables | Time Span | Records |
|----------|--------|-----------|---------|
| Healthcare Workforce | 7 | 2006-2019 | 390 |
| Healthcare Facilities | 4 | 2009-2020 | 408 |
| Health Outcomes & Mortality | 3 | 1990-2019 | 90 |
| Public Health & Prevention | 6 | 2003-2020 | 213 |
| Healthcare Utilization | 3 | 2006-2020 | 353 |
| Healthcare Expenditure | 1 | 2006-2018 | 13 |
| Nutrition Surveys | 3 | 2004, 2010 | 54 |
| **Total** | **27** | **1990-2020** | **1,521** |

---

## Data Access & Connection

### Method 1: Download Full Dataset (Recommended)

```python
import kagglehub
import pandas as pd
from pathlib import Path

# Download entire dataset (cached locally)
dataset_path = kagglehub.dataset_download(
    "subhamjain/health-dataset-complete-singapore"
)

print(f"Dataset cached at: {dataset_path}")

# Load specific table
doctors_df = pd.read_csv(
    Path(dataset_path) / "number-of-doctors" / "number-of-doctors.csv"
)
```

**Advantages:**
- ✓ Faster for multiple table access
- ✓ Works offline after first download
- ✓ Full directory structure preserved
- ✓ Includes metadata files

### Method 2: Direct Pandas Loading (Individual Tables)

```python
import kagglehub
from kagglehub import KaggleDatasetAdapter

# Load single table directly into pandas
df = kagglehub.load_dataset(
    KaggleDatasetAdapter.PANDAS,
    "subhamjain/health-dataset-complete-singapore",
    file_path="number-of-doctors/number-of-doctors.csv"
)
```

**Advantages:**
- ✓ Simpler for single table
- ✓ Direct to DataFrame
- ✓ Less disk space

### Method 3: Automated ETL Pipeline

```python
# Use project's ETL pipeline
from src.data_processing.kaggle_connector import KaggleConnector

connector = KaggleConnector()
all_tables = connector.extract_all()

# or load specific categories
workforce_tables = connector.extract_category('workforce')
```

**Advantages:**
- ✓ Standardized column names
- ✓ Automatic validation
- ✓ Logging and error handling
- ✓ Database loading included

---

## Authentication Setup

### Prerequisites

1. **Kaggle Account** (free): https://www.kaggle.com/
2. **Python 3.7+**
3. **Required packages**:
   ```bash
   pip install kagglehub pandas
   ```

### Option A: API Key File (Recommended)

**Step 1**: Generate API Key
1. Login to Kaggle → Account Settings
2. Scroll to "API" section
3. Click "Create New API Token"
4. Download `kaggle.json`

**Step 2**: Install API Key
```bash
# Create Kaggle directory
mkdir -p ~/.kaggle

# Move downloaded file
mv ~/Downloads/kaggle.json ~/.kaggle/

# Set permissions (security requirement)
chmod 600 ~/.kaggle/kaggle.json
```

**File contents** (`~/.kaggle/kaggle.json`):
```json
{
  "username": "your_kaggle_username",
  "key": "your_api_key_here"
}
```

### Option B: Environment Variables

```bash
# Add to ~/.bashrc or ~/.zshrc
export KAGGLE_USERNAME="your_username"
export KAGGLE_KEY="your_api_key"

# Or set for current session
export KAGGLE_USERNAME="your_username"
export KAGGLE_KEY="your_api_key"
```

### Verification

```python
import kagglehub

# Test connection
try:
    dataset_path = kagglehub.dataset_download(
        "subhamjain/health-dataset-complete-singapore"
    )
    print("✓ Authentication successful!")
    print(f"Dataset location: {dataset_path}")
except Exception as e:
    print(f"✗ Authentication failed: {e}")
```

---

## Data Documentation Resources

### Quick References

| Document | Purpose | Location |
|----------|---------|----------|
| **Comprehensive Data Catalog** | Complete table schemas & descriptions | [`docs/data_dictionary/COMPREHENSIVE_DATA_CATALOG.md`](../data_dictionary/COMPREHENSIVE_DATA_CATALOG.md) |
| **Table Quick Reference** | Fast lookup & code templates | [`docs/data_dictionary/TABLE_QUICK_REFERENCE.md`](../data_dictionary/TABLE_QUICK_REFERENCE.md) |
| **Extraction Automation Guide** | ETL pipelines & scheduling | [`docs/DATA_EXTRACTION_AUTOMATION_GUIDE.md`](../DATA_EXTRACTION_AUTOMATION_GUIDE.md) |
| **Kaggle Quick Start** | Step-by-step setup | [`docs/KAGGLE_QUICK_START.md`](../KAGGLE_QUICK_START.md) |
| **Dataset Exploration (JSON)** | Machine-readable metadata | [`data/dataset_exploration.json`](../../data/dataset_exploration.json) |

### Key Tables by Use Case

**Workforce Planning:**
- `number-of-doctors.csv` (78 records, 2006-2019)
- `number-of-nurses-and-midwives.csv` (126 records, 2008-2019)
- `number-of-pharmacists.csv` (42 records, 2006-2019)

**Capacity Management:**
- `health-facilities-and-beds-in-inpatient-facilities-public-not-for-profit-private.csv` (180 records)
- `health-facilities-primary-care-dental-clinics-and-pharmacies.csv` (96 records)

**Disease Burden Analysis:**
- `age-standardised-mortality-rate-for-cancer.csv` (30 years, 1990-2019)
- `age-standardised-mortality-rate-for-stroke.csv` (30 years, 1990-2019)
- `age-standardised-mortality-rate-for-ischaemic-heart-disease.csv` (30 years)

**Public Health Programs:**
- `common-health-problems-of-students-examined-obesity-annual.csv` (48 records)
- `vaccination-and-immunisation-of-students-annual.csv` (33 records)
- `dental-index-dental-health-status-of-the-school-children-at-12-and-15-years-old.csv` (36 records)

**Financial Analysis:**
- `government-health-expenditure.csv` (13 years, 2006-2018)

**Healthcare Utilization:**
- `hospital-admission-rate-by-age-and-sex.csv` (216 records, detailed demographics)
- `residential-long-term-care-admissions.csv` (25 records)

---

## Data Quality & Characteristics

### Quality Metrics

| Metric | Value |
|--------|-------|
| Completeness | 100% (no missing values) |
| Consistency | High (standardized formats) |
| Timeliness | Annual updates (most recent: 2020) |
| Accuracy | Official government source |
| Granularity | Annual, with demographic breakdowns |

### Data Limitations

1. **Update Frequency**: Annual (not real-time)
2. **Latest Data**: 2019-2020 (dataset last updated April 2020)
3. **Geographic Granularity**: National level only (no regional breakdowns)
4. **Limited Demographics**: Primarily age, gender, race
5. **Sparse Data**: Some tables have limited years (e.g., physiotherapists: 2014-2019)

### Known Issues

- **Column naming**: Inconsistent capitalization/spacing across tables
- **Year alignment**: Expenditure uses `financial_year`, others use `year`
- **Rate bases**: Some rates per 1,000, others per 10,000
- **Sector naming**: Slight variations ("Public Sector" vs "Public")

---

## Integration Points

### Database Schema Mapping

```yaml
# Recommended database structure
schemas:
  kaggle_raw:
    description: "Raw data as extracted from Kaggle"
    tables: 35
    
  kaggle_staging:
    description: "Standardized column names, data types"
    tables: 35
    
  kaggle_analytics:
    description: "Aggregated views, derived metrics"
    views: 15+
```

### Project Integration

**Config Files:**
```yaml
# config/database.yml
data_sources:
  kaggle:
    dataset_id: "subhamjain/health-dataset-complete-singapore"
    schema: "kaggle_raw"
    refresh_schedule: "daily"
    priority: "high"
```

**ETL Scripts:**
- [`scripts/load_kaggle_data.py`](../../scripts/load_kaggle_data.py) - Main extraction script
- [`scripts/explore_kaggle_dataset.py`](../../scripts/explore_kaggle_dataset.py) - Dataset exploration
- [`src/data_processing/kaggle_connector.py`](../../src/data_processing/kaggle_connector.py) - Connector class

---

## Usage Examples

### Example 1: Quick Data Exploration

```python
import kagglehub
import pandas as pd

# Download dataset
dataset_path = kagglehub.dataset_download(
    "subhamjain/health-dataset-complete-singapore"
)

# Load and explore doctors data
doctors = pd.read_csv(f"{dataset_path}/number-of-doctors/number-of-doctors.csv")

print(f"Data shape: {doctors.shape}")
print(f"Years covered: {doctors['year'].min()} - {doctors['year'].max()}")
print(f"\nSector distribution (latest year):\n{doctors[doctors['year'] == doctors['year'].max()]['sector'].value_counts()}")
```

### Example 2: Multi-Table Analysis

```python
# Load related workforce tables
doctors = pd.read_csv(f"{dataset_path}/number-of-doctors/number-of-doctors.csv")
nurses = pd.read_csv(f"{dataset_path}/number-of-nurses-and-midwives/number-of-nurses-and-midwives.csv")

# Aggregate by year and sector
workforce_trend = pd.concat([
    doctors.groupby(['year', 'sector'])['count'].sum().rename('doctors'),
    nurses.groupby(['year', 'sector'])['count'].sum().rename('nurses')
], axis=1).reset_index()

print(workforce_trend.head())
```

### Example 3: Automated ETL

```bash
# Run full ETL pipeline
python scripts/load_kaggle_data.py

# Or use project's ETL module
python -c "from src.data_processing.etl_pipeline import run_kaggle_etl; run_kaggle_etl()"
```

---

## Support & Contact

**Dataset Issues**: https://www.kaggle.com/datasets/subhamjain/health-dataset-complete-singapore/discussion  
**Original Data Source**: Ministry of Health Singapore (data.gov.sg)  
**Project Documentation**: See [`docs/`](../) directory  
**Technical Support**: Contact project data team

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-30 | Initial comprehensive documentation |
| 0.1 | 2020-04-20 | Dataset first documented by source |

---

**Document maintained by:** Data Analytics Team  
**Last verified:** 30 January 2026  
**Next review:** Quarterly or when dataset updates

