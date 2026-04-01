---
type: prompt
description: MultiAgent Orchestration Prompt for Execution of the Implementation Plan
rule_name: execute-implementation-plan
stage: Development
model: Claude Sonnet 4.5
---

# Single Task Delivery Pipeline

Deliver the task provided in `$ARGUMENTS` by orchestrating the agents below. Each agent runs via the **Task tool** as a subagent with its own context. Agents communicate through shared documents in `docs/`, not through direct context passing — always pass file paths, never large code blocks.

Before starting, gather the following context:
- **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
- **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{name}/` (Implementation details)
- **Domain Knowledge**: `docs/domain-knowledge/` — **MUST review and apply relevant guides**
- create the following directory structure if it does not exist, and place all generated code artifacts in the correct locations:

### Directory Structure (MANDATORY)

**✅ CORRECT - All artifacts in ONE problem-statement directory:**
```
problem-statement/ps-{num}-{descriptive-name}/
├── notebooks/              # ALL notebooks for this PS
├── src/                    # Problem-specific code
│   ├── data_processing/    # ETL and cleaning code
│   ├── analysis/           # Analysis modules
│   └── scripts/            # Executable scripts
├── data/                   # Problem-specific data
│   ├── 3_interim/          # Intermediate data
│   └── 4_processed/        # Final processed data
├── results/                # Analysis results
│   ├── tables/
│   └── metrics/
├── reports/                # Reports and figures
│   └── figures/
├── models/                 # Trained models
├── config/                 # Problem-specific config
├── tests/                  # Integration tests
├── logs/                   # Execution logs
└── README.md              # Execution instructions
```

ALWAYS pass the problem statement to all agents as context, and ensure they understand the specific user story being implemented. This keeps all agents aligned on the same goal.

## Phase 1: Extraction

Spawn extractor agent and execute, Wait for extractor to complete before Phase 2.

**data-extractor**:
- Subagent type: `data-extractor`
- Input: 
    1. **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{slug}/01-extract-*.md`
    3. **Data Dictionary**: `docs/data-dictionary/` (for schema definitions)
    4. **Data Sources**: `docs/project-context/data-sources.md` (for source details)
    5. **Folder Structure**: `problem-statements/ps-{num}-{name}/README.md` and `shared/README.md` 
- Expected outputs: `ocs/agent-handoffs/extraction/ps-{num}-{name}/extraction_to_{next_agent}_{timestamp}.json` + jupyter notebooks and respective extracted raw datasets

## Phase 2: Data Cleaning and Validation (parallel)

Spawn these agents in parallel using the Task tool. Wait for both to complete before Phase 3.

**data-cleaning**:
- Subagent type: `data-cleaning`
- Input: 
    1. **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{name}/` (relevant story)
    3. **Validation Handoff**: `docs/agent-handoffs/validation/ps-{num}-{name}/*`
    4. **Extractor Handoff**: `ocs/agent-handoffs/extraction/ps-{num}-{name}/*`
- Expected outputs: `ocs/agent-handoffs/extraction/ps-{num}-{name}/cleaning_to_{next_agent}_{timestamp}.json` + jupyter notebooks and cleaned datasets

**data-validation**:
- Subagent type: `data-validation`
- Input: 
  1. **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
  2. **Extractor Handoff**: `ocs/agent-handoffs/extraction/ps-{num}-{name}/*`
- Expected outputs: `ocs/agent-handoffs/extraction/ps-{num}-{name}/validation_to_{next_agent}_{timestamp}.json` + jupyter notebooks

## Phase 3: Analysis and Visualization (parallel)

Spwan these agents in parallel using the Task tool. Wait for both to complete before Phase 4.

**exploratory-analysis**
- Subagent type: `exploratory-analysis`
- Input : 
  1. problem statement: `docs/objectives/problem_statements/ps-{num}-{name}.md`
  2. **Handoff File**: `docs/agent-handoffs/data-cleaning/ps-{num}-{name}/*` & `docs/agent-handoffs/validation/ps-{num}-{name}/*`, `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/*`
  3. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{slug}/` (relevant story)
- Expected outputs: `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/exploratory_to_{next_agent}_{timestamp}.json` + jupyter notebooks

**feature-engineer**
- Subagent type: `feature-engineer`
- Input : 
  1. **Handoff File**: `docs/agent-handoffs/exploratory-analysis/*`
  2. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{slug}/{num}-engineer-**.md` (relevant story)
- Expected outputs: `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/exploratory_to_{next_agent}_{timestamp}.json` + jupyter notebooks and interim datasets with new features

# Phase 4: model-forecasting
Spawn model-forecasting agent and execute, Wait for model forecasting to complete before Phase 5.

**model-forecasting**:
- Subagent type: `model-forecasting`
- Input: 
    1. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{slug}/{num}-**-forecast-**.md`
    2. **Handoff File**: `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/*` and `docs/agent-handoffs/feature-engineering/ps-{num}-{name}/handoff_{timestamp}.md`
- Expected outputs: `docs/agent-handoffs/model-forecasting/ps-{num}-{name}/forecasting_to_{next_agent}_{timestamp}.json` + jupyter notebooks and respective extracted raw datasets

# Phase 5: dashboard-visualization
Spawn dashboard-visualization agent and execute, Wait for model forecasting to complete before Phase 6.

**dashboard-visualization**:
- Subagent type: `dashboard-visualization`
- Input: 
    1. **User Story**: `docs/objectives/user_stories/problem-statement-{num}-{slug}/{num}-**-**-dashboard.md` 
    2. **Handoff File**: `docs/agent-handoffs/exploratory-analysis/ps-{num}-{name}/*`, `docs/agent-handoffs/model-forecasting/ps-{num}-{name}/*`
- Expected outputs: `docs/agent-handoffs/dashboard-visualization/ps-{num}-{name}/dashboard_to_{next_agent}_{timestamp}.json` + dashboards scripts

# Phase 6: Quality Gate (sequential, mandatory)

Run these steps in order. Each depends on the previous.

### Step 1 — code-simplifier

- Subagent type: `code-simplifier`
- Input: review all code changes from Phase 1-5
- Purpose: clean up generated code, reduce complexity, preserve functionality

### Step 2 — code-reviewer (MANDATORY)

> **This step cannot be skipped. It is essential for ensuring all code works.**

- Subagent type: `code-reviewer`
- Input: Review all code changes from Phase 1-5, execute all code, fix ALL errors found. This is a critical step — you MUST execute all code and fix any errors before proceeding. Fix all issues immediately.

## Phase 7: Summary

After all phases complete, present a delivery report:

```
## Story Delivery Report: [title]

### Artifacts
[List all the data files, notebooks, scripts, dashboards, and other deliverables created in this implementation]

### Files Changed
[list all created/modified source files]

### Code Quality Summary:
[summary of code quality improvements, simplifications, and fixes applied during code review]

### Status: READY FOR REVIEW | BLOCKED
```

## Rules

1. **code-reviewer is mandatory** — every task must be executable without errors. No exceptions.
2. **Agents run via Task tool** — never inline agent work in the main conversation.
3. **Agents communicate through files** — pass file paths between agents, not code.
4. **Respect the dependency chain** — never start a phase before dependencies complete.