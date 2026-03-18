---
mode: ask
description: "Convert a validated ADR into an atomic implementation checklist. Paste the ADR to begin. Output requires your approval before any code is written."
tools:
  - codebase
  - readFiles
  - search
---

Convert this Architecture Decision Record into a precise, ordered implementation checklist. No code — only specifications.

**ADR**:
${input:adr:Paste the Architecture Decision Record here}

Each step must be one atomic unit: one function, one class, one config change, or one file scaffold.

Per-step spec must include:
- Full file path
- Function/class name
- Input types and output types
- Behavior description
- Edge cases to handle
- Dependency on prior steps
- Convention note (matches existing conventions, or deviation with reason)
- ⚠️ Risk flag if the step is tricky or has broad impact

Output format:
```markdown
# Implementation Plan
**Feature**: [name] | **ADR date**: [date] | **Total steps**: [N]

## Dependencies to add
[pyproject.toml changes — or "none"]

## Phase 1: [name]
- [ ] **Step 1**: Create `src/pkg/module.py` — implement `function_name`
  - Inputs: `param: Type`
  - Returns: `Type`
  - Behavior: [description]
  - Edge cases: [list]
  - Depends on: nothing / Step N
  - Convention note: [matches / deviation: reason]
  - ⚠️ Risk: [if applicable]

## Test checkpoints
- After Step N: `python -m pytest tests/test_X.py -v`

---
**Gate note**: [decisions warranting explicit confirmation — or "none, proceed when ready"]
```

⬆️ **Review this plan carefully. Only proceed to implementation after you have approved it.**
