# User Story: 9 - Interactive Disease Burden Forecast Dashboard

**As a** policy maker,  
**I want** an interactive web dashboard showing mortality forecasts and scenarios,  
**so that** I can explore projections dynamically and communicate insights to stakeholders.

## 1. 🎯 Acceptance Criteria

1. **Multi-Disease Forecast Visualization** - Line charts with confidence bands for all 3 diseases
2. **Scenario Comparison View** - Toggle between baseline/optimistic/pessimistic scenarios
3. **Capacity Planning Calculator** - Translate mortality forecasts to capacity needs interactively
4. **Actual-vs-Forecast Tracker** - Display historical accuracy (will update as new data arrives)
5. **Export Functionality** - Download charts and data tables for presentations

## 2. 🔒 Technical Constraints

- **Dashboard platform**: Plotly Dash (Python-based, integrates with Databricks)
- **Deployment**: Databricks Dashboard or standalone Dash app
- **Accessibility**: Responsive design, accessible to MOH strategic planning division

## 3. 📚 Domain Knowledge References

- Visualization best practices: Uncertainty communication (show prediction intervals, not just point forecasts)
- Dashboard design principles: Clarity, interactivity, export capabilities

## 4. 📦 Dependencies

**External Packages**:
- `plotly>=5.14.0` - Interactive visualizations
- `dash>=2.14.0` - Web dashboard framework

**Input Data**:
- Forecasts from User Story 6
- Scenarios from User Story 7
- Capacity requirements from User Story 8

## 5. ✅ Implementation Tasks

### Dashboard Development
- ⬜ **Setup Dash app structure** (layout, callbacks)
- ⬜ **Create forecast visualization component** (line chart with prediction intervals)
- ⬜ **Implement scenario selector** (dropdown: baseline/optimistic/pessimistic)
- ⬜ **Build capacity calculator widget** (input: year/disease, output: bed/specialist needs)

### Interactivity
- ⬜ **Add disease filter** (select cancer/stroke/heart disease)
- ⬜ **Implement zoom/pan controls** for time series charts
- ⬜ **Create export buttons** (download PNG, CSV)

### Deployment
- ⬜ **Deploy to Databricks Dashboard** or standalone server
- ⬜ **Configure access permissions** (MOH users only)
- ⬜ **Test cross-browser compatibility**

## 6. Notes

**Dashboard Features Priority**:
- **Must-have**: Forecast visualization, scenario comparison, export
- **Nice-to-have**: Capacity calculator, sensitivity sliders

**Stakeholder Engagement**:
- Demo dashboard to key users (Healthcare Planning Division, Finance Division)
- Gather feedback and iterate

**Next Steps**:
- Dashboard updated annually when new mortality data released
- Actual-vs-forecast tracker populated in User Story 10

---

## Implementation Plan

### 1. Feature Overview

Build interactive web dashboard showing mortality forecasts, scenarios, and capacity planning calculator using Plotly Dash.

### 2. Component Analysis & Reuse Strategy

**Input**: All outputs from US-06, US-07, US-08

**New**: `problem-statements/ps-006-forecasting/dashboard/app.py`

### 4-6. Code Specifications

```python
# problem-statements/ps-006-forecasting/dashboard/app.py

import plotly.graph_objects as go
import polars as pl
from dash import Dash, dcc, html, Input, Output
from pathlib import Path

app = Dash(__name__)

# Load data
forecasts = pl.read_parquet('shared/data/4_processed/disease_burden_forecasts_2020_2025.parquet')
scenarios = pl.read_parquet('shared/data/4_processed/disease_burden_scenarios_2020_2025.parquet')

app.layout = html.Div([
    html.H1('Singapore Disease Burden Forecasts 2020-2025'),
    
    dcc.Dropdown(
        id='disease-selector',
        options=[{'label': d.title(), 'value': d} for d in ['cancer', 'stroke', 'heart_disease']],
        value='cancer'
    ),
    
    dcc.Dropdown(
        id='scenario-selector',
        options=[{'label': s.title(), 'value': s} for s in ['baseline', 'optimistic', 'pessimistic']],
        value='baseline'
    ),
    
    dcc.Graph(id='forecast-chart'),
    
    html.Div(id='capacity-calculator')
])

@app.callback(
    Output('forecast-chart', 'figure'),
    [Input('disease-selector', 'value'),
     Input('scenario-selector', 'value')]
)
def update_chart(disease, scenario):
    data = scenarios.filter(
        (pl.col('disease') == disease) &
        (pl.col('scenario') == scenario)
    ).to_pandas()
    
    fig = go.Figure()
    
    # Forecast line
    fig.add_trace(go.Scatter(
        x=data['year'],
        y=data['forecast_mean'],
        mode='lines+markers',
        name='Forecast'
    ))
    
    # Confidence interval
    fig.add_trace(go.Scatter(
        x=data['year'].tolist() + data['year'].tolist()[::-1],
        y=data['upper_95'].tolist() + data['lower_95'].tolist()[::-1],
        fill='toself',
        fillcolor='rgba(0,100,200,0.2)',
        line=dict(color='rgba(255,255,255,0)'),
        name='95% CI'
    ))
    
    fig.update_layout(
        title=f'{disease.title()} Mortality Forecast ({scenario.title()} Scenario)',
        xaxis_title='Year',
        yaxis_title='Age-Standardized Mortality Rate (per 100k)'
    )
    
    return fig

if __name__ == '__main__':
    app.run_server(debug=True, port=8050)
```

### 7. Implementation Steps

- [ ] Setup Dash app structure
- [ ] Create forecast visualization component (line + CI bands)
- [ ] Implement disease and scenario selectors
- [ ] Build capacity calculator widget
- [ ] Add export functionality (download PNG, CSV)
- [ ] Deploy to Databricks Dashboard or standalone server
- [ ] Test cross-browser compatibility

✅ **PLAN READY**
