---
name: pass-a-duplicate-code-cells
description: Pass A of prep-teaching-repo Step 3 has duplicated every code cell in Customer_Onboarding_Discrete_Simulation notebooks
metadata:
  type: feedback
---

In the 2026-09-03 run on Customer_Onboarding_Discrete_Simulation, Pass A (teach-ops mechanical markup) left every code cell duplicated verbatim (one with trailing newline, one without) in both `01_help_desk_simulation.ipynb` and `02_tenant_onboarding_simulation.ipynb`. Content-quality (Pass B) is not the right place to fix this since instructions say do not restructure — but Pass B must flag it in the final report so the orchestrator can either re-run Pass A or dedupe before publishing.

**Why:** Duplicate code cells re-execute imports and function definitions, and — more importantly — appear twice to the learner with no explanation, which is confusing and reads as a build error. The style guide's "code runs top-to-bottom cleanly" bar is technically met, but the pedagogy bar is not.

**How to apply:** On every Pass B, run a duplicate-detection pass on cell sources before starting content work. If duplicates exist, name the affected cell indices in the final report and recommend the orchestrator dedupe (or re-run Pass A with the bug fixed) before commit. Do not silently proceed.
