# User Story: 10 - Forecast Monitoring and Model Update System

**As an** analytics manager,  
**I want** an automated system to monitor forecast accuracy and trigger model retraining,  
**so that** forecasts remain reliable and stakeholders are alerted to significant deviations.

## 1. 🎯 Acceptance Criteria

1. **Actual-vs-Forecast Comparison** - Automated quarterly reports comparing new mortality data to forecasts
2. **Deviation Alerts** - Flag when actuals exceed 95% prediction interval (significant deviation)
3. **Forecast Accuracy Tracking** - Calculate rolling MAPE as new data arrives
4. **Model Drift Detection** - Statistical tests for forecast drift (bias accumulation)
5. **Retraining Triggers** - Recommend model update when drift detected or annually

## 2. 🔒 Technical Constraints

- **Automation**: Databricks scheduled jobs (quarterly or when new data available)
- **Alerting**: Email/Slack notifications for significant deviations
- **Logging**: Track all monitoring runs in audit logs

## 3. 📚 Domain Knowledge References

- [Time Series Forecasting Methods](../../../domain_knowledge/time-series-forecasting-methods.md) - Model validation section, forecast monitoring
- Statistical process control: Control chart concepts for forecast monitoring

## 4. 📦 Dependencies

**External Packages**:
- `smtplib` or Slack API - Alerting
- `schedule` or Databricks Jobs API - Automation

**Input**:  
- Historical forecasts (from User Story 6)
- New annual mortality data (as released by MOH)

**Output**:
- Monitoring dashboard: `reports/figures/problem-statement-006/forecast_monitoring_dashboard.html`
- Alert logs: `logs/audit/forecast_monitoring.log`

## 5. ✅ Implementation Tasks

### Monitoring Logic
- ⬜ **Create forecast-vs-actual comparison function**
- ⬜ **Calculate forecast errors** (actual - forecast)
- ⬜ **Check if actuals within prediction intervals**
- ⬜ **Detect systematic bias** (mean error significantly different from 0)

### Alerting
- ⬜ **Implement alert conditions** (actual outside 95% PI, or MAPE > threshold)
- ⬜ **Setup email/Slack notifications** to analytics team
- ⬜ **Create alert message templates** (include disease, year, severity)

### Automation
- ⬜ **Create Databricks job** for quarterly monitoring
- ⬜ **Schedule execution** (e.g., every Q1 when new data expected)
- ⬜ **Configure job failure notifications**

### Retraining Workflow
- ⬜ **Define retraining triggers** (annual, or if drift detected)
- ⬜ **Document model update procedure** (retrain with expanded data, re-validate)
- ⬜ **Archive old model versions** (model versioning)

### Deliverables
- ⬜ **Monitoring dashboard** (actual-vs-forecast tracker)
- ⬜ **Alert system operational** (tested with historical data)
- ⬜ **Retraining procedure documented** in `docs/methodology/`

## 6. Notes

**Monitoring Frequency**:
- Quarterly checks (align with typical MOH data release schedule)
- Ad-hoc checks when major events occur (e.g., pandemic, policy changes)

**Success Criteria**:
- Alerts triggered within 24 hours of new data indicating deviation
- Model retraining completed within 1 week when triggered
- Forecast accuracy reported to stakeholders quarterly

**Value**:
- **Early warning**: Detect emerging disease burden shifts before crisis
- **Stakeholder trust**: Transparent tracking of forecast performance
- **Continuous improvement**: Adaptive models that learn from new data

**Integration**:
- Monitoring results displayed in dashboard (User Story 9)
- Retraining process uses pipeline from User Stories 1-6

---

## Implementation Plan

### 1. Feature Overview

Automated system to monitor forecast accuracy, detect deviations, and trigger model retraining when new mortality data arrives.

### 2. Component Analysis & Reuse Strategy

**New**: `shared/src/models/forecast_monitoring.py`

### 4-6. Code Specifications

```python
# shared/src/models/forecast_monitoring.py

import polars as pl
import numpy as np
from loguru import logger
from datetime import datetime
from typing import Dict
import smtplib
from email.message import EmailMessage


def check_forecast_accuracy(
    actuals: pl.DataFrame,
    forecasts: pl.DataFrame
) -> Dict:
    """Compare actuals to forecasts, calculate errors.
    
    Args:
        actuals: Actual mortality data (new observations)
        forecasts: Historical forecast predictions
        
    Returns:
        Dict with accuracy metrics and alert status
    """
    logger.info("Checking forecast accuracy")
    
    # Join actuals and forecasts
    comparison = actuals.join(
        forecasts,
        on=['year', 'disease'],
        how='inner'
    )
    
    # Calculate errors
    comparison = comparison.with_columns([
        (pl.col('asmr') - pl.col('forecast_mean')).alias('error'),
        (pl.col('asmr') - pl.col('forecast_mean')).abs().alias('abs_error'),
        ((pl.col('asmr') - pl.col('forecast_mean')) / pl.col('asmr') * 100).abs().alias('ape')
    ])
    
    # Check if actuals within 95% PI
    comparison = comparison.with_columns([
        ((pl.col('asmr') >= pl.col('lower_95')) & (pl.col('asmr') <= pl.col('upper_95')))
        .alias('within_pi')
    ])
    
    # Calculate summary metrics
    n_obs = comparison.height
    within_pi_count = comparison['within_pi'].sum()
    coverage = (within_pi_count / n_obs) * 100 if n_obs > 0 else 0
    
    mape = comparison['ape'].mean()
    
    # Detect deviations
    significant_deviations = comparison.filter(~pl.col('within_pi'))
    
    result = {
        'n_observations': n_obs,
        'pi_coverage': coverage,
        'mape': mape,
        'significant_deviations': significant_deviations.height,
        'alert_triggered': (coverage < 90 or significant_deviations.height > 0),
        'timestamp': datetime.now().isoformat()
    }
    
    if result['alert_triggered']:
        logger.warning(f"⚠️ ALERT: Coverage={coverage:.1f}%, Deviations={significant_deviations.height}")
    else:
        logger.info(f"✅ Forecast performance OK: Coverage={coverage:.1f}%, MAPE={mape:.1f}%")
    
    return result


def send_alert(alert_data: Dict, recipients: list[str]) -> None:
    """Send email alert for forecast deviations."""
    
    msg = EmailMessage()
    msg['Subject'] = '⚠️ Forecast Monitoring Alert'
    msg['From'] = 'analytics@moh.gov.sg'
    msg['To'] = ', '.join(recipients)
    
    body = f"""Forecast monitoring detected significant deviations:
    
    - Observations checked: {alert_data['n_observations']}
    - Prediction interval coverage: {alert_data['pi_coverage']:.1f}% (target: 95%)
    - Significant deviations: {alert_data['significant_deviations']}
    - MAPE: {alert_data['mape']:.1f}%
    
    Action required: Review forecast models and consider retraining.
    
    Timestamp: {alert_data['timestamp']}
    """
    
    msg.set_content(body)
    
    # Send via SMTP (configure server details)
    # with smtplib.SMTP('smtp.moh.gov.sg') as server:
    #     server.send_message(msg)
    
    logger.info(f"Alert email sent to {recipients}")
```

### 7. Implementation Steps

- [ ] Create `shared/src/models/forecast_monitoring.py`
- [ ] Implement actual-vs-forecast comparison function
- [ ] Implement deviation detection (outside 95% PI)
- [ ] Setup email/Slack alerting system
- [ ] Create Databricks scheduled job (quarterly execution)
- [ ] Define retraining triggers (annual + drift-based)
- [ ] Document retraining procedure
- [ ] Test monitoring with historical data

✅ **PLAN READY**
