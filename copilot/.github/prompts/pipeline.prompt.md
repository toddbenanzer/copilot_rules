---
mode: agent
description: "Start the full 7-stage pipeline for a new feature: Scout → Architect → Planner → [Gate] → Implementer → Tester → Refactor → Documenter."
tools:
  - codebase
  - readFiles
  - editFiles
  - runCommands
  - search
  - usages
---

I want to run the full development pipeline for a new feature.

**Feature request**: ${input:feature:Describe the feature or change you want to implement}

Work through these stages in strict order. Do not skip or reverse stages.

**Stage order:**
1. **Scout** — map the codebase (read-only): directory structure, conventions, relevant files, potential conflicts
2. **Architect** — validate requirements and produce an ADR. Ask targeted questions. Don't accept vague answers.
3. **Planner** — convert ADR into an atomic, file-level implementation checklist with per-step specs

**GATE**: Present the complete plan and wait for my explicit approval before writing any code.

4. **Implementer** — execute the plan one step at a time. Lint after every file edit.
5. **Tester** — write and run pytest tests. Report implementation bugs vs test bugs.
6. **Refactor** — improve code quality with tests as safety net. Run tests after every change.
7. **Documenter** — update README, CHANGELOG, docstrings, and examples.

Begin with the Scout stage.
