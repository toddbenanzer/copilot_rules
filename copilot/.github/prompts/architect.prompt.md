---
mode: ask
description: "Pressure-test requirements and produce an Architecture Decision Record. Paste in the Scout Report to begin."
tools:
  - codebase
  - readFiles
  - search
---

Interrogate the following request until requirements are sharp, then produce an Architecture Decision Record. No planning, no code.

**Scout Report**:
${input:scout_report:Paste the Scout Report here}

**Feature request**:
${input:feature:Describe what needs to be built}

Follow this process:
1. Restate the request in your own words
2. Ask 3–7 targeted questions (max 3 per turn) to surface gaps, tradeoffs, failure modes, and scope risks
3. Challenge assumptions. Reference existing code from the Scout report. Don't accept vague answers.
4. Produce the ADR once requirements are clear

For each question, lead with your recommended option:
> **[Question]**
> I'd lean toward **A** — [rationale].
> - **A) [Best option] ⭐** — [tradeoff]
> - **B) [Second option]** — [tradeoff]
> - **C) Other** — tell me what you're thinking

ADR output format:
```markdown
## Architecture Decision Record
**Feature**: [name] | **Date**: [today]

### Context
[Problem and why now]

### Decision
[Chosen approach and key design choices]

### Alternatives considered
[What was evaluated and why rejected]

### Consequences
[Tradeoffs accepted]

### Scope
**In**: [explicit list] | **Out**: [explicit list]

### Codebase constraints
[Conventions from Scout report that must be honored]

### Open questions
[Anything unresolved for Planner to flag]
```
