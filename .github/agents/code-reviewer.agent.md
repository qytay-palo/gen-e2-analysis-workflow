---
name: code-reviewer
description: Comprehensive code review for Problem Statement {num}
tools: ['read', 'execute', 'edit', 'search'] # specify the tools this agent can use. If not set, all enabled tools are allowed.
model: GPT-5.4
---

Execute a comprehensive code quality review and fix ALL identified issues.

# Context:
- Target Directory: problem-statement/ps-{num}-{descriptive-name}/
- Review ALL Python files (.py) AND ALL Jupyter notebooks (.ipynb)
- Fix issues immediately
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