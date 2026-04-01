# Create Interactive Workforce Planning Dashboard (Lifecycle Stage: Visualization & Insights)

**Story ID**: PS-001-US-10  
**Epic**: Healthcare Workforce Sustainability Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: L (8-9 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Workforce Planning Division Manager at MOH**,  
I want **an interactive dashboard that visualizes workforce trends, forecasts through 2030, supply-demand gaps, and scenario comparisons in an intuitive interface**,  
So that **my team can explore workforce projections dynamically, export visualizations for presentations, and justify policy interventions to senior leadership and Ministers**.

---

## 🎯 Acceptance Criteria

1. **Dashboard functionality implemented**
   - Platform: Plotly Dash or Databricks Dashboard (stakeholder preference)
   - Pages/Tabs:
     1. **Overview**: High-level workforce metrics, key insights summary
     2. **Historical Trends**: Interactive trend charts (2006-2019) with profession selection
     3. **Forecasts 2020-2030**: Forecast visualizations with confidence intervals
     4. **Supply-Demand Gaps**: Gap analysis charts, shortage timelines
     5. **Scenario Comparison**: Interactive scenario comparison tool
   - Filters: profession, sector (public/private), year range, scenario selection
   - Export capability: download charts as PNG/PDF, export data as CSV

2. **Visualizations included**
   - Line charts: workforce counts over time (historical + forecast)
   - Bar charts: CAGR by profession, gap magnitude by profession
   - Heatmaps: growth rates by year and profession, gap severity matrix
   - Waterfall charts: scenario impact decomposition
   - Benchmark comparison: Singapore vs WHO/OECD standards
   - All charts interactive (hover tooltips, zoom, pan)

3. **User experience quality**
   - Responsive design (works on desktop and tablet)
   - Loading time < 3 seconds for chart updates
   - Clear labeling: all charts have titles, axis labels, legends, units
   - Color scheme: consistent, accessible (colorblind-friendly palette)
   - Help documentation: in-dashboard tooltips explaining metrics

4. **Data output requirement**
   - Dashboard URL: Hosted on Databricks or internal MOH server
   - Source data: Automatically refreshes from `shared/data/4_processed/` Parquet files
   - Export folder: `reports/dashboard_exports/` for user-downloaded charts and data
   - Dashboard documentation: `docs/dashboard_user_guide.md`

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9 (or Databricks Dashboard native)
- **Primary Library**: Polars 0.20+ (data loading), Plotly/Dash (visualization)
- **Hosting**: Databricks Dashboard (preferred) or standalone Dash app
- **Authentication**: MOH SSO integration (if applicable)
- **Logging**: loguru (NOT print statements)
- **Testing**: pytest + Dash testing utilities for dashboard functionality

---

## 📚 Domain Knowledge References

- [Healthcare Workforce Metrics & KPIs](../../../../domain_knowledge/healthcare-workforce-metrics-kpis.md) - Metrics to display in dashboard
- [Problem Statement PS-001](../../../problem_statements/ps-001-healthcare-workforce-sustainability.md#expected-outcomes-and-deliverables) - Dashboard requirements from problem statement

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`: Data loading and processing
- `plotly>=5.18.0`: Interactive charts
- `dash>=2.14.0`: Dashboard framework (if Plotly Dash chosen)
- `dash-bootstrap-components>=1.5.0`: Bootstrap styling for Dash
- `loguru>=0.7.0`: Structured logging
- `pyyaml>=6.0`: Configuration management

### Internal Dependencies
- **Upstream**: 
  - PS-001-US-03 (Trends - data for historical charts)
  - PS-001-US-04 (Benchmarks - data for comparison charts)
  - PS-001-US-06 (Forecasts - data for projection charts)
  - PS-001-US-07 (Gap analysis - data for shortage visualizations)
  - PS-001-US-08 (Scenarios - data for intervention comparisons)
- **Data Sources**: 
  - `shared/data/3_interim/workforce_integrated_clean.parquet` (historical)
  - `shared/data/4_processed/workforce_forecasts_2020_2030.parquet` (forecasts)
  - `results/tables/workforce_gap_analysis.csv` (gaps)
  - `results/tables/workforce_scenario_analysis.csv` (scenarios)
  - `results/tables/workforce_density_benchmarks.csv` (benchmarks)
- **Config Files**: `config/analysis.yml` (dashboard styling, color schemes)

---

## ✅ Implementation Tasks

### Dashboard Architecture Setup
- [ ] Choose dashboard platform: Plotly Dash vs Databricks Dashboard (consult stakeholders)
- [ ] Set up project structure: `shared/src/visualization/workforce_dashboard/`
- [ ] Define dashboard layout: multi-page or single-page with tabs
- [ ] Configure color scheme: colorblind-friendly palette, consistent brand colors
- [ ] Set up data loading: connect to Parquet files, implement caching for performance

### Page 1: Overview Dashboard
- [ ] Create KPI cards:
  - Total workforce 2019, projected 2030
  - Overall growth rate 2006-2019
  - Critical professions count (with shortages)
  - Top 3 priority interventions
- [ ] Summary insights box: auto-generated key findings text
- [ ] Quick links to detailed pages

### Page 2: Historical Trends (2006-2019)
- [ ] Line chart: workforce count over time (faceted or filter by profession)
- [ ] Bar chart: CAGR by profession and sector
- [ ] Heatmap: YoY growth rates (year × profession)
- [ ] Filters: profession selector, sector toggle (public/private/both)
- [ ] Data table: summary statistics (growth rates, volatility)

### Page 3: Forecasts 2020-2030
- [ ] Line chart: historical (2006-2019) + forecast (2020-2030) with confidence intervals
- [ ] Shaded CI bands: 80% and 95% confidence intervals
- [ ] Profession selector: dropdown to switch between professions
- [ ] Forecast table: point estimates and CI bounds by year
- [ ] Export button: download forecast data as CSV

### Page 4: Supply-Demand Gaps
- [ ] Gap magnitude chart: bar chart showing 2025 and 2030 gaps by profession
- [ ] Gap timeline: line chart showing when each profession hits critical shortage
- [ ] Gap severity heatmap: profession × year matrix (color = severity)
- [ ] Scenario selector: baseline demand vs OECD target demand
- [ ] Priority ranking table: professions ranked by gap severity

### Page 5: Scenario Comparison
- [ ] Interactive scenario selector: checkboxes for scenarios to compare
  - Baseline (no intervention)
  - Training expansion scenarios (10%, 20%, 30%)
  - Retention improvement scenarios
  - Immigration scenarios
  - Combined optimal scenario
- [ ] Comparative line chart: overlay multiple scenarios on same chart
- [ ] Gap closure waterfall: contribution of each intervention component
- [ ] Scenario impact table: gap reduction % per scenario
- [ ] Recommendation box: suggested intervention mix per profession

### Interactivity & User Experience
- [ ] Implement filters: profession, sector, year range, scenario
- [ ] Add hover tooltips: show exact values on charts
- [ ] Enable zoom/pan on all time series charts
- [ ] Cross-filtering: selecting a profession updates all charts
- [ ] Reset button: restore default view
- [ ] Loading spinners: indicate when data is refreshing

### Export & Sharing
- [ ] Chart export: download as PNG, PDF, SVG
- [ ] Data export: download underlying data as CSV
- [ ] Share view: generate shareable link with current filter state
- [ ] Print view: optimized layout for printing reports

### Performance Optimization
- [ ] Implement data caching: avoid reloading Parquet files on every interaction
- [ ] Lazy loading: load chart data only when tab is selected
- [ ] Optimize chart rendering: use Plotly's fast rendering options
- [ ] Test with full dataset: ensure <3 second load times

### Testing & Quality Assurance
- [ ] Functional tests: validate all filters work correctly
- [ ] UI tests: test chart interactions (hover, zoom, pan)
- [ ] Cross-browser testing: Chrome, Firefox, Safari, Edge
- [ ] Mobile responsiveness: test on tablet screen sizes
- [ ] Accessibility: keyboard navigation, screen reader compatibility
- [ ] Load testing: performance with multiple concurrent users

### Documentation
- [ ] User guide: `docs/dashboard_user_guide.md`
  - Dashboard overview and navigation
  - Page-by-page feature descriptions
  - How to interpret charts and metrics
  - Export and sharing instructions
  - FAQ section
- [ ] Admin guide: `docs/dashboard_admin_guide.md`
  - Data refresh procedures
  - Configuration options
  - Troubleshooting common issues
- [ ] Code documentation: Docstrings for all dashboard components (Google style)
- [ ] README: `shared/src/visualization/workforce_dashboard/README.md`

### Deployment
- [ ] Deploy to Databricks workspace or MOH server
- [ ] Configure authentication: MOH SSO integration (if applicable)
- [ ] Set up access permissions: restrict to MOH workforce planning division
- [ ] Create user training materials: screenshots, video walkthrough
- [ ] Conduct user acceptance testing with stakeholders

---

## 📌 Notes

**Dashboard Technology Decision**:
- **Plotly Dash**: More customization, richer interactivity, requires separate hosting
- **Databricks Dashboard**: Integrated with platform, easier deployment, limited customization
- **Recommendation**: Start with Databricks Dashboard for MVP, migrate to Plotly Dash if advanced features needed

**Color Scheme (Colorblind-Friendly)**:
```python
# Define in config/analysis.yml
color_scheme:
  professions:
    doctors: "#1f77b4"  # blue
    nurses_midwives: "#ff7f0e"  # orange
    pharmacists: "#2ca02c"  # green
    dentists: "#d62728"  # red
    allied_health: "#9467bd"  # purple
  sectors:
    public: "#0072B2"  # dark blue
    private: "#D55E00"  # vermillion
  severity:
    critical: "#CC0000"
    moderate: "#FF9900"
    manageable: "#FFCC00"
    surplus: "#009900"
```

**Dashboard Layout Example (Plotly Dash)**:
```python
import dash
from dash import dcc, html
import dash_bootstrap_components as dbc
import polars as pl
import plotly.express as px

# Load data
df_historical = pl.read_parquet('shared/data/3_interim/workforce_integrated_clean.parquet')
df_forecast = pl.read_parquet('shared/data/4_processed/workforce_forecasts_2020_2030.parquet')

# Dashboard app
app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])

app.layout = dbc.Container([
    dbc.Row([
        dbc.Col(html.H1("Healthcare Workforce Planning Dashboard"), width=12)
    ]),
    dbc.Row([
        dbc.Col([
            dcc.Tabs(id="tabs", value='tab-overview', children=[
                dcc.Tab(label='Overview', value='tab-overview'),
                dcc.Tab(label='Historical Trends', value='tab-historical'),
                dcc.Tab(label='Forecasts 2020-2030', value='tab-forecasts'),
                dcc.Tab(label='Supply-Demand Gaps', value='tab-gaps'),
                dcc.Tab(label='Scenario Comparison', value='tab-scenarios'),
            ])
        ], width=12)
    ]),
    html.Div(id='tab-content')
])

# Callback for tab switching
@app.callback(
    Output('tab-content', 'children'),
    Input('tabs', 'value')
)
def render_content(tab):
    if tab == 'tab-overview':
        return html.Div([
            # Overview page content
        ])
    # ... other tabs
    
if __name__ == '__main__':
    app.run_server(debug=True)
```

**Key Metrics to Display**:
- **Overview Page**: Total workforce, CAGR, shortage professions count, priority interventions
- **Historical**: Growth rates, sector composition, trend charts
- **Forecasts**: 2025 and 2030 projections, confidence intervals, forecast accuracy (MAPE)
- **Gaps**: Shortage magnitude, shortage timing, priority ranking
- **Scenarios**: Gap closure %, intervention impact, optimal intervention mix

**User Training Plan**:
1. Live demo session with workforce planning team (1 hour)
2. Recorded video tutorial (15 minutes) covering all features
3. Quick reference guide (1-page PDF) with common tasks
4. Hands-on sandbox environment for practice

**Success Metrics for Dashboard**:
- Adoption: ≥80% of workforce planning team uses dashboard monthly
- Export usage: ≥20 chart exports per month (indicates use in presentations)
- User satisfaction: ≥4/5 rating in user survey
- Stakeholder impact: Dashboard cited in ≥3 policy documents or budget proposals within 6 months

---

## Implementation Plan

### 1. Dashboard Overview

Build interactive Plotly Dash dashboard for workforce planning. Features: historical trends, 2020-2030 forecasts, supply-demand gaps, scenario comparisons. Enable profession/sector filtering, chart exports, confidence interval toggles.

### 2. Component Analysis & Reuse Strategy

**Reuse**: All processed datasets (US-02 through US-09), visualization utilities (US-03)
**Create**: `shared/src/visualization/workforce_dashboard.py`, Dash app layout, callbacks

### 3. Key Implementation Components

**[CREATE] `shared/src/visualization/workforce_dashboard.py`**

```python
"""
Workforce Planning Dashboard
=============================

Interactive Plotly Dash dashboard for workforce analytics.

Author: Gen-E2 Team
Date: 2026-03-11
"""

import polars as pl
import plotly.express as px
import plotly.graph_objects as go
from dash import Dash, dcc, html, Input, Output, State
import dash_bootstrap_components as dbc
from pathlib import Path
from loguru import logger


# Load data
DATA_DIR = Path("shared/data")

df_historical = pl.read_parquet(DATA_DIR / "3_interim/workforce_integrated_clean.parquet")
df_forecast = pl.read_parquet(DATA_DIR / "4_processed/workforce_forecasts_2020_2030.parquet")
df_gaps = pl.read_parquet(DATA_DIR / "4_processed/workforce_gaps_2020_2030.parquet")
df_scenarios = pl.read_parquet(DATA_DIR / "4_processed/scenario_results.parquet")
df_validation = pl.read_parquet(DATA_DIR / "4_processed/forecast_validation_results.parquet")

logger.info("Loaded all dashboard datasets")

# Initialize Dash app
app = Dash(
    __name__,
    external_stylesheets=[dbc.themes.FLATLY],  # Modern Bootstrap theme
    title="Healthcare Workforce Planning"
)

# ============================================================================
# Layout Components
# ============================================================================

def create_overview_metrics() -> dbc.Row:
    """Create overview KPI cards."""
    # Calculate metrics
    total_workforce_2019 = df_historical.filter(pl.col('year') == 2019)['count'].sum()
    total_forecast_2030 = df_forecast.filter(pl.col('year') == 2030)['forecast'].sum()
    
    critical_shortages = df_gaps.filter(
        (pl.col('year') == 2030) &
        (pl.col('severity_category') == 'critical')
    ).height
    
    avg_confidence = df_validation['confidence_rating'].value_counts()
    
    return dbc.Row([
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H4(f"{total_workforce_2019:,.0f}", className="card-title"),
                    html.P("Total Workforce (2019)", className="card-text")
                ])
            ], color="primary", inverse=True)
        ], width=3),
        
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H4(f"{total_forecast_2030:,.0f}", className="card-title"),
                    html.P("Forecasted 2030", className="card-text")
                ])
            ], color="info", inverse=True)
        ], width=3),
        
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H4(f"{critical_shortages}", className="card-title"),
                    html.P("Critical Shortages (2030)", className="card-text")
                ])
            ], color="danger", inverse=True)
        ], width=3),
        
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H4("See Report", className="card-title"),
                    html.P("Forecast Confidence", className="card-text")
                ])
            ], color="success", inverse=True)
        ], width=3)
    ], className="mb-4")


def create_filters() -> dbc.Row:
    """Create filter controls."""
    professions = df_historical['profession'].unique().sort().to_list()
    sectors = df_historical['sector'].unique().to_list()
    
    return dbc.Row([
        dbc.Col([
            html.Label("Profession", className="fw-bold"),
            dcc.Dropdown(
                id='profession-filter',
                options=[{'label': p.title(), 'value': p} for p in professions],
                value=professions[0],
                clearable=False
            )
        ], width=4),
        
        dbc.Col([
            html.Label("Sector", className="fw-bold"),
            dcc.Dropdown(
                id='sector-filter',
                options=[{'label': s.title(), 'value': s} for s in sectors],
                value='total',
                clearable=False
            )
        ], width=4),
        
        dbc.Col([
            html.Label("Options", className="fw-bold"),
            dbc.Checklist(
                id='chart-options',
                options=[
                    {'label': 'Show Confidence Intervals', 'value': 'ci'},
                    {'label': 'Show Grid', 'value': 'grid'}
                ],
                value=['ci', 'grid'],
                inline=True
            )
        ], width=4)
    ], className="mb-4")


# ============================================================================
# App Layout
# ============================================================================

app.layout = dbc.Container([
    # Header
    dbc.Row([
        dbc.Col([
            html.H1("Healthcare Workforce Planning Dashboard", className="text-primary mt-4"),
            html.P("Singapore 2006-2019 Historical | 2020-2030 Forecasts", className="lead")
        ], width=12)
    ]),
    
    html.Hr(),
    
    # Overview metrics
    create_overview_metrics(),
    
    # Tabs
    dbc.Row([
        dbc.Col([
            dcc.Tabs(id="tabs", value='tab-historical', children=[
                dcc.Tab(label='📊 Historical Trends', value='tab-historical'),
                dcc.Tab(label='🔮 Forecasts 2020-2030', value='tab-forecasts'),
                dcc.Tab(label='⚠️ Supply-Demand Gaps', value='tab-gaps'),
                dcc.Tab(label='🎯 Scenario Comparison', value='tab-scenarios'),
                dcc.Tab(label='✅ Validation Report', value='tab-validation')
            ])
        ], width=12)
    ]),
    
    html.Div(id='tab-content', className="mt-4")
    
], fluid=True)


# ============================================================================
# Callbacks
# ============================================================================

@app.callback(
    Output('tab-content', 'children'),
    [Input('tabs', 'value'),
     Input('profession-filter', 'value'),
     Input('sector-filter', 'value'),
     Input('chart-options', 'value')]
)
def render_tab_content(tab, profession, sector, options):
    """Render content for selected tab."""
    
    show_ci = 'ci' in options
    show_grid = 'grid' in options
    
    if tab == 'tab-historical':
        return render_historical_tab(profession, sector, show_grid)
    
    elif tab == 'tab-forecasts':
        return render_forecast_tab(profession, sector, show_ci, show_grid)
    
    elif tab == 'tab-gaps':
        return render_gaps_tab(profession, sector, show_grid)
    
    elif tab == 'tab-scenarios':
        return render_scenarios_tab(profession, sector, show_grid)
    
    elif tab == 'tab-validation':
        return render_validation_tab()


def render_historical_tab(profession: str, sector: str, show_grid: bool) -> html.Div:
    """Render historical trends tab."""
    
    # Filter data
    df_filtered = df_historical.filter(
        (pl.col('profession') == profession) &
        (pl.col('sector') == sector)
    ).sort('year')
    
    # Create line chart
    fig = px.line(
        df_filtered.to_pandas(),
        x='year',
        y='count',
        title=f"{profession.title()} Workforce Trend ({sector})",
        labels={'count': 'Workforce Count', 'year': 'Year'},
        markers=True
    )
    
    fig.update_layout(
        template='plotly_white',
        showlegend=False,
        xaxis=dict(showgrid=show_grid),
        yaxis=dict(showgrid=show_grid)
    )
    
    # Calculate CAGR
    first_year = df_filtered['year'].min()
    last_year = df_filtered['year'].max()
    first_count = df_filtered.filter(pl.col('year') == first_year)['count'][0]
    last_count = df_filtered.filter(pl.col('year') == last_year)['count'][0]
    cagr = ((last_count / first_count) ** (1 / (last_year - first_year)) - 1) * 100
    
    return html.Div([
        create_filters(),
        dbc.Row([
            dbc.Col([
                dcc.Graph(figure=fig)
            ], width=8),
            dbc.Col([
                dbc.Card([
                    dbc.CardBody([
                        html.H5("Trend Summary", className="card-title"),
                        html.P(f"CAGR ({first_year}-{last_year}): {cagr:.2f}%"),
                        html.P(f"Total Growth: {((last_count/first_count - 1) * 100):.1f}%"),
                        html.P(f"Starting ({first_year}): {first_count:,.0f}"),
                        html.P(f"Ending ({last_year}): {last_count:,.0f}")
                    ])
                ])
            ], width=4)
        ])
    ])


def render_forecast_tab(profession: str, sector: str, show_ci: bool, show_grid: bool) -> html.Div:
    """Render forecast tab with historical + forecast."""
    
    # Historical data
    df_hist = df_historical.filter(
        (pl.col('profession') == profession) &
        (pl.col('sector') == sector)
    ).sort('year').select(['year', 'count']).rename({'count': 'value'})
    
    df_hist = df_hist.with_columns([pl.lit('Historical').alias('series')])
    
    # Forecast data
    df_fcst = df_forecast.filter(
        (pl.col('profession') == profession) &
        (pl.col('sector') == sector)
    ).sort('year').select(['year', 'forecast', 'lower_95', 'upper_95']).rename({'forecast': 'value'})
    
    df_fcst = df_fcst.with_columns([pl.lit('Forecast').alias('series')])
    
    # Combine
    df_combined = pl.concat([
        df_hist.select(['year', 'value', 'series']),
        df_fcst.select(['year', 'value', 'series'])
    ])
    
    # Create figure
    fig = go.Figure()
    
    # Historical line
    hist_data = df_combined.filter(pl.col('series') == 'Historical')
    fig.add_trace(go.Scatter(
        x=hist_data['year'].to_list(),
        y=hist_data['value'].to_list(),
        mode='lines+markers',
        name='Historical',
        line=dict(color='#2E86AB', width=2)
    ))
    
    # Forecast line
    fcst_data = df_combined.filter(pl.col('series') == 'Forecast')
    fig.add_trace(go.Scatter(
        x=fcst_data['year'].to_list(),
        y=fcst_data['value'].to_list(),
        mode='lines+markers',
        name='Forecast',
        line=dict(color='#A23B72', width=2, dash='dash')
    ))
    
    # Confidence intervals
    if show_ci:
        fig.add_trace(go.Scatter(
            x=df_fcst['year'].to_list() + df_fcst['year'].reverse().to_list(),
            y=df_fcst['upper_95'].to_list() + df_fcst['lower_95'].reverse().to_list(),
            fill='toself',
            fillcolor='rgba(162, 59, 114, 0.2)',
            line=dict(color='rgba(255,255,255,0)'),
            name='95% CI',
            showlegend=True
        ))
    
    fig.update_layout(
        title=f"{profession.title()} Forecast 2020-2030 ({sector})",
        xaxis_title="Year",
        yaxis_title="Workforce Count",
        template='plotly_white',
        xaxis=dict(showgrid=show_grid),
        yaxis=dict(showgrid=show_grid)
    )
    
    return html.Div([
        create_filters(),
        dcc.Graph(figure=fig)
    ])


def render_gaps_tab(profession: str, sector: str, show_grid: bool) -> html.Div:
    """Render supply-demand gap analysis tab."""
    
    df_gap = df_gaps.filter(
        (pl.col('profession') == profession) &
        (pl.col('sector') == sector)
    ).sort('year')
    
    # Create gap chart
    fig = go.Figure()
    
    fig.add_trace(go.Scatter(
        x=df_gap['year'].to_list(),
        y=df_gap['supply_forecast'].to_list(),
        mode='lines+markers',
        name='Supply (Forecast)',
        line=dict(color='#2E86AB')
    ))
    
    fig.add_trace(go.Scatter(
        x=df_gap['year'].to_list(),
        y=df_gap['demand_baseline'].to_list(),
        mode='lines+markers',
        name='Demand (Baseline)',
        line=dict(color='#F18F01')
    ))
    
    fig.update_layout(
        title=f"{profession.title()} Supply vs Demand Gap ({sector})",
        xaxis_title="Year",
        yaxis_title="Workforce Count",
        template='plotly_white',
        xaxis=dict(showgrid=show_grid),
        yaxis=dict(showgrid=show_grid)
    )
    
    # Gap severity by year
    gap_severity = df_gap.groupby('severity_category').agg(pl.count()).sort('count', descending=True)
    
    return html.Div([
        create_filters(),
        dbc.Row([
            dbc.Col([
                dcc.Graph(figure=fig)
            ], width=8),
            dbc.Col([
                dbc.Card([
                    dbc.CardBody([
                        html.H5("Gap Summary", className="card-title"),
                        html.P(f"2030 Gap: {df_gap.filter(pl.col('year')==2030)['gap_baseline'][0]:,.0f}"),
                        html.P(f"2030 Gap %: {df_gap.filter(pl.col('year')==2030)['gap_pct_baseline'][0]:.1f}%"),
                        html.P(f"Severity: {df_gap.filter(pl.col('year')==2030)['severity_category'][0].upper()}")
                    ])
                ])
            ], width=4)
        ])
    ])


def render_scenarios_tab(profession: str, sector: str, show_grid: bool) -> html.Div:
    """Render scenario comparison tab."""
    
    df_scen = df_scenarios.filter(
        (pl.col('profession') == profession) &
        (pl.col('sector') == sector)
    )
    
    # Create scenario comparison chart
    fig = px.line(
        df_scen.to_pandas(),
        x='year',
        y='scenario_supply',
        color='scenario_name',
        title=f"{profession.title()} Scenario Comparison ({sector})",
        labels={'scenario_supply': 'Workforce Count', 'year': 'Year', 'scenario_name': 'Scenario'}
    )
    
    fig.update_layout(
        template='plotly_white',
        xaxis=dict(showgrid=show_grid),
        yaxis=dict(showgrid=show_grid)
    )
    
    return html.Div([
        create_filters(),
        dcc.Graph(figure=fig)
    ])


def render_validation_tab() -> html.Div:
    """Render forecast validation report."""
    
    # Create validation table
    table_header = [
        html.Thead(html.Tr([
            html.Th("Profession"),
            html.Th("MAPE"),
            html.Th("RMSE"),
            html.Th("Validation"),
            html.Th("Confidence")
        ]))
    ]
    
    rows = []
    for row in df_validation.iter_rows(named=True):
        rows.append(html.Tr([
            html.Td(row['profession'].title()),
            html.Td(f"{row['mape']:.2f}%"),
            html.Td(f"{row['rmse']:.1f}"),
            html.Td("✅" if row['validation_pass'] else "❌"),
            html.Td(html.Span(
                row['confidence_rating'].upper(),
                className=f"badge bg-{'success' if row['confidence_rating']=='high' else 'warning' if row['confidence_rating']=='moderate' else 'danger'}"
            ))
        ]))
    
    table_body = [html.Tbody(rows)]
    
    return html.Div([
        html.H3("Forecast Validation Report"),
        dbc.Table(table_header + table_body, bordered=True, hover=True, responsive=True, striped=True),
        html.P("✅ Validation Pass: MAPE < 20%, statistical tests pass", className="text-success"),
        html.P("❌ Validation Fail: MAPE ≥ 20% or test failures", className="text-danger")
    ])


# ============================================================================
# Run App
# ============================================================================

if __name__ == '__main__':
    logger.info("Starting Workforce Planning Dashboard...")
    app.run_server(debug=True, host='0.0.0.0', port=8050)
```

### 4. Dashboard Configuration

**[CREATE] `shared/config/dashboard_config.yml`**

```yaml
# Dashboard Configuration
# =========================

app:
  title: "Healthcare Workforce Planning Dashboard"
  theme: "flatly"  # Bootstrap theme
  port: 8050
  debug: true

colors:
  primary: "#2E86AB"
  secondary: "#A23B72"
  success: "#06A77D"
  warning: "#F18F01"
  danger: "#D62246"

charts:
  default_height: 500
  default_template: "plotly_white"
  show_grid: true
  show_legend: true

data_sources:
  historical: "shared/data/3_interim/workforce_integrated_clean.parquet"
  forecasts: "shared/data/4_processed/workforce_forecasts_2020_2030.parquet"
  gaps: "shared/data/4_processed/workforce_gaps_2020_2030.parquet"
  scenarios: "shared/data/4_processed/scenario_results.parquet"
  validation: "shared/data/4_processed/forecast_validation_results.parquet"
```

### 5. Implementation Steps

**Phase 1: Dashboard Infrastructure (Days 1-2)**
- [ ] Set up Plotly Dash app with Bootstrap theme
- [ ] Create layout structure (header, filters, tabs)
- [ ] Load all required datasets (historical, forecasts, gaps, scenarios)
- [ ] Create filter controls (profession, sector, options)

**Phase 2: Tab Development (Days 2-4)**
- [ ] Historical Trends tab: Line charts, CAGR calculations
- [ ] Forecasts tab: Historical + forecast with CI ribbons
- [ ] Supply-Demand Gaps tab: Supply vs demand comparison
- [ ] Scenario Comparison tab: Multi-scenario overlay
- [ ] Validation Report tab: Metrics table, confidence ratings

**Phase 3: Interactivity & Polish (Days 4-5)**
- [ ] Implement all callbacks (tab switching, filter updates)
- [ ] Add chart export functionality
- [ ] Add tooltip enhancements
- [ ] Optimize dashboard performance (data caching)

**Phase 4: Testing & Deployment (Days 5-6)**
- [ ] Test all interactions and filters
- [ ] Test across different browsers and screen sizes
- [ ] Create user documentation
- [ ] Deploy to Databricks or cloud hosting
- [ ] Conduct user training session

### 6. Success Metrics

- ✅ Dashboard loads all datasets without errors
- ✅ All 5 tabs functional with correct visualizations
- ✅ Filters work correctly (profession, sector)
- ✅ Charts exportable as PNG/PDF
- ✅ User training completed with ≥4/5 satisfaction rating
- ✅ Dashboard accessible to all stakeholders (MOH workforce planning team)

### 7. User Training Plan

1. **Live Demo Session** (1 hour)
   - Walkthrough of all dashboard features
   - Q&A with workforce planning team
   - Hands-on practice exercises

2. **Video Tutorial** (15 minutes)
   - Recorded screen capture covering all tabs
   - Common use cases demonstrated
   - Troubleshooting tips

3. **Quick Reference Guide** (1-page PDF)
   - Filter usage
   - Chart interpretation
   - Export instructions

### 8. References

- [Plotly Dash Documentation](https://dash.plotly.com/)
- [Dash Bootstrap Components](https://dash-bootstrap-components.opensource.faculty.ai/)
- US-02 through US-09 (all data sources)

---

**Implementation Plan Complete for US-10**  
**Dependencies**: US-02 through US-09 (all datasets)  
**Completes**: Healthcare Workforce Sustainability Epic (PS-001)
