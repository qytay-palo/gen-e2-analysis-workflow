# Build Healthcare Expenditure Analysis Dashboard (Lifecycle Stage: Visualization)

**Story ID**: PS-004-US-09  
**Epic**: Healthcare Expenditure Drivers & Cost Control Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: L (7-8 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Healthcare Financial Planner tracking expenditure trends**,  
I want **an interactive dashboard displaying 15-year expenditure trends, driver decomposition, benchmarking, and cost control opportunities**,  
So that **I can monitor cost dynamics, explore drivers interactively, and communicate financial insights to budget committees and policymakers**.

---

## 🎯 Acceptance Criteria

1. **Core visualizations**
   - Expenditure trends: total, per capita, growth rates (2006-2018)
   - Driver decomposition: demographic, utilization, intensity contributions
   - Correlation analysis: expenditure vs utilization/demographics
   - Benchmarking: Singapore vs OECD comparisons
   - Cost control opportunities: priority matrix, savings potential

2. **Interactive features**
   - Year range slider
   - Metric selector: total expenditure, per capita, % GDP
   - Driver toggle: show/hide decomposition components
   - Benchmark toggle: overlay international comparisons
   - Opportunity filter: by category, savings potential

3. **User experience**
   - Responsive design
   - Tooltips with detailed metrics
   - Export: charts (PNG/PDF), data (CSV)
   - Auto-generated insights
   - Help documentation

4. **Deployment**
   - Dashboard deployed and accessible
   - User documentation provided
   - Code documentation

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9; Dash/Plotly deployed on cloud
- **Primary Library**: Plotly Dash
- **Data Backend**: Polars 0.20+
- **Visualization**: Plotly
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Problem Statement PS-004](../../../problem_statements/ps-004-healthcare-expenditure-drivers.md) - Dashboard objectives

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `plotly>=5.18.0`, `dash>=2.14.0`, `dash-bootstrap-components>=1.5.0`, `gunicorn>=21.0.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-004-US-03 through PS-004-US-08 (All analyses - BLOCKING)
- **Data Sources**: All PS-004 results tables
- **Config Files**: `config/dashboard.yml`

---

## ✅ Implementation Tasks

### Dashboard Architecture
- [ ] Initialize Dash app
- [ ] Design multi-tab layout: Overview, Trends, Drivers, Benchmarking, Opportunities
- [ ] Data loading and caching
- [ ] Component hierarchy

### Core Visualizations
- [ ] Expenditure trends: line charts
- [ ] Driver decomposition: waterfall/stacked bar
- [ ] Correlation: scatter plots
- [ ] Benchmarking: bar/line comparisons
- [ ] Opportunities: priority matrix

### Interactivity
- [ ] Year range slider
- [ ] Metric selector dropdown
- [ ] Driver component toggles
- [ ] Benchmark comparison toggle
- [ ] Opportunity filters

### Data Callbacks
- [ ] Filter data based on selections
- [ ] Update charts
- [ ] Generate narratives
- [ ] Export functionality

### Styling & UX
- [ ] Bootstrap theme
- [ ] Tooltips
- [ ] Loading spinners
- [ ] Error handling

### Deployment
- [ ] Dockerize
- [ ] Deploy to cloud
- [ ] HTTPS and authentication
- [ ] Performance optimization

### Testing & Documentation
- [ ] Unit tests
- [ ] Integration tests
- [ ] User guide
- [ ] Technical documentation

---

## 📌 Notes

**Dashboard Tabs**:
1. **Overview**: Key metrics, summary statistics
2. **Trends**: Expenditure growth over time
3. **Drivers**: Decomposition and correlation analysis
4. **Benchmarking**: International comparisons
5. **Opportunities**: Cost control initiatives
6. **About**: Methodology, glossary

**Key Features**:
- **Trend Extrapolation**: Project future expenditure based on historical trends
- **Driver Simulator**: Adjust utilization/demographics, see expenditure impact
- **Savings Calculator**: Select opportunities, estimate total savings

---

## Implementation Plan

### 1. Feature Overview

Build a five-tab Plotly Dash dashboard that integrates all PS-004 analytical outputs (trends, decomposition, correlation, benchmarking, cost-control opportunities) into a single interactive application. Primary user role: **Healthcare Financial Planner**.

---

### 2. Component Analysis & Reuse Strategy

| Component | Status | Action |
|-----------|--------|--------|
| `results/tables/problem-statement-004/expenditure_growth_analysis_2006_2018.csv` | US-03 output | **Reuse** |
| `results/tables/problem-statement-004/expenditure_decomposition_*.csv` | US-04 output | **Reuse** |
| `results/tables/problem-statement-004/expenditure_correlation_analysis_*.csv` | US-05 output | **Reuse** |
| `results/tables/problem-statement-004/international_expenditure_benchmarking_*.csv` | US-07 output | **Reuse** |
| `results/tables/problem-statement-004/cost_control_opportunities_*.csv` | US-08 output | **Reuse** |
| `shared/data/3_interim/expenditure_drivers_integrated.parquet` | US-02 output | **Reuse** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/src/dashboard/app.py` | Missing | **Create** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/src/dashboard/data_loader.py` | Missing | **Create** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/src/dashboard/layouts/` | Missing | **Create** |
| `problem-statements/ps-004-healthcare-expenditure-drivers/config/dashboard.yml` | Missing | **Create** |

---

### 4. Affected Files

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/dashboard/app.py`**
  - `create_app(config_path)` → `dash.Dash`
  - Multi-tab layout wiring
  - Tab-switching callback

- **[CREATE] `problem-statements/ps-004-healthcare-expenditure-drivers/src/dashboard/data_loader.py`**
  - `DashboardDataLoader` class
  - `load_trends()`, `load_decomposition()`, `load_correlations()`, `load_benchmarks()`, `load_opportunities()`, `load_integrated()`, `load_all()` → `dict`

- **[CREATE] `src/dashboard/layouts/overview.py`** — KPI cards
- **[CREATE] `src/dashboard/layouts/trends.py`** — line + YoY bar + slider callbacks
- **[CREATE] `src/dashboard/layouts/drivers.py`** — decomposition stacked bar + toggle
- **[CREATE] `src/dashboard/layouts/benchmarking.py`** — horizontal peer bar
- **[CREATE] `src/dashboard/layouts/opportunities.py`** — priority matrix + savings bar
- **[CREATE] `config/dashboard.yml`**
- **[CREATE] `tests/unit/test_dashboard_data_loader.py`**
- **[CREATE] `tests/integration/test_dashboard_callbacks.py`**

---

### 5. Data Pipeline

```
On app startup:
  DashboardDataLoader.load_all()
    glob latest results CSV per category (mtime-sorted)
    read with Polars → convert to list[dict] for dcc.Store

Dash dcc.Store (client-side):
  store-trends, store-decomp, store-correlations, store-benchmarks, store-opportunities

Callbacks:
  [year-range-slider] + [metric-selector] → filter store-trends → update trend graphs
  [component-toggle]  → include/exclude series in decomp stacked bar
  [benchmark-toggle]  → overlay/hide peer lines in benchmarking chart
  [category-filter]   → filter store-opportunities → update priority matrix + savings bar
```

---

### 6. Code Generation Specifications

#### 6.0 `dashboard.yml`

```yaml
dashboard:
  title: "Singapore Healthcare Expenditure Analysis"
  port: 8050
  debug: false
  theme: "BOOTSTRAP"

data:
  results_base: "results/tables/problem-statement-004"
  integrated_parquet: "shared/data/3_interim/expenditure_drivers_integrated.parquet"
  file_patterns:
    trends: "expenditure_growth_analysis_2006_2018.csv"
    decomposition: "expenditure_decomposition_*.csv"
    correlations: "expenditure_correlation_analysis_*.csv"
    benchmarks: "international_expenditure_benchmarking_*.csv"
    opportunities: "cost_control_opportunities_*.csv"

colors:
  primary: "#1f77b4"
  accent: "#d62728"
  highlight: "#ff7f0e"
  neutral: "#7f7f7f"
```

#### 6.1 Complete Implementation — `data_loader.py`

```python
"""
Dashboard Data Loader
=====================

Loads all PS-004 analysis result tables for the expenditure dashboard.

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
from pathlib import Path
from typing import Optional

import polars as pl
import yaml

logger = logging.getLogger(__name__)


def _read_latest_glob(pattern: str) -> Optional[pl.DataFrame]:
    """
    Find the most recently modified file matching a glob pattern and read it.

    Args:
        pattern: Glob pattern relative to cwd.

    Returns:
        Polars DataFrame, or None if no file found.
    """
    matches = sorted(
        Path(".").glob(pattern),
        key=lambda p: p.stat().st_mtime,
        reverse=True,
    )
    if not matches:
        logger.warning(f"No files matched pattern: {pattern}")
        return None
    logger.info(f"Loading: {matches[0]}")
    return pl.read_csv(str(matches[0]))


class DashboardDataLoader:
    """
    Load all PS-004 result tables required by the expenditure dashboard.

    Example:
        >>> loader = DashboardDataLoader("config/dashboard.yml")
        >>> data = loader.load_all()
    """

    def __init__(self, config_path: str = "config/dashboard.yml") -> None:
        """
        Initialise loader from dashboard YAML config.

        Args:
            config_path: Path to dashboard.yml.
        """
        config_file = Path(config_path)
        if not config_file.exists():
            raise FileNotFoundError(f"Dashboard config not found: {config_path}")
        with open(config_file) as f:
            self.config = yaml.safe_load(f)
        self.results_base = Path(self.config["data"]["results_base"])
        self.patterns = self.config["data"]["file_patterns"]
        self.integrated_path = Path(self.config["data"]["integrated_parquet"])

    def load_trends(self) -> pl.DataFrame:
        """Load expenditure trends CSV (US-03 output)."""
        path = self.results_base / self.patterns["trends"]
        if not path.exists():
            logger.warning(f"Trends file not found: {path}")
            return pl.DataFrame()
        return pl.read_csv(str(path))

    def load_decomposition(self) -> pl.DataFrame:
        """Load latest expenditure decomposition CSV (US-04 output)."""
        df = _read_latest_glob(str(self.results_base / self.patterns["decomposition"]))
        return df if df is not None else pl.DataFrame()

    def load_correlations(self) -> pl.DataFrame:
        """Load latest correlation analysis CSV (US-05 output)."""
        df = _read_latest_glob(str(self.results_base / self.patterns["correlations"]))
        return df if df is not None else pl.DataFrame()

    def load_benchmarks(self) -> pl.DataFrame:
        """Load latest benchmarking CSV (US-07 output)."""
        df = _read_latest_glob(str(self.results_base / self.patterns["benchmarks"]))
        return df if df is not None else pl.DataFrame()

    def load_opportunities(self) -> pl.DataFrame:
        """Load latest cost control opportunities CSV (US-08 output)."""
        df = _read_latest_glob(str(self.results_base / self.patterns["opportunities"]))
        return df if df is not None else pl.DataFrame()

    def load_integrated(self) -> pl.DataFrame:
        """Load integrated Parquet (US-02 output) for real-time slider filtering."""
        if not self.integrated_path.exists():
            logger.warning(f"Integrated Parquet not found: {self.integrated_path}")
            return pl.DataFrame()
        return pl.read_parquet(str(self.integrated_path))

    def load_all(self) -> dict:
        """
        Load all data sources.

        Returns:
            Dict with keys: trends, decomposition, correlations,
            benchmarks, opportunities, integrated.
        """
        data = {
            "trends": self.load_trends(),
            "decomposition": self.load_decomposition(),
            "correlations": self.load_correlations(),
            "benchmarks": self.load_benchmarks(),
            "opportunities": self.load_opportunities(),
            "integrated": self.load_integrated(),
        }
        for key, df in data.items():
            logger.info(f"Loaded '{key}': {len(df)} rows")
        return data
```

#### 6.2 Complete Implementation — `app.py`

```python
"""
PS-004 Healthcare Expenditure Dashboard
========================================

Plotly Dash application integrating all PS-004 analysis outputs.
Launch: python -m .../src/dashboard/app

Author: Gen-E2 Team
Date: 2026-03-25
"""

import logging
import sys
from pathlib import Path

import dash
import dash_bootstrap_components as dbc
import yaml
from dash import Input, Output, dcc, html

from .data_loader import DashboardDataLoader
from .layouts.benchmarking import benchmarking_layout
from .layouts.drivers import drivers_layout
from .layouts.opportunities import opportunities_layout
from .layouts.overview import overview_layout
from .layouts.trends import trends_layout

logger = logging.getLogger(__name__)


def create_app(config_path: str = "config/dashboard.yml") -> dash.Dash:
    """
    Create and configure the Dash application.

    Args:
        config_path: Path to dashboard YAML configuration.

    Returns:
        Configured Dash app instance.
    """
    with open(config_path) as f:
        config = yaml.safe_load(f)

    theme_name = config["dashboard"].get("theme", "BOOTSTRAP")
    theme = getattr(dbc.themes, theme_name, dbc.themes.BOOTSTRAP)

    app = dash.Dash(
        __name__,
        external_stylesheets=[theme],
        suppress_callback_exceptions=True,
        title=config["dashboard"]["title"],
    )

    loader = DashboardDataLoader(config_path)
    data = loader.load_all()

    app.layout = dbc.Container(
        fluid=True,
        children=[
            dbc.Row(
                dbc.Col(
                    html.H3(config["dashboard"]["title"], className="text-primary my-3"),
                    width=12,
                )
            ),
            dbc.Tabs(
                id="main-tabs",
                active_tab="tab-overview",
                children=[
                    dbc.Tab(label="Overview", tab_id="tab-overview"),
                    dbc.Tab(label="Trends", tab_id="tab-trends"),
                    dbc.Tab(label="Drivers", tab_id="tab-drivers"),
                    dbc.Tab(label="Benchmarking", tab_id="tab-benchmarking"),
                    dbc.Tab(label="Opportunities", tab_id="tab-opportunities"),
                ],
            ),
            html.Div(id="tab-content", className="mt-3"),
            dcc.Store(id="store-trends", data=data["trends"].to_dicts()),
            dcc.Store(id="store-decomp", data=data["decomposition"].to_dicts()),
            dcc.Store(id="store-correlations", data=data["correlations"].to_dicts()),
            dcc.Store(id="store-benchmarks", data=data["benchmarks"].to_dicts()),
            dcc.Store(id="store-opportunities", data=data["opportunities"].to_dicts()),
            dcc.Store(id="store-integrated", data=data["integrated"].to_dicts()),
        ],
    )

    @app.callback(Output("tab-content", "children"), Input("main-tabs", "active_tab"))
    def render_tab(active_tab: str) -> html.Div:
        """Switch visible tab layout based on active tab selection."""
        if active_tab == "tab-overview":
            return overview_layout(data)
        if active_tab == "tab-trends":
            return trends_layout(data)
        if active_tab == "tab-drivers":
            return drivers_layout(data)
        if active_tab == "tab-benchmarking":
            return benchmarking_layout(data)
        if active_tab == "tab-opportunities":
            return opportunities_layout(data)
        return html.Div("Unknown tab")

    return app


if __name__ == "__main__":
    config_arg = sys.argv[1] if len(sys.argv) > 1 else "config/dashboard.yml"
    with open(config_arg) as f:
        cfg = yaml.safe_load(f)
    application = create_app(config_arg)
    application.run(
        debug=cfg["dashboard"].get("debug", False),
        port=cfg["dashboard"].get("port", 8050),
    )
```

#### 6.3 Layout Modules

```python
# layouts/overview.py
"""Overview tab: KPI cards for key expenditure metrics."""

import polars as pl
import dash_bootstrap_components as dbc
from dash import html


def _kpi_card(title: str, value: str, colour: str = "primary") -> dbc.Card:
    return dbc.Card(
        dbc.CardBody([
            html.H6(title, className="card-subtitle text-muted"),
            html.H4(value, className=f"text-{colour} fw-bold"),
        ]),
        className="shadow-sm",
    )


def overview_layout(data: dict) -> html.Div:
    """
    Render the Overview tab with KPI cards.

    Args:
        data: Pre-loaded data dict from DashboardDataLoader.

    Returns:
        Dash layout component.
    """
    df = pl.DataFrame(data.get("trends", []))
    if len(df) > 0 and "total_expenditure" in df.columns:
        latest_exp = f"SGD {df['total_expenditure'].max():,.0f}M"
        avg_growth = (
            f"{df['yoy_growth_pct'].drop_nulls().mean():.1f}%"
            if "yoy_growth_pct" in df.columns
            else "N/A"
        )
        years = f"{df['year'].min()} \u2013 {df['year'].max()}"
    else:
        latest_exp, avg_growth, years = "N/A", "N/A", "2006 \u2013 2018"

    return html.Div([
        dbc.Row([
            dbc.Col(_kpi_card("Latest Expenditure (2018)", latest_exp, "primary"), width=4),
            dbc.Col(_kpi_card("Avg. YoY Growth", avg_growth, "warning"), width=4),
            dbc.Col(_kpi_card("Analysis Period", years, "info"), width=4),
        ], className="mb-4"),
        dbc.Alert(
            "Based on Singapore Ministry of Health data (2006\u20132018). "
            "Source: Kaggle \u2014 subhamjain/health-dataset-complete-singapore.",
            color="light",
        ),
    ])
```

```python
# layouts/trends.py
"""Trends tab: line chart + YoY bar chart with year range slider."""

import polars as pl
import plotly.graph_objects as go
import dash_bootstrap_components as dbc
from dash import Input, Output, callback, dcc, html


def trends_layout(data: dict) -> html.Div:
    """
    Render the Trends tab.

    Args:
        data: Pre-loaded data dict.

    Returns:
        Dash layout component.
    """
    df = pl.DataFrame(data.get("trends", []))
    min_year = int(df["year"].min()) if len(df) > 0 else 2006
    max_year = int(df["year"].max()) if len(df) > 0 else 2018

    return html.Div([
        dbc.Row([
            dbc.Col([
                html.Label("Year Range"),
                dcc.RangeSlider(
                    id="year-range-slider",
                    min=min_year, max=max_year, step=1,
                    value=[min_year, max_year],
                    marks={yr: str(yr) for yr in range(min_year, max_year + 1, 2)},
                ),
            ], width=8),
            dbc.Col([
                html.Label("Metric"),
                dbc.Select(
                    id="metric-selector",
                    options=[
                        {"label": "Total Expenditure (SGD M)", "value": "total_expenditure"},
                        {"label": "Per Capita Expenditure", "value": "per_capita_expenditure"},
                        {"label": "YoY Growth (%)", "value": "yoy_growth_pct"},
                    ],
                    value="total_expenditure",
                ),
            ], width=4),
        ], className="mb-3"),
        dbc.Row([
            dbc.Col(dcc.Graph(id="trend-line-chart"), width=8),
            dbc.Col(dcc.Graph(id="yoy-bar-chart"), width=4),
        ]),
    ])


@callback(
    Output("trend-line-chart", "figure"),
    Output("yoy-bar-chart", "figure"),
    Input("year-range-slider", "value"),
    Input("metric-selector", "value"),
    Input("store-trends", "data"),
)
def update_trends(year_range: list, metric: str, store_data: list) -> tuple:
    """
    Update trend charts based on year range and metric selection.

    Args:
        year_range: [start_year, end_year] from slider.
        metric: Selected metric column name.
        store_data: Serialised trends data from dcc.Store.

    Returns:
        Tuple of (line_figure, bar_figure).
    """
    df = pl.DataFrame(store_data).filter(
        (pl.col("year") >= year_range[0]) & (pl.col("year") <= year_range[1])
    )
    empty = go.Figure()
    empty.update_layout(title="No data available", template="simple_white")
    if len(df) == 0 or metric not in df.columns:
        return empty, empty

    years = df["year"].to_list()
    values = df[metric].to_list()

    line_fig = go.Figure(go.Scatter(
        x=years, y=values, mode="lines+markers",
        line={"color": "#1f77b4", "width": 2.5}, marker={"size": 7},
        hovertemplate="%{x}: %{y:.1f}<extra></extra>",
    ))
    line_fig.update_layout(
        title=f"{metric.replace('_', ' ').title()} Over Time",
        xaxis_title="Year", yaxis_title=metric.replace("_", " ").title(),
        template="simple_white",
    )

    yoy_vals = df["yoy_growth_pct"].to_list() if "yoy_growth_pct" in df.columns else values
    bar_fig = go.Figure(go.Bar(
        x=years, y=yoy_vals, marker_color="#ff7f0e",
        hovertemplate="%{x}: %{y:.1f}%<extra></extra>",
    ))
    bar_fig.update_layout(
        title="YoY Growth Rate (%)", xaxis_title="Year",
        yaxis_title="Growth (%)", template="simple_white",
    )
    return line_fig, bar_fig
```

```python
# layouts/drivers.py
"""Drivers tab: decomposition stacked bar with component toggle."""

import polars as pl
import plotly.graph_objects as go
import dash_bootstrap_components as dbc
from dash import Input, Output, callback, dcc, html


def drivers_layout(data: dict) -> html.Div:
    """
    Render the Drivers tab.

    Args:
        data: Pre-loaded data dict.

    Returns:
        Dash layout component.
    """
    return html.Div([
        dbc.Row([
            dbc.Col([
                html.Label("Show Components"),
                dbc.Checklist(
                    id="component-toggle",
                    options=[
                        {"label": "Demographic", "value": "demographic_effect"},
                        {"label": "Volume", "value": "volume_effect"},
                        {"label": "Price/Intensity", "value": "price_effect"},
                    ],
                    value=["demographic_effect", "volume_effect", "price_effect"],
                    inline=True,
                ),
            ]),
        ], className="mb-3"),
        dbc.Row([
            dbc.Col(dcc.Graph(id="decomp-stacked-bar"), width=8),
            dbc.Col(dcc.Graph(id="correlation-scatter"), width=4),
        ]),
    ])


@callback(
    Output("decomp-stacked-bar", "figure"),
    Input("component-toggle", "value"),
    Input("store-decomp", "data"),
)
def update_decomp(components: list, store_data: list) -> go.Figure:
    """
    Update decomposition chart based on component toggle.

    Args:
        components: List of selected decomposition column names.
        store_data: Serialised decomposition data from dcc.Store.

    Returns:
        Plotly stacked bar figure.
    """
    df = pl.DataFrame(store_data)
    fig = go.Figure()
    colors = {
        "demographic_effect": "#1f77b4",
        "volume_effect": "#ff7f0e",
        "price_effect": "#d62728",
    }
    if len(df) > 0 and "year" in df.columns:
        years = df["year"].to_list()
        for comp in components:
            if comp in df.columns:
                fig.add_trace(go.Bar(
                    name=comp.replace("_", " ").title(),
                    x=years, y=df[comp].to_list(),
                    marker_color=colors.get(comp, "#7f7f7f"),
                    hovertemplate="%{x}: %{y:.1f}<extra></extra>",
                ))
    fig.update_layout(
        barmode="relative", title="Expenditure Growth Decomposition",
        xaxis_title="Year", yaxis_title="SGD Millions",
        template="simple_white",
    )
    return fig
```

```python
# layouts/benchmarking.py
"""Benchmarking tab: Singapore vs peer countries horizontal bar."""

import polars as pl
import plotly.graph_objects as go
import dash_bootstrap_components as dbc
from dash import dcc, html


def benchmarking_layout(data: dict) -> html.Div:
    """
    Render the Benchmarking tab.

    Args:
        data: Pre-loaded data dict.

    Returns:
        Dash layout component.
    """
    df = pl.DataFrame(data.get("benchmarks", []))
    fig = go.Figure()
    if len(df) > 0 and "country" in df.columns and "che_per_capita_usd_ppp" in df.columns:
        df_sorted = df.sort("che_per_capita_usd_ppp")
        fig.add_trace(go.Bar(
            x=df_sorted["che_per_capita_usd_ppp"].to_list(),
            y=df_sorted["country"].to_list(),
            orientation="h",
            marker_color=[
                "#d62728" if c == "Singapore" else "#1f77b4"
                for c in df_sorted["country"].to_list()
            ],
            hovertemplate="%{y}: USD %{x:,.0f} PPP<extra></extra>",
        ))
    fig.update_layout(
        title="Healthcare Expenditure Per Capita (USD PPP) \u2014 Peer Comparison",
        xaxis_title="USD PPP Per Capita", template="simple_white",
    )
    return html.Div([
        dbc.Row(dbc.Col(dcc.Graph(figure=fig), width=12)),
        dbc.Alert(
            "Source: WHO Global Health Expenditure Database. Singapore value: MOH data.",
            color="light", className="mt-3",
        ),
    ])
```

```python
# layouts/opportunities.py
"""Opportunities tab: priority matrix scatter + savings grouped bar."""

import polars as pl
import plotly.graph_objects as go
import dash_bootstrap_components as dbc
from dash import Input, Output, callback, dcc, html


def opportunities_layout(data: dict) -> html.Div:
    """
    Render the Opportunities tab.

    Args:
        data: Pre-loaded data dict.

    Returns:
        Dash layout component.
    """
    df = pl.DataFrame(data.get("opportunities", []))
    categories = ["All"] + (df["category"].unique().to_list() if "category" in df.columns else [])
    return html.Div([
        dbc.Row([
            dbc.Col([
                html.Label("Filter by Category"),
                dbc.Select(
                    id="opportunity-category-filter",
                    options=[{"label": c, "value": c} for c in categories],
                    value="All",
                ),
            ], width=4),
        ], className="mb-3"),
        dbc.Row([
            dbc.Col(dcc.Graph(id="priority-matrix-scatter"), width=6),
            dbc.Col(dcc.Graph(id="savings-bar-chart"), width=6),
        ]),
    ])


@callback(
    Output("priority-matrix-scatter", "figure"),
    Output("savings-bar-chart", "figure"),
    Input("opportunity-category-filter", "value"),
    Input("store-opportunities", "data"),
)
def update_opportunities(category: str, store_data: list) -> tuple:
    """
    Update opportunity charts based on category filter.

    Args:
        category: Selected category or 'All'.
        store_data: Serialised opportunities data from dcc.Store.

    Returns:
        Tuple of (priority_matrix_figure, savings_bar_figure).
    """
    df = pl.DataFrame(store_data)
    if category != "All" and "category" in df.columns:
        df = df.filter(pl.col("category") == category)

    empty = go.Figure()
    empty.update_layout(title="No data available", template="simple_white")
    if len(df) == 0:
        return empty, empty

    matrix_fig = go.Figure()
    if "implementation_difficulty" in df.columns and "savings_sgd_m_annual_optimistic" in df.columns:
        matrix_fig.add_trace(go.Scatter(
            x=df["implementation_difficulty"].to_list(),
            y=df["savings_sgd_m_annual_optimistic"].to_list(),
            mode="markers+text",
            text=df["opportunity_id"].to_list() if "opportunity_id" in df.columns else [],
            textposition="top center",
            marker={"size": 14, "color": "#1f77b4"},
            hovertemplate="ID: %{text}<br>Difficulty: %{x}<br>Savings: SGD %{y:.0f}M<extra></extra>",
        ))
    matrix_fig.update_layout(
        title="Cost Control Priority Matrix",
        xaxis_title="Implementation Difficulty (1-10)",
        yaxis_title="Annual Savings (SGD Millions)",
        template="simple_white",
    )

    bar_fig = go.Figure()
    if "opportunity_id" in df.columns and "savings_sgd_m_annual_optimistic" in df.columns:
        bar_fig.add_trace(go.Bar(
            x=df["opportunity_id"].to_list(),
            y=df["savings_sgd_m_annual_optimistic"].to_list(),
            marker_color="#ff7f0e", name="Optimistic",
            hovertemplate="%{x}: SGD %{y:.0f}M<extra></extra>",
        ))
        if "savings_sgd_m_annual_conservative" in df.columns:
            bar_fig.add_trace(go.Bar(
                x=df["opportunity_id"].to_list(),
                y=df["savings_sgd_m_annual_conservative"].to_list(),
                marker_color="#aec7e8", name="Conservative",
            ))
    bar_fig.update_layout(
        barmode="group", title="Annual Savings Potential (SGD Millions)",
        xaxis_title="Opportunity", yaxis_title="SGD Millions",
        template="simple_white",
    )
    return matrix_fig, bar_fig
```

#### 6.5 Test Specifications

```python
# tests/unit/test_dashboard_data_loader.py

import polars as pl
import pytest
from pathlib import Path
from problem_statements.ps_004_healthcare_expenditure_drivers.src.dashboard.data_loader import (
    DashboardDataLoader,
    _read_latest_glob,
)


@pytest.fixture
def config_file(tmp_path: Path) -> Path:
    cfg = tmp_path / "dashboard.yml"
    cfg.write_text(f"""
dashboard:
  title: "Test Dashboard"
  port: 8050
  debug: false
  theme: BOOTSTRAP
data:
  results_base: "{tmp_path}/results"
  integrated_parquet: "{tmp_path}/integrated.parquet"
  file_patterns:
    trends: "expenditure_growth_analysis_2006_2018.csv"
    decomposition: "expenditure_decomposition_*.csv"
    correlations: "expenditure_correlation_analysis_*.csv"
    benchmarks: "international_expenditure_benchmarking_*.csv"
    opportunities: "cost_control_opportunities_*.csv"
""")
    return cfg


@pytest.fixture
def loader_with_data(tmp_path: Path, config_file: Path) -> DashboardDataLoader:
    results = tmp_path / "results"
    results.mkdir(parents=True)
    pl.DataFrame({"year": [2006, 2018], "total_expenditure": [3500.0, 6500.0]}).write_csv(
        str(results / "expenditure_growth_analysis_2006_2018.csv")
    )
    pl.DataFrame({"year": [2006], "demographic_effect": [100.0]}).write_csv(
        str(results / "expenditure_decomposition_20260325.csv")
    )
    pl.DataFrame({"country": ["Singapore", "Japan"],
                  "che_per_capita_usd_ppp": [2500.0, 3500.0]}).write_csv(
        str(results / "international_expenditure_benchmarking_20260325.csv")
    )
    pl.DataFrame({"opportunity_id": ["OPP-01"], "category": ["Efficiency"],
                  "savings_sgd_m_annual_optimistic": [600.0],
                  "savings_sgd_m_annual_conservative": [300.0],
                  "implementation_difficulty": [5]}).write_csv(
        str(results / "cost_control_opportunities_20260325.csv")
    )
    pl.DataFrame({"year": [2006, 2018], "total_expenditure": [3500.0, 6500.0]}).write_parquet(
        str(tmp_path / "integrated.parquet")
    )
    return DashboardDataLoader(str(config_file))


def test_load_trends_returns_dataframe(loader_with_data: DashboardDataLoader) -> None:
    df = loader_with_data.load_trends()
    assert isinstance(df, pl.DataFrame)
    assert len(df) == 2


def test_load_benchmarks_two_rows(loader_with_data: DashboardDataLoader) -> None:
    df = loader_with_data.load_benchmarks()
    assert len(df) == 2
    assert "country" in df.columns


def test_load_all_six_keys(loader_with_data: DashboardDataLoader) -> None:
    data = loader_with_data.load_all()
    assert set(data.keys()) == {"trends", "decomposition", "correlations",
                                "benchmarks", "opportunities", "integrated"}


def test_missing_file_returns_empty_dataframe(loader_with_data: DashboardDataLoader) -> None:
    df = loader_with_data.load_correlations()
    assert isinstance(df, pl.DataFrame)
    assert len(df) == 0


def test_read_latest_glob_returns_newest(tmp_path: Path) -> None:
    import time
    f1 = tmp_path / "data_20240101.csv"
    f1.write_text("col\n1\n")
    time.sleep(0.05)
    f2 = tmp_path / "data_20240201.csv"
    f2.write_text("col\n2\n")
    df = _read_latest_glob(str(tmp_path / "data_*.csv"))
    assert df is not None
    assert df["col"][0] == 2
```

```python
# tests/integration/test_dashboard_callbacks.py

import polars as pl
import pytest
from problem_statements.ps_004_healthcare_expenditure_drivers.src.dashboard.layouts.trends import (
    update_trends,
)
from problem_statements.ps_004_healthcare_expenditure_drivers.src.dashboard.layouts.opportunities import (
    update_opportunities,
)
from problem_statements.ps_004_healthcare_expenditure_drivers.src.dashboard.layouts.drivers import (
    update_decomp,
)


@pytest.fixture
def trends_store() -> list:
    return pl.DataFrame({
        "year": list(range(2006, 2019)),
        "total_expenditure": [3500.0 + i * 250 for i in range(13)],
        "yoy_growth_pct": [None] + [7.0] * 12,
    }).to_dicts()


@pytest.fixture
def opportunities_store() -> list:
    return pl.DataFrame({
        "opportunity_id": ["OPP-01", "OPP-02"],
        "category": ["Efficiency", "Care Setting"],
        "savings_sgd_m_annual_optimistic": [600.0, 150.0],
        "savings_sgd_m_annual_conservative": [300.0, 75.0],
        "implementation_difficulty": [5, 3],
    }).to_dicts()


@pytest.fixture
def decomp_store() -> list:
    return pl.DataFrame({
        "year": list(range(2007, 2019)),
        "demographic_effect": [50.0] * 12,
        "volume_effect": [80.0] * 12,
        "price_effect": [120.0] * 12,
    }).to_dicts()


def test_update_trends_returns_two_figures(trends_store: list) -> None:
    line_fig, bar_fig = update_trends([2006, 2018], "total_expenditure", trends_store)
    assert len(line_fig.data) == 1
    assert len(bar_fig.data) == 1


def test_update_trends_year_filter(trends_store: list) -> None:
    line_fig, _ = update_trends([2010, 2015], "total_expenditure", trends_store)
    x_vals = list(line_fig.data[0].x)
    assert min(x_vals) >= 2010
    assert max(x_vals) <= 2015


def test_update_opportunities_all_returns_two_points(opportunities_store: list) -> None:
    matrix_fig, _ = update_opportunities("All", opportunities_store)
    assert len(matrix_fig.data[0].x) == 2


def test_update_opportunities_category_filter(opportunities_store: list) -> None:
    matrix_fig, _ = update_opportunities("Efficiency", opportunities_store)
    assert len(matrix_fig.data[0].x) == 1


def test_update_decomp_all_components(decomp_store: list) -> None:
    fig = update_decomp(
        ["demographic_effect", "volume_effect", "price_effect"],
        decomp_store,
    )
    assert len(fig.data) == 3


def test_update_decomp_partial_components(decomp_store: list) -> None:
    fig = update_decomp(["demographic_effect"], decomp_store)
    assert len(fig.data) == 1
    assert fig.data[0].name == "Demographic Effect"
```

#### 6.6 Package Management

```bash
uv pip install polars>=0.20.0 "plotly>=5.18.0" "dash>=2.14.0" dash-bootstrap-components>=1.5.0 pyyaml>=6.0 gunicorn>=21.0.0 loguru>=0.7.0
uv pip freeze > requirements.txt
```

---

### 9. Styling & Visualisation

**Theme**: `dash-bootstrap-components` `BOOTSTRAP`

| Tab | Primary Chart Type | Controls |
|-----|--------------------|---------|
| Overview | KPI cards (3-col) | — |
| Trends | Line + YoY bar | Year slider, metric dropdown |
| Drivers | Stacked relative bar | Component checklist |
| Benchmarking | Horizontal bar (peer comparison) | — |
| Opportunities | Priority scatter + savings grouped bar | Category filter |

**Colours**: primary `#1f77b4`, accent `#d62728`, highlight `#ff7f0e`, neutral `#7f7f7f`  
**Singapore highlight**: Always `#d62728` in peer comparison bar.  
**Responsive**: Fluid container + Bootstrap 12-column grid; all graphs in `dbc.Col`.

---

### 10. Testing Strategy

**Unit** (`tests/unit/test_dashboard_data_loader.py`, 5 tests):
- `load_trends()` returns non-empty DataFrame with correct row count
- `load_all()` returns exactly 6 keys
- Missing file returns empty DataFrame (not exception)
- `_read_latest_glob()` returns newest file by mtime

**Integration** (`tests/integration/test_dashboard_callbacks.py`, 6 tests):
- `update_trends()` returns two Plotly figures with ≥1 trace
- Year range filter constrains x-axis correctly
- Opportunity category filter reduces data points
- Decomposition component toggle controls trace count

---

### 11. Implementation Steps

#### Phase 1: Foundation
- [ ] Create `src/dashboard/` directory with `__init__.py`
- [ ] Create `config/dashboard.yml` with correct paths to US-03–08 outputs
- [ ] Implement `data_loader.py` — verify `load_all()` returns 6 keys with data

#### Phase 2: Layout Modules
- [ ] Implement `layouts/overview.py` — KPI cards populated from trends data
- [ ] Implement `layouts/trends.py` — line chart + YoY bar with year slider + metric dropdown
- [ ] Implement `layouts/drivers.py` — decomposition stacked bar + component toggle
- [ ] Implement `layouts/benchmarking.py` — horizontal peer comparison bar
- [ ] Implement `layouts/opportunities.py` — priority matrix + savings grouped bar with category filter

#### Phase 3: App Wiring
- [ ] Implement `app.py` — `create_app()`, tab-switching callback, `dcc.Store` population
- [ ] Launch locally: `python -m …/src/dashboard/app` → verify `http://localhost:8050`
- [ ] Confirm each tab renders without console errors
- [ ] Test year slider, metric dropdown, and category filter interactively

#### Phase 4: Testing
- [ ] Write and run unit tests for `DashboardDataLoader`
- [ ] Write and run integration tests for callbacks
- [ ] Confirm ≥80% test coverage

#### Phase 5: Documentation
- [ ] Create `notebooks/09-create-expenditure-dashboard.ipynb` with app launch cell
- [ ] Add `README.md` to `src/dashboard/` documenting tab structure and data dependencies

---

### 12. Adaptive Implementation Strategy

- **If any upstream results CSV is missing** → `DashboardDataLoader` returns empty `pl.DataFrame`; affected tab shows "No data available" alert — app does not crash
- **If `dash-bootstrap-components` conflicts** → fall back to Bootstrap CDN URL in `external_stylesheets`
- **If Plotly PNG export needed** → add `uv pip install kaleido`; expose server-side export callback
- **If port 8050 occupied** → update `port:` in `dashboard.yml`; restart app
- **If data updated** → restart app; `_read_latest_glob()` picks newest file by `mtime` automatically

---

### 13. Code Generation Order

1. `config/dashboard.yml`
2. `src/dashboard/__init__.py`
3. `src/dashboard/data_loader.py`
4. `src/dashboard/layouts/overview.py`
5. `src/dashboard/layouts/trends.py`
6. `src/dashboard/layouts/drivers.py`
7. `src/dashboard/layouts/benchmarking.py`
8. `src/dashboard/layouts/opportunities.py`
9. `src/dashboard/app.py`
10. `tests/unit/test_dashboard_data_loader.py`
11. `tests/integration/test_dashboard_callbacks.py`
12. `notebooks/09-create-expenditure-dashboard.ipynb`

---

### 17. UI/Dashboard Testing

**Manual checklist**:
- [ ] All 5 tabs render without browser console errors
- [ ] Year slider filters trend data correctly (no out-of-range years shown)
- [ ] Metric selector changes y-axis label and values
- [ ] Decomposition checklist show/hides individual bar traces
- [ ] Singapore bar highlighted red in benchmarking chart
- [ ] Category filter narrows opportunity matrix to correct subset
- [ ] Tooltips display correct formatted values on hover
- [ ] Fluid layout reflows at 768px width (Bootstrap responsive)

---

### 19. References

- [03-analyze-expenditure-trends.md](03-analyze-expenditure-trends.md) — Trends CSV spec
- [04-decompose-expenditure-growth.md](04-decompose-expenditure-growth.md) — Decomposition CSV spec
- [07-international-expenditure-benchmarking.md](07-international-expenditure-benchmarking.md) — Benchmarking CSV spec
- [08-cost-control-opportunities.md](08-cost-control-opportunities.md) — Opportunities CSV spec
- [Plotly Dash docs](https://dash.plotly.com/) — Callback and dcc.Store reference
- [dash-bootstrap-components](https://dash-bootstrap-components.opensource.faculty.ai/) — Grid and theme reference

---

### 20. Security & Privacy

- Dashboard serves **aggregate, public healthcare data** — no PII/PHI
- No auth required for internal deployment
- For external access: wrap with `dash-auth` BasicAuth or reverse proxy with HTTP auth
- `dcc.Store` serialises data as JSON in browser; contains only aggregate statistics

---

### 21. Version Control

```
Branch: feature/ps-004-us-09-expenditure-dashboard
Commits:
  feat(ps-004): add DashboardDataLoader with mtime-sorted glob resolution
  feat(ps-004): 5-tab Dash layout with overview, trends, drivers, benchmarking, opportunities
  feat(ps-004): trends tab year slider and metric selector callbacks
  feat(ps-004): opportunities tab priority matrix and savings bar with category filter
  test(ps-004): unit tests for data loader and integration tests for callbacks
  docs(ps-004): dashboard config guide and notebook launch instructions
```
