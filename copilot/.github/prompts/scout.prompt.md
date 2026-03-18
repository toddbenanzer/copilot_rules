---
mode: ask
description: "Map the existing codebase before any design decisions. Produces a Scout Report for use in Architect stage."
tools:
  - codebase
  - readFiles
  - search
  - usages
---

Map this codebase to ground all downstream decisions in reality. Read-only — no design, planning, or code suggestions.

**Context**: ${input:context:What feature or change is being considered? (used to focus the scan)}

Map the following:
1. Directory structure (skip `.venv`, `__pycache__`, `.git`, `dist`)
2. Conventions: naming, imports, type hints, error handling, async usage, docstring style
3. Dependencies (`pyproject.toml`)
4. Test setup: framework, fixture patterns, mock style
5. Files most likely touched by the requested change
6. Potential conflicts: naming collisions, tight coupling, missing abstractions
7. Non-negotiable conventions any new code must match

Output format:
```
# Scout Report

## Project structure
[Condensed tree]

## Conventions in use
- Naming: [pattern]
- Imports: [style]
- Types: [approach]
- Errors: [handling pattern]
- Async: [usage]
- Tests: [framework + patterns]

## Relevant existing code
[File path: reason it matters]

## Potential conflicts
[Specific friction points — or "none identified"]

## Conventions new code must match
[Bulleted non-negotiables]
```

No speculation. If unsure, say so.
