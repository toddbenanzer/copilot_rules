# Project context

Environment-specific facts every mode needs to operate correctly.
Update this file whenever your project structure, tooling, or environment changes.

---

## Runtime environment

- **OS**: Linux (Ubuntu-based Docker container on WSL2)
- **File paths**: Always Linux-style (`/workspace/src/...`). Never assume
  Windows paths. Never use backslashes.
- **Python interpreter**: `/usr/local/bin/python` (or `.venv` if activated)
- **Virtual environment**: `/workspace/.venv`
  - Activate: `source /workspace/.venv/bin/activate`
  - Install editable: `pip install -e ".[dev,test]"`
- **Project root mounted at**: `/workspace`

---

## WSL2 / Docker operational rules

- **Always use `python -m <tool>`** rather than calling tools directly.
  This ensures the correct interpreter and environment are used.
  - ✅ `python -m pytest`
  - ✅ `python -m ruff check src/`
  - ✅ `python -m mypy src/`
  - ❌ `pytest`, `ruff`, `mypy` (may resolve to wrong environment)

- **File permissions**: Files created inside Docker may have root ownership
  on the WSL2 host. If permission errors occur, use:
  `docker exec --user $(id -u):$(id -g) ...`

- **Line endings**: Always LF. If editing from the Windows side of WSL2,
  confirm `.gitattributes` enforces `* text=auto eol=lf`.

- **Filesystem performance**: Keep project files in the WSL2 native
  filesystem (`~/projects/`), not under `/mnt/c/`. Operations on `/mnt/c/`
  are ~10x slower.

- **Port binding**: Use `0.0.0.0` when binding ports inside Docker containers
  for WSL2 accessibility. `localhost` inside Docker is not accessible from WSL2.

---

## Package structure

This project uses the `src/` layout:

```
project-root/
├── AGENTS.md
├── .kilocodemodes
├── .kilocodeignore
├── .kilocode/
│   ├── rules/
│   │   ├── python-standards.md     ← universal code quality rules
│   │   └── project-context.md      ← this file
│   ├── rules-implementer-bp/
│   │   └── coding-conventions.md
│   ├── rules-tester-bp/
│   │   └── testing-standards.md
│   └── rules-refactor-bp/
│       └── refactoring-guidelines.md
├── pyproject.toml
├── src/
│   └── <package_name>/
│       ├── __init__.py             ← public API exports + __all__
│       ├── py.typed                ← PEP 561 marker (empty file)
│       ├── exceptions.py           ← custom exception hierarchy
│       ├── models.py               ← Pydantic/dataclass data models
│       └── ...
└── tests/
    ├── conftest.py                 ← shared fixtures
    └── test_*.py
```

**The `src/` layout is mandatory.** Never place package files at the
project root. This prevents accidental imports of the development copy
and ensures installed behavior matches local behavior.

---

## Key commands

Run all commands from the project root with the virtual environment active.

```bash
# Install in editable mode with all dependency groups
pip install -e ".[dev,test]"

# Run tests (always use this form)
python -m pytest tests/ -v --tb=short

# Run tests with coverage report
python -m pytest tests/ --cov=src --cov-report=term-missing

# Run linter
python -m ruff check src/ tests/

# Run formatter (check only)
python -m black --check src/ tests/

# Run formatter (apply)
python -m black src/ tests/

# Run type checker
python -m mypy src/

# Run everything (pre-commit style)
python -m ruff check src/ tests/ && python -m black --check src/ tests/ && python -m mypy src/ && python -m pytest tests/ -v
```

---

## pyproject.toml baseline

Every project managed by this pipeline must have a `pyproject.toml`
conforming to this baseline. Update `[project]` fields per project.

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "<package-name>"
version = "0.1.0"
description = "<short description>"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [{ name = "<author>", email = "<email>" }]
dependencies = []

[project.optional-dependencies]
dev  = ["ruff>=0.4", "black>=24.0", "mypy>=1.10", "pre-commit"]
test = ["pytest>=8.0", "pytest-cov>=5.0", "pytest-mock>=3.14", "pytest-asyncio>=0.23"]
docs = ["mkdocs-material", "mkdocstrings[python]"]

[tool.hatch.build.targets.wheel]
packages = ["src/<package_name>"]

[tool.ruff]
target-version = "py311"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]
ignore = []

[tool.black]
line-length = 88
target-version = ["py311"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_unreachable = true
pretty = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"
asyncio_mode = "auto"

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
show_missing = true
skip_covered = false
fail_under = 80
```

---

## Tooling decisions (already resolved — do not re-debate)

These choices were made for this project and are not open for revision
during a pipeline run. If you believe a change is warranted, raise it
in a new Architect session, not mid-implementation.

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Build backend | `hatchling` | Modern, no `setup.py`, PEP 621 native |
| Formatter | `black` | Non-negotiable, zero-config |
| Linter | `ruff` | Replaces flake8+isort+pyupgrade in one tool |
| Type checker | `mypy` (strict) | Full type safety enforcement |
| Test framework | `pytest` | Ecosystem standard |
| HTTP client | `httpx` | Async-capable, replaces `requests` |
| Data validation (external) | `pydantic` v2 | Fast, typed, schema generation |
| Data containers (internal) | `dataclasses` | No dependency, slots support |
| Async framework | `asyncio` (stdlib) | No framework lock-in |

---

## What to update in this file

Update `project-context.md` when:

- The package name changes → update the directory tree
- A new tool is added to the stack → add it to "Tooling decisions"
- The Docker base image or Python version changes → update "Runtime environment"
- New standard commands are established → add to "Key commands"
- The project structure diverges from the baseline → update the tree

Do **not** use this file to override `python-standards.md` rules.
If a project-specific deviation from the universal standards is truly
necessary, document it here with an explicit justification and flag it
to the Architect at the start of any pipeline run.
