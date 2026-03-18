# Python standards

These rules apply to ALL modes in this pipeline. Every file produced,
modified, or reviewed must comply. No exceptions without explicit justification
documented in the ADR.

---

## Language and runtime

- **Target: Python 3.11+.** Use modern syntax throughout:
  - `X | Y` union types instead of `Union[X, Y]`
  - `X | None` instead of `Optional[X]`
  - `list[X]`, `dict[K, V]`, `tuple[X, ...]` instead of `typing` equivalents
  - `match / case` for multi-branch dispatch on structure
  - `tomllib` (stdlib) for TOML parsing
  - `Self` type from `typing` for fluent builder patterns

- **No compatibility shims** for Python < 3.11. Do not import from
  `typing_extensions` unless using a feature not yet in stdlib.

---

## Type system

- Type hints are **mandatory** on all function signatures and return types.
  No untyped public functions, ever.

- Use `collections.abc` types for parameters:
  `Sequence`, `Mapping`, `Callable`, `Iterable`, `Iterator`
  (prefer these over `list`, `dict` in signatures — accept broad, return specific)

- Use `TypeAlias` for complex type aliases:
  ```python
  JsonValue: TypeAlias = str | int | float | bool | None | dict[str, "JsonValue"] | list["JsonValue"]
  ```

- Use `TypeVar` and `Generic` for reusable generic containers.

- Use `Protocol` for structural typing instead of ABCs where duck typing
  is the actual intent.

- `Any` is a code smell. If you must use it, add a `# type: ignore[assignment]`
  comment with a brief explanation of why it's unavoidable.

- `cast()` should be rare and always accompanied by a comment explaining
  why the type checker can't infer it.

---

## Code style

- **Formatter: Black**, line length 88. Do not fight the formatter.
  If Black's output looks wrong, the structure needs to change, not the config.

- **Linter: Ruff** with default rule set. All warnings must be resolved
  before a step is considered complete. Run: `python -m ruff check src/`

- **Import order** (enforced by Ruff):
  1. Standard library
  2. Third-party packages
  3. Local/project imports
  Separated by blank lines. Never mix groups.

- **Absolute imports only.** No `from . import x` or `from .. import y`
  except within the same subpackage where relative imports are clearer.

- **f-strings** for all string interpolation. No `.format()`, no `%` formatting.

- **Raw strings** (`r"..."`) for all regex patterns.

- **No magic numbers.** Named constants or `enum.Enum` members only.

- **No walrus operator** (`:=`) unless it meaningfully reduces complexity.
  Clarity over cleverness.

---

## Documentation

- **Google-style docstrings** on all public functions, classes, and modules.
  Required sections:

  ```python
  def process(items: Sequence[str], limit: int = 100) -> list[str]:
      """Process a sequence of items up to a given limit.

      Args:
          items: The items to process. Order is preserved.
          limit: Maximum number of items to return. Must be positive.

      Returns:
          Processed items, deduplicated and trimmed, in original order.

      Raises:
          ValueError: If limit is not positive.
          ProcessingError: If any item fails validation.
      """
  ```

- **Module-level docstrings** on every `__init__.py` and every substantive
  module file. Explain purpose, key exports, and usage in 2–5 sentences.

- **No placeholder docstrings.** `"""TODO"""` or `"""..."""` are not acceptable.
  If you don't know what a function does yet, you haven't finished understanding
  the step — ask before implementing.

---

## Structure

- **Functions: ≤ 30 lines.** If longer, extract helper functions.
  Long functions are always doing too many things.

- **Classes: ≤ 200 lines.** If larger, split responsibilities into
  separate classes or move logic to module-level functions.

- **Files: ≤ 400 lines.** If larger, split into a subpackage.

- **`__init__.py`** controls the public API. Import key symbols explicitly.
  Use `__all__` to define what `from package import *` exposes.

- **`py.typed`** marker file must exist at the package root (PEP 561).
  An empty file is sufficient: `touch src/<package>/py.typed`

- **`exceptions.py`** for custom exception hierarchies. One file per
  package, with a base exception class the others inherit from:
  ```python
  class PackageError(Exception): ...
  class ConfigError(PackageError): ...
  class ValidationError(PackageError): ...
  ```

---

## Error handling

- **Never use bare `except:`** or `except Exception:` without immediately
  re-raising or logging with full traceback context.

- **Catch the most specific exception type possible.** If you're catching
  `OSError`, say so — don't use `Exception` and ignore the 50 other things
  that could raise it.

- **Include context in error messages.** Not: `"Invalid input"`.
  Yes: `f"Expected positive integer for limit, got {limit!r}"`.

- **Use `logging`** for all diagnostic output. Never `print()`.
  Module-level logger: `logger = logging.getLogger(__name__)`

- **`contextlib.suppress()`** is acceptable for intentionally swallowed
  exceptions when the intent is obvious from context.

---

## Patterns

**Prefer these:**

| Instead of                          | Use                                      |
|-------------------------------------|------------------------------------------|
| `os.path.join(a, b)`               | `Path(a) / b`                            |
| `os.path.exists(p)`                | `Path(p).exists()`                       |
| `open(f)` without context manager  | `with Path(f).open() as fh:`            |
| `if len(x) == 0:`                  | `if not x:`                              |
| `try/except KeyError` on dict      | `dict.get(key, default)`                 |
| Manual index loop                   | `for i, item in enumerate(items):`       |
| `isinstance(x, (A, B))`           | `isinstance(x, A \| B)` (3.10+)         |
| `typing.Optional[X]`              | `X \| None`                              |
| `typing.List[X]`                  | `list[X]`                                |
| Nested if/elif chains              | `match / case`                           |
| `functools.reduce` (complex)       | Explicit loop (more readable)            |
| Mutable default arguments          | `field(default_factory=list)` / `None`  |

**Use dataclasses for data containers:**
```python
from dataclasses import dataclass, field

@dataclass(frozen=True, slots=True)
class Config:
    host: str
    port: int = 8080
    tags: list[str] = field(default_factory=list)

    def __post_init__(self) -> None:
        if not 1 <= self.port <= 65535:
            raise ValueError(f"Port must be 1–65535, got {self.port}")
```

**Use `enum.Enum` for fixed value sets:**
```python
from enum import Enum, auto

class Status(Enum):
    PENDING = auto()
    RUNNING = auto()
    COMPLETE = auto()
    FAILED = auto()
```

**Use `contextlib.contextmanager` for resource patterns:**
```python
from contextlib import contextmanager
from collections.abc import Generator

@contextmanager
def managed_resource(config: Config) -> Generator[Resource, None, None]:
    resource = Resource(config)
    try:
        yield resource
    finally:
        resource.close()
```

---

## Dependencies

- **`pyproject.toml` only** (PEP 621). No `setup.py`, `setup.cfg`, or
  `requirements.txt` as the source of truth for dependencies.

- **Version ranges in `pyproject.toml`**, pinned exact versions in lock
  files only. Example: `"httpx>=0.27,<1.0"` not `"httpx==0.27.3"`.

- **Optional dependency groups:**
  ```toml
  [project.optional-dependencies]
  dev  = ["ruff", "black", "mypy", "pre-commit"]
  test = ["pytest>=8.0", "pytest-cov", "pytest-mock", "pytest-asyncio"]
  docs = ["mkdocs-material", "mkdocstrings[python]"]
  ```

- **Prefer stdlib** over third-party for simple tasks. Add a dependency
  only when the third-party library provides substantial value.

- **HTTP client: `httpx`** (async-capable). Not `requests` for new code.

- **Data validation: `pydantic` v2** for external data (API responses, config
  files). Plain `dataclasses` for internal data structures.
