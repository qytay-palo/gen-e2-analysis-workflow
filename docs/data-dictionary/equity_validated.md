# Equity Datasets -- Validated Data Dictionary
## PS-005: Healthcare Access Equity & Demographic Disparities Analysis

*Extends [`equity_extracted.md`](equity_extracted.md) with quality findings and preprocessing guidance.*

---

## Data Quality Assessment

**Validation Date**: 2026-04-10
**Overall Quality Score**: 98/100

| Dimension    | Score  | Status |
|--------------|--------|--------|
| Completeness | 100%  | PASS |
| Accuracy     | 97%  | PASS |
| Consistency  | 96%  | PASS |
| Validity     | 100%  | PASS |
| Timeliness   | 100% | PASS |

---

### Known Issues

1. **LOW** -- `age_group` (Consistency): hospital_admissions: 9 age groups use space-padded separators (e.g. '0 - 4'). Recommended action: Clean to '0-4' / '75+' format in cleaning stage
2. **INFO** -- `N/A` (Coverage): ltc_admissions has no sex stratification — limits gender equity analysis for long-term care. Recommended action: Document as known limitation; request sex-stratified LTC data for future release
3. **MEDIUM** -- `rate` (Accuracy): Rate bases differ: hospital admissions (per 1,000) vs mortality (per 100,000) — not directly comparable without normalisation. Recommended action: Normalise all rates to per-100,000 in cleaning stage

---

### Recommended Preprocessing

1. Standardise `age_group` labels: replace `" - "` with `"-"` (e.g. `"0 - 4"` to `"0-4"`, `"75 & Over"` to `"75+"`)
2. Validate `sex` values: retain only `Male` / `Female` -- strip totals or mixed-case variants
3. Normalise rate bases: convert `admission_rate_per_1000` to per 100,000 for cross-dataset comparison
4. Align temporal coverage: create 2006-2019 subset for cross-dataset joins (utilization ∩ outcomes)
5. Create composite join key: `year_sex` string column for multi-dataset equity joins
6. Calculate disparity ratios: `male_rate / female_rate` per age group per year
7. Flag temporal outliers: rows where `|rate - rolling_mean_3yr| > 3 sigma` per demographic group
8. Document LTC limitation: note sex stratification absence in all downstream reports

---

### Quality Gates

- [x] **minimum_completeness** -- threshold=90.0, actual=100.0, **PASSED**
- [x] **no_critical_issues** -- threshold=0, actual=0, **PASSED**
- [x] **business_rule_compliance** -- threshold=95.0, actual=100.0, **PASSED**
- [x] **data_freshness_utilization** -- threshold=2020, actual=2020, **PASSED**
- [x] **data_freshness_outcomes** -- threshold=2019, actual=2019, **PASSED**
- [x] **overall_quality_score** -- threshold=80.0, actual=98.45, **PASSED**

---

### Profiling Summary per Dataset

| Dataset | Rows | Cols | Null Rate | PK Dupes | Year Range | Status |
|---------|------|------|-----------|----------|------------|--------|
| hospital_admissions | 270 | 4 | 0% | 0 | 2006-2020 | PASS |
| polyclinic_attendances | 66 | 4 | 0% | 0 | 2009-2019 | PASS |
| ltc_admissions | 30 | 3 | 0% | 0 | 2006-2020 | PASS |
| mortality_cancer | 60 | 4 | 0% | 0 | 1990-2019 | PASS |
| mortality_stroke | 60 | 4 | 0% | 0 | 1990-2019 | PASS |
| mortality_heart_disease | 60 | 4 | 0% | 0 | 1990-2019 | PASS |

---

### Cross-Dataset Relationship Validation

- **Year join**: All 6 datasets joinable on `year` (integer) -- OK
- **Sex join**: hospital_admissions, polyclinic_attendances, all mortality datasets share `sex in {Male, Female}` -- OK
- **LTC limitation**: ltc_admissions has no `sex` column -- cannot participate in gender equity joins
- **Rate base heterogeneity**: Admission rates (per 1,000) vs mortality rates (per 100,000) -- normalisation required

---

**Validated by**: data-validation-agent
**Next stage**: data-cleaning-agent
**Handoff**: `docs/agent-handoffs/validation/ps-005-healthcare-equity-disparities/`