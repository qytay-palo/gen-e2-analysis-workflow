# Identify Critical Workforce Gaps & Shortage Risks (Lifecycle Stage: Advanced Analysis & Modeling)

**Story ID**: PS-001-US-07  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5-6 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Workforce Planning Policy Analyst**,  
I want **to compare projected workforce supply against estimated demand based on population aging and utilization trends to identify critical gaps, shortage timelines, and priority professions**,  
So that **I can provide evidence-based recommendations on which professions require immediate intervention and the magnitude of shortages expected by 2025 and 2030**.

---

## 🎯 Acceptance Criteria

1. **Demand estimation completed**
   - Demand estimated using workforce-to-population ratios: maintain 2019 ratio vs improve to OECD average
   - Scenarios: (1) Baseline demand (maintain 2019 density), (2) OECD target (converge to OECD average by 2030)
   - Demographic adjustments: account for population growth and aging in demand projections
   - Utilization-based adjustment: scale demand by historical admission rate growth

2. **Supply-demand gap analysis**
   - Annual gaps calculated 2020-2030: `gap = projected_supply - estimated_demand`
   - Gap metrics: absolute shortage (headcount), relative shortage (% below demand), first year of shortage
   - Severity classification: critical (>20% shortage), moderate (10-20%), manageable (<10%)
   - Sector-specific gaps: public vs private sector shortage analysis

3. **Priority profession identification**
   - Professions ranked by shortage severity and urgency (which gaps appear soonest?)
   - Top 3 critical professions identified (largest absolute and relative gaps)
   - Timeline analysis: when does each profession hit critical shortage threshold?

4. **Data output requirement**
   - Output file: `results/tables/workforce_gap_analysis.csv`
   - Format: CSV with columns (profession, year, supply_forecast, demand_baseline, demand_oecd, gap_baseline, gap_oecd, shortage_pct, severity_category)
   - Schema documented: Yes
   - Gap visualization: `reports/figures/workforce_supply_demand_gaps.png`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for gap calculations)
- **Visualization**: Matplotlib or Plotly for supply-demand charts
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for gap calculation functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#2-physicians-density) - OECD benchmark targets for demand estimation
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#objectives) - Objective 3: Identify critical workforce gaps and shortage risks

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Gap calculations and scenario modeling
- `matplotlib>=3.8.0` or `plotly>=5.18.0`: Supply-demand gap visualizations
- `loguru>=0.7.0`: Structured logging
- `pyyaml>=6.0`: Load scenario configurations

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-06 (Forecast models - BLOCKING)
  - PS-001-US-04 (Workforce density ratios - BLOCKING for demand estimation)
- **Data Sources**: 
  - `shared/data/4_processed/workforce_forecasts_2020_2030.parquet`
  - `results/tables/workforce_density_benchmarks.csv`
- **Config Files**: `config/analysis.yml` (demand scenarios: baseline vs OECD targets)
- **External Data**: Population projections (Singapore Dept of Statistics or extrapolate from historical trends)

---

## ✅ Implementation Tasks

### Demand Estimation
- [ ] Load population projections for 2020-2030 (extrapolate from historical trends or use external data)
- [ ] Calculate baseline demand: `demand = projected_population * density_2019 / 10,000`
- [ ] Calculate OECD target demand: `demand = projected_population * oecd_benchmark / 10,000`
- [ ] Create demand scenarios dataset with annual demand estimates per profession
- [ ] Document demand calculation methodology and assumptions

### Supply-Demand Gap Analysis
- [ ] Join supply forecasts with demand estimates by profession and year
- [ ] Calculate absolute gaps: `gap = supply_forecast - demand_estimate`
- [ ] Calculate relative gaps: `gap_pct = (gap / demand_estimate) * 100`
- [ ] Identify first year of shortage per profession (gap < 0)
- [ ] Classify severity: critical (gap_pct < -20%), moderate (-20% to -10%), manageable (-10% to 0%)

### Timeline & Urgency Analysis
- [ ] Identify professions with immediate shortages (gap < 0 in 2020)
- [ ] Identify professions with near-term shortages (gap < 0 by 2025)
- [ ] Identify professions with long-term shortages (gap < 0 by 2030)
- [ ] Calculate cumulative shortage: sum of annual gaps 2020-2030

### Priority Profession Ranking
- [ ] Rank professions by absolute gap magnitude (largest workforce shortage)
- [ ] Rank professions by relative gap (largest % shortfall)
- [ ] Rank professions by urgency (earliest shortage year)
- [ ] Create composite priority score: weighted average of magnitude, severity, urgency
- [ ] Identify top 3 priority professions requiring immediate intervention

### Sector-Specific Gap Analysis
- [ ] Calculate public vs private gaps separately (if sector-level forecasts available)
- [ ] Identify if shortages concentrated in public or private sector
- [ ] Recommend sector-targeted interventions (e.g., public sector recruitment drives)

### Visualization
- [ ] Create supply-demand gap charts: line plots showing supply forecast vs demand scenarios
- [ ] Create heatmap: gap severity by profession and year
- [ ] Create ranking chart: top professions by shortage magnitude
- [ ] Create timeline chart: when each profession hits critical shortage
- [ ] Export figures in PNG and PDF formats

### Testing & Validation
- [ ] Unit tests for gap calculation: validate formula correctness
- [ ] Test edge cases: professions with supply > demand (surplus scenarios)
- [ ] Validate severity classification logic
- [ ] Integration test: end-to-end gap analysis pipeline
- [ ] Sensitivity test: how do gaps change if OECD target varies ±10%?

### Documentation
- [ ] Docstrings for gap calculation functions (Google style)
- [ ] Gap analysis report: `results/workforce_gap_analysis_insights.md`
  - Top 3 priority professions with shortage details
  - Shortage timelines and severity classifications
  - Scenario comparison (baseline vs OECD targets)
  - Policy recommendations
- [ ] Update README: `shared/src/analysis/README.md`

---

## 📌 Notes

**Demand Calculation Formula**:
```python
import polars as pl

# Baseline demand (maintain 2019 density)
df_demand = df_population.with_columns([
    (pl.col('population_2020') * pl.col('density_2019') / 10_000).alias('demand_baseline_2020'),
    (pl.col('population_2025') * pl.col('density_2019') / 10_000).alias('demand_baseline_2025'),
    (pl.col('population_2030') * pl.col('density_2019') / 10_000).alias('demand_baseline_2030')
])

# OECD target demand
df_demand = df_demand.with_columns([
    (pl.col('population_2030') * pl.col('oecd_benchmark') / 10_000).alias('demand_oecd_2030')
])
```

**Gap Calculation (Polars)**:
```python
# Join supply forecasts with demand
df_gap = (
    df_supply
    .join(df_demand, on=['profession', 'year'], how='left')
    .with_columns([
        (pl.col('forecast_count') - pl.col('demand_baseline')).alias('gap_baseline'),
        (pl.col('forecast_count') - pl.col('demand_oecd')).alias('gap_oecd'),
        ((pl.col('forecast_count') - pl.col('demand_baseline')) / pl.col('demand_baseline') * 100)
        .alias('gap_pct_baseline')
    ])
)

# Severity classification
df_gap = df_gap.with_columns([
    pl.when(pl.col('gap_pct_baseline') < -20).then(pl.lit('critical'))
      .when(pl.col('gap_pct_baseline') < -10).then(pl.lit('moderate'))
      .when(pl.col('gap_pct_baseline') < 0).then(pl.lit('manageable'))
      .otherwise(pl.lit('surplus'))
      .alias('severity_category')
])
```

**Priority Scoring Example**:
```python
# Composite priority score (0-100 scale)
df_priority = df_gap.with_columns([
    (
        0.5 * pl.col('gap_magnitude_normalized') +  # 50% weight on shortage size
        0.3 * pl.col('gap_severity_normalized') +   # 30% weight on % shortfall
        0.2 * pl.col('urgency_normalized')          # 20% weight on timeline
    ).alias('priority_score')
])

# Rank professions
df_ranked = df_priority.sort('priority_score', descending=True)
```

**Expected Insights**:
- Nurses likely to show largest absolute gap (largest workforce category)
- Dentists/pharmacists may show earlier shortages (smaller base, slower growth)
- Public sector likely more constrained than private (budget limits)
- Critical threshold likely reached 2025-2027 for priority professions

**Demand Scenarios**:
1. **Baseline**: Maintain 2019 workforce-to-population ratios (assumes no quality improvement)
2. **OECD Target**: Converge to OECD average densities by 2030 (aspirational quality standard)
3. **Sensitivity**: Test ±10% variation in demand assumptions (uncertainty analysis)

**Policy Recommendations Framework**:
- **Critical professions** (gap > 20%): Immediate training program expansion, overseas recruitment
- **Moderate professions** (gap 10-20%): Retention improvements, phased capacity increase
- **Manageable professions** (gap < 10%): Monitor trends, maintain current programs

---

## Implementation Plan

### 1. Gap Analysis Overview

Compare workforce supply forecasts (US-06) against projected demand (population growth + density targets). Identify shortages, quantify gaps, rank professions by priority. Output actionable insights for policymakers.

### 2. Component Analysis & Reuse Strategy

**Reuse**: Forecasts (US-06), population ratios (US-04), OECD benchmarks (US-04)
**Create**: `shared/src/analysis/demand_gap_analyzer.py`, gap analysis tests

### 3. Key Implementation Components

**[CREATE] `shared/src/analysis/demand_gap_analyzer.py`**

```python
"""
Workforce Demand-Supply Gap Analysis
====================================

Compare forecasted supply against demographic demand projections.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import numpy as np
from pathlib import Path
from typing import Dict, List
from dataclasses import dataclass
from loguru import logger
import yaml


@dataclass
class GapAnalysisResult:
    """Container for gap analysis outputs."""
    profession: str
    sector: str
    year: int
    supply_forecast: float
    demand_baseline: float
    demand_oecd: float
    gap_baseline: float
    gap_oecd: float
    gap_pct_baseline: float
    severity_category: str
    priority_score: float


class DemandGapAnalyzer:
    """
    Analyze workforce demand-supply gaps.
    
    Demand Scenarios:
    - Baseline: Maintain 2019 density ratios
    - OECD: Converge to OECD average densities
    - Sensitivity: ±10% variation
    """
    
    def __init__(
        self,
        forecast_df: pl.DataFrame,
        population_df: pl.DataFrame,
        density_df: pl.DataFrame,
        benchmark_config_path: str
    ):
        """
        Initialize gap analyzer.
        
        Args:
            forecast_df: Workforce forecasts from US-06
            population_df: Population projections by year
            density_df: Historical density ratios from US-04
            benchmark_config_path: Path to OECD benchmarks YAML
        """
        self.forecasts = forecast_df
        self.population = population_df
        self.density = density_df
        self.benchmarks = self._load_benchmarks(benchmark_config_path)
        logger.info("Initialized DemandGapAnalyzer")
    
    def _load_benchmarks(self, config_path: str) -> dict:
        """Load OECD benchmark densities."""
        with open(config_path, 'r') as f:
            benchmarks = yaml.safe_load(f)
        logger.info(f"Loaded benchmarks from {config_path}")
        return benchmarks
    
    def calculate_baseline_demand(self) -> pl.DataFrame:
        """
        Calculate baseline demand (maintain 2019 densities).
        
        Formula: demand = population × density_2019 / 10,000
        
        Returns:
            DataFrame with baseline demand projections
        """
        logger.info("Calculating baseline demand (2019 density)")
        
        # Get 2019 density for each profession
        density_2019 = (
            self.density
            .filter(pl.col('year') == 2019)
            .select(['profession', 'sector', 'density'])
            .rename({'density': 'density_2019'})
        )
        
        # Calculate demand for each future year
        demand_rows = []
        for year in range(2020, 2031):
            pop_year = self.population.filter(pl.col('year') == year)
            
            for row in density_2019.iter_rows(named=True):
                demand = (
                    pop_year['total_population'][0] * 
                    row['density_2019'] / 10000
                )
                demand_rows.append({
                    'profession': row['profession'],
                    'sector': row['sector'],
                    'year': year,
                    'demand_baseline': demand
                })
        
        df_demand = pl.DataFrame(demand_rows)
        logger.info(f"✓ Calculated baseline demand for {len(df_demand)} records")
        return df_demand
    
    def calculate_oecd_demand(self) -> pl.DataFrame:
        """
        Calculate OECD target demand (use OECD benchmark densities).
        
        Returns:
            DataFrame with OECD-standard demand projections
        """
        logger.info("Calculating OECD target demand")
        
        demand_rows = []
        for year in range(2020, 2031):
            pop_year = self.population.filter(pl.col('year') == year)
            
            for profession, benchmark_density in self.benchmarks['oecd_densities'].items():
                for sector in ['public', 'private', 'total']:
                    demand = pop_year['total_population'][0] * benchmark_density / 10000
                    demand_rows.append({
                        'profession': profession,
                        'sector': sector,
                        'year': year,
                        'demand_oecd': demand
                    })
        
        df_demand_oecd = pl.DataFrame(demand_rows)
        logger.info(f"✓ Calculated OECD demand for {len(df_demand_oecd)} records")
        return df_demand_oecd
    
    def calculate_gaps(
        self,
        demand_baseline: pl.DataFrame,
        demand_oecd: pl.DataFrame
    ) -> pl.DataFrame:
        """
        Calculate supply-demand gaps.
        
        Args:
            demand_baseline: Baseline demand projections
            demand_oecd: OECD target demand projections
            
        Returns:
            DataFrame with gap analysis results
        """
        logger.info("Calculating supply-demand gaps")
        
        # Join forecasts with demand projections
        df_gap = (
            self.forecasts
            .select(['profession', 'sector', 'year', 'forecast'])
            .join(demand_baseline, on=['profession', 'sector', 'year'], how='left')
            .join(demand_oecd, on=['profession', 'sector', 'year'], how='left')
        )
        
        # Calculate gaps
        df_gap = df_gap.with_columns([
            # Absolute gap
            (pl.col('forecast') - pl.col('demand_baseline')).alias('gap_baseline'),
            (pl.col('forecast') - pl.col('demand_oecd')).alias('gap_oecd'),
            
            # Percentage gap
            ((pl.col('forecast') - pl.col('demand_baseline')) / 
             pl.col('demand_baseline') * 100).alias('gap_pct_baseline'),
            ((pl.col('forecast') - pl.col('demand_oecd')) / 
             pl.col('demand_oecd') * 100).alias('gap_pct_oecd')
        ])
        
        logger.info("✓ Calculated gaps")
        return df_gap
    
    def classify_severity(self, df_gap: pl.DataFrame) -> pl.DataFrame:
        """
        Classify gap severity into categories.
        
        Thresholds:
        - Critical: gap < -20%
        - Moderate: gap -20% to -10%
        - Manageable: gap -10% to 0%
        - Surplus: gap > 0%
        
        Args:
            df_gap: DataFrame with calculated gaps
            
        Returns:
            DataFrame with severity classifications
        """
        logger.info("Classifying gap severity")
        
        df_classified = df_gap.with_columns([
            pl.when(pl.col('gap_pct_baseline') < -20).then(pl.lit('critical'))
              .when(pl.col('gap_pct_baseline') < -10).then(pl.lit('moderate'))
              .when(pl.col('gap_pct_baseline') < 0).then(pl.lit('manageable'))
              .otherwise(pl.lit('surplus'))
              .alias('severity_category')
        ])
        
        # Count by severity
        severity_counts = (
            df_classified
            .groupby('severity_category')
            .agg(pl.count().alias('count'))
        )
        logger.info(f"Severity distribution:\n{severity_counts}")
        
        return df_classified
    
    def calculate_priority_scores(self, df_gap: pl.DataFrame) -> pl.DataFrame:
        """
        Calculate priority scores for professions.
        
        Score = 0.5 × gap_magnitude + 0.3 × gap_severity + 0.2 × urgency
        
        Args:
            df_gap: DataFrame with gaps and severity
            
        Returns:
            DataFrame with priority scores
        """
        logger.info("Calculating priority scores")
        
        # Normalize components to 0-100 scale
        df_priority = df_gap.with_columns([
            # Gap magnitude (absolute shortage)
            (pl.col('gap_baseline').abs() / 
             pl.col('gap_baseline').abs().max() * 100)
            .alias('gap_magnitude_normalized'),
            
            # Gap severity (% shortfall)
            (pl.col('gap_pct_baseline').abs() / 100 * 100)
            .alias('gap_severity_normalized'),
            
            # Urgency (earlier years = higher urgency)
            ((2030 - pl.col('year')) / 10 * 100)
            .alias('urgency_normalized')
        ])
        
        # Calculate composite score
        df_priority = df_priority.with_columns([
            (
                0.5 * pl.col('gap_magnitude_normalized') +
                0.3 * pl.col('gap_severity_normalized') +
                0.2 * pl.col('urgency_normalized')
            ).alias('priority_score')
        ])
        
        logger.info("✓ Calculated priority scores")
        return df_priority
    
    def identify_critical_professions(
        self,
        df_priority: pl.DataFrame,
        threshold_year: int = 2025
    ) -> pl.DataFrame:
        """
        Identify professions with critical shortages by 2025.
        
        Args:
            df_priority: DataFrame with priority scores
            threshold_year: Year to check for critical gaps
            
        Returns:
            DataFrame with critical professions ranked by priority
        """
        logger.info(f"Identifying critical professions by {threshold_year}")
        
        critical_professions = (
            df_priority
            .filter(
                (pl.col('year') == threshold_year) &
                (pl.col('severity_category') == 'critical')
            )
            .sort('priority_score', descending=True)
        )
        
        logger.warning(
            f"⚠ {len(critical_professions)} critical shortages identified by {threshold_year}"
        )
        
        return critical_professions
    
    def run_full_analysis(self) -> pl.DataFrame:
        """
        Execute complete gap analysis pipeline.
        
        Returns:
            DataFrame with full gap analysis results
        """
        logger.info("=" * 60)
        logger.info("Running Full Demand-Supply Gap Analysis")
        logger.info("=" * 60)
        
        # Calculate demand scenarios
        demand_baseline = self.calculate_baseline_demand()
        demand_oecd = self.calculate_oecd_demand()
        
        # Calculate gaps
        df_gap = self.calculate_gaps(demand_baseline, demand_oecd)
        
        # Classify severity
        df_gap = self.classify_severity(df_gap)
        
        # Calculate priorities
        df_gap = self.calculate_priority_scores(df_gap)
        
        logger.success("✓ Gap analysis complete")
        return df_gap
```

### 4. Gap Analysis Configuration

**[CREATE] `shared/config/gap_analysis_config.yml`**

```yaml
# Demand-Supply Gap Analysis Configuration
# =========================================

severity_thresholds:
  critical: -20  # Gap < -20%
  moderate: -10  # Gap -20% to -10%
  manageable: 0  # Gap -10% to 0%

priority_weights:
  gap_magnitude: 0.5  # Absolute shortage size
  gap_severity: 0.3   # Percentage shortfall
  urgency: 0.2        # Timeline to shortage

demand_scenarios:
  baseline:
    description: "Maintain 2019 workforce densities"
    density_year: 2019
  
  oecd_target:
    description: "Converge to OECD average densities by 2030"
    use_benchmarks: true
  
  sensitivity_high:
    description: "Baseline + 10% demand increase"
    multiplier: 1.10
  
  sensitivity_low:
    description: "Baseline - 10% demand decrease"
    multiplier: 0.90

reporting_years:
  - 2025  # 5-year horizon
  - 2030  # 10-year horizon
```

### 5. Gap Analysis Tests

```python
# shared/tests/unit/test_demand_gap_analyzer.py

import pytest
import polars as pl
from shared.src.analysis.demand_gap_analyzer import DemandGapAnalyzer


def test_baseline_demand_calculation():
    """Test baseline demand uses 2019 densities."""
    # Mock data
    forecast_df = pl.DataFrame()
    population_df = pl.DataFrame({
        'year': [2020, 2021],
        'total_population': [5600000, 5700000]
    })
    density_df = pl.DataFrame({
        'profession': ['doctors', 'doctors'],
        'sector': ['total', 'total'],
        'year': [2019, 2019],
        'density': [25.0, 25.0]  # 25 per 10k
    })
    
    analyzer = DemandGapAnalyzer(
        forecast_df, population_df, density_df,
        benchmark_config_path="shared/config/who_oecd_benchmarks.yml"
    )
    
    demand = analyzer.calculate_baseline_demand()
    
    # 2020: 5.6M × 25 / 10k = 14,000 doctors
    row_2020 = demand.filter(pl.col('year') == 2020)
    assert row_2020['demand_baseline'][0] == pytest.approx(14000, abs=10)


def test_severity_classification():
    """Test gap severity categories."""
    df_gap = pl.DataFrame({
        'profession': ['doctors', 'nurses', 'pharmacists'],
        'year': [2025, 2025, 2025],
        'gap_pct_baseline': [-25, -15, -5]
    })
    
    analyzer = DemandGapAnalyzer(
        pl.DataFrame(), pl.DataFrame(), pl.DataFrame(),
        benchmark_config_path="shared/config/who_oecd_benchmarks.yml"
    )
    
    df_classified = analyzer.classify_severity(df_gap)
    
    assert df_classified.filter(pl.col('profession') == 'doctors')['severity_category'][0] == 'critical'
    assert df_classified.filter(pl.col('profession') == 'nurses')['severity_category'][0] == 'moderate'
    assert df_classified.filter(pl.col('profession') == 'pharmacists')['severity_category'][0] == 'manageable'
```

### 6. Implementation Steps

**Phase 1: Demand Projections (Days 1-2)**
- [ ] Load population projections (from external sources or interpolate)
- [ ] Calculate baseline demand (2019 density × future population)
- [ ] Calculate OECD target demand (OECD density × future population)
- [ ] Validate demand calculations (spot-check 2025, 2030)

**Phase 2: Gap Calculation (Days 2-3)**
- [ ] Load workforce forecasts (US-06 output)
- [ ] Join forecasts with demand projections
- [ ] Calculate absolute and percentage gaps
- [ ] Classify severity (critical/moderate/manageable/surplus)

**Phase 3: Priority Ranking (Day 3)**
- [ ] Calculate priority scores (magnitude + severity + urgency)
- [ ] Identify critical professions by 2025
- [ ] Rank professions by priority
- [ ] Generate shortage timelines

**Phase 4: Output & Visualization (Days 3-4)**
- [ ] Export gap analysis: `shared/data/4_processed/workforce_gaps_2020_2030.parquet`
- [ ] Generate gap comparison charts (supply vs demand)
- [ ] Create priority ranking table for policymakers
- [ ] Document policy recommendations

### 7. Success Metrics

- ✅ Gaps calculated for all profession-sector-year combinations
- ✅ Critical professions identified by 2025
- ✅ Priority scores calculated and validated
- ✅ Policy recommendations generated for top 5 priority professions
- ✅ Gap analysis validated against known Singapore healthcare planning reports

### 8. References

- [Singapore Healthcare Manpower Plan 2020](https://www.moh.gov.sg/)
- [OECD Health Statistics](https://www.oecd.org/health/health-data.htm)
- US-04 (density benchmarks), US-06 (forecasts)

---

**Implementation Plan Complete for US-07**  
**Dependencies**: US-06 (forecasts), US-04 (density ratios)  
**Enables**: US-08 (scenario modeling to address gaps)
