# Calculate Workforce-to-Population Ratios & Benchmarks (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-001-US-04  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: S (3-4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Policy Analyst evaluating workforce adequacy**,  
I want **to calculate workforce-to-population density ratios (per 10,000 population) for all professions and compare against WHO/OECD international benchmarks**,  
So that **I can identify Singapore's workforce gaps or surpluses relative to international standards and prioritize professions needing intervention**.

---

## 🎯 Acceptance Criteria

1. **Population data integrated**
   - Population estimates derived from hospital admission rate data (using denominators)
   - Annual population estimates validated against external sources (Singapore Dept of Statistics)
   - Population data available for 2006-2019 matching workforce years

2. **Workforce density metrics calculated**
   - Density calculated: workforce per 10,000 population for each profession
   - Metrics: physicians density, nurses density, pharmacists density, dentists density
   - Calculated annually (2006-2019) to show density trends over time
   - Sector-specific densities: public vs private workforce density

3. **International benchmarking completed**
   - WHO minimum thresholds incorporated (physicians: 10/10k, nurses: 30/10k, total: 44.5/10k)
   - OECD benchmarks incorporated (physicians: 34/10k avg, nurses: 50-180/10k range)
   - Gap analysis: Singapore vs WHO minimums and OECD targets
   - Ranking: Singapore's position relative to high-income country benchmarks

4. **Data output requirement**
   - Output file: `results/tables/workforce_density_benchmarks.csv`
   - Format: CSV with columns (profession, year, workforce_count, population, density_per_10k, who_threshold, oecd_avg, gap_to_oecd)
   - Schema documented: Yes
   - Benchmark visualization: `reports/figures/workforce_density_benchmarks.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for data processing)
- **Visualization**: Matplotlib or Plotly for benchmark comparison charts
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for density calculation functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#1-health-workers-density) - Formula for density calculations and international benchmarks
- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#2-physicians-density) - WHO and OECD physician benchmarks
- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#3-nurses--midwives-density) - Nursing workforce benchmarks

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data joining and density calculations
- `matplotlib>=3.8.0` or `plotly>=5.18.0`: Benchmark comparison charts
- `loguru>=0.7.0`: Structured logging
- `pyyaml>=6.0`: Load benchmark thresholds from config

### Internal Dependencies
- **Upstream**: PS-001-US-02 (Clean workforce data - BLOCKING)
- **Data Sources**: 
  - `shared/data/3_interim/workforce_integrated_clean.parquet`
  - `shared/data/1_raw/utilization/hospital-admission-rate-by-age-and-sex.csv` (for population denominators)
- **Config Files**: `config/analysis.yml` (international benchmark values)
- **External References**: `shared/data/2_external/who_oecd_benchmarks.yml` (create this file with benchmark data)

---

## ✅ Implementation Tasks

### Population Data Preparation
- [ ] Extract population data from hospital admission rate table (admission_rate * denominator logic)
- [ ] Validate population estimates against Singapore Dept of Statistics (external validation)
- [ ] Create annual population estimates dataframe (years 2006-2019)
- [ ] Handle missing years if necessary (interpolation or external data)

### Density Calculations
- [ ] Join workforce data with population data by year
- [ ] Calculate density: `(workforce_count / population) * 10,000`
- [ ] Calculate total health workers density (sum all professions)
- [ ] Calculate sector-specific densities (public vs private)
- [ ] Create time series of density trends (2006-2019)

### Benchmark Integration
- [ ] Create benchmark configuration file: `shared/data/2_external/who_oecd_benchmarks.yml`
- [ ] Document WHO thresholds: physicians (10/10k), nurses (30/10k), total (44.5/10k)
- [ ] Document OECD averages: physicians (~34/10k), nurses (~50-180/10k)
- [ ] Join benchmarks with Singapore density data
- [ ] Calculate gaps: Singapore density - benchmark threshold

### Comparative Analysis
- [ ] Gap analysis: professions below WHO minimum thresholds
- [ ] Comparison: Singapore vs OECD averages (percentage difference)
- [ ] Trend analysis: is Singapore converging toward or diverging from benchmarks?
- [ ] Identify priority professions: largest gaps requiring intervention

### Visualization
- [ ] Create benchmark comparison bar chart (Singapore vs WHO vs OECD)
- [ ] Create time series chart: Singapore density trends with benchmark lines
- [ ] Create gap analysis chart: bar chart showing gap magnitude per profession
- [ ] Export figures in PNG and PDF formats

### Testing & Validation
- [ ] Unit tests for density calculation: `assert abs(calculated_density - expected) < 0.1`
- [ ] Validate population estimates: compare against official statistics
- [ ] Test edge cases: missing years, zero population (should not occur)
- [ ] Validate benchmarks: ensure WHO/OECD values match published standards

### Documentation
- [ ] Docstrings for density calculation functions (Google style)
- [ ] Document benchmark sources in `shared/data/2_external/README.md`
- [ ] Analysis summary: `results/workforce_density_insights.md`
- [ ] Update data dictionary with density metrics

---

## 📌 Notes

**Density Calculation Formula** (from domain knowledge):
```python
Density = (Number of health workers / Total population) × 10,000
```

**Polars Implementation**:
```python
import polars as pl

# Join workforce with population
df_density = (
    df_workforce
    .join(df_population, on='year', how='left')
    .with_columns([
        (pl.col('count') / pl.col('population') * 10_000).alias('density_per_10k')
    ])
)

# Add benchmarks
df_density = df_density.with_columns([
    pl.when(pl.col('profession') == 'doctors')
      .then(10.0)  # WHO minimum
      .when(pl.col('profession') == 'nurses_midwives')
      .then(30.0)
      .otherwise(None)
      .alias('who_threshold'),
    
    pl.when(pl.col('profession') == 'doctors')
      .then(34.0)  # OECD average
      .when(pl.col('profession') == 'nurses_midwives')
      .then(100.0)
      .otherwise(None)
      .alias('oecd_avg')
])

# Calculate gaps
df_density = df_density.with_columns([
    (pl.col('density_per_10k') - pl.col('oecd_avg')).alias('gap_to_oecd')
])
```

**International Benchmark Values** (create in `shared/data/2_external/who_oecd_benchmarks.yml`):
```yaml
who_thresholds:
  total_health_workers: 44.5
  physicians: 10.0
  nurses_midwives: 30.0

oecd_averages_2024:
  physicians: 34.0
  nurses_midwives: 100.0  # midpoint of 50-180 range
  pharmacists: 8.0
  dentists: 6.5

high_income_targets:
  physicians: [30, 40]  # range
  nurses_midwives: [80, 120]
```

**Expected Insights**:
- Singapore likely meets WHO minimums but may be below OECD averages
- Identify specific professions with largest gaps (e.g., nurses if density <100/10k)
- Provide evidence for "which professions need priority recruitment"

**Validation Sources**:
- WHO Global Health Observatory: https://www.who.int/data/gho
- OECD Health Statistics: https://www.oecd.org/health/health-data.htm
- Singapore Dept of Statistics population data

---

## Implementation Plan

### 1. Feature Overview

Calculate workforce-to-population density ratios and benchmark against international standards (WHO/OECD). Primary goal is to identify Singapore's workforce gaps or surpluses relative to international benchmarks and prioritize professions needing intervention.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Cleaned workforce data (US-02), trend analysis methods (US-03)
**Create**: `shared/src/analysis/density_metrics.py`, `shared/data/2_external/who_oecd_benchmarks.yml`

### 3. Key Implementation Components

**[CREATE] `shared/src/analysis/density_metrics.py`**
```python
def calculate_workforce_density(
    workforce_count: int,
    population: int,
    per_x_population: int = 10000
) -> float:
    """
    Calculate workforce density per X population.
    
    Formula: (workforce_count / population) × per_x_population
    
    Args:
        workforce_count: Total workforce in profession
        population: Total population
        per_x_population: Denominator (default 10,000)
        
    Returns:
        Density ratio
        
    Example:
        >>> calculate_workforce_density(1000, 5000000, 10000)
        2.0  # 2 workers per 10,000 population
    """
    return (workforce_count / population) * per_x_population


def compare_to_benchmarks(
    actual_density: float,
    who_threshold: float,
    oecd_avg: float
) -> dict:
    """
    Compare actual density against benchmarks.
    
    Returns:
        Dict with gap analysis and classification
    """
    return {
        'actual_density': actual_density,
        'who_threshold': who_threshold,
        'oecd_avg': oecd_avg,
        'gap_vs_who': actual_density - who_threshold,
        'gap_vs_oecd': actual_density - oecd_avg,
        'meets_who_min': actual_density >= who_threshold,
        'meets_oecd_avg': actual_density >= oecd_avg
    }
```

**[CREATE] `shared/data/2_external/who_oecd_benchmarks.yml`**
```yaml
# WHO and OECD Workforce Density Benchmarks
# Source: WHO Global Health Observatory, OECD Health Statistics 2024

who_benchmarks:
  total_health_workers:
    threshold: 44.5  # per 10,000 population
    description: "WHO minimum for essential health coverage"
  
  physicians:
    threshold: 10.0  # per 10,000
    description: "WHO minimum physicians density"
  
  nurses_midwives:
    threshold: 30.0  # per 10,000
    description: "WHO minimum nursing density"

oecd_benchmarks:
  physicians:
    average: 34.0  # per 10,000
    range: [20, 50]
    high_performers: [44, 50]  # Top quartile (e.g., Norway, Germany)
  
  nurses_midwives:
    average: 90.0  # per 10,000
    range: [50, 180]
    high_performers: [150, 180]  # Top quartile (e.g., Norway, Switzerland)
  
  pharmacists:
    average: 8.5  # per 10,000
    range: [5, 12]
  
  dentists:
    average: 6.5  # per 10,000
    range: [4, 10]
```

### 4. Data Pipeline

**Population Data Integration**:
```python
# Extract population from hospital admission rate denominators
population_df = (
    pl.read_csv("shared/data/1_raw/utilization/hospital-admission-rate-by-age-and-sex.csv")
    .groupby('year')
    .agg([
        pl.col('denominator').sum().alias('total_population')
    ])
)
```

**Density Calculation Pipeline**:
```python
density_df = (
    workforce_df
    .join(population_df, on='year', how='inner')
    .with_columns([
        ((pl.col('count') / pl.col('total_population')) * 10000)
        .alias('density_per_10k')
    ])
    .join(benchmark_df, on='profession', how='left')
    .with_columns([
        (pl.col('density_per_10k') - pl.col('who_threshold')).alias('gap_vs_who'),
        (pl.col('density_per_10k') - pl.col('oecd_avg')).alias('gap_vs_oecd')
    ])
)
```

### 5. Visualization Specifications

**Chart 1: Density vs Benchmarks**
- Type: Dot plot with threshold lines
- X-axis: Profession
- Y-axis: Density (per 10,000)
- Points: Singapore actual (2019)
- Lines: WHO minimum (dashed red), OECD average (solid blue)
- Annotations: Gap percentages

**Chart 2: Density Trends 2006-2019**
- Type: Line chart per profession
- Show density evolution over time
- Compare to static benchmark lines

### 6. Implementation Steps

**Phase 1: Data Preparation (Day 1)**
- [ ] Extract population estimates from hospital admission data
- [ ] Validate population against Singapore Dept of Statistics
- [ ] Create annual population DataFrame (2006-2019)

**Phase 2: Benchmark Setup (Day 1)**
- [ ] Create `who_oecd_benchmarks.yml`
- [ ] Document benchmark sources and dates
- [ ] Load benchmarks into analysis pipeline

**Phase 3: Density Calculations (Day 2)**
- [ ] Join workforce + population data
- [ ] Calculate density per 10,000 for all professions
- [ ] Calculate total health workers density
- [ ] Calculate sector-specific densities (public vs private)

**Phase 4: Gap Analysis (Days 2-3)**
- [ ] Compare Singapore densities vs WHO thresholds
- [ ] Compare vs OECD averages
- [ ] Rank professions by gap severity
- [ ] Identify priority professions (largest gaps)

**Phase 5: Visualization & Reporting (Days 3-4)**
- [ ] Generate benchmark comparison charts
- [ ] Create density trend visualizations
- [ ] Export results to `results/tables/workforce_density_benchmarks.csv`
- [ ] Document findings in analysis report

### 7. Testing Strategy

```python
def test_density_calculation():
    """Test density calculation formula."""
    workforce = 1000
    population = 5000000
    
    density = calculate_workforce_density(workforce, population, 10000)
    
    # Expected: (1000 / 5,000,000) × 10,000 = 2.0
    assert density == 2.0


def test_benchmark_comparison():
    """Test gap analysis vs benchmarks."""
    actual = 25.0  # physicians per 10k
    who_min = 10.0
    oecd_avg = 34.0
    
    result = compare_to_benchmarks(actual, who_min, oecd_avg)
    
    assert result['meets_who_min'] == True
    assert result['meets_oecd_avg'] == False
    assert result['gap_vs_oecd'] == -9.0  # 9 below OECD average
```

### 8. Success Metrics

- ✅ Density calculated for all professions (2006-2019)
- ✅ Comparison against WHO minimums and OECD averages
- ✅ Priority professions identified (top 3 gaps)
- ✅ Visualization of Singapore vs international benchmarks

### 9. References

- [Healthcare Workforce Metrics & KPIs](../../../domain_knowledge/healthcare-workforce-metrics-kpis.md) - Density formulas
- WHO Global Health Observatory
- OECD Health Statistics 2024

---

**Implementation Plan Complete for US-04**  
**Dependencies**: US-02 (cleaned data), Population data from utilization tables  
**Enables**: US-07 (Gap analysis uses density benchmarks for demand estimation)
