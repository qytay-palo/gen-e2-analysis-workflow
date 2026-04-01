# Explore Workforce Trends & Growth Patterns (Lifecycle Stage: Exploratory Data Analysis)

**Story ID**: PS-001-US-03  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (5-6 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Public Health Analyst examining workforce sustainability**,  
I want **to analyze 14-year workforce growth patterns (2006-2019) across all healthcare professions and sectors to identify trends, variations, and notable changes**,  
So that **I can understand historical workforce dynamics and identify professions with concerning growth stagnation or rapid expansion**.

---

## 🎯 Acceptance Criteria

1. **Annual growth rates calculated**
   - Year-over-year growth rates calculated for each profession and sector
   - Compound annual growth rates (CAGR) computed for full 2006-2019 period
   - Growth rate statistics: mean, median, std dev, min/max per profession
   - Outlier years identified (>2 std dev from mean growth)

2. **Sector comparisons completed**
   - Public vs private sector growth analyzed for each profession
   - Sector share trends calculated (% of total workforce in public vs private)
   - Divergence analysis: professions where public/private growth differs significantly

3. **Trend analysis deliverables**
   - Trend charts: workforce count over time (faceted by profession)
   - Growth rate charts: YoY growth % by profession and sector
   - Summary statistics table: CAGR, median growth, volatility by profession
   - Inflection point detection: years where growth accelerated/decelerated

4. **Data output requirement**
   - Output file: `results/tables/workforce_growth_analysis.csv`
   - Format: CSV with columns (profession, sector, year, count, yoy_growth_pct, cagr)
   - Schema documented: Yes in data dictionary
   - Figures saved: `reports/figures/workforce_trends_by_profession.png` (and PDF for publication)

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+ (MANDATORY for data processing)
- **Visualization**: Matplotlib/Seaborn or Plotly for trend charts
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest with ≥80% coverage for trend calculation functions

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md#key-performance-indicators-kpis) - Understanding workforce density and benchmarks
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#objectives) - Objective 1: Quantify historical workforce growth patterns

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data processing and trend calculations
- `matplotlib>=3.8.0`: Publication-quality trend visualizations
- `seaborn>=0.13.0`: Statistical visualizations (optional)
- `plotly>=5.18.0`: Interactive charts (optional for dashboard preview)
- `loguru>=0.7.0`: Structured logging

### Internal Dependencies
- **Upstream**: PS-001-US-02 (Clean workforce data - BLOCKING)
- **Data Sources**: `shared/data/3_interim/workforce_integrated_clean.parquet`
- **Config Files**: `config/analysis.yml` (chart styling and color schemes)

---

## ✅ Implementation Tasks

### Exploratory Data Analysis
- [ ] Load cleaned workforce data using Polars
- [ ] Create annual summary: total workforce by profession and year
- [ ] Calculate year-over-year growth rates: `(count_t - count_t-1) / count_t-1 * 100`
- [ ] Calculate compound annual growth rates (CAGR): `(end_value / start_value)^(1/years) - 1`
- [ ] Compute growth statistics: mean, median, std dev per profession
- [ ] Identify outlier years (>2 std dev from mean growth)
- [ ] Detect inflection points using segmented regression (optional advanced feature)

### Sector Comparison Analysis
- [ ] Calculate sector share: % public vs private for each profession
- [ ] Compare public vs private growth rates (side-by-side comparison)
- [ ] Identify professions with divergent sector growth (e.g., private growing faster)
- [ ] Statistical test: t-test for significant public-private growth differences

### Visualization
- [ ] Create line charts: workforce count over time (one panel per profession)
- [ ] Create bar charts: CAGR by profession and sector
- [ ] Create heatmap: YoY growth rates (year × profession matrix)
- [ ] Create sector comparison: stacked area chart showing public/private composition
- [ ] Style charts for publication quality (figure size, labels, legends)
- [ ] Export figures: PNG (for reports) and PDF (for publication)

### Statistical Analysis
- [ ] Summary statistics table: CAGR, volatility, sector share per profession
- [ ] Correlation analysis: which professions grow together?
- [ ] Trend significance testing: is growth statistically significant?

### Testing & Validation
- [ ] Unit tests for growth rate calculations (test with known data)
- [ ] Validate CAGR formula: `assert abs(calculated_cagr - expected_cagr) < 0.01`
- [ ] Test edge cases: professions with missing years (nurses pre-2008)
- [ ] Validate output: all professions present, no missing years

### Documentation
- [ ] Docstrings for all analysis functions (Google style)
- [ ] Analysis report: `results/workforce_eda_insights.md` (key findings)
- [ ] README updated in `shared/src/analysis/` with usage examples
- [ ] Figures documented: captions and interpretation in report

---

## 📌 Notes

**Growth Rate Calculation (Polars)**:
```python
import polars as pl

df_growth = (
    df.sort(['profession', 'sector', 'year'])
    .with_columns([
        # Year-over-year growth
        ((pl.col('count') - pl.col('count').shift(1)) / pl.col('count').shift(1) * 100)
        .over(['profession', 'sector'])
        .alias('yoy_growth_pct')
    ])
)

# CAGR calculation
df_cagr = (
    df.group_by(['profession', 'sector']).agg([
        pl.col('count').first().alias('start_count'),
        pl.col('count').last().alias('end_count'),
        (pl.col('year').max() - pl.col('year').min()).alias('years')
    ])
    .with_columns([
        ((pl.col('end_count') / pl.col('start_count')) ** (1 / pl.col('years')) - 1) * 100
        .alias('cagr_pct')
    ])
)
```

**Expected Insights** (hypotheses to explore):
- Nurses likely have higher growth than doctors (shorter training pipeline)
- Private sector may grow faster than public (market expansion)
- Allied health professions may have higher volatility (newer categories)
- Post-2010 may show accelerated growth (aging population impact)

**Visualization Best Practices**:
- Use consistent color scheme per profession (define in config)
- Public = blue, Private = orange (standard sector colors)
- Include confidence intervals if calculating forecast preview
- Annotate notable events (e.g., policy changes, if known)

**Key Questions to Answer**:
1. Which profession has fastest growth? Slowest?
2. Is public or private sector growing faster?
3. Are there periods of stagnation or rapid acceleration?
4. Which professions have most stable vs volatile growth?

---

## Implementation Plan

### 1. Feature Overview

This user story implements exploratory data analysis (EDA) for healthcare workforce trends. The primary goal is to analyze 14-year growth patterns (2006-2019), calculate annual and compound growth rates, compare public vs private sector dynamics, and identify professions with concerning trends or rapid expansion.

**Primary User Role**: Public Health Analyst examining workforce sustainability

**Key Deliverable**: Comprehensive trend analysis with growth statistics, sector comparisons, and publication-quality visualizations documenting historical workforce dynamics.

### 2. Component Analysis & Reuse Strategy

**Existing Components Available for Reuse**:
- **✅ REUSE AS-IS**: Cleaned workforce data from US-02 (`workforce_integrated_clean.parquet`)
- **✅ REUSE AS-IS**: SchemaValidator for data quality checks
- **✅ REUSE AS-IS**: Logging infrastructure (loguru patterns from US-01)

**New Components Required**:
- **CREATE**: `shared/src/analysis/trend_analysis.py` - Growth rate calculations, trend detection
- **CREATE**: `shared/src/visualization/workforce_plots.py` - Workforce-specific plotting functions
- **CREATE**: `shared/config/visualization.yml` - Chart styling configuration
- **CREATE**: Unit tests for trend calculation logic

### 3. Affected Files

**[CREATE] `shared/src/analysis/trend_analysis.py`**
- Functions: `calculate_yoy_growth()`, `calculate_cagr()`, `detect_inflection_points()`, `identify_outlier_years()`
- Dependencies: `polars`, `scipy`, `numpy`, `loguru`

**[CREATE] `shared/src/visualization/workforce_plots.py`**
- Functions: `plot_workforce_trends()`, `plot_growth_rates()`, `plot_sector_comparison()`, `plot_growth_heatmap()`
- Dependencies: `matplotlib`, `seaborn`, `polars`

**[CREATE] `shared/config/visualization.yml`**
- Color schemes, font sizes, figure dimensions, publication settings

**[CREATE] `results/tables/workforce_growth_analysis.csv`**
- Output: Profession, year, count, yoy_growth_pct, cagr

**[CREATE] `reports/figures/workforce_trends_*.png`**
- Publication-quality trend visualizations

### 4. Data Pipeline

**Input**: `shared/data/3_interim/workforce_integrated_clean.parquet` (from US-02)

**Transformations**:
1. Aggregate by profession, sector, year
2. Calculate year-over-year growth rates
3. Calculate compound annual growth rates (2006-2019)
4. Compute growth statistics (mean, median, std dev, min, max)
5. Identify outlier years (>2 std dev from mean)
6. Compare public vs private sector growth
7. Detect inflection points (growth acceleration/deceleration)

**Outputs**:
- `results/tables/workforce_growth_analysis.csv` - Growth metrics
- `reports/figures/workforce_trends_by_profession.png` - Line charts
- `reports/figures/workforce_growth_rates_heatmap.png` - YoY growth heatmap
- `reports/figures/sector_comparison.png` - Public vs private analysis

### 5. Key Statistical Methods

**Year-over-Year Growth Rate**:
```python
def calculate_yoy_growth(df: pl.DataFrame) -> pl.DataFrame:
    """Calculate year-over-year growth percentage."""
    return df.with_columns([
        ((pl.col('count') - pl.col('count').shift(1)) / pl.col('count').shift(1) * 100)
        .over(['profession', 'sector'])
        .alias('yoy_growth_pct')
    ])
```

**Compound Annual Growth Rate (CAGR)**:
```python
def calculate_cagr(
    start_value: float,
    end_value: float,
    years: int
) -> float:
    """Calculate CAGR: (end_value / start_value)^(1/years) - 1"""
    return (end_value / start_value) ** (1 / years) - 1
```

**Outlier Detection**:
```python
def identify_outlier_years(df: pl.DataFrame, threshold: float = 2.0) -> pl.DataFrame:
    """Identify years with growth >threshold std devs from mean."""
    return df.with_columns([
        (pl.col('yoy_growth_pct') - pl.col('yoy_growth_pct').mean().over('profession'))
        / pl.col('yoy_growth_pct').std().over('profession')
        .alias('growth_zscore')
    ]).filter(pl.col('growth_zscore').abs() > threshold)
```

### 6. Visualization Specifications

**Chart 1: Workforce Trends Over Time**
- Type: Multi-panel line chart (one panel per profession)
- X-axis: Year (2006-2019)
- Y-axis: Workforce count
- Lines: Public (blue), Private (orange), Total (gray)
- Size: 12" x 8"
- Save as: `workforce_trends_by_profession.png` (PNG + PDF)

**Chart 2: Growth Rate Heatmap**
- Type: Heatmap (year × profession matrix)
- Values: Year-over-year growth percentage
- Colormap: RdYlGn (red=negative, yellow=neutral, green=positive)
- Annotations: Show percentage values in cells
- Save as: `workforce_growth_rates_heatmap.png`

**Chart 3: CAGR Bar Chart**
- Type: Grouped bar chart
- X-axis: Profession
- Y-axis: CAGR (%)
- Groups: Public sector, Private sector
- Error bars: None (single CAGR value per profession-sector)
- Save as: `workforce_cagr_by_profession.png`

**Chart 4: Sector Share Evolution**
- Type: Stacked area chart
- X-axis: Year
- Y-axis: Percentage of total workforce
- Areas: Public (bottom), Private (top)
- Facets: One panel per profession
- Save as: `workforce_sector_share_evolution.png`

### 7. Code Implementation

```python
"""
Workforce Trend Analysis
=========================

Calculate growth rates and analyze workforce trends.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
from typing import Tuple, Dict
from loguru import logger
import yaml


class WorkforceTrendAnalyzer:
    """
    Analyze workforce growth trends and patterns.
    
    Example:
        >>> analyzer = WorkforceTrendAnalyzer()
        >>> growth_df = analyzer.calculate_growth_metrics()
        >>> analyzer.generate_all_visualizations()
    """
    
    def __init__(
        self,
        data_path: str = "shared/data/3_interim/workforce_integrated_clean.parquet",
        config_path: str = "shared/config/visualization.yml"
    ):
        """Initialize trend analyzer."""
        self.data_path = Path(data_path)
        self.config_path = Path(config_path)
        
        # Load data
        self.df = pl.read_parquet(data_path)
        logger.info(f"Loaded {len(self.df)} workforce records")
        
        # Load visualization config
        self.viz_config = self._load_viz_config()
    
    def _load_viz_config(self) -> dict:
        """Load visualization configuration."""
        if self.config_path.exists():
            with open(self.config_path, 'r') as f:
                return yaml.safe_load(f)
        return self._default_viz_config()
    
    def _default_viz_config(self) -> dict:
        """Default visualization settings."""
        return {
            'colors': {
                'public': '#1f77b4',  # Blue
                'private': '#ff7f0e',  # Orange
                'total': '#7f7f7f'    # Gray
            },
            'figure_size': (12, 8),
            'dpi': 300,
            'font_size': 10
        }
    
    def calculate_yoy_growth(self) -> pl.DataFrame:
        """Calculate year-over-year growth rates."""
        logger.info("Calculating year-over-year growth rates...")
        
        df_growth = (
            self.df
            .sort(['profession', 'sector', 'year'])
            .with_columns([
                ((pl.col('count') - pl.col('count').shift(1)) / 
                 pl.col('count').shift(1) * 100)
                .over(['profession', 'sector'])
                .alias('yoy_growth_pct')
            ])
        )
        
        logger.info(f"✓ Calculated YoY growth for {df_growth.select('profession').n_unique()} professions")
        return df_growth
    
    def calculate_cagr(self) -> pl.DataFrame:
        """Calculate compound annual growth rates (2006-2019)."""
        logger.info("Calculating CAGR (2006-2019)...")
        
        # Get start and end values per profession-sector
        start_values = (
            self.df
            .filter(pl.col('year') == 2006)
            .select(['profession', 'sector', 'count'])
            .rename({'count': 'count_2006'})
        )
        
        end_values = (
            self.df
            .filter(pl.col('year') == 2019)
            .select(['profession', 'sector', 'count'])
            .rename({'count': 'count_2019'})
        )
        
        cagr_df = (
            start_values
            .join(end_values, on=['profession', 'sector'], how='inner')
            .with_columns([
                ((pl.col('count_2019') / pl.col('count_2006')) ** (1 / 13) - 1)
                .alias('cagr')
            ])
            .select(['profession', 'sector', 'cagr'])
        )
        
        logger.info(f"✓ Calculated CAGR for {len(cagr_df)} profession-sector combinations")
        return cagr_df
    
    def compute_growth_statistics(self, df_growth: pl.DataFrame) -> pl.DataFrame:
        """Compute summary statistics for growth rates."""
        logger.info("Computing growth statistics...")
        
        stats_df = (
            df_growth
            .filter(pl.col('yoy_growth_pct').is_not_null())
            .groupby(['profession', 'sector'])
            .agg([
                pl.col('yoy_growth_pct').mean().alias('mean_growth'),
                pl.col('yoy_growth_pct').median().alias('median_growth'),
                pl.col('yoy_growth_pct').std().alias('std_growth'),
                pl.col('yoy_growth_pct').min().alias('min_growth'),
                pl.col('yoy_growth_pct').max().alias('max_growth')
            ])
        )
        
        logger.info("✓ Growth statistics computed")
        return stats_df
    
    def identify_outlier_years(self, df_growth: pl.DataFrame, threshold: float = 2.0) -> pl.DataFrame:
        """Identify years with outlier growth rates."""
        logger.info(f"Identifying outlier years (threshold={threshold} std dev)...")
        
        outliers = (
            df_growth
            .filter(pl.col('yoy_growth_pct').is_not_null())
            .with_columns([
                ((pl.col('yoy_growth_pct') - pl.col('yoy_growth_pct').mean().over('profession')) /
                 pl.col('yoy_growth_pct').std().over('profession'))
                .alias('growth_zscore')
            ])
            .filter(pl.col('growth_zscore').abs() > threshold)
            .sort(['profession', 'year'])
        )
        
        logger.info(f"✓ Found {len(outliers)} outlier year-profession combinations")
        return outliers
    
    def plot_workforce_trends(self, save_path: str = "reports/figures/workforce_trends_by_profession.png"):
        """Generate multi-panel trend charts."""
        logger.info("Generating workforce trend visualizations...")
        
        professions = self.df.select('profession').unique().to_series().to_list()
        n_professions = len(professions)
        
        fig, axes = plt.subplots(
            nrows=(n_professions + 1) // 2,
            ncols=2,
            figsize=self.viz_config['figure_size'],
            sharex=True
        )
        axes = axes.flatten()
        
        for idx, profession in enumerate(professions):
            ax = axes[idx]
            
            # Filter data for this profession
            prof_data = self.df.filter(pl.col('profession') == profession)
            
            # Plot each sector
            for sector in ['public', 'private', 'total']:
                sector_data = prof_data.filter(pl.col('sector') == sector).sort('year')
                
                if len(sector_data) > 0:
                    ax.plot(
                        sector_data['year'].to_list(),
                        sector_data['count'].to_list(),
                        label=sector.capitalize(),
                        color=self.viz_config['colors'].get(sector, 'gray'),
                        marker='o',
                        linewidth=2
                    )
            
            ax.set_title(f"{profession.replace('_', ' ').title()}", fontsize=12)
            ax.set_ylabel('Workforce Count')
            ax.legend(loc='best')
            ax.grid(True, alpha=0.3)
        
        # Hide unused subplots
        for idx in range(n_professions, len(axes)):
            axes[idx].set_visible(False)
        
        plt.xlabel('Year', fontsize=12)
        plt.tight_layout()
        
        # Save figure
        save_path = Path(save_path)
        save_path.parent.mkdir(parents=True, exist_ok=True)
        plt.savefig(save_path, dpi=self.viz_config['dpi'], bbox_inches='tight')
        plt.savefig(save_path.with_suffix('.pdf'), bbox_inches='tight')  # Also save PDF
        
        logger.success(f"✓ Saved trend chart: {save_path}")
        plt.close()
    
    def generate_all_visualizations(self):
        """Generate all trend analysis visualizations."""
        logger.info("=" * 60)
        logger.info("Generating all workforce trend visualizations")
        logger.info("=" * 60)
        
        self.plot_workforce_trends()
        # Add other plot methods here
        
        logger.success("✓ All visualizations generated")
```

### 8. Testing Strategy

```python
# shared/tests/unit/test_trend_analysis.py

def test_calculate_yoy_growth():
    """Test YoY growth calculation."""
    df = pl.DataFrame({
        'profession': ['doctors', 'doctors', 'doctors'],
        'sector': ['public', 'public', 'public'],
        'year': [2006, 2007, 2008],
        'count': [1000, 1100, 1210]
    })
    
    analyzer = WorkforceTrendAnalyzer()
    df_growth = analyzer.calculate_yoy_growth()
    
    # Expected: 10% growth 2006->2007, 10% growth 2007->2008
    assert df_growth.filter(pl.col('year') == 2007)['yoy_growth_pct'][0] == pytest.approx(10.0, rel=0.01)
    assert df_growth.filter(pl.col('year') == 2008)['yoy_growth_pct'][0] == pytest.approx(10.0, rel=0.01)


def test_calculate_cagr():
    """Test CAGR calculation."""
    # CAGR = (1210 / 1000)^(1/2) - 1 ≈ 10%
    start_value = 1000
    end_value = 1210
    years = 2
    
    cagr = (end_value / start_value) ** (1 / years) - 1
    
    assert cagr == pytest.approx(0.10, abs=0.001)
```

### 9. Implementation Steps

**Phase 1: Analysis Setup (Day 1)**
- [ ] Create `shared/src/analysis/trend_analysis.py`
- [ ] Create `shared/src/visualization/workforce_plots.py`
- [ ] Create `shared/config/visualization.yml`
- [ ] Load cleaned workforce data
- [ ] Verify data quality (no nulls, expected professions)

**Phase 2: Growth Calculations (Days 1-2)**
- [ ] Implement `calculate_yoy_growth()`
- [ ] Implement `calculate_cagr()`
- [ ] Implement `compute_growth_statistics()`
- [ ] Implement `identify_outlier_years()`
- [ ] Save growth metrics to `results/tables/workforce_growth_analysis.csv`

**Phase 3: Sector Analysis (Day 2)**
- [ ] Implement public vs private sector comparison
- [ ] Calculate sector share trends
- [ ] Identify sectors with divergent growth
- [ ] Statistical tests for sector differences (t-test)

**Phase 4: Visualization (Days 3-4)**
- [ ] Implement `plot_workforce_trends()` - Line charts
- [ ] Implement `plot_growth_heatmap()` - YoY growth matrix
- [ ] Implement `plot_cagr_bars()` - CAGR comparison
- [ ] Implement `plot_sector_comparison()` - Sector evolution
- [ ] Save all figures to `reports/figures/`

**Phase 5: Testing & Documentation (Days 4-5)**
- [ ] Unit tests for growth calculations
- [ ] Integration tests for full analysis pipeline
- [ ] Verify all visualizations generated
- [ ] Document key findings in analysis report
- [ ] Commit and create PR

### 10. Success Metrics

**Analytical Completeness**:
- ✅ YoY growth calculated for all profession-sector-year combinations
- ✅ CAGR calculated for 2006-2019 period
- ✅ Growth statistics computed (mean, median, std dev, min, max)
- ✅ Outlier years identified and documented

**Visualization Quality**:
- ✅ All charts publication-ready (300 DPI, proper labels)
- ✅ Consistent color scheme across all charts
- ✅ Both PNG and PDF versions saved

**Key Insights Documented**:
- Identify fastest and slowest growing professions
- Document public vs private growth differentials
- Highlight periods of stagnation or acceleration
- Quantify growth volatility per profession

### 11. References

- [Healthcare Workforce Metrics & KPIs](../../../domain_knowledge/healthcare-workforce-metrics-kpis.md) - Growth rate benchmarks
- [Problem Statement PS-001](../../problem_statements/ps-001-healthcare-workforce-sustainability.md) - Objective 1
- US-02 Implementation Plan - Cleaned data schema
- Matplotlib Documentation: https://matplotlib.org/
- Seaborn Statistical Visualizations: https://seaborn.pydata.org/

---

**Implementation Plan Complete for US-03**  
**Dependencies**: US-02 (cleaned data)  
**Enables**: US-04, US-05 (insights inform feature engineering)
