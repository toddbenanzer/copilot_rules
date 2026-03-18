# Workspace instructions

## Pipeline

Use the **Pipeline** agent for new features. Use **Quick Fix** for targeted bug fixes.
Stage order: Scout → Architect → Planner → [Gate: your approval] → Implementer → Tester → Refactor → Documenter.

## Environment

- Python 3.11+, Docker container on WSL2, project root at `/workspace`
- Venv: `/workspace/.venv` — activate: `source /workspace/.venv/bin/activate`
- Always use `python -m <tool>` — never bare `pytest`, `ruff`, `mypy`
- Linux paths only. Install: `pip install -e ".[dev,test]"`

## Toolchain (resolved — do not re-debate)

| Concern | Tool |
|---------|------|
| Build | hatchling / pyproject.toml (PEP 621) |
| Format | black, line-length 88 |
| Lint | ruff (default rules) |
| Types | mypy --strict |
| Tests | pytest + pytest-cov + pytest-mock |
| HTTP | httpx (not requests) |
| External data | pydantic v2 |
| Internal data | dataclasses |

## Key commands

```bash
python -m pytest tests/ -v --tb=short
python -m pytest tests/ --cov=src --cov-report=term-missing
python -m ruff check src/ tests/
python -m black --check src/ tests/
python -m mypy src/
```

## Questioning convention

When you need input before proceeding, use ranked multiple-choice — options ordered best to worst, free-response always last.

> **[Question]**
> I'd go with **A** — [rationale].
> - **A) [Best option] ⭐** — [tradeoff]
> - **B) [Second option]** — [tradeoff]
> - **C) Other** — tell me what you're thinking
