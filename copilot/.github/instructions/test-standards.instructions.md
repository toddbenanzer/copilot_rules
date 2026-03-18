---
applyTo: "{tests/**/*.py,**/test_*.py,**/conftest.py}"
---

# Test standards — test files

## Commands
```bash
python -m pytest tests/ -v --tb=short                              # full suite
python -m pytest tests/ --cov=src --cov-report=term-missing        # with coverage
python -m pytest tests/test_module.py::test_fn_scenario -v         # targeted
python -m pytest -k "keyword" -v                                   # by keyword
```
Coverage targets: 80% line minimum; 100% on all public API functions.

## Conventions
- Mirror source: `src/pkg/module.py` → `tests/test_module.py`
- Names: `test_<function>_<scenario>`
- Shared fixtures in `conftest.py` — one per directory level
- `pytest.mark.parametrize` for 3+ input variations
- `mocker` fixture (pytest-mock) — not `unittest.mock.patch` decorators
- Assert mock call counts and arguments, not just that mocks were called
- `pytest.raises(SpecificException, match="pattern")` for error paths
- `pytest.approx()` for floats
- Mock at boundaries (filesystem, network, DB) — not internal functions
- Test public API only — never private functions (prefix `_`)
