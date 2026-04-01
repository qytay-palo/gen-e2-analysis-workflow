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
- Expected outputs: `ocs/agent-handoffs/extraction/ps-{num}-{name}/cleaning_to_{next_agent}_{timestamp}.json` + jupyter notebooks and respective datasets

**data-validation**:
- Subagent type: `data-validation`
- Input: 
    1. **Problem Statement**: `docs/objectives/problem_statements/ps-{num}-{name}.md`
    2. **Extractor Handoff**: `ocs/agent-handoffs/extraction/ps-{num}-{name}/*`
- Expected outputs: `ocs/agent-handoffs/extraction/ps-{num}-{name}/validation_to_{next_agent}_{timestamp}.json` + jupyter notebooks and respective datasets

### Step 1 — code-simplifier

- Subagent type: `code-simplifier`
- Input: review all code changes from Phase 2
- Purpose: clean up generated code, reduce complexity, preserve functionality

### Step 2 — test-runner (MANDATORY)

> **This step cannot be skipped. It is the oracle gate. A task is not done until tests pass.**

- Subagent type: `test-runner`
- Input: run the full test suite (Go + Next.js)
- Expected output: structured pass/fail report
- If tests **PASS** → go to Step 4
- If tests **FAIL** → go to Step 3

### Step 3 — debugger (only if tests failed)

- Subagent type: `debugger`
- Input: the test failure report from Step 2
- After the debugger completes, re-run **test-runner** (Step 2)
- Retry up to **3 times**. If tests still fail after 3 debug cycles → **STOP** and report failures to the user. Do not continue.

### Step 4 — security-advisor

- Subagent type: `security-advisor`
- Input: all code changes + architecture doc path
- Expected output: `docs/reviews/<feature-name>-security-review.md`
- If **critical findings** → present them to the user and ask whether to proceed or fix first. Do not silently continue.

### Step 5 — ui-reviewer (MANDATORY for any feature with UI)

> **This step always runs if the feature has a frontend.** It does not require a design doc. At minimum it verifies pages render, navigation works, and there are no console errors. If a design doc exists, it also does a full visual comparison.

- Subagent type: `ui-reviewer`
- Input: the local app URL (ask the user if not known) and, if available, the design doc path
- Expected output: `docs/reviews/<feature-name>-ui-review.md`
- If the page **fails to render or shows a blank screen** → treat this as a **Critical** finding, equivalent to a test failure. Spawn `debugger` to investigate the routing/rendering issue before proceeding.

## Phase 4: Summary

After all phases complete, present a delivery report:

```
## Story Delivery Report: [title]

### Artifacts
- Architecture: docs/architecture/<name>-architecture.md
- Design: docs/design/<name>-design.md (if applicable)
- Backend plan: docs/plans/<name>-backend-plan.md
- Frontend plan: docs/plans/<name>-frontend-plan.md
- Security review: docs/reviews/<name>-security-review.md
- UI review: docs/reviews/<name>-ui-review.md (if applicable)

### Files Changed
[list all created/modified source files]

### Test Results
- Backend: X passed, Y failed
- Frontend: X passed, Y failed
- Debug cycles: N

### Security: X critical, X high, X medium, X low
### UI Review: [summary or N/A]

### Status: READY FOR REVIEW | BLOCKED
```

## Rules

1. **test-runner is mandatory** — every task must pass tests. No exceptions.
2. **Security criticals block delivery** — report to user, do not proceed silently.
3. **Agents run via Task tool** — never inline agent work in the main conversation.
4. **Agents communicate through files** — pass file paths between agents, not code.
5. **Respect the dependency chain** — never start a phase before dependencies complete.
6. **Debug loop has a limit** — 3 attempts max, then the human decides.



=====================================

---
type: prompt
description: Prompt for Execution of the Implementation Plan
rule_name: execute-implementation-plan
stage: Development
model: Claude Sonnet 4.5
---
# AI Agent Prompt: Execute Implementation Plan

## Role

Execute a detailed implementation plan accurately and verify its completion according to specifications for an end-to-end data analysis project.

## Terminology

| Term | Definition | Example |
|------|------------|---------|
| **Problem Statement (PS)** | Top-level analysis objective | problem-statement-001: Workforce Capacity Mismatch |
| **User Story (US)** | Deliverable subset of a problem statement | US-001 within problem-statement-001 |
| **Subagent** | Specialist AI agent invoked via `#runSubagent` | ExtractionAgent, Code Reviewer Agent |
| **Handoff File** | JSON file agents use to pass validated outputs | `extraction_to_profiling_20260306.json` |
---

## 🚨 CRITICAL RULES - READ FIRST

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

## 1. MCP Tools Quick Reference

### Filesystem Tools (REQUIRED)
Use for ALL file operations:

```
Read data:     "Use filesystem tools to read shared/data/1_raw/input.csv"
Create dir:    "Use filesystem tools to create directory problem-statement/ps-001-{name}/results/"
Write file:    "Use filesystem tools to save results to problem-statement/ps-001-{name}/results/tables/output.csv"
List files:    "Use filesystem tools to list files in problem-statement/ps-001-{name}/"
```

### SQLite Tools (when applicable)
Use for database operations:

```
Query:         "Query using SQLite tools: SELECT * FROM patients WHERE year >= 2020"
Create table:  "Use SQLite tools to create summary table"
```

### Context7 MCP (REQUIRED for Library/Framework Usage)
Use for fetching **current documentation** instead of relying on training data:

**🚨 CRITICAL RULE**: When implementing code that uses libraries, frameworks, or APIs, you MUST use Context7 to fetch current documentation. This ensures API accuracy, correct syntax, and version compatibility.

**WHEN TO USE Context7 (MANDATORY):**

| Scenario | Action | Example |
|----------|--------|----------|
| **Writing data processing code** | Fetch Polars/pandas docs | `resolve-library-id` → "polars" → `query-docs` "lazy evaluation scan_csv" |
| **Implementing forecasting models** | Fetch statsmodels/prophet docs | `resolve-library-id` → "statsmodels" → `query-docs` "ARIMA model fit predict" |
| **Building dashboards** | Fetch Plotly Dash docs | `resolve-library-id` → "plotly" → `query-docs` "Dash app layout callbacks" |
| **Using ML libraries** | Fetch scikit-learn docs | `resolve-library-id` → "scikit-learn" → `query-docs` "train_test_split cross validation" |
| **Databricks integration** | Fetch Databricks docs | `resolve-library-id` → "databricks" → `query-docs` "scheduled jobs DBFS" |
| **API integrations** | Fetch API docs | `resolve-library-id` → "requests" → `query-docs` "session retry timeout" |

**WHEN NOT TO USE Context7:**
- Writing specifications/documentation (no code implementation)
- High-level architecture planning
- Business logic that doesn't depend on external libraries
- Simple Python standard library usage (pathlib, datetime, logging)

**Context7 Workflow:**

```bash
# Step 1: Identify the library/framework
Library: "polars" (for data processing)

# Step 2: Resolve library ID
resolve-library-id({library: "polars", question: "How to use lazy evaluation with scan_csv?"})
# Returns: [{id: "polars", name: "Polars", version: "0.20.0"}]

# Step 3: Query specific documentation
query-docs({libraryId: "polars", question: "scan_csv lazy evaluation collect filter"})
# Returns: Current API syntax, examples, best practices

# Step 4: Implement using fetched documentation
# Use exact syntax from docs (not training data)
```

**Verification After Context7 Usage:**
- ✅ API method signatures match current documentation
- ✅ Parameter names and types are accurate
- ✅ Deprecated methods avoided
- ✅ Best practices from docs followed
- ✅ Version-specific features used correctly

**🔗 Details**: See [MCP Tools Reference Guide](.github/guides/mcp-tools-reference.md)


### 2. Continuous Code Quality Analysis (MANDATORY)

**🚨 CRITICAL REQUIREMENT**: You MUST execute quality review and Jupyter Notebook Execution Agent before this task can be marked as complete. Quality checks are NOT optional and NOT end-of-implementation tasks. Failure to execute these subagents means the implementation is INCOMPLETE.

**EXECUTION WORKFLOW (MANDATORY):**
```
For EACH implementation stage:
1. Implement the code/module
2. IMMEDIATELY RUN THE CODE to verify zero errors
3. Fix ALL errors found
4. Delegate quality review and jupyter notebook execution
5. Fix issues found
6. [If visualization user story] → Run VisualizationAgent
   → THEN CONTINUE: run Code Review, Notebook Execution,
     Import Validation, Final Review — ALL REQUIRED
7. Proceed to next stage ONLY after all quality gates pass
```

> **⚠️ KEY RULE**: VisualizationAgent is a PIPELINE STEP, not a terminal step. The full quality workflow applies AFTER it completes, not instead of it.

**🔴 ZERO TOLERANCE FOR EXECUTION ERRORS:**
- **ALL Python scripts (.py) MUST execute successfully with exit code 0**
- **ALL Jupyter notebooks (.ipynb) MUST execute from start to finish with zero cell errors**
- **ALL expected output files MUST be generated and validated**
- **ANY ImportError, TypeError, AttributeError, FileNotFoundError, or other exception is a BLOCKER**
- **Implementation is INCOMPLETE until ALL execution errors are resolved**

**WHEN TO EXECUTE SUBAGENTS (NON-NEGOTIABLE):**

| Implementation Stage | Subagent to Execute | Timing |
|---------------------|---------------------|--------|
| **BEFORE writing any code** | Directory Structure Validation | PREVENT duplicate directories, files and import path chaos |
| **After creating each module** | Code Reviewer Agent Code Review | Review error handling, types, security |
| **After completing 2-3 modules** | Dead Code Elimination | Clean up unused code before it accumulates |
| **After completing 2-3 modules** | Import Path Validation | Verify all imports match actual directory structure |
| **For visualization/dashboard user stories** | VisualizationAgent | NON-BLOCKING |
| **After VisualizationAgent returns** | Code Review Agent | Review ALL visualization files produced |
| **After VisualizationAgent returns** | Jupyter Notebook Execution Agent| Execute ALL notebooks including new visualization notebooks |
| **Before marking work complete** | Code Reviewer Agent Comprehensive Review| Final validation of all code quality standards |

**MANDATORY SUBAGENT EXECUTIONS:**

Execute these subagents in **DURING** implementation. Each subagent call is REQUIRED:

**a. Import Path Validation Agent (REQUIRED - Execute AFTER creating files, BEFORE marking complete):**
```javascript
#runSubagent({
  description: "Validate import paths match directory structure",
  prompt: `CRITICAL: Verify all import statements match actual directory structure.

Problem Statement Directory: problem-statement/ps-{num}-{descriptive-name}/

Tasks:
1. **Validate Import Paths Against Actual Filesystem**: For each import, resolve the path relative to the importing file's location, verify the module EXISTS on filesystem, check naming consistency (hyphens vs underscores), and flag mismatches

2. **Validate Relative Imports Don't Cross Boundaries**: Files should import from their own problem-statement directory; flag if ps-001 imports from ps-002 (should use shared/src/ for shared code)

Return:
- Import_Validation_Status: "ALL_VALID" | "MISMATCHES_FOUND"
- Resolve all mismatched imports`
})
```

**b. Jupyter Notebook Execution Agent (REQUIRED - Execute AFTER creating/updating notebooks):**
```javascript
#runSubagent({
  description: "Execute all Jupyter notebooks and fix errors for Problem Statement {num}",
  prompt: `Read and execute the Jupyter Notebook Execution protocol from .github/prompts/6-test-execuetion-code.prompt.md

Context:
- Problem Statement: {num}
- Target Directory: problem-statement/ps-{num}-{descriptive-name}/
- Notebooks Location: problem-statement/ps-{num}-{descriptive-name}/notebooks/
- **MISSION**: Zero cell execution errors across all notebooks

Instructions:
1. Read the complete testing protocol from .github/prompts/6-test-execuetion-code.prompt.md
2. Follow ALL phases in the "Testing Protocol" section:
   - Phase 1: Discovery (find all notebooks)
   - Phase 2: Pre-Execution Validation (imports, dependencies, environment)
   - Phase 3: Execution (jupyter nbconvert --execute)
   - Phase 4: Error Handling (diagnose and fix ALL errors)
   - Phase 5: Output Verification (cells, files, visualizations)
   - Phase 6: Documentation Updates (prerequisites, error prevention)
3. Apply the "Error Resolution & Code Adjustments" protocol for ALL errors encountered
4. Iterate until ZERO errors across all notebooks
5. Generate comprehensive report as specified in the prompt

**CRITICAL REQUIREMENTS:**
- ✅ ALL notebooks must execute from start to finish with ZERO errors
- ✅ ALL cells must have outputs (no empty execution_count)
- ✅ ALL expected files must be generated and validated
- ✅ Import cells positioned correctly (first executable cell)
- ✅ Prerequisites documented in markdown cells
- ✅ Error prevention checks added to notebooks

**FAILURE CONDITION:**
If ANY notebook cannot be fixed to execute successfully:
1. Report persistent error in detail
2. Mark status as "EXECUTION_FAILED"
3. Block further implementation
4. Escalate to primary agent

Reference: See .github/prompts/6-test-execuetion-code.prompt.md for complete testing protocol and error resolution strategies.`
})
```

**c. Code Review Agent (REQUIRED):**
```javascript
#runSubagent({
  description: "Comprehensive code review for Problem Statement {num}",
  prompt: `Execute a comprehensive code quality review and fix ALL identified issues.

Context:
- Target Directory: problem-statement/ps-{num}-{descriptive-name}/
- Review ALL Python files (.py) AND ALL Jupyter notebooks (.ipynb)
- Fix issues immediately - do not just report them
- **CRITICAL**: Any errors detected during code execution MUST BE FIXED before proceeding

**Tasks:**
**🔴 CRITICAL EXECUTION REQUIREMENT - HIGHEST PRIORITY:**
Before ANY other review tasks, you MUST execute ALL code to verify zero errors:

1. **Run ALL Python Scripts (MANDATORY):**
   - List ALL .py files in the problem-statement directory (src/scripts/, src/data_processing/, src/analysis/, etc.)
   - Determine correct execution order based on dependencies
   - Execute EVERY .py file one by one
   - Check exit code for each script (must be 0)
   - Verify zero ImportError, TypeError, AttributeError, FileNotFoundError, KeyError, ValueError
   - **STOP IMMEDIATELY if any script fails** - do not proceed to next script
   - Fix ALL errors immediately before proceeding
   - Re-run failed script to confirm fix
   - Document execution results: script path, exit code, execution time, outputs generated

2. **Execute ALL Jupyter Notebooks (MANDATORY):**
   - List ALL .ipynb files in problem-statement/ps-{num}-{descriptive-name}/notebooks/
   - For EACH notebook:
     * Check that all prerequisite scripts have been run (data files exist)
     * Execute notebook: `jupyter nbconvert --execute --to notebook --inplace {notebook_path}`
     * Verify exit code is 0
     * Check that ALL cells executed successfully (no exception outputs)
     * Verify all expected outputs are generated (figures, CSV files, etc.)
   - **STOP IMMEDIATELY if any notebook cell fails**
   - Fix ALL errors (missing imports, file paths, data issues)
   - Re-run notebook to confirm fix
   - Document execution results: notebook path, cell count, execution time, outputs generated

3. **Context7 Documentation Validation (MANDATORY - NEW):**
   - For EVERY library/framework import, verify against current documentation using Context7
   - Use `resolve-library-id` + `query-docs` to fetch current API syntax
   - Check for:
     * Deprecated methods (replace with current alternatives)
     * Incorrect parameter names/types (fix to match current docs)
     * Missing required parameters (add with correct defaults)
     * Version-incompatible features (update or remove)
   - Document which libraries were validated and any fixes applied
   - **DO NOT SKIP**: This prevents runtime errors from outdated API usage

4. **Verify All Outputs Created (MANDATORY):**
   - Check that all expected output files exist (from both scripts AND notebooks)
   - Verify file contents are valid (not empty, correct format, correct schema)
   - Verify file sizes are reasonable (not 0 bytes, not suspiciously small)
   - Open CSV/JSON files and inspect first few rows/records
   - Fix missing outputs immediately
   - Re-run code if outputs are missing or invalid

**IF ANY ERRORS FOUND: STOP - FIX - RE-RUN - VERIFY ZERO ERRORS**
**DO NOT PROCEED TO REMAINING TASKS UNTIL ALL CODE EXECUTES SUCCESSFULLY**

**After execution verification passes (ALL code runs with zero errors), proceed with additional tasks:**

5. **Code Formatting (Python Files AND Notebooks - MANDATORY):**
   - Format Python files: `ruff format problem-statement/ps-{num}-{descriptive-name}/ --check`
   - **Format Jupyter notebooks**: `ruff format problem-statement/ps-{num}-{descriptive-name}/**/*.ipynb --check`
   - Auto-format both file types if needed
   - **DO NOT SKIP notebook formatting**

6. **Parallel Code Review Perspectives:** Run these subagents in parallel:
   - **Data structure validator (CRITICAL)**: Execute all data loading code, inspect actual columns/keys/dtypes, verify every subsequent reference matches reality exactly (case-sensitive). Flag any KeyError, ColumnNotFoundError, or assumptions about data structure
   - Correctness reviewer: logic errors, edge cases, type issues
   - Code quality reviewer: readability, naming, duplication
   - Security reviewer: input validation, injection risks, data exposure
   - Architecture reviewer: codebase patterns, design consistency, structural alignment
   - Notebook documentation reviewer: Template variables replaced, markdown cells accurate, prerequisites clear
`
})
```

**MANDATORY ACTION AFTER EACH SUBAGENT REPORT:**

You MUST take these actions immediately after receiving subagent findings:

1. **REVIEW findings** - Read the complete subagent report
2. **TRIAGE issues** - Categorize by severity (critical → high → medium → low)
3. **FIX CRITICAL/HIGH immediately** - Do not proceed until security issues, data loss risks, and high-priority bugs are fixed
4. **RE-RUN ALL CODE AFTER FIXES** - Execute all .py scripts and .ipynb notebooks again to verify fixes work
5. **REFACTOR duplicates** - If 3+ occurrences of duplicate code found, extract to shared function/module
6. **UPDATE code** - Apply all recommendations from subagent before next implementation stage
7. **DOCUMENT deferred items** - Add any medium/low priority improvements to TODO.md with issue IDs
8. **VERIFY fixes** - Re-run subagent if critical issues were found to confirm resolution

**VERIFICATION REQUIREMENT:**

Before marking implementation complete, you MUST document:
- ✅ **ALL Python scripts executed successfully with exit code 0** (list each script with execution status)
- ✅ **ALL Jupyter notebooks executed successfully with zero cell errors** (list each notebook with execution status)
- ✅ **ALL expected output files generated and validated** (list file paths and sizes)
- ✅ List of all subagent executions with timestamps
- ✅ Jupyter Notebook Execution Agent report (all notebooks passing with zero errors)
- ✅ Summary of findings from each subagent report
- ✅ Actions taken to address critical/high priority issues
- ✅ Confirmation that all security vulnerabilities are fixed
- ✅ Refactoring completed for duplicate code (3+ occurrences)
- ✅ Final quality scores from comprehensive review
- ✅ Any deferred improvements documented in TODO.md
- ✅ **Post-fix execution verification** (all code re-run after fixes to confirm zero errors)

### 3. Naming Convention (MANDATORY)

**Format**: `problem-statement/ps-{num}-{descriptive-name}/`
- `{num}`: Zero-padded problem statement number (e.g., `001`, `002`)
- `{descriptive-name}`: Kebab-case description (e.g., `workforce-capacity-mismatch`)

**✅ Examples:**
- `problem-statement/ps-001-workforce-capacity-mismatch/`
- `problem-statement/ps-002-seasonal-disease-forecasting/`

### 4. Organization Rules

1. **One Problem Statement = One Directory**: All user stories for the same problem statement share ONE directory
2. **No Shared Notebooks Directory**: Do NOT create notebooks in `notebooks/1_exploratory/`, `notebooks/2_analysis/`, etc.
3. **Problem-Statement Outputs**: Save outputs within problem statement directory: `problem-statement/ps-{num}-{name}/reports/figures/`, `problem-statement/ps-{num}-{name}/results/tables/`

## Input Requirements

The input will consist of:
- A detailed implementation plan (typically in Markdown format)
- User story and acceptance criteria
- Design specifications and requirements

## Output Requirements

The output MUST include:
- **Problem-Statement-Specific Directory Structure**: Created per CRITICAL RULES above
- Implementation of all required files and changes
- Verification that specifications have been met
- Completed Design Implementation Verification Checklist

## Review Requirements

Before implementation, the implementation plan MUST be reviewed for:
- Clarity and completeness of all steps
- Availability of all necessary information
- Alignment with project guidelines and technical stack
- Completeness of design specifications (colors, spacing, typography)

If any part of the plan is unclear or missing information, clarification MUST be requested before proceeding.

## 5. Search for Existing Implementations

**MANDATORY STEP**: Before creating new features, search for similar implementations.

### a. Search Process

```javascript
// Check for existing dashboards
grep_search("dashboard", isRegexp=false, includePattern="dashboards/**")

// Check for similar analysis modules (search in both shared and problem-specific)
grep_search("forecasting|prediction", isRegexp=true, includePattern="{shared/src,problem-statement/ps-*}/**/analysis/**")

// Check for data processing patterns
semantic_search("data cleaning workflow for disease surveillance")
```

### b. Relevance Assessment

Ask these questions to determine if feature should extend existing implementation:

| Question | EXTEND if... | CREATE NEW if... |
|----------|-------------|-----------------|
| **Same domain?** | Same disease types, same health outcomes | Different health domains (workforce vs disease) |
| **Same audience?** | Same stakeholders/decision-makers | Different user groups (clinicians vs policymakers) |
| **Same workflow?** | Same analytical pipeline stages | Different methodologies |
| **Compatible config?** | Uses same config files/parameters | Requires conflicting configurations |

### c. Extension Patterns

**Pattern 1: Dashboard Extension (Multi-Page Apps)**
```python
# Existing: dashboards/workforce_capacity_dashboard.py
# New Feature: Pharmacist analysis (RELATED to workforce)
# Action: Add new page
dashboards/pages/03_💊_Pharmacist_Analysis.py
```

**Pattern 2: Module Extension (Add Functions)**
```python
# Existing: problem-statement/ps-001/analysis/burden_calculator.py
# New Feature: Calculate burden with demographic breakdown
# Action: Add function to existing module
def calculate_burden_by_demographics(df: pl.DataFrame) -> pl.DataFrame:
    """Calculate disease burden stratified by age/gender."""
```

### d. Integration Requirements

When extending existing implementations:

1. **Maintain consistency**: Same naming, code style, error handling, logging
2. **Preserve functionality**: DO NOT modify existing functions unless fixing bugs
3. **Use shared configs**: Read from existing `shared/config/*.yml` or `problem-statement/ps-{num}-{name}/config/*.yml`, add entries if needed
4. **Update documentation**: Update README with new features added

**Verification Checklist (see §8.2):**
- [ ] Searched for existing implementations
- [ ] Assessed relevance (domain, audience, workflow)
- [ ] Extended existing ONLY if relevant
- [ ] New code follows existing patterns
- [ ] No regressions in existing functionality


## 6. Implementation Requirements

The implementation MUST:
- **FIRST: Create the problem-statement-specific directory structure per CRITICAL RULES** (see top of document)
- Follow the staged implementation approach outlined below
- Adhere to file paths, code structures, and configurations specified in the plan
- **ALL code files, notebooks, and scripts MUST be placed within `problem-statement/ps-{num}-{descriptive-name}/`**
- Follow project coding standards and best practices
- **Leverage MCP (Model Context Protocol) tools for all file and data operations as specified below**
- **Implement ALL code blocks provided in the implementation plan verbatim (see Code Implementation Fidelity below)**
- **Update README files to document the code running flow and execution instructions (see README Documentation Requirements below)**
- **Create at least one Jupyter notebook for each user story** to facilitate user viewing of outputs and results
  - **Notebook Location**: Place notebooks in `problem-statement/ps-{num}-{name}/notebooks/` directory
  - **CRITICAL Dependency Requirements**:
    - **BEFORE creating notebooks**: Execute all prerequisite scripts/code to generate required data
    - **Document dependencies**: Include markdown cell at top listing all prerequisite scripts
    - **Test execution flow**: Run all dependencies → then run notebook → verify outputs
    - **Verify data exists**: Check that all expected input files exist before running notebook cells
    - **Check file existence explicitly**: Use `Path(file).exists()` before reading files to provide clear error messages instead of letting exceptions occur
  - Include markdown cells documenting each analysis step
  - **🚨 CRITICAL NOTEBOOK FORMATTING RULE**: When creating Jupyter notebooks, NEVER use literal `\n` characters in cell source code.

### Code Implementation Fidelity

**CRITICAL**: When an implementation plan includes code blocks with complete function implementations, these MUST be used exactly as specified:

✅ **REQUIRED - Implement Verbatim:**
- ALL code blocks provided in the implementation plan
- All type hints, docstrings, and parameter definitions
- All error handling, retry logic, and validation checks
- All logging statements and audit trail tracking
- All unit tests specified with full test coverage
- All helper functions, utility modules, and configuration files
- All security best practices (credential handling, input validation)

**Rationale**: Implementation plan code blocks are:
- Production-ready and tested against edge cases
- Include proper error handling for real-world scenarios
- Follow security best practices
- Implement comprehensive logging for debugging and audit trails
- Support reproducibility and long-term maintainability

**Verification After Implementation**:
1. Every function specified in the plan exists and is fully implemented
2. All type hints and docstrings match the specification exactly
3. All unit tests run successfully with expected coverage
4. No functions are stubs (contain only `pass` or `NotImplementedError`)
5. All error handling, retry logic, and validation checks are present
6. All logging and metadata tracking code is functional

**How to Invoke Specialist Agents:**

Use the `#runSubagent` tool to delegate specific stages to specialist agents. Each agent reads its template from `.agents/` and follows the instructions there.

**Sequential Pipeline (With Handoffs):**
```javascript
// Step 1: Extract
#runSubagent({...}) → extraction_handoff.json

// Step 2: Profile (reads extraction handoff)
#runSubagent({
  prompt: `Execute ProfilingAgent with input from: shared/data/3_interim/agent_handoffs/extraction_handoff.json`
}) → profiling_handoff.json

// Step 3: Clean (reads profiling handoff)
#runSubagent({...}) → cleaned data + cleaning_handoff.json
```

### §5.4 Handoff Protocol

**Handoff File Format:**
```json
{
  "agent_name": "ProfilingAgent",
  "timestamp": "20260306_141530",
  "stage": 3,
  "validation_status": "passed",
  "outputs": {
    "report": "problem-statement/ps-001-{name}/results/tables/data_quality_report.md",
    "metrics": "problem-statement/ps-001-{name}/results/metrics/quality_metrics.json"
  },
  "findings": {
    "overall_quality_score": 87.5,
    "recommended_cleaning_actions": ["impute_missing_values", "handle_outliers"]
  },
  "recommended_next_step": "cleaning"
}
```

**Location**: `shared/data/3_interim/agent_handoffs/`

See [`.agents/README.md`](.agents/README.md) for complete documentation.


**Agent Invocation Template:**

When invoking any agent with `#runSubagent`, use this pattern:

```
#runSubagent(
  description: "{AgentName} for Problem Statement {num}",
  prompt: "Read and execute the {AgentName} template from .agents/{agent}_agent.md
  
  Context:
  - Problem Statement: {num}
  - Problem Title: {title}
  - Previous Handoff: {path_to_handoff_file} (if applicable)
  - Input Data: {path_to_input_data}
  - Additional Context: {any_specific_requirements}
  
  Instructions:
  1. Read the agent template completely
  2. Follow all steps in the template's 'Your Responsibilities' section
  3. Use the specified instruction files from the template
  4. Apply domain skills from .agents/skills/ as referenced
  5. Generate all required outputs as specified
  6. Create handoff file with validation status and findings
  7. Return path to handoff file and summary of work completed
  "
)
```

See [`.agents/README.md`](.agents/README.md) for detailed documentation.

**Adaptive Implementation Workflow:**

```
1. Review implementation plan stage
2. Identify knowledge gaps or uncertainties
3. Select appropriate data plugin command to gather context
4. Execute command and analyze results
5. Adjust implementation approach based on findings
6. Generate code with informed decisions
7. Validate outputs using `/validate`
8. Proceed to next stage
```

This approach ensures that code generation is **data-driven and context-aware**, reducing rework and improving implementation quality.

---

## 7. VisualizationAgent (MANDATORY for Dashboard / Visualization User Stories)

**🚨 CRITICAL RULE**: If the user story involves **any** of the following, you **MUST** invoke the VisualizationAgent via `#runSubagent` in [../agents/visualization.agent.md](../agents/visualization.agent.md) **before** marking the user story complete:

- Creating or updating an interactive dashboard (HTML, Plotly Dash, Streamlit, etc.)
- Generating publication-quality charts or figures for stakeholder reports
- Producing visualizations that surface EDA or model results
- Any user story whose title or acceptance criteria include keywords: `dashboard`, `visualiz`, `chart`, `plot`, `figure`, `KPI`, `report`, `interactive`
- Run this tool: #runSubagent [../.agents/visualization_agent.md]tements/ps-{num}-{name}/reports/dashboards/*.html`
- [ ] Dashboard opens in browser without errors
- [ ] All KPIs display correct values (spot-check 2–3 against source data)
- [ ] All PS objectives are answered by at least one dashboard component
- [ ] Handoff JSON created at `problem-statements/ps-{num}-{name}/data/3_interim/agent_handoffs/`
- [ ] Shared code evaluation documented in handoff JSON (`shared_code_decisions` field)

---

## Verification Requirements

After implementation, the following verifications MUST be completed:

1. **Acceptance Criteria Verification**
   - Each acceptance criterion MUST be verified as met
   - Any discrepancies MUST be documented

2. **Code Execution Verification (MANDATORY - FIRST PRIORITY)**
   - **REQUIRED**: ALL Python scripts (.py) executed successfully with exit code 0
   - **REQUIRED**: ALL Jupyter notebooks (.ipynb) executed from start to finish with zero cell errors
   - **REQUIRED**: ALL expected output files generated (CSV, JSON, figures, reports)
   - **REQUIRED**: Output files validated (non-empty, correct format, correct schema)
   - **REQUIRED**: ALL execution errors fixed and code re-run to verify fixes
   
   **Code Execution Verification Checklist:**
   ```
   - [ ] Listed ALL .py files in problem-statement directory
   - [ ] Executed EVERY .py script in correct dependency order
   - [ ] Verified exit code 0 for all scripts
   - [ ] Listed ALL .ipynb files in problem-statement directory
   - [ ] Executed EVERY notebook with jupyter nbconvert --execute
   - [ ] Verified zero cell errors in all notebooks
   - [ ] All expected output files exist (list file paths)
   - [ ] Output files validated (not empty, correct format)
   - [ ] ALL ImportError, TypeError, AttributeError, FileNotFoundError fixed
   - [ ] ALL KeyError, ValueError, IndexError fixed
   - [ ] Code re-run after fixes to confirm zero errors
   - [ ] Execution log documenting all runs (script/notebook, exit code, execution time)
   ```

3. **Quality Subagent Execution Verification (MANDATORY)**
   - **REQUIRED**: Verify that quality review subagents were executed during implementation
   - **REQUIRED**: Document all subagent executions with timestamps and findings
   - **REQUIRED**: Confirm critical/high priority issues from subagent reports were fixed
   - **REQUIRED**: Provide evidence that security vulnerabilities were addressed
   - **REQUIRED**: Show refactoring completed for duplicate code (3+ occurrences)
   
   **Quality Verification Checklist:**
   ```
   - [ ] Code Reviewer Agent Code Review executed after 2-3 major module
   - [ ] Dead Code Elimination subagent executed during implementation
   - [ ] Comprehensive Code Reviewer Agent review executed before marking complete
   - [ ] All critical issues from quality reports fixed
   - [ ] All high priority issues from quality reports fixed
   - [ ] Security vulnerabilities addressed (credentials, SQL injection, etc.)
   - [ ] Duplicate code refactored (3+ occurrences)
   - [ ] Medium/low priority improvements documented in TODO.md
   - [ ] Final quality report shows >80% functions with type hints
   - [ ] Final quality report shows >80% functions with docstrings
   - [ ] Final quality report shows consistent error handling patterns
   ```

4. **Design Implementation Verification**
   - Complete the Design Implementation Verification Checklist
   - The checklist MUST include the sections below

- Explicitly check that all service and API integration logic is implemented, not just stubbed.
- During verification, confirm that all functions required to fetch, process, and return data are fully implemented and tested.
- **ALL code must have been executed successfully before design verification begins**
- If any function is a stub or placeholder, the implementation is NOT complete. Document this as a failure and halt further verification until resolved.
- If any code fails to execute, the implementation is NOT complete. Fix errors and re-run before proceeding.

## README Documentation Requirements

After implementing code, README files MUST be updated to reflect the actual code running flow and execution instructions. This ensures that future users (including stakeholders, team members, and the AI agent itself) can understand and execute the code correctly.

### README Update Locations

Update README files at appropriate levels based on the scope of implementation:

1. **Problem Statement Level**: `problem-statement/ps-{num}-{name}/README.md`
   - Overall execution flow for the entire problem statement
   - High-level orchestration of analysis stages
   - Dependencies between user stories/waves
   - Environment setup requirements

2. **Module Level**: Individual module docstrings and inline comments
   - Function-level execution details
   - Parameter descriptions and examples
   - Edge cases and error handling

### Required README Sections

Each README MUST include the following sections based on code running flow:

#### 1. Quick Start
```markdown
## Quick Start

#### 2. Execution Flow

**Critical Execution Rules**:
- **BLOCKER**: Step N cannot start until Step N-1 completes successfully
- **MANDATORY SEQUENCE**: Extract → Profile → Clean → Analyze/Model → Visualize
- **PARALLEL ALLOWED**: Analysis and modeling can run in parallel after cleaning completes
- **NOTEBOOKS**: Can only run after their input data files exist (check file paths first)
- **NEVER SKIP STAGES**: Each stage validates data for the next; skipping causes downstream failures


#### 3. Input/Output Specifications
```markdown
## Input/Output Specifications

### Inputs
| File | Location | Format | Required Fields |
|------|----------|--------|----------------|
| Raw disease data | `shared/data/1_raw/disease_data.csv` | CSV | date, disease, case_count, region |
| Configuration | `shared/config/base.yml` or `problem-statement/ps-{num}-{name}/config/config.yml` | YAML | target_diseases, date_range |

### Outputs
| File | Location | Format | Description |
|------|----------|--------|-------------|
| Cleaned data | `problem-statement/ps-{num}-{name}/data/4_processed/cleaned_data.csv` | CSV | Standardized disease cases |
| Quality report | `problem-statement/ps-{num}-{name}/results/tables/data_quality_report.md` | Markdown | Data profiling results |
| Visualizations | `problem-statement/ps-{num}-{name}/reports/figures/seasonal_patterns.png` | PNG | Time series plots |
```

#### 4. Environment Setup
```markdown
## Environment Setup

### 1. Python Environment
```bash
# Activate virtual environment
source .venv/bin/activate

# Install dependencies (use uv, not pip)
uv pip install -r requirements.txt
```

### 2. Configuration
```bash
# Copy example config and customize (for problem-specific config)
cp problem-statement/ps-{num}-{name}/config/config.example.yml problem-statement/ps-{num}-{name}/config/config.yml

# Edit config to set:
# - target_diseases: ["Dengue", "HFMD", "COVID-19"]
# - date_range: {start: "2012-01-01", end: "2023-12-31"}
```

### 3. API Keys (if applicable)
```bash
# Set required API keys as environment variables
export API_KEY="your_api_key"
export API_SECRET="your_api_secret"

# For persistent configuration, add to ~/.zshrc or ~/.bashrc
echo 'export API_KEY="your_api_key"' >> ~/.zshrc
echo 'export API_SECRET="your_api_secret"' >> ~/.zshrc
source ~/.zshrc
```

#### 5. Execution Modes
```markdown
## Execution Modes

### Development Mode
Run individual scripts for testing and debugging:
```bash
python problem-statement/ps-001-{name}/wave-1/02_profile_data.py --debug --sample-size 1000
```

### Production Mode
Run complete pipeline with all data:
```bash
python problem-statement/ps-001-{name}/run_all.py --mode production
```

### Notebook Mode
Interactive exploration:
```bash
jupyter notebook problem-statement/ps-001-{name}/notebooks/2_analysis/seasonal_analysis.ipynb
```

#### 6. Troubleshooting
```markdown
## Troubleshooting

### Common Issues

**Issue**: ImportError: No module named 'polars'
**Solution**: Install packages using `uv pip install -r requirements.txt`

**Issue**: FileNotFoundError: shared/data/1_raw/disease_data.csv
**Solution**: Run data extraction first: `python problem-statement/ps-{num}-{name}/src/scripts/01_extract_data.py`

### Logs
Check execution logs for detailed error information:
- ETL logs: `problem-statement/ps-{num}-{name}/logs/etl/`
- Error logs: `problem-statement/ps-{num}-{name}/logs/errors/`
- Audit logs: `problem-statement/ps-{num}-{name}/logs/audit/`
```

#### 7. Performance Considerations
```markdown
## Performance Considerations

### Data Size Recommendations
- **Small datasets** (< 100MB): Run on local machine
- **Medium datasets** (100MB - 1GB): Use lazy evaluation with Polars
- **Large datasets** (> 1GB): Consider Databricks cluster execution

### Optimization Tips
1. Use `pl.scan_csv()` instead of `pl.read_csv()` for lazy loading
2. Filter data early in the pipeline to reduce memory usage
3. Use `Categorical` dtype for disease names to save memory
4. Enable Polars' parallel processing for aggregations
```

### README Update Process

When updating README files, follow this process:

1. **Execute All Dependencies First**: Run prerequisite scripts to generate required data
2. **Execute the Code**: Run all implemented scripts/notebooks to verify actual execution flow
3. **Document Dependency Chain**: Record which scripts must run before notebooks/other scripts
4. **Document Actual Flow**: Record the actual sequence, inputs, outputs, and timings observed
5. **Capture Error Messages**: Note common errors encountered during testing (especially missing dependencies)
6. **Verify Instructions**: Test README instructions on a fresh environment to ensure accuracy
7. **Test Notebook Prerequisites**: Verify notebooks fail gracefully if dependencies haven't run

### README Update Verification

After updating README files, verify:

- ✅ Quick Start commands execute successfully on a fresh environment
- ✅ Execution Flow diagram matches actual code execution sequence
- ✅ **Execution Flow includes "Must run before" dependencies for each stage**
- ✅ **Notebook prerequisites are clearly documented (scripts that must run first)**
- ✅ All input/output file paths are accurate and exist
- ✅ Environment setup instructions are complete and tested
- ✅ Execution modes work as documented
- ✅ **Troubleshooting includes solutions for missing dependencies/data files**
- ✅ Troubleshooting section addresses actual errors encountered
- ✅ All code snippets in README are syntactically correct
- ✅ Dependencies listed match `requirements.txt`
- ✅ Configuration examples match actual config files
- ✅ **Test complete execution flow**: Run all dependencies → Run notebooks → Verify outputs
- ✅ **Verify notebooks fail gracefully** if prerequisites haven't been run (with helpful error messages)

## Documentation Requirements

The final output MUST include:
- Confirmation of completion if successful
- Results of all verification steps
- Any command outputs or test results
- The completed Design Implementation Verification Checklist
- Any noted discrepancies or issues
- **Updated README files documenting code execution flow** (see README Documentation Requirements above)
- Verification that README instructions have been tested and work correctly