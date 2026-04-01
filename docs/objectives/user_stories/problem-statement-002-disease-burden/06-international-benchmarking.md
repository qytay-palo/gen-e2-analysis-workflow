# International Benchmarking Against WHO Standards (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-002-US-06  
**Epic**: National Disease Burden Temporal Trends Analysis  
**Priority**: P1 (High)  
**Effort Estimate**: M (5-6 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Public Health Policy Maker evaluating Singapore's healthcare system performance**,  
I want **to benchmark Singapore's disease mortality trends against WHO global averages and high-income country standards (OECD, IHME data)**,  
So that **I can identify areas where Singapore outperforms international peers (success stories) and areas needing improvement (performance gaps)**.

---

## 🎯 Acceptance Criteria

1. **External benchmark data acquired**
   - WHO Global Health Observatory (GHO) mortality data downloaded for cancer, stroke, IHD
   - OECD Health Statistics mortality rates extracted for comparable countries
   - High-income country peer group defined (e.g., Japan, South Korea, Australia, UK, USA)
   - Benchmark data standardized to match Singapore's metrics (age-standardized per 100k)

2. **Comparative analysis completed**
   - Singapore vs WHO global average: mortality rate differences calculated for each year
   - Singapore vs high-income average: performance gap quantified
   - Peer country rankings: Singapore's rank among comparable nations
   - Trend convergence/divergence: is Singapore gap widening or narrowing over time?

3. **Performance assessment delivered**
   - Outperformance areas: diseases where Singapore has lower mortality than benchmarks
   - Underperformance areas: diseases where Singapore lags behind peers
   - Trend comparison: is Singapore improving faster or slower than global trends?
   - Statistical significance testing: are differences statistically meaningful?

4. **Deliverables generated**
   - Output file: `results/tables/international_mortality_benchmarking.csv`
   - Visualization: Multi-panel comparison (Singapore vs benchmarks over time)
   - Benchmarking report: Performance summary with international context
   - Executive brief: 3-page summary for policy makers

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ for data integration and analysis
- **External Data**: WHO API or CSV downloads, OECD data portal access
- **Visualization**: Matplotlib/Plotly for international comparisons
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for benchmarking functions

---

## 📚 Domain Knowledge References

- [Disease Burden Feature Engineering Guide](../../../../domain_knowledge/disease-burden-feature-engineering-guide.md#age-standardized-mortality-rate-asmr) - Understanding ASMR for international comparisons
- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#international-benchmarks-disease-burden) - International benchmark sources
- [Problem Statement PS-002](../../../problem_statements/ps-002-disease-burden-temporal-trends.md#objective-3) - Objective: Benchmark against international standards

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data integration and comparative analytics
- `requests>=2.31.0`: API calls to WHO GHO, OECD data portal
- `matplotlib>=3.8.0`: Benchmark visualization
- `seaborn>=0.13.0`: Statistical comparison charts
- `scipy>=1.11.0`: Statistical significance testing
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: PS-002-US-03 (Analyze mortality trends - BLOCKING)
- **Data Sources**: 
  - Internal: `shared/data/3_interim/mortality_trends_integrated_clean.parquet`
  - External: WHO GHO, OECD Health Statistics, IHME GBD database
- **Config Files**: `config/analysis.yml` (peer country list, API endpoints)

---

## ✅ Implementation Tasks

### External Data Acquisition
- [ ] Identify WHO GHO API endpoints for cancer, stroke, IHD mortality
- [ ] Download WHO global mortality data (1990-2019 if available)
- [ ] Access OECD Health Statistics via API or manual download
- [ ] Extract IHME Global Burden of Disease (GBD) estimates for comparison
- [ ] Define peer country list: Japan, South Korea, Taiwan, Australia, UK, Germany, USA
- [ ] Save external data to `shared/data/2_external/who_oecd_mortality_benchmarks.csv`

### Data Standardization
- [ ] Verify age-standardization: ensure WHO/OECD data uses same standard population
- [ ] Align time periods: match overlapping years (likely 1990-2018 for OECD)
- [ ] Standardize disease categories: map WHO ICD codes to Singapore categories
- [ ] Integrate Singapore data with benchmark data (common schema)
- [ ] Handle missing data: interpolation or flagging for incomplete benchmarks

### Comparative Analysis
- [ ] Calculate Singapore - WHO global average difference for each year
- [ ] Calculate Singapore - high-income average difference
- [ ] Rank Singapore among peer countries for each disease and year
- [ ] Compute convergence metric: is gap narrowing over time?
- [ ] Statistical testing: t-tests for significant differences

### Performance Assessment
- [ ] Identify outperformance: diseases where Singapore < peer average
- [ ] Identify underperformance: diseases where Singapore > peer average
- [ ] Quantify magnitude: % difference from benchmarks
- [ ] Trend comparison: Singapore APC vs global APC
- [ ] Generate performance scorecard: summary metrics

### Visualization
- [ ] Multi-panel time series: Singapore vs WHO vs high-income average
- [ ] Scatter plot: Singapore vs peer countries (latest year)
- [ ] Gap chart: difference from benchmark over time
- [ ] Rank chart: Singapore's position among peers
- [ ] Save figures: `reports/figures/international_mortality_benchmarking_*.png/pdf`

### Reporting
- [ ] Executive summary: key performance insights (3 pages)
- [ ] Detailed benchmarking report: methodology, findings, interpretation
- [ ] Success stories: areas where Singapore leads
- [ ] Improvement opportunities: areas to target based on peer performance
- [ ] Policy implications: what explains Singapore's position?

### Testing & Validation
- [ ] Unit tests for data integration functions
- [ ] Validate benchmark data: spot-check against published WHO/OECD reports
- [ ] Test gap calculation logic with sample data
- [ ] Validate rankings: ensure consistent ranking methodology
- [ ] Docstrings for all benchmarking functions

---

## 📌 Notes

**Data Sources**:
1. **WHO Global Health Observatory (GHO)**
   - URL: https://www.who.int/data/gho
   - Indicator codes: age-standardized NCD mortality rates
   - API: https://ghoapi.azureedge.net/api/

2. **OECD Health Statistics**
   - URL: https://data-explorer.oecd.org/
   - Dataset: Causes of mortality
   - Format: CSV download or API access
**Benchmarking Analysis Example (Polars)**:
```python
import polars as pl
from loguru import logger

# Load Singapore data
df_singapore = pl.read_parquet("shared/data/3_interim/mortality_trends_integrated_clean.parquet")

# Load WHO benchmark (after extraction)
df_who = pl.read_csv("shared/data/2_external/who_mortality_benchmarks.csv")

# Calculate performance gap
df_comparison = (
    df_singapore
    .join(df_who.rename({'rate': 'who_global_avg'}), on=['year', 'disease_category'], how='left')
    .with_columns([
        (pl.col('mortality_rate') - pl.col('who_global_avg')).alias('gap_from_who'),
        ((pl.col('mortality_rate') / pl.col('who_global_avg') - 1) * 100).alias('gap_pct')
    ])
)

logger.info(f"✓ Benchmarking complete: {len(df_comparison)} year-disease comparisons")
```

**Peer Country Selection Criteria**:
- High-income status (World Bank classification)
- Similar healthcare system sophistication
- Comparable demographic characteristics (aging population)
- Data availability (complete time series)

**Suggested Peers**: Japan, South Korea, Taiwan, Australia, New Zealand, UK, Germany, France

**Expected Findings** (hypotheses):
- **Cancer**: Singapore likely outperforms global average, close to best-in-class
- **Stroke**: Singapore may lead due to excellent hypertension control programs
- **IHD**: Possibly middle-of-pack among high-income countries

**Interpretation Caveats**:
- Age-standardization differences across sources
- ICD code version changes over time
- Data completeness varies by country
- Cultural factors (e.g., diet, smoking prevalence) affect comparability

**Output Schema**:
```yaml
# results/tables/international_mortality_benchmarking.csv
columns:
  year: int32
  disease_category: categorical
  singapore_rate: float64
  who_global_avg: float64
  high_income_avg: float64
  singapore_rank: int  # among peer countries
  gap_from_who: float64  # negative = outperformance
  gap_from_peers: float64
  gap_trend: string  # converging, diverging, stable
```

---

## Implementation Plan

### 1. Feature Overview & Component Analysis

**Purpose**: Benchmark Singapore's disease mortality trends against WHO global averages and high-income country standards to identify performance gaps and success areas.

**Primary User Role**: Public Health Policy Maker

**Reuse Strategy**:
- ✅ **`shared/data/3_interim/mortality_trends_integrated_clean.parquet`** - Singapore mortality data (US-02)
- ✅ **`shared/src/utils/logger.py`** - Logging utilities
- 🆕 **`shared/src/data_processing/external_data_fetcher.py`** - WHO/OECD API client
- 🆕 **`shared/data/2_external/who_mortality_benchmarks.csv`** - WHO benchmark data
- 🆕 **`shared/data/2_external/oecd_mortality_benchmarks.csv`** - OECD peer country data
- 🆕 **`shared/src/analysis/international_benchmarking.py`** - Benchmarking analytics
- 🆕 **`shared/src/visualization/benchmarking_viz.py`** - Benchmarking visualizations

### 3. ML Model Evaluation & Selection

**N/A** - Descriptive comparative analytics only.

### 4-6. Code Specifications

**Key Functions** (`shared/src/data_processing/external_data_fetcher.py`):

```python
import polars as pl
import requests
from pathlib import Path
from loguru import logger
from typing import List, Optional
from datetime import datetime

def fetch_who_mortality_data(
    disease_codes: List[str],
    start_year: int = 1990,
    end_year: int = 2019
) -> pl.DataFrame:
    """Fetch WHO Global Health Observatory mortality data.
    
    Args:
        disease_codes: List of WHO GHO indicator codes
        start_year: Starting year
        end_year: Ending year
        
    Returns:
        DataFrame with [year, disease_category, who_global_avg]
    """
    base_url = "https://ghoapi.azureedge.net/api"
    
    all_data = []
    for code in disease_codes:
        try:
            url = f"{base_url}/{code}"
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            
            data = response.json()
            # Parse WHO JSON structure
            for record in data.get('value', []):
                year = int(record.get('TimeDim', 0))
                if start_year <= year <= end_year:
                    all_data.append({
                        'year': year,
                        'disease_code': code,
                        'who_global_avg': float(record.get('NumericValue', 0))
                    })
            
            logger.info(f"✓ Fetched WHO data for {code}")
        except requests.RequestException as e:
            logger.error(f"Failed to fetch WHO data for {code}: {e}")
            raise
    
    df = pl.DataFrame(all_data)
    logger.info(f"Fetched {df.height} WHO records across {len(disease_codes)} diseases")
    return df


def fetch_oecd_mortality_data(
    countries: List[str],
    diseases: List[str]
) -> pl.DataFrame:
    """Fetch OECD Health Statistics mortality data.
    
    Args:
        countries: List of ISO3 country codes (e.g., ['JPN', 'KOR', 'AUS'])
        diseases: Disease categories
        
    Returns:
        DataFrame with peer country mortality rates
    """
    # Note: OECD API may require authentication or CSV download
    # Placeholder for actual implementation
    logger.warning("OECD data fetching requires manual download or authenticated API - implement based on data availability")
    
    # Manual CSV loading fallback
    oecd_file = Path("shared/data/2_external/oecd_mortality_manual_download.csv")
    if oecd_file.exists():
        df = pl.read_csv(oecd_file)
        logger.info(f"Loaded OECD data from manual CSV: {df.height} records")
        return df
    else:
        raise FileNotFoundError("OECD data not available - download manually from https://data-explorer.oecd.org/")
```

**Benchmarking Analysis** (`shared/src/analysis/international_benchmarking.py`):

```python
import polars as pl
from loguru import logger
from typing import Dict, List

def calculate_benchmark_gaps(
    df_singapore: pl.DataFrame,
    df_benchmarks: pl.DataFrame,
    benchmark_name: str = 'WHO'
) -> pl.DataFrame:
    """Calculate Singapore's gap from benchmark rates.
    
    Args:
        df_singapore: Singapore mortality data
        df_benchmarks: Benchmark mortality data
        benchmark_name: Name of benchmark source
        
    Returns:
        DataFrame with gap metrics
    """
    df_comparison = df_singapore.join(
        df_benchmarks,
        on=['year', 'disease_category'],
        how='left',
        suffix=f'_{benchmark_name.lower()}'
    ).with_columns([
        (pl.col('mortality_rate') - pl.col(f'mortality_rate_{benchmark_name.lower()}')).alias(f'gap_from_{benchmark_name.lower()}'),
        ((pl.col('mortality_rate') / pl.col(f'mortality_rate_{benchmark_name.lower()}') - 1) * 100).alias(f'gap_pct_{benchmark_name.lower()}')
    ])
    
    logger.info(f"Calculated {benchmark_name} benchmark gaps for {df_comparison.height} records")
    return df_comparison


def rank_among_peers(
    df_all_countries: pl.DataFrame,
    singapore_country_code: str = 'SGP'
) -> pl.DataFrame:
    """Rank Singapore among peer countries.
    
    Args:
        df_all_countries: Mortality rates for all countries
        singapore_country_code: Singapore ISO code
        
    Returns:
        DataFrame with Singapore ranks per year/disease
    """
    df_ranked = df_all_countries.with_columns([
        pl.col('mortality_rate').rank(method='ordinal').over(['year', 'disease_category']).alias('rank')
    ])
    
    df_singapore_ranks = df_ranked.filter(pl.col('country_code') == singapore_country_code)
    
    logger.info(f"Ranked Singapore for {df_singapore_ranks.height} year-disease pairs")
    return df_singapore_ranks
```

### 10. Testing Strategy

```python
# shared/tests/unit/test_international_benchmarking.py

def test_fetch_who_data_mock(mocker):
    """Test WHO API fetch with mocked response."""
    mock_response = {
        'value': [
            {'TimeDim': 1990, 'NumericValue': 150.0},
            {'TimeDim': 2019, 'NumericValue': 120.0}
        ]
    }
    
    mocker.patch('requests.get', return_value=mocker.Mock(json=lambda: mock_response, status_code=200))
    
    df = fetch_who_mortality_data(['CANCER_RATE'], 1990, 2019)
    
    assert df.height == 2
    assert df['year'].to_list() == [1990, 2019]


def test_calculate_benchmark_gaps():
    """Test benchmark gap calculation."""
    df_sg = pl.DataFrame({
        'year': [1990, 2019],
        'disease_category': ['cancer', 'cancer'],
        'mortality_rate': [150.0, 120.0]
    })
    
    df_who = df_sg.rename({'mortality_rate': 'mortality_rate_who'}).with_columns([
        pl.col('mortality_rate_who') * 1.2  # WHO 20% higher
    ])
    
    result = calculate_benchmark_gaps(df_sg, df_who, 'WHO')
    
    assert 'gap_from_who' in result.columns
    assert result['gap_from_who'][0] < 0  # Negative = Singapore better
```

### 11. Implementation Steps

**Phase 1: External Data Acquisition**
- [ ] Identify WHO GHO API indicator codes for cancer, stroke, IHD
- [ ] Implement `fetch_who_mortality_data()` with API calls
- [ ] Download OECD mortality data manually (or implement API if available)
- [ ] Save external data to `shared/data/2_external/`
- [ ] Validate external data completeness and quality

**Phase 2: Benchmarking Analysis**
- [ ] Implement `calculate_benchmark_gaps()` function
- [ ] Implement `rank_among_peers()` function
- [ ] Calculate `gap trends (converging/diverging) over time
- [ ] Generate performance scorecard

**Phase 3: Visualization & Reporting**
- [ ] Create multi-panel time series charts (Singapore vs WHO vs high-income avg)
- [ ] Create gap charts showing differences from benchmarks
- [ ] Create ranking visualizations
- [ ] Generate 3-page executive brief
- [ ] Document methodology and interpretation caveats

**Phase 4: Testing & Validation**
- [ ] Unit tests for all functions
- [ ] Mock external API calls for testing
- [ ] Validate benchmark calculations
- [ ] Integration test full pipeline

### 14. Data Quality & Validation

**External Data Quality Checks**:
- Verify WHO data completeness (may have gaps for early years)
- Check OECD data availability across all peer countries
- Validate age-standardization methodology alignment
- Document data source limitations and caveats

**Critical Validation**:
- Singapore rates should match internal data (cross-check)
- Benchmark averages should be reasonable (sanity checks)
- Rankings should be consistent with published reports

### 18. Success Metrics

- ✅ WHO global average data obtained for 1990-2019 (or best available)
- ✅ At least 5 peer countries included in benchmarking
- ✅ Singapore ranked for all 3 diseases
- ✅ Performance gaps quantified (absolute and relative)
- ✅ Executive brief generated and reviewed

### 20. Security & Privacy

**API Authentication**:
```python
# Store API keys in environment variables
import os
from dotenv import load_dotenv

load_dotenv()

WHO_API_KEY = os.getenv('WHO_API_KEY', None)  # If required
OECD_API_KEY = os.getenv('OECD_API_KEY', None)  # If required
```

**No PII/PHI** - All data aggregated at national level.

---

✅ **US-06 Implementation Plan Complete** - Ready for external data acquisition and benchmarking analysis
3. **IHME Global Burden of Disease (GBD)**
   - URL: http://ghdx.healthdata.org/gbd-results-tool
   - Comprehensive disease burden estimates
   - Download tool: can specify countries, diseases, years

**Polars Integration Example**:
```python
import polars as pl
import requests
from loguru import logger

# Example: WHO GHO API call
def fetch_who_mortality(indicator_code, year_range):
    """Fetch WHO mortality data via API"""
    base_url = "https://ghoapi.azureedge.net/api"
    endpoint = f"{base_url}/{indicator_code}"
    response = requests.get(endpoint)
    data = response.json()
    # Parse and convert to Polars DataFrame
    df_who = pl.DataFrame(data['value'])
    return df_who

# Integrate with Singapore data
df_singapore = pl.read_parquet("shared/data/3_interim/mortality_trends_integrated_clean.parquet")
df_who = fetch_who_mortality("cancer_mortality", "1990-2019")

# Calculate performance gap
df_comparison = (
    df_singapore
    .join(df_who.rename({'rate': 'who_global_avg'}), on=['year', 'disease_category'], how='left')
    .with_columns([
        (pl.col('mortality_rate') - pl.col('who_global_avg')).alias('gap_from_who'),
        ((pl.col('mortality_rate') / pl.col('who_global_avg') - 1) * 100).alias('gap_pct')
    ])
)

logger.info(f"✓ Benchmarking complete: {len(df_comparison)} year-disease comparisons")
```

**Peer Country Selection Criteria**:
- High-income status (World Bank classification)
- Similar healthcare system sophistication
- Comparable demographic characteristics (aging population)
- Data availability (complete time series)

**Suggested Peers**: Japan, South Korea, Taiwan, Australia, New Zealand, UK, Germany, France

**Expected Findings** (hypotheses):
- **Cancer**: Singapore likely outperforms global average, close to best-in-class
- **Stroke**: Singapore may lead due to excellent hypertension control programs
- **IHD**: Possibly middle-of-pack among high-income countries

**Interpretation Caveats**:
- Age-standardization differences across sources
- ICD code version changes over time
- Data completeness varies by country
- Cultural factors (e.g., diet, smoking prevalence) affect comparability

**Output Schema**:
```yaml
# results/tables/international_mortality_benchmarking.csv
columns:
  year: int32
  disease_category: categorical
  singapore_rate: float64
  who_global_avg: float64
  high_income_avg: float64
  singapore_rank: int  # among peer countries
  gap_from_who: float64  # negative = outperformance
  gap_from_peers: float64
  gap_trend: string  # converging, diverging, stable
```
