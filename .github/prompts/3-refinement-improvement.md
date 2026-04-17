---
description: Data Analysis Project Refinements and Improvements
model: claude-sonnet-4.5
stage: Refinement & Improvement
---

# Tasks

As a senior data analyst, your task is to refine and improve an existing delivered analytics implementation based on stakeholder feedback, follow-up requests, defects, or findings from prior analysis.

This refinement flow is for work that already belongs to an existing problem statement and its existing delivery folder. Reuse and improve the current implementation instead of creating a new problem-statement delivery unless the user explicitly asks for a new problem statement.

## 1. Identify the target problem statement first

Before doing anything else, analyze the user input and determine whether a problem statement is already identified.

Accepted identifiers include any of the following:
- Problem statement number and slug, such as `ps-001-seasonal-pattern-forecasting`
- A direct path to `docs/objectives/problem_statements/ps-{num}-{name}.md`
- A direct path to `artifacts/ps-{num}-{name}/`
- Clear natural-language references that uniquely match one existing problem statement

### Mandatory rule

If no problem statement can be identified with confidence, stop and ask the user which problem statement the request belongs to.

Do **not** guess.

## 2. Use the existing refinement scope

Once the problem statement is identified, select the matching existing folder:

`artifacts/ps-{num}-{name}/`

All refinement outputs must stay inside that existing problem-statement folder. Only create missing subfolders inside it when required by the updated implementation.

## 3. Mandatory inputs for refinement

For every refinement request, gather and use all of the following inputs before selecting the execution stage:

1. **User request / stakeholder feedback**
2. **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
3. **Current artifact folder**: `artifacts/ps-{num}-{name}/`
4. **Latest previous agent handoffs** for the same problem statement from:
    - `docs/agent-handoffs/data-extractor/ps-{num}-{name}/`
	- `docs/agent-handoffs/data-validation/ps-{num}-{name}/`
	- `docs/agent-handoffs/data-cleaning/ps-{num}-{name}/`
	- `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/`
	- `docs/agent-handoffs/feature-engineering/ps-{num}-{name}/`
	- `docs/agent-handoffs/model-forecasting/ps-{num}-{name}/`
	- `docs/agent-handoffs/narrative-compiler/ps-{num}-{name}/`
	- `docs/agent-handoffs/dashboard-visualization/ps-{num}-{name}/`
5. **Relevant user stories** in `docs/objectives/user_stories/problem-statement-{num}-{name}/`
6. **Relevant domain knowledge** in `docs/domain-knowledge/`
7. **Execution flow reference**: analyze [2-execution-stage.md](./2-execution-stage.md) to understand the purpose, ordering, validation gates, and dependencies of each phase

When multiple handoff files exist for the same stage, always use the latest successful handoff.

## 4. Determine the correct stage to start from

Use the user request, the current artifact contents, and the latest previous handoffs to determine the **earliest impacted stage**.

### Stage routing guide

| Earliest impacted need | Start from phase | Primary subagent(s) |
|---|---|---|
| Source system changes, extraction bug, missing raw data, changed source schema | Phase 1 | `data-extractor` |
| Validation rules, schema conformance, data quality issues, cleaning logic, standardization fixes | Phase 2 | `data-validation`, `data-cleaning` |
| EDA refresh, new metrics, segmentation logic, derived columns, feature creation | Phase 3 | `exploratory-analysis`, `feature-engineering` |
| Forecast logic, model tuning, retraining, prediction outputs | Phase 4 | `model-forecasting` |
| Narrative rewrite, report conclusions, executive summary, interpretation updates | Phase 5 | `narrative-compiler` |
| Dashboard UI, charts, filters, layout, interaction, dashboard-specific metric presentation | Phase 6 | `dashboard-visualization` |

### Dependency rule

Always start from the **earliest stage impacted by the requested change**.

Then apply both of these rules:

1. **Upstream prerequisite rule**: if the selected start stage depends on earlier-stage handoffs that are missing, failed, stale, or incompatible with the new request, re-run the necessary upstream stage(s) first.
2. **Downstream consistency rule**: after the chosen start stage is completed, continue through all downstream dependent stages so the final artifacts, reports, and dashboards remain consistent with the updated outputs.

In short: choose the earliest impacted stage, repair any broken prerequisites, then continue forward using the same phase order as [2-execution-stage.md](./2-execution-stage.md).

## 5. Refinement execution flow

After selecting the start phase, trigger only the required phases, but follow the exact dependency pattern and verification discipline from [2-execution-stage.md](./2-execution-stage.md).

### Required behavior

- Use `runSubagent` for every agent invocation; never inline the agent work in the main conversation
- Reuse the selected `artifacts/ps-{num}-{name}/` folder rather than generating a separate delivery tree
- Pass the problem statement path to every subagent call
- Pass the user request and the current artifact folder to every subagent call
- Pass the latest relevant handoff JSON paths to every downstream subagent call
- Preserve the same phase ordering, parallelization rules, and quality gates defined in [2-execution-stage.md](./2-execution-stage.md)

## 6. Phase trigger rules for refinement

Use these refinement trigger rules:

### If Phase 1 is selected

Trigger, in order:
1. `data-extractor`
2. `code-quality`
3. `code-reviewer`
4. Phase 2 onward using the same flow as [2-execution-stage.md](./2-execution-stage.md)

### If Phase 2 is selected

Verify Phase 1 handoff suitability first. If unsuitable, rerun Phase 1.

Then trigger:
1. `data-validation` and `data-cleaning` in parallel where applicable
2. `code-quality`
3. `code-reviewer`
4. Phase 3 onward using the same flow as [2-execution-stage.md](./2-execution-stage.md)

### If Phase 3 is selected

Verify Phases 1-2 handoffs first. If unsuitable, rerun the required upstream phases.

Then trigger:
1. `exploratory-analysis` and `feature-engineering` in parallel where applicable
2. `code-quality`
3. `code-reviewer`
4. Phase 4 onward if applicable, then Phases 5 and 6

### If Phase 4 is selected

Verify Phase 3 feature outputs first. If unsuitable, rerun the required upstream phases.

Then trigger:
1. `model-forecasting`
2. `code-quality`
3. `code-reviewer`
4. Phases 5 and 6 if those deliverables depend on forecast outputs

### If Phase 5 is selected

Verify all prior results needed for reporting. If unsuitable, rerun the required upstream phases.

Then trigger:
1. `narrative-compiler`
2. If the dashboard depends on updated narrative-driven metrics or messaging, also trigger Phase 6

### If Phase 6 is selected

Verify narrative and analytical inputs first. If unsuitable, rerun the required upstream phases.

Then trigger:
1. `dashboard-visualization`
2. `code-quality`
3. `code-reviewer`

## 7. Handoff verification is mandatory after every triggered agent

After every `runSubagent` call, verify completion before moving on.

Check all of the following:

| Check | Requirement |
|---|---|
| Handoff JSON exists | Present in the expected `docs/agent-handoffs/{agent}/ps-{num}-{name}/` folder |
| Handoff status | `status` must be `success` |
| Claimed outputs exist | Every path in `outputs` exists |
| Notebooks are valid | Any `.ipynb` is non-empty, larger than 1 KB, and contains executed cells with output |
| Data files are valid | Any CSV/Parquet is non-empty and has at least 1 data row |
| Scripts are runnable | Produced scripts pass code-quality and code-review checks |

If any verification fails, re-run the failed subagent immediately with explicit fix instructions.

## 8. Data-analysis best practices are mandatory

Ensure all refinement work adheres to project and analytics best practices:

- Revalidate data assumptions whenever upstream data, schema, or filters change
- Prefer modifying the minimum necessary scope, but do not leave downstream deliverables inconsistent
- Preserve reproducibility across notebooks, scripts, and result files
- Keep notebook cell order correct: constants/paths -> data loading -> helper functions -> main logic -> outputs
- Empty notebooks are failed delivery
- Document any changed methodology, assumptions, caveats, and limitations
- Keep outputs inside the correct artifact subfolders
- Update dependency files if new packages are introduced
- Ensure results remain stakeholder-ready and internally consistent across tables, metrics, narrative, and dashboard views

## 9. Completion summary

After all required refinement phases finish, present a concise delivery report:

```markdown
## Refinement Delivery Report: [problem statement title]

### Start Phase Chosen
[Phase number and reason]

### Inputs Reviewed
- User request
- Current artifact folder
- Latest handoffs used

### Artifacts Updated
[List notebooks, scripts, datasets, results, reports, dashboard files]

### Files Changed
[List created/modified files]

### Quality Checks
[Summary of validation, code-quality, and code-review outcomes]

### Status
READY FOR REVIEW | BLOCKED
```

## Rules

1. Never start refinement before identifying the problem statement.
2. If no problem statement is identified, ask the user which one to use.
3. Always refine the existing `artifacts/ps-{num}-{name}/` folder for that problem statement.
4. Always use the latest successful handoffs as baseline context.
5. Choose the earliest impacted phase, not the latest visible symptom.
6. If prerequisites are missing or stale, rerun the necessary upstream phases first.
7. After the selected start phase, continue through downstream dependent phases as needed for consistency.
8. Follow the same agent sequencing, validation gates, and quality checks defined in [2-execution-stage.md](./2-execution-stage.md).
9. `runSubagent` is mandatory for every triggered agent.
10. Best-practice compliance is required for every refinement delivery.