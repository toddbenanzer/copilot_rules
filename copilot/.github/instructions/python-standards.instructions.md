---
applyTo: "src/**/*.py"
---

# Python standards — src files

## Types (mandatory)
- Hints on all signatures and return types. `X | None` not `Optional[X]`. `list[X]` not `List[X]`.
- Parameters: use `collections.abc` types (`Sequence`, `Mapping`, `Callable`).
- `Any` requires an inline comment explaining why. `cast()` should be rare.

## Style
- Black-formatted (line-length 88). Ruff clean before marking any step done.
- Absolute imports only. f-strings for interpolation. Raw strings for regex.

## Docstrings (Google style, all public symbols)
```python
def fn(items: Sequence[str], limit: int = 100) -> list[str]:
    """One-line summary.

    Args:
        items: Description.
        limit: Must be positive.

    Returns:
        Deduplicated items in original order.

    Raises:
        ValueError: If limit is not positive.
    """
```

## Structure
- Functions ≤ 30 lines. Classes ≤ 200 lines. Files ≤ 400 lines.
- `pathlib.Path` over `os.path`. `logging` over `print()`.
- Specific exception types only — never bare `except:`.
- Custom exceptions in `exceptions.py` with a base class.
- `enum.Enum` for fixed value sets.

## Prefer
| Instead of | Use |
|---|---|
| `os.path.join(a, b)` | `Path(a) / b` |
| `Optional[X]` | `X \| None` |
| `List[X]` / `Dict[K,V]` | `list[X]` / `dict[K,V]` |
| `isinstance(x, (A, B))` | `isinstance(x, A \| B)` |
| `if len(x) == 0:` | `if not x:` |
| Nested if/elif chains | `match / case` |
