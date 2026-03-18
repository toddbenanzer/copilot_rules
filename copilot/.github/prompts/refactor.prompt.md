---
mode: agent
description: "Improve code quality with a passing test suite as safety net. Runs tests after every change."
tools:
  - codebase
  - readFiles
  - editFiles
  - runCommands
  - usages
---

Review and improve code quality using the passing test suite as a safety net. No new features. No public API changes.

**Files to review**:
${input:files:List the files or modules to refactor (e.g., src/pkg/module.py)}

Hard constraint: Run `python -m pytest tests/ -v` after **every individual change**. If a change breaks tests: revert immediately and report.

Review checklist (work through in this priority order):
1. **DRY** — duplicated logic to extract
2. **Naming** — unclear or inconsistent identifiers
3. **Complexity** — cyclomatic complexity > 10
4. **Type safety** — missing hints, overuse of `Any`, unnecessary `cast()`
5. **Error handling** — bare excepts, swallowed exceptions, missing error paths
6. **Python idioms** — non-Pythonic patterns
7. **Docstrings** — missing, outdated, or mismatched signatures
8. **Security** — hardcoded secrets, unsafe deserialization, path traversal
9. **Performance** — N+1 patterns, unnecessary copies

Apply High and Medium findings without asking. For Low severity: ask first (batch at end, max 3 per turn).

Finding format:
```
### Finding N: [title]
**File**: `path/to/file.py` lines X–Y | **Severity**: High / Medium / Low
**Issue**: [why it's a problem]
**Fix**: [refactored code]
**Tests after fix**: ✅ passing / ❌ [what broke — reverting]
```

Summary format when done:
```
## Refactor Summary
Findings fixed: N (H: N, M: N) | Low suggestions deferred: [list or "none"]
Final test status: N passing, N% coverage
```
