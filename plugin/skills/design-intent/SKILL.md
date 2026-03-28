---
name: design-intent
description: >
  Document design intent. Record key design decisions,
  trade-offs, intentional exclusions, and caveats.
disable-model-invocation: true
context: fork
---

# Design Intent

Document the design intent so that future reviewers understand WHY decisions were made, not just WHAT was built.

## Prerequisites

- `.goose-artifacts/{branch}/design.md` must exist.
- `.goose-artifacts/{branch}/architecture.md` must exist.

## Procedure

1. **Read** design.md and architecture.md.

2. **Draft intent.md** covering:
   - Task overview (what is being built and why)
   - Key design decisions with alternatives considered
   - Intentional exclusions (what was deliberately NOT built)
   - Caveats and known limitations

3. **Present the draft** to the user along with a list of ambiguous discussion points that need clarification. Format discussion points as:
   - `[DISCUSS] {topic}: {why this needs clarification}`

4. **Incorporate feedback** and save to `.goose-artifacts/{branch}/intent.md`.

## Artifact Format

```
# Design Intent

## Task Overview
{What is being built and the motivation behind it}

## Key Design Decisions

### Decision 1: {title}
- **Options considered:** {A vs B vs C}
- **Chosen:** {which option}
- **Rationale:** {why this option was selected}

## Intentional Exclusions
- {Feature/approach X} — {why it was excluded}

## Caveats
- {Known limitation or risk and its mitigation}
```

## Rules

- This skill runs in a forked context. Only create artifact files — do NOT modify source code.
- Always surface discussion points. Do not silently resolve ambiguity.
- Resolve the branch name via `git branch --show-current` for the artifact path.
