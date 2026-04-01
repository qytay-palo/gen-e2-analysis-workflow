# Benchmark Against International Expenditure Standards (Lifecycle Stage: Advanced Analysis)

**Story ID**: PS-004-US-07  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P1 (High)  
**Effort Estimate**: M (5-6 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Policy Maker evaluating Singapore's cost performance**,  
I want **to benchmark Singapore's healthcare expenditure against OECD and WHO standards including expenditure as % of GDP, per capita spending, and growth rates**,  
So that **I can assess whether Singapore is spending efficiently relative to peer countries and identify areas of over/under-spending**.

---

## 🎯 Acceptance Criteria

1. **External benchmark data acquired**
   - OECD Health Statistics: expenditure data for peer countries
   - WHO Global Health Expenditure Database: international comparisons
   - Key metrics: expenditure % GDP, per capita, public vs private share
   - Peer countries: Japan, South Korea, Australia, UK, USA, Germany

2. **Comparative analysis completed**
   - Singapore vs OECD average: expenditure gaps quantified
   - Singapore vs high-performing peers: best practice identification
   - Trend comparison: Singapore growth vs international growth
   - Public-private mix comparison

3. **Performance assessment**
   - Outperformance areas: where Singapore spends less for similar outcomes
   - Underperformance areas: where Singapore spends more
   - Efficiency indicators: expenditure per health outcome unit

4. **Deliverables**
   - Output: `results/tables/international_expenditure_benchmarking.csv`
   - Figures: Benchmarking charts, peer country comparisons
   - Report: International expenditure performance assessment

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **External Data**: OECD API, WHO database
- **Visualization**: Matplotlib/Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#international-expenditure-benchmarks) - OECD/WHO benchmarking methods
- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md#objective-3) - International benchmarking

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `requests>=2.31.0`, `matplotlib>=3.8.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-03 (Expenditure trends - BLOCKING)
- **Data Sources**: 
  - Internal: `shared/data/3_interim/expenditure_drivers_integrated.parquet`
  - External: OECD database, WHO GHED
- **Config Files**: `config/analysis.yml`

---

## ✅ Implementation Tasks

### External Data Acquisition
- [ ] Access OECD Health Statistics (API or download)
- [ ] Access WHO Global Health Expenditure Database
- [ ] Extract expenditure metrics for peer countries
- [ ] Save to `shared/data/2_external/oecd_who_expenditure.csv`

### Benchmark Calculations
- [ ] Calculate Singapore expenditure as % GDP
- [ ] Calculate per capita expenditure in USD PPP
- [ ] Compare with OECD averages and peer countries
- [ ] Calculate gaps and rankings

### Trend Comparison
- [ ] Compare Singapore growth rates with international trends
- [ ] Assess convergence or divergence
- [ ] Identify periods where Singapore diverged from peers

### Visualization
- [ ] Bar charts: Singapore vs peers (latest year)
- [ ] Time series: Singapore vs OECD average over time
- [ ] Scatter plot: expenditure % GDP vs health outcomes (if data available)
- [ ] Save figures

### Testing & Documentation
- [ ] Validate external data quality
- [ ] Unit tests for benchmarking calculations
- [ ] Docstrings
- [ ] Benchmarking report

---

## 📌 Notes

**Data Sources**:
- **OECD Health Statistics**: https://data-explorer.oecd.org/
- **WHO GHED**: https://apps.who.int/nha/database

**Key Metrics**:
- Expenditure as % GDP (typical range: 6-12% for high-income countries)
- Per capita expenditure in USD PPP (purchasing power parity)
- Public vs private expenditure share

**Expected Findings**:
- Singapore likely lower expenditure % GDP than US, similar to Asian peers
- Efficient spending: good outcomes for moderate expenditure

---

## Implementation Plan

### 1. Feature Overview

Acquire OECD/WHO international health expenditure data for peer countries, compute key benchmark metrics (% GDP, per-capita USD PPP), compare Singapore against peers and OECD average, and produce gap analysis charts. Primary role: **Healthcare Financial Policy Maker**.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | US-02 | **Reuse** Singapore base |
| `shared/data/2_external/` | Exists (dir) | **Create** WHO/OECD CSV here |
| `ps-004/src/data_processing/benchmark_extractor.py` | Missing | **Create** |
| `ps-004/src/analysis/international_benchmarking.py` | Missing | **Create** |
| `results/tables/problem-statement-004/international_expenditure_benchmarking.csv` | Missing | **Create** |

**Note**: OECD API requires registration. WHO GHED provides open CSV downloads. Plan uses WHO GHED CSV as primary; OECD as optional enhancement.

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/data_processing/benchmark_extractor.py`**
  - `BenchmarkExtractor` class
  - `download_who_ghed()` → `pl.DataFrame`
  - `load_static_fallback()` → `pl.DataFrame`
  - `save_external()` → `Path`

- **[CREATE] `ps-004/src/analysis/international_benchmarking.py`**
  - `InternationalBenchmarker` class
  - `prepare_singapore_metrics()` → `pl.DataFrame`
  - `compute_gaps()` → `pl.DataFrame`
  - `rank_peers()` → `pl.DataFrame`
  - `save_results()` → `None`

- **[CREATE] `shared/data/2_external/who_ghed_peer_countries.csv`** (static fallback)

- **[CREATE] `ps-004/tests/unit/test_international_benchmarking.py`**

- **[CREATE] `ps-004/notebooks/07-international-expenditure-benchmarking.ipynb`**

---

### 5. Data Pipeline

```
External: WHO GHED (https://apps.who.int/nha/database)
  → Download/load CSV with columns: country, year, che_gdp (% GDP), che_cap_usd (per capita USD)
  → Filter to peer countries: SGP, JPN, KOR, AUS, GBR, USA, DEU
  → Filter to 2006-2018 overlap
  → Save to shared/data/2_external/who_ghed_peer_countries.csv

Singapore internal:
  shared/data/3_interim/expenditure_drivers_integrated.parquet
  → Calculate SGD → USD conversion (use static PPP rate ~0.74 for approximate)
  → Calculate % GDP (if GDP data available from external source)

Join: peer + Singapore
  → compute_gaps(): Singapore vs OECD average, vs each peer
  → rank_peers(): percentile rank of Singapore

  → SAVE: results/tables/problem-statement-004/international_expenditure_benchmarking_*.csv
  → SAVE: reports/figures/problem-statement-004/expenditure_benchmark_bar_*.png
  → SAVE: reports/figures/problem-statement-004/expenditure_benchmark_timeseries_*.png
```

---

### 6. Code Generation Specifications

#### 6.1 Complete Implementation — `benchmark_extractor.py`

```python
"""
International Benchmark Data Extractor
=======================================

Downloads or loads WHO Global Health Expenditure Database (GHED) data
for Singapore and peer countries (Japan, South Korea, Australia, UK, USA, Germany).

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from pathlib import Path

import polars as pl
import requests

logger = logging.getLogger(__name__)

PEER_COUNTRIES = ["Singapore", "Japan", "Korea", "Australia",
                   "United Kingdom", "United States", "Germany"]
PEER_ISO3 = ["SGP", "JPN", "KOR", "AUS", "GBR", "USA", "DEU"]

WHO_GHED_URL = "https://apps.who.int/nha/database/Select/Indicators/en"

# Static fallback: approximate WHO GHED values (2018, per WHO Health Statistics 2022)
STATIC_BENCHMARK_DATA = {
    "country": ["Singapore", "Japan", "South Korea", "Australia",
                "United Kingdom", "United States", "Germany"],
    "iso3": ["SGP", "JPN", "KOR", "AUS", "GBR", "USA", "DEU"],
    "year": [2018] * 7,
    "che_gdp_pct": [4.5, 10.9, 7.6, 9.3, 10.0, 16.9, 11.7],          # % of GDP
    "che_per_capita_usd_ppp": [2527, 4267, 2497, 5187, 4070, 10623, 5468],  # USD PPP
    "source": ["WHO_GHED_static_2022"] * 7,
}


class BenchmarkExtractor:
    """
    Extract and cache international health expenditure benchmark data.

    Example:
        >>> extractor = BenchmarkExtractor()
        >>> df = extractor.run()
    """

    def __init__(
        self,
        output_path: str = "shared/data/2_external/who_ghed_peer_countries.csv",
    ) -> None:
        self.output_path = Path(output_path)
        self.output_path.parent.mkdir(parents=True, exist_ok=True)

    def load_static_fallback(self) -> pl.DataFrame:
        """
        Return hardcoded WHO GHED snapshot (2018) as fallback.

        Returns:
            DataFrame with peer country benchmark metrics.
        """
        df = pl.DataFrame(STATIC_BENCHMARK_DATA)
        logger.info(
            f"Loaded static benchmark fallback: {len(df)} countries "
            "(WHO GHED 2022 snapshot)"
        )
        return df

    def try_download_who_ghed(
        self, timeout: int = 10
    ) -> pl.DataFrame | None:
        """
        Attempt to download WHO GHED CSV export.
        Returns None if download fails (network unavailable, API changed).

        Args:
            timeout: Request timeout in seconds.

        Returns:
            DataFrame if successful, None otherwise.
        """
        try:
            resp = requests.get(WHO_GHED_URL, timeout=timeout)
            resp.raise_for_status()
            # Note: WHO GHED requires form submission; direct download requires
            # manual setup. This is a placeholder for organisations with API access.
            logger.info("WHO GHED live download attempted (requires API setup)")
            return None
        except Exception as exc:
            logger.warning(f"WHO GHED download unavailable: {exc} — using static fallback")
            return None

    def save_external(self, df: pl.DataFrame) -> Path:
        """
        Persist benchmark data to shared/data/2_external/.

        Args:
            df: Benchmark DataFrame to save.

        Returns:
            Path to saved CSV.
        """
        df.write_csv(str(self.output_path))
        logger.info(f"✓ Benchmark data saved: {self.output_path}")
        print(f"✓ External benchmark saved: {self.output_path}")
        return self.output_path

    def run(self) -> pl.DataFrame:
        """Try live download; fall back to static; save to external directory."""
        if self.output_path.exists():
            logger.info(f"Loading cached benchmark from {self.output_path}")
            return pl.read_csv(str(self.output_path))

        df = self.try_download_who_ghed()
        if df is None:
            df = self.load_static_fallback()
        self.save_external(df)
        return df
```

#### 6.1b Complete Implementation — `international_benchmarking.py`

```python
"""
International Expenditure Benchmarking
=========================================

Compares Singapore government health expenditure against OECD/WHO peer
countries: Japan, South Korea, Australia, UK, USA, Germany.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from datetime import datetime
from pathlib import Path

import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import polars as pl

logger = logging.getLogger(__name__)

# Approximate SGD to USD PPP conversion (2018 IMF estimate)
SGD_USD_PPP_RATE = 0.74
SINGAPORE_POPULATION_2018 = 5_638_700  # Approximate


class InternationalBenchmarker:
    """
    Compare Singapore expenditure against international peers.

    Example:
        >>> benchmarker = InternationalBenchmarker(df_singapore, df_peers)
        >>> df_gaps = benchmarker.run()
    """

    def __init__(
        self,
        df_singapore: pl.DataFrame,
        df_peers: pl.DataFrame,
        results_dir: str = "results/tables/problem-statement-004",
        figures_dir: str = "reports/figures/problem-statement-004",
    ) -> None:
        """
        Args:
            df_singapore: Singapore integrated DataFrame (year, total_expenditure).
            df_peers: Peer benchmark DataFrame with che_gdp_pct, che_per_capita_usd_ppp.
            results_dir: Output directory for tables.
            figures_dir: Output directory for figures.
        """
        self.df_sg = df_singapore.sort("year")
        self.df_peers = df_peers
        self.results_dir = Path(results_dir)
        self.figures_dir = Path(figures_dir)
        self.results_dir.mkdir(parents=True, exist_ok=True)
        self.figures_dir.mkdir(parents=True, exist_ok=True)

    def prepare_singapore_metrics(self, benchmark_year: int = 2018) -> dict:
        """
        Calculate Singapore per-capita (USD PPP) from internal expenditure data.

        Args:
            benchmark_year: Year to use for comparison.

        Returns:
            Dict with Singapore che_per_capita_usd_ppp and
            total_expenditure_sgd_m for the benchmark year.
        """
        sg_row = self.df_sg.filter(pl.col("year") == benchmark_year)
        if len(sg_row) == 0:
            # Use latest available year
            sg_row = self.df_sg.tail(1)
        exp_sgd_m = sg_row["total_expenditure"][0]
        exp_usd_per_capita = (exp_sgd_m * 1e6 * SGD_USD_PPP_RATE) / SINGAPORE_POPULATION_2018

        result = {
            "country": "Singapore (internal)",
            "year": int(sg_row["year"][0]),
            "total_expenditure_sgd_m": round(exp_sgd_m, 1),
            "che_per_capita_usd_ppp": round(exp_usd_per_capita, 0),
        }
        logger.info("Singapore metrics: %s", result)
        return result

    def compute_gaps(
        self, sg_metrics: dict, benchmark_year: int = 2018
    ) -> pl.DataFrame:
        """
        Compute per-capita gap between Singapore and each peer country.

        Args:
            sg_metrics: Dict from prepare_singapore_metrics().
            benchmark_year: Year to filter peer data.

        Returns:
            DataFrame with gap analysis per country.
        """
        df_year = self.df_peers.filter(pl.col("year") == benchmark_year)
        if len(df_year) == 0:
            df_year = self.df_peers  # Use all if year filter returns empty

        sg_cap = sg_metrics["che_per_capita_usd_ppp"]
        oecd_avg = df_year["che_per_capita_usd_ppp"].mean()

        df_gaps = df_year.with_columns([
            pl.lit(sg_cap).alias("sg_per_capita_usd_ppp"),
            (pl.col("che_per_capita_usd_ppp") - sg_cap).alias("gap_vs_singapore_usd"),
            pl.lit(oecd_avg).alias("peer_avg_per_capita_usd_ppp"),
        ])

        logger.info(
            f"Peer avg: ${oecd_avg:,.0f} USD PPP vs Singapore: ${sg_cap:,.0f}"
        )
        return df_gaps.sort("che_per_capita_usd_ppp", descending=True)

    def plot_benchmark_bar(
        self, df_gaps: pl.DataFrame, output_path: Path
    ) -> None:
        """
        Horizontal bar chart: per-capita expenditure for all countries.

        Args:
            df_gaps: Gap analysis DataFrame.
            output_path: Path to save PNG.
        """
        countries = df_gaps["country"].to_list()
        values = df_gaps["che_per_capita_usd_ppp"].to_list()
        sg_cap = df_gaps["sg_per_capita_usd_ppp"][0]

        colors = [
            "#e74c3c" if c == "Singapore" else "#3498db" for c in countries
        ]

        fig, ax = plt.subplots(figsize=(10, 5))
        bars = ax.barh(countries, values, color=colors, edgecolor="white")
        ax.axvline(sg_cap, color="red", linestyle="--", linewidth=1.2,
                   label=f"Singapore: ${sg_cap:,.0f}")
        ax.set_title(
            "Per-Capita Health Expenditure (USD PPP) — International Comparison",
            fontsize=12, fontweight="bold", pad=10
        )
        ax.set_xlabel("Per-Capita Expenditure (USD PPP)")
        ax.xaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f"${x:,.0f}"))
        ax.legend(fontsize=9)
        ax.grid(axis="x", alpha=0.3)
        plt.tight_layout()
        output_path.parent.mkdir(parents=True, exist_ok=True)
        fig.savefig(str(output_path), dpi=150, bbox_inches="tight")
        plt.close(fig)
        logger.info(f"✓ Benchmark bar chart saved: {output_path}")
        print(f"✓ Figure saved: {output_path}")

    def save_results(self, df_gaps: pl.DataFrame) -> None:
        """Persist gap analysis CSV with timestamp."""
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        out_path = self.results_dir / f"international_expenditure_benchmarking_{ts}.csv"
        df_gaps.write_csv(str(out_path))
        logger.info(f"✓ Benchmarking saved: {out_path}")
        print(f"✓ Results saved: {out_path}")

    def run(self, benchmark_year: int = 2018) -> pl.DataFrame:
        """Full pipeline: Singapore metrics → gap analysis → chart → save."""
        sg_metrics = self.prepare_singapore_metrics(benchmark_year)
        df_gaps = self.compute_gaps(sg_metrics, benchmark_year)
        self.plot_benchmark_bar(
            df_gaps,
            self.figures_dir / f"expenditure_benchmark_bar_{datetime.now().strftime('%Y%m%d_%H%M%S')}.png"
        )
        self.save_results(df_gaps)
        logger.info("International benchmarking complete")
        return df_gaps
```

#### 6.3 Static Fallback Data Schema

```yaml
# shared/data/2_external/README.md addition
who_ghed_peer_countries.csv:
  source: "WHO Global Health Expenditure Database (static 2018 snapshot)"
  url: "https://apps.who.int/nha/database"
  date_retrieved: "2026-03-25"
  columns:
    country: "Country name"
    iso3: "ISO 3166-1 alpha-3 code"
    year: "Reference year"
    che_gdp_pct: "Current health expenditure as % of GDP"
    che_per_capita_usd_ppp: "Per capita expenditure USD PPP"
    source: "Data source reference"
```

---

### 10. Testing Strategy

```python
# ps-004/tests/unit/test_international_benchmarking.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.data_processing.benchmark_extractor import (
    BenchmarkExtractor,
    STATIC_BENCHMARK_DATA,
)
from problem_statements.ps_004_healthcare_expenditure_drivers.src.analysis.international_benchmarking import (
    InternationalBenchmarker,
)


@pytest.fixture
def df_peers() -> pl.DataFrame:
    return pl.DataFrame(STATIC_BENCHMARK_DATA)


@pytest.fixture
def df_singapore() -> pl.DataFrame:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 250 for i in range(13)],
    })


@pytest.fixture
def benchmarker(
    tmp_path: Path, df_singapore: pl.DataFrame, df_peers: pl.DataFrame
) -> InternationalBenchmarker:
    return InternationalBenchmarker(
        df_singapore=df_singapore,
        df_peers=df_peers,
        results_dir=str(tmp_path / "results"),
        figures_dir=str(tmp_path / "figures"),
    )


def test_static_fallback_seven_countries() -> None:
    extractor = BenchmarkExtractor.__new__(BenchmarkExtractor)
    extractor.output_path = Path("/tmp/test_benchmark.csv")
    df = extractor.load_static_fallback()
    assert len(df) == 7
    assert "SGP" in df["iso3"].to_list()


def test_prepare_singapore_metrics_positive(
    benchmarker: InternationalBenchmarker
) -> None:
    sg = benchmarker.prepare_singapore_metrics(2018)
    assert sg["che_per_capita_usd_ppp"] > 0
    assert sg["year"] == 2018


def test_compute_gaps_contains_gap_column(
    benchmarker: InternationalBenchmarker
) -> None:
    sg = benchmarker.prepare_singapore_metrics(2018)
    df_gaps = benchmarker.compute_gaps(sg, benchmark_year=2018)
    assert "gap_vs_singapore_usd" in df_gaps.columns
    assert len(df_gaps) == 7


def test_save_creates_csv(
    benchmarker: InternationalBenchmarker,
    df_peers: pl.DataFrame
) -> None:
    sg = benchmarker.prepare_singapore_metrics(2018)
    df_gaps = benchmarker.compute_gaps(sg, 2018)
    benchmarker.save_results(df_gaps)
    files = list(Path(benchmarker.results_dir).glob("international_expenditure_benchmarking_*.csv"))
    assert len(files) == 1
```

---

### 11. Implementation Steps

#### Phase 1: External Data
- [ ] Implement `BenchmarkExtractor.run()` — check for cached CSV first
- [ ] If no live download available, use static fallback and save to `shared/data/2_external/who_ghed_peer_countries.csv`
- [ ] Document source and date in `shared/data/2_external/README.md`

#### Phase 2: Singapore Metrics
- [ ] Calculate Singapore per-capita USD PPP from `total_expenditure` (2018)
- [ ] Cross-check against known values (~$2,500 USD PPP in 2018)

#### Phase 3: Gap Analysis
- [ ] Compute `gap_vs_singapore_usd` for all peers
- [ ] Identify countries Singapore is above/below
- [ ] Compute OECD peer average as reference

#### Phase 4: Visualisations
- [ ] Horizontal bar chart (per-capita) → `expenditure_benchmark_bar_*.png`
- [ ] Time series: Singapore expenditure % GDP trend if GDP data sourced

#### Phase 5: Output & Tests
- [ ] Save `results/tables/problem-statement-004/international_expenditure_benchmarking_*.csv`
- [ ] Run unit tests; confirm ≥80% coverage
- [ ] Create `notebooks/07-international-expenditure-benchmarking.ipynb`

---

### 12. Adaptive Implementation Strategy

- **If WHO GHED download unavailable** → static 2018 fallback is pre-built in `benchmark_extractor.py`; document limitation in notebook
- **If GDP data unavailable** → skip `che_gdp_pct` comparison; focus on per-capita USD PPP only
- **If Singapore internal per-capita deviates significantly from WHO figure** → log discrepancy; compare internal vs WHO-reported Singapore to understand accounting differences (government-only vs total health expenditure)

---

### 21. Version Control

```
Branch: feature/ps-004-us-07-international-benchmarking
Commits:
  feat(ps-004): add BenchmarkExtractor with WHO GHED static fallback
  feat(ps-004): add InternationalBenchmarker with gap analysis and bar chart
  test(ps-004): unit tests for benchmark extraction and gap computation
```
