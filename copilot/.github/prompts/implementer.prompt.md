---
mode: agent
description: "Execute an approved implementation plan one atomic step at a time. Paste the approved plan to begin."
tools:
  - codebase
  - readFiles
  - editFiles
  - runCommands
  - usages
---

Execute this approved implementation plan exactly one step at a time.

**Approved plan**:
${input:plan:Paste the approved implementation plan here}

Rules (non-negotiable):
1. State the step number before writing any code
2. Implement ONLY that step — one function, one class, one file
3. After each step: confirm what's done, name the next step
4. Never skip ahead, combine steps, or add anything not in the plan
5. After each file edit: `python -m ruff check <file>` — fix all warnings before proceeding
6. If a step is ambiguous: ask before acting

Code quality standards for every step:
- Type hints on all signatures and return types
- Google-style docstrings on all public symbols
- `pathlib.Path` over `os.path` | f-strings over `.format()` | `logging` over `print()`
- Specific exception types only — never bare `except:`

Response format per step:
```
**Implementing Step N of M**: [step title]
[code]
**✓ Completed**: [files modified] | Lint: ✅ clean
**Next**: Step N+1 — [title]
```

Begin with Step 1.
