# python-ai-dev-pipeline

A 7-stage AI-assisted Python development pipeline with parallel configurations for **Kilo Code** (VS Code extension via OpenRouter) and **GitHub Copilot Enterprise** (VS Code native).

Both tools run the same workflow with consistent behavior, model routing, and quality standards — so you can switch between them or use both on the same project.

---

## The pipeline

```
Scout → Architect → Planner → [Gate: your approval] → Implementer → Tester → Refactor → Documenter
```

| Stage | Role | Reads code? | Writes code? |
|-------|------|:-----------:|:------------:|
| 🔍 Scout | Maps existing codebase before any decisions | ✅ | ❌ |
| 🏛️ Architect | Pressure-tests requirements, produces ADR | ✅ | ❌ |
| 📋 Planner | Converts ADR into atomic checklist | ✅ | ❌ |
| **[Gate]** | **You review and approve the plan** | — | — |
| ⚡ Implementer | Executes plan one step at a time | ✅ | ✅ |
| 🧪 Tester | Writes and runs pytest suite | ✅ | tests only |
| 🔧 Refactor | Improves quality with tests as safety net | ✅ | ✅ |
| 📝 Documenter | Updates README, docstrings, CHANGELOG | ✅ | docs only |

Also included: **⚡ Quick Fix** — a bypass mode for targeted bug fixes that scope-guards itself to ≤3 files.

---

## Repository structure

```
python-ai-dev-pipeline/
├── AGENTS.md                          ← pipeline reference (cross-tool)
│
├── kilo-code/                         ← copy contents to your project root
│   ├── .kilocodemodes                 ← all 9 mode definitions
│   └── .kilocode/
│       └── rules/
│           ├── python-standards.md    ← universal Python quality rules
│           └── project-context.md     ← environment, paths, toolchain
│
└── copilot/                           ← copy contents to your project root
    ├── .github/
    │   ├── copilot-instructions.md    ← always-on workspace context
    │   ├── agents/                    ← 9 custom agents (.agent.md)
    │   └── instructions/              ← scoped instruction files
    └── .vscode/
        └── settings.json              ← Copilot + Python toolchain config
```

---

## Setup

### Kilo Code

1. Install [Kilo Code](https://marketplace.visualstudio.com/items?itemName=kilocode.kilo-code) in VS Code
2. Copy `kilo-code/` contents into your project root:
   ```bash
   cp -r python-ai-dev-pipeline/kilo-code/. your-project/
   ```
3. Configure OpenRouter in Kilo Code settings (gear icon → API Configuration)
4. Update `.kilocode/rules/project-context.md` with your package name and paths
5. Start a task with the **🔄 Pipeline** mode

### GitHub Copilot

1. Ensure you have Copilot Enterprise with Claude and Gemini models enabled by your org admin
2. Copy `copilot/` contents into your project root:
   ```bash
   cp -r python-ai-dev-pipeline/copilot/. your-project/
   ```
3. Merge `.vscode/settings.json` with any existing settings you have
4. Open Copilot Chat → agents dropdown → select **Pipeline**

---

## Model routing

### Kilo Code (OpenRouter)

| Stage | Model | Cost |
|-------|-------|------|
| Scout | `google/gemini-2.5-flash-preview` | ~$0.001 |
| Architect | `anthropic/claude-sonnet-4-5` | ~$0.015 |
| Planner | `google/gemini-2.5-pro-preview` | ~$0.010 |
| Implementer | `deepseek/deepseek-chat-v3-0324` | ~$0.001 |
| Tester | `anthropic/claude-3-5-haiku` | ~$0.004 |
| Refactor | `anthropic/claude-sonnet-4-5` | ~$0.015 |
| Documenter | `google/gemini-2.5-flash-preview` | ~$0.001 |
| Quick Fix | `deepseek/deepseek-chat-v3-0324` | ~$0.001 |

Typical feature cost: **$2–8 total** end-to-end.

### GitHub Copilot Enterprise (premium request multipliers)

| Stage | Model | Multiplier |
|-------|-------|:----------:|
| Pipeline | GPT-4.1 | **0x** (free) |
| Scout | GPT-4.1 | **0x** (free) |
| Architect | Claude Sonnet 4.5 | 1x |
| Planner | GPT-4.1 | **0x** (free) |
| Implementer | GPT-4.1 | **0x** (free) |
| Tester | Claude Haiku 4.5 | 0.33x |
| Refactor | Claude Sonnet 4.5 | 1x |
| Documenter | GPT-4.1 | **0x** (free) |
| Quick Fix | GPT-4.1 | **0x** (free) |

Only Architect and Refactor spend premium requests. A full pipeline run costs **~2–4 premium requests** depending on conversation length.

> **Note for Enterprise admins:** Claude Sonnet 4.5 and Claude Haiku 4.5 must be enabled in your org's Copilot model policy before they appear in VS Code.

---

## Interaction convention

Every stage uses ranked multiple-choice questions when it needs your input — options ordered best to worst, with a free-response option always last:

```
> **[Question]**
> I'd go with **A** — [rationale].
> - **A) [Best option] ⭐** — [tradeoff]
> - **B) [Second option]** — [tradeoff]
> - **C) Other** — tell me what you're thinking
```

You're never locked into a lettered option. Free-text answers are always accepted.

---

## Customizing for your project

**Required changes before first use:**

- `kilo-code/.kilocode/rules/project-context.md` — update package name, Docker paths, Python version
- `copilot/.github/copilot-instructions.md` — update package name and any project-specific toolchain choices

**Optional:**

- Add mode-specific rules to `.kilocode/rules-{slug}/` directories
- Add `.kilocode/skills/` directories for reusable context packs
- Adjust model assignments in `.kilocodemodes` or agent frontmatter to match your quality/cost preferences

---

## Python standards enforced

All code produced by this pipeline must comply:

- Python 3.11+ | `pyproject.toml` (PEP 621) | `hatchling` build backend
- Type hints on all public signatures | Google-style docstrings
- `black` (line-length 88) + `ruff` (default rules) + `mypy --strict`
- `pathlib.Path` over `os.path` | `logging` over `print()` | `httpx` over `requests`
- Functions ≤ 30 lines | Classes ≤ 200 lines | Files ≤ 400 lines
- `src/` layout | `py.typed` marker | Custom exceptions in `exceptions.py`

---

## Contributing

Improvements welcome. If you adjust model assignments, update the routing table in this README to keep it accurate.
