---
mode: agent
description: "Write and run pytest tests for the implementation. Triages failures as implementation bugs vs test bugs."
tools:
  - codebase
  - readFiles
  - editFiles
  - runCommands
  - usages
---

Write and run pytest tests to verify the implementation matches the ADR and plan.

**ADR + Plan summary**:
${input:context:Paste the ADR and/or plan summary so the expected behavior is clear}

Process:
1. Read the ADR/plan (expected behavior, edge cases, function contracts)
2. Read the implemented source code
3. Write tests covering: happy path, edge cases, error paths, and ADR integration points
4. Run the tests and triage failures:
   - **Implementation bug**: code doesn't match the ADR/plan → report; do not fix
   - **Test bug**: wrong assumption in the test → fix and re-run

Test conventions:
- `tests/` mirrors `src/` | names: `test_<fn>_<scenario>` | shared fixtures in `conftest.py`
- `pytest.mark.parametrize` for 3+ input variations
- `mocker` fixture (pytest-mock) — assert call counts and arguments
- `pytest.raises(SpecificException, match="pattern")` for error paths
- `pytest.approx()` for floats
- Mock at boundaries (filesystem, network, DB) — not internal functions

Commands:
```bash
python -m pytest tests/ -v --tb=short
python -m pytest tests/ --cov=src --cov-report=term-missing
```

Output format:
```
# Test Results
**Total**: N | **Passed**: N | **Failed**: N | **Coverage**: N%

## Failures
- `test_name`: Expected [X], got [Y]
  → **Implementation bug**: [description]
  → **Test bug**: [fixed this run]

## Coverage gaps
[Public functions below 100% — or "none"]
```

Coverage targets: 80% line minimum; 100% on all public API functions.
