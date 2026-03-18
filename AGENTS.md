# AGENTS.md

This file describes the AI-assisted development workflow for this project.
It is read by Kilo Code, and is compatible with other agentic coding tools
(OpenAI Codex, Claude Code, etc.) that recognize the AGENTS.md convention.

---

## Overview

This project uses a **7-stage development pipeline** implemented as custom
modes in Kilo Code (VS Code). Every non-trivial change — new features, new
modules, significant refactors — should flow through the full pipeline.

The pipeline is designed around one principle: **catch mistakes at the
cheapest possible stage.** A wrong assumption costs 30 seconds in Scout.
It costs hours if discovered during testing.

```
🔄 Pipeline (orchestrator)
    │
    ├── 1. 🔍 Scout        — map the codebase before any decisions are made
    ├── 2. 🏛️ Architect    — validate requirements, produce ADR
    ├── 3. 📋 Planner      — convert ADR into atomic implementation checklist
    ├── G. [GATE]          — human approves plan before any code is written
    ├── 4. ⚡ Implementer  — execute plan one step at a time
    ├── 5. 🧪 Tester       — write and run tests; establish safety net
    ├── 6. 🔧 Refactor     — improve quality with tests as safety net
    └── 7. 📝 Documenter   — update docs, docstrings, CHANGELOG
```

---

## How to start the pipeline

Open Kilo Code in VS Code, select the **🔄 Pipeline** mode, and describe
what you want to build. The orchestrator will ask two scoping questions
then drive all seven stages automatically, pausing at the Gate for your
approval before any code is written.

To run a stage directly (bypassing the orchestrator), switch to its mode
using the `/slug` command:

```
/scout-bp
/architect-bp
/planner-bp
/implementer-bp
/tester-bp
/refactor-bp
/documenter-bp
```

---

## Stage reference

### 🔍 Scout (`scout-bp`)
**Purpose**: Read and map the existing codebase before any design decisions.
**Input**: User request + read access to the project.
**Output**: Scout Report — directory map, conventions in use, relevant files,
potential conflicts, conventions the new code must honor.
**Key constraint**: Read-only. No design, no planning, no code.
**When to skip**: Never. Even on new projects it establishes conventions.

---

### 🏛️ Architect (`architect-bp`)
**Purpose**: Pressure-test the request and produce a validated Architecture
Decision Record (ADR).
**Input**: Scout Report + user request.
**Output**: ADR with Context, Decision, Alternatives Considered, Consequences,
Scope (In/Out), Constraints from codebase, and Open Questions.
**Key constraint**: Read-only. Asks ranked multiple-choice questions to surface
tradeoffs. Does not proceed to ADR until requirements are sharp.
**Gate check**: ADR must contain all required sections before Planner runs.

---

### 📋 Planner (`planner-bp`)
**Purpose**: Convert the validated ADR into a precise, ordered implementation
checklist with file-level granularity.
**Input**: ADR + Scout Report.
**Output**: Numbered checklist where every step names a file path, function,
input/output types, edge cases, dependencies, and convention notes.
**Key constraint**: Read-only. Each step is one atomic unit of work.
**Gate check**: Plan must have ≥3 steps with file-level detail. This output
is presented to the user for approval before implementation begins.

---

### [GATE] — Human approval
**Purpose**: Explicit human sign-off on the plan before any code is written.
**The orchestrator stops here.** It presents the plan and waits for approval.
Accepted signals: "yes", "proceed", "looks good", "approved", or equivalent.
If changes are requested, the plan goes back to Planner and is re-presented.
This is the last cheap moment to change direction.

---

### ⚡ Implementer (`implementer-bp`)
**Purpose**: Execute the approved plan exactly one step at a time.
**Input**: Approved plan + ADR + Scout Report.
**Output**: Implemented source files, one step per turn.
**Key constraint**: Implements ONE step per response. Does not skip, combine,
or add anything not in the plan. Asks a ranked multiple-choice question only
when a step is a genuine blocker — otherwise assumes and proceeds.
**Recovery**: If tester-bp finds implementation bugs, a targeted fix task is
sent back here for only the failing items.

---

### 🧪 Tester (`tester-bp`)
**Purpose**: Write and run pytest tests to verify correctness. Runs BEFORE
Refactor to establish a safety net.
**Input**: Implemented source + ADR + plan.
**Output**: Test suite in `tests/` + Test Results report (passed/failed/coverage).
**Key constraint**: Can only edit files matching `tests/|test_|conftest\.py`.
Does not fix implementation bugs — triages and reports them.
**Coverage targets**: 80% line coverage minimum; 100% on all public API functions.

---

### 🔧 Refactor (`refactor-bp`)
**Purpose**: Improve code quality with a passing test suite as a safety net.
**Input**: Passing test results + source code.
**Output**: Refactored source with all High/Medium findings resolved. Tests
must still pass after every individual change.
**Key constraint**: Runs tests after each change. Reverts any change that
breaks the suite. Does not apply Low severity findings without user approval
(asks via ranked multiple-choice, batched at end of run).
**Sequencing note**: Deliberately runs AFTER Tester, not before. You refactor
code that is already proven correct, never untested code.

---

### 📝 Documenter (`documenter-bp`)
**Purpose**: Update all documentation artifacts after code is final.
**Input**: Final source + ADR + test results + refactor summary.
**Output**: Updated docstrings, README section, CHANGELOG entry, and
usage examples (scope depends on package vs script, confirmed via question).
**Key constraint**: Documentation only — does not touch source code.
All usage examples must be runnable.

---

## Model routing

Each mode runs on a different OpenRouter model, chosen to balance quality
and cost for that stage's computational demands.

| Mode           | Recommended model                    | Why                                      |
|----------------|--------------------------------------|------------------------------------------|
| scout-bp       | `google/gemini-2.5-flash-preview`    | Fast, long context, read-heavy work      |
| architect-bp   | `anthropic/claude-sonnet-4-5`        | Deep reasoning, design tradeoffs         |
| planner-bp     | `google/gemini-2.5-pro-preview`      | Strong structured output, long context   |
| implementer-bp | `deepseek/deepseek-chat-v3-0324`     | Near-frontier code quality, 1/20th cost  |
| tester-bp      | `anthropic/claude-3-5-haiku`         | Fast iteration, strong SWE-bench score   |
| refactor-bp    | `anthropic/claude-sonnet-4-5`        | Multi-file reasoning, intent inference   |
| documenter-bp  | `google/gemini-2.5-flash-preview`    | Fast, cheap, write-heavy task            |

Configure models in Kilo Code: gear icon → API Configuration → select
OpenRouter → set model per mode using the sticky model feature.

---

## Questioning convention

Every mode in this pipeline uses a consistent ranked multiple-choice format
when it needs input from the user. Options are ordered from most recommended
to least. A free-response option is always available as the final choice.

```
> **[Question]**
> I'd go with **A** because [rationale].
> - **A) [Best option] ⭐ recommended** — [tradeoff summary]
> - **B) [Second option]** — [tradeoff summary]
> - **C) Other** — tell me what you're thinking
```

You are never locked into the lettered options. Free-text responses are
always accepted and honored.

---

## File structure

```
project-root/
├── AGENTS.md                          ← this file
├── .kilocodemodes                     ← all 8 mode definitions (YAML)
├── .kilocodeignore                    ← excludes .venv, __pycache__, etc.
│
├── .kilocode/
│   ├── rules/                         ← apply to ALL modes
│   │   ├── python-standards.md        ← universal Python quality rules
│   │   └── project-context.md         ← environment, paths, package config
│   │
│   ├── rules-implementer-bp/          ← implementer-only rules
│   │   └── coding-conventions.md
│   ├── rules-tester-bp/               ← tester-only rules
│   │   └── testing-standards.md
│   └── rules-refactor-bp/             ← refactor-only rules
│       └── refactoring-guidelines.md
│
├── src/
│   └── <package_name>/
│       ├── __init__.py
│       ├── py.typed
│       └── ...
│
└── tests/
    ├── conftest.py
    └── ...
```

### Rules hierarchy

- **Universal rules** (`.kilocode/rules/`) — injected into every mode's
  system prompt. Keep these lean. Only put things here that truly apply
  everywhere.
- **Mode-specific rules** (`.kilocode/rules-{slug}/`) — injected only into
  that mode. Use for constraints that would be noise in other modes.
- **`customInstructions`** in `.kilocodemodes` — behavioral rules baked
  into the mode definition itself (questioning protocol, output format).
  Do not duplicate these in rules files.

---

## Conventions enforced by this pipeline

The following are non-negotiable and enforced across all modes via rules files.
Any code produced by this pipeline must comply.

- **Python 3.11+** — use modern syntax (`X | Y` unions, `match`, `tomllib`)
- **Type hints** on all public function signatures and return types
- **Google-style docstrings** on all public functions, classes, and modules
- **Black** formatter (line length 88); **Ruff** linter (default rules)
- **`pathlib.Path`** over `os.path`; f-strings over `.format()`
- **Specific exception types** — never bare `except:`
- **`logging` module** — never `print()` for diagnostics
- **`src/` layout** — no flat layout packages
- **Absolute imports** — no relative imports except within the same subpackage
- **Functions ≤ 30 lines**, classes ≤ 200 lines, files ≤ 400 lines

---

## Adding or modifying modes

1. Edit `.kilocodemodes` at the project root.
2. Update this file's Stage Reference section to reflect the change.
3. If the mode has universal rules implications, update
   `.kilocode/rules/python-standards.md` or `project-context.md`.
4. If adding a new mode-specific constraint, create a new
   `.kilocode/rules-{new-slug}/` directory with a Markdown file.

Kilo Code picks up `.kilocodemodes` changes on next reload (Cmd/Ctrl+Shift+P
→ "Kilo Code: Reload Configuration").

---

## What this pipeline does NOT handle

- **Database migrations** — run these manually; never delegate to an agent
- **Secret management** — never put credentials in any file the agent can read
- **Deployment / CI** — pipeline produces code; shipping is a separate concern
- **Breaking API changes** — the Refactor mode is constrained from changing
  public signatures; intentional breaking changes require a new Architect run
