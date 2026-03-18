---
mode: agent
description: "Targeted fix for a bug, typo, or small edit. Self-scope-guards to ≤3 files. Use the full pipeline for anything larger."
tools:
  - codebase
  - readFiles
  - editFiles
  - runCommands
  - usages
---

I have a small, targeted fix needed.

**Issue**: ${input:issue:Describe the bug, typo, or small change}

Follow this process in strict order:
1. **Read** the affected files first — understand the current code before touching anything
2. **Confirm** your proposed fix using ranked multiple-choice before making any edit
3. **Apply** the fix one file at a time
4. **Lint**: `python -m ruff check <file>` after each edit
5. **Test** using the most targeted command possible (single test function preferred)
6. **Report** using the Quick Fix Report format:

```
**Quick Fix Report**
Issue: [what was wrong]
Fix: [what changed — file + line range]
Lint: ✅ clean / ⚠️ [resolved]
Test: `[command]` → ✅ passing / ❌ [what failed]
Scope: [N files — must be ≤3]
```

**Scope guard**: If the fix requires touching more than 3 files or involves design decisions, stop and tell me this needs the full pipeline instead.
