# Prioritize Vulnerable Demographic Populations (Lifecycle Stage: Feature Engineering & Insights)

**Story ID**: PS-005-US-08  
**Epic**: Healthcare Access Equity & Demographic Disparities Analysis  
**Priority**: P0 (Critical)  
**Effort Estimate**: M (4 days)  
**Created**: March 11, 2026

---

## 📝 User Story Description

As a **Population Health Policy Director allocating equity program resources**,  
I want **to prioritize vulnerable demographic populations based on disparity magnitude, outcome impact, and intervention feasibility**,  
So that **I can target limited resources toward populations with greatest equity gaps and potential for improvement**.

---

## 🎯 Acceptance Criteria

1. **Vulnerability scoring framework**
   - Disparity magnitude score
   - Outcome impact score (absolute number affected)
   - Trend urgency score (widening vs narrowing disparities)
   - Intervention feasibility score

2. **Population prioritization**
   - Composite vulnerability score
   - Priority ranking: top vulnerable populations
   - Priority quadrants: high disparity + high impact
   - Quick wins identified

3. **Resource allocation recommendations**
   - Investment priorities by demographic
   - Intervention type recommendations
   - Estimated impact per population

4. **Deliverables**
   - Output: `results/tables/vulnerable_population_priorities.csv`
   - Figures: Priority matrix, vulnerability scores
   - Policy brief: Equity intervention priorities

---

## 🔒 Technical Constraints

- **Platform**: Databricks Runtime 13.3.x, Python 3.9
- **Primary Library**: Polars 0.20+
- **Visualization**: Matplotlib
- **Logging**: loguru
- **Testing**: pytest ≥80% coverage

---

## 📚 Domain Knowledge References

- [Domain Knowledge Research](../../../problem_statements/DOMAIN_KNOWLEDGE_RESEARCH.md#equity-prioritization)
- [Problem Statement PS-005](../../../problem_statements/ps-005-healthcare-equity-disparities.md#objective-5)

---

## 📦 Dependencies

### External Packages
- `polars>=0.20.0`, `scikit-learn>=1.3.0`, `matplotlib>=3.8.0`, `loguru>=0.7.0`

### Internal Dependencies
- **Upstream**: PS-005-US-03 through PS-005-US-07 (All equity analyses - BLOCKING)
- **Data Sources**: All PS-005 analysis outputs

---

## ✅ Implementation Tasks

### Scoring Framework
- [ ] Disparity magnitude: normalize rate ratios
- [ ] Outcome impact: affected population size
- [ ] Trend urgency: worsening disparities scored higher
- [ ] Feasibility: addressability of barriers

### Composite Scoring
- [ ] Weighted composite: 0.3×disparity + 0.3×impact + 0.2×urgency + 0.2×feasibility
- [ ] Rank populations by composite score
- [ ] Identify top 5-10 priority populations

### Priority Quadrants
- [ ] High disparity + high impact: URGENT
- [ ] High disparity + low impact: Targeted programs
- [ ] Low disparity + high impact: Maintenance
- [ ] Low disparity + low impact: Low priority

### Recommendations
- [ ] Intervention priorities
- [ ] Resource allocation suggestions
- [ ] Implementation roadmap

### Visualization
- [ ] Priority matrix scatter plot
- [ ] Vulnerability score charts
- [ ] Save figures

### Policy Brief
- [ ] Executive summary: top priorities
- [ ] Population profiles
- [ ] Recommendations

### Testing & Documentation
- [ ] Validate scoring logic
- [ ] Docstrings
- [ ] Policy brief

---

## 📌 Notes

**Composite Scoring Example**:
```python
priority_score = (
    0.3 * disparity_score +      # How large is the gap?
    0.3 * impact_score +          # How many people affected?
    0.2 * trend_urgency_score +   # Is it getting worse?
    0.2 * feasibility_score       # Can we address it?
)
```

**Expected Priorities**:
- Elderly populations (large disparities, high impact)
- Lower socioeconomic groups (if data shows disparities)
- Specific age-sex combinations with access barriers
