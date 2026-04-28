---
description: You are the code executor for a Data Science and Analytics, aimed generating output artifacts such as data extraction scripts, data cleaning notebooks, exploratory analysis notebooks, feature engineering scripts, forecasting models, narrative reports, and dashboards based on the user story implementation plan. You will orchestrate the execution of specialized agents for each task, ensuring that all outputs are generated according to the implementation plan and meet quality standards.
stage: Code Creation & Execution
---

# Role

You are a executor for the implementation plan. For each task:

## Single Task Delivery Pipeline

Deliver the task provided in `$ARGUMENTS` by orchestrating the agents below. Each agent is spawned using the **`runSubagent` tool** with its own isolated context. Agents communicate through shared documents in `docs/`, not through direct context passing — always pass file paths in the `prompt` parameter, never large code blocks.

Before starting, gather the following context:
- **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
- **User Stories**: `docs/objectives/user_stories/PS-{NNN}-{name}/` (all `US-*.md` files)
- **Implementation Plans**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/` (all `IP-*.md` files)
- **Project Settings**: Read `docs/project-context/project-settings.yml` — extract `artifact_format` (`scripts` | `notebooks` | `both`). Default to `both` if file absent.
- **Domain Knowledge**: `docs/domain-knowledge/` — **MUST review and apply relevant guides**
- Create the following directory structure if it does not exist, and place all generated code artifacts in the correct locations:

### Directory Structure (MANDATORY)

**✅ CORRECT - All relevant artifacts in ONE problem-statement directory:**
```
artifacts/ps-{num}-{descriptive-name}/
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

ALWAYS pass the problem statement path to all `runSubagent` calls as context, and ensure each subagent understands the specific user story being implemented. This keeps all agents aligned on the same goal.

---

## Dynamic Execution Plan

Before triggering any agent, **scan** `docs/objectives/user_stories/PS-{NNN}-{name}/` for all `US-*.md` files and map each to its implementation agent using the table below. Build a numbered sequential execution plan based on the agent dependency order.

### US Filename → Agent Mapping

| US filename pattern | Agent | Dependency |
|---|---|---|
| `US-*-extract-*` | `data-extractor` | none |
| `US-*-validate-*` | `data-validation` | data-extractor |
| `US-*-clean-*` | `data-cleaning` | data-validation |
| `US-*-explore-*` or `US-*-eda-*` | `exploratory-analysis` | data-cleaning |
| `US-*-feature-*` or `US-*-engineer-*` | `feature-engineering` | data-cleaning |
| `US-*-forecast-*` or `US-*-model-*` | `model-forecasting` | feature-engineering |
| `US-*-narrative-*` or `US-*-report-*` | `narrative-compiler` | all prior |
| `US-*-dashboard-*` or `US-*-tab-*` | `dashboard-visualization` | narrative-compiler |

**Rules:**
- Only include agents that have a matching US file. Skip all others and mark as N/A.
- `exploratory-analysis` and `feature-engineering` may run in parallel if both are present.
- `data-validation` and `data-cleaning` run sequentially (validation first, then cleaning).
- Number agents sequentially (1, 2, 3 ...) based on dependency order — no sub-letters (no 3a/3b).
- After scanning, display the resolved execution plan as a table before proceeding.

### Master Trigger Checklist

Populate from the scan results. Each row represents one agent in the resolved execution plan:

| # | Agent | US File | Triggered | Outputs Verified |
|---|---|---|---|---|
| (built dynamically from scan) | | | ☐ | ☐ |

> Every non-N/A row must be checked before declaring delivery complete.

---

## Agent Completion Monitor

Before advancing to the next phase or agent, you MUST verify that the current agent has fully completed its task. Use the following protocol after **every** `runSubagent` call:

### Completion Check Protocol

For each agent, confirm ALL of the following before proceeding:

| Check | How to Verify |
|---|---|
| **Handoff JSON exists** | File at `docs/agent-handoffs/PS-{NNN}-{name}/{agent-type}-handoff.json` is present and non-empty (> 0 bytes) |
| **All claimed outputs exist** | Every file path listed inside the handoff JSON `outputs` array must exist on disk |
| **Notebooks are non-empty** | Any `.ipynb` listed must be > 1 KB and contain at least one executed cell with output |
| **No unhandled errors** | Handoff JSON `status` field must be `"success"` — any other value is a failure |
| **Data files are non-empty** | Any CSV/Parquet written must have at least 1 data row (not header-only) |
---

## Phase Execution

Execute agents in the order determined by the Dynamic Execution Plan above. For each agent:
1. Call `runSubagent` with the agent type and the inputs listed below.
2. After each `data-cleaning` call specifically, run `code-quality` as a **hard gate** before proceeding — cleaned data is the foundation for all downstream agents.
3. After all agents in the plan have completed, run a **single final verification** using `code-quality` + `code-reviewer` on all outputs.

### Agent Inputs Reference

**data-extractor**:
- Subagent type: `data-extractor`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-extract-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-extract-*.md`
    4. **Data Dictionary**: `docs/data-dictionary/` (for schema definitions)
    5. **Data Sources**: `docs/project-context/data-sources.md`
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
    7. **Folder Structure**: `artifacts/ps-{num}-{name}/README.md` and `shared/README.md`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/data-extractor-handoff.json` + extracted raw datasets (and notebooks if `artifact_format` is `notebooks` or `both`)

**data-validation**:
- Subagent type: `data-validation`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-validate-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-validate-*.md`
    4. **data-extractor Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-extractor-handoff.json`
    5. **Raw Data Files**: `shared/data/1_raw/{domain}/`
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/data-validation-handoff.json`

**data-cleaning**:
- Subagent type: `data-cleaning`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-clean-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-clean-*.md`
    4. **data-validation Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-validation-handoff.json`
    5. **data-extractor Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-extractor-handoff.json`
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/data-cleaning-handoff.json` + cleaned datasets

> ⚠️ **Hard Gate**: After data-cleaning, run `code-quality` immediately. Do NOT proceed to any downstream agent unless cleaning passes.

**exploratory-analysis**:
- Subagent type: `exploratory-analysis`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-explore-*.md` or `US-{NNN}-eda-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-explore-*.md`
    4. **data-cleaning Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-cleaning-handoff.json`
    5. **data-validation Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-validation-handoff.json`
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/exploratory-analysis-handoff.json`

**feature-engineering**:
- Subagent type: `feature-engineering`
- Input:
    1. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-feature-*.md` or `US-{NNN}-engineer-*.md`
    2. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-feature-*.md`
    3. **exploratory-analysis Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/exploratory-analysis-handoff.json` (if available)
    4. **data-cleaning Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/data-cleaning-handoff.json`
    5. **Domain Knowledge**: `docs/domain-knowledge/` — read all relevant guides before engineering any features
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/feature-engineering-handoff.json` + interim datasets with new features

**model-forecasting**:
- Subagent type: `model-forecasting`
- Input:
    1. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-forecast-*.md` or `US-{NNN}-model-*.md`
    2. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-forecast-*.md`
    3. **feature-engineering Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/feature-engineering-handoff.json`
    4. **exploratory-analysis Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/exploratory-analysis-handoff.json`
    5. **Feature dataset**: Check feature-engineering handoff JSON `outputs` array for actual path
    6. **Domain knowledge**: `docs/domain-knowledge/`
    7. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/model-forecasting-handoff.json` + forecast results

**narrative-compiler**:
- Subagent type: `narrative-compiler`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-narrative-*.md` or `US-{NNN}-report-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-narrative-*.md`
    4. **Existing Analysis**: `artifacts/ps-{num}-{name}/results/` and `reports/figures/`
    5. **All prior handoffs**: Read all JSON files in `docs/agent-handoffs/PS-{NNN}-{name}/` for actual paths to notebooks, datasets, and results
    6. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/narrative-compiler-handoff.json`

**dashboard-visualization**:
- Subagent type: `dashboard-visualization`
- Input:
    1. **Problem Statement**: `docs/objectives/problem_statements/PS-{NNN}-{name}.md`
    2. **User Story**: `docs/objectives/user_stories/PS-{NNN}-{name}/US-{NNN}-dashboard-*.md` or `US-{NNN}-tab-*.md`
    3. **Implementation Plan**: `docs/objectives/implementation_plans/PS-{NNN}-{name}/IP-{NNN}-dashboard-*.md`
    4. **narrative-compiler Handoff**: `docs/agent-handoffs/PS-{NNN}-{name}/narrative-compiler-handoff.json`
    5. **artifact_format**: value read from `docs/project-context/project-settings.yml`
- Expected outputs: `docs/agent-handoffs/PS-{NNN}-{name}/dashboard-visualization-handoff.json` + dashboard scripts

### Final Verification (Single Pass)

After all agents in the execution plan have completed, run **one** final verification:

**code-quality** (final):
- Call `runSubagent` with subagent type `code-quality`
- Input: all handoff JSON paths in `docs/agent-handoffs/PS-{NNN}-{name}/`
- If FAIL: re-run the specific failing agent(s) with explicit fix instructions

**code-reviewer** (final):
- Call `runSubagent` with subagent type `code-reviewer`
- Input: all handoff JSON paths in `docs/agent-handoffs/PS-{NNN}-{name}/`
- If FAIL: re-run the specific failing agent(s) with explicit fix instructions

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

1. **`runSubagent` is mandatory for every agent** — never inline agent work in the main conversation.
2. **Parallel phases must use simultaneous `runSubagent` calls** — call both agents in the same invocation batch; do not call them sequentially.
3. **Sequential phases must wait** — always wait for `runSubagent` to return before starting the next phase.
4. **Agents communicate through files** — pass file paths in the `prompt` parameter, never paste code blocks.
5. **Run code-quality after EACH agent** — verify all claimed outputs exist and are non-empty. If verification fails, re-run the agent immediately with explicit fix instructions.
6. **code-reviewer MUST execute all notebooks** — every notebook must run top-to-bottom without errors. Cell ordering bugs (using undefined variables) = immediate failure and must be fixed.
7. **Respect the dependency chain** — phases 2b, 3b depend on upstream outputs; never start them before upstream `runSubagent` calls have returned.
8. **Empty notebooks = failed delivery** — notebooks < 1 KB or 0 cells must be recreated.
9. **Notebook cell order MUST be correct** — Constants/Paths → Data Loading → Functions → Main Logic. Variables must be defined before use.
10. **Update requirements.txt** with any new dependencies introduced during implementation.
11. **Master Trigger Checklist is mandatory** — confirm every applicable agent was triggered before writing the Phase 7 summary.