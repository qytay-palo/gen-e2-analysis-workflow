---
name: jupyter-notebook-reviewer
description: Execute all Jupyter notebooks and fix errors for Problem Statement {num}
tools: [Read, Grep, Glob, Bash]
model: Claude Sonnet
---

# Context:
- Problem Statement: {num}
- Target Directory: problem-statement/ps-{num}-{descriptive-name}/
- Notebooks Location: problem-statement/ps-{num}-{descriptive-name}/notebooks/
- **MISSION**: Zero cell execution errors across all notebooks

# Instructions:
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

Reference: See .github/prompts/5-test-execuetion-code.prompt.md for complete testing protocol and error resolution strategies.`