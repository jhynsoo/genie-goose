---
name: rub
description: >
  Rub the lamp — refine requirements through collaborative brainstorming.
  Identify purpose, constraints, and success criteria,
  then propose 2-3 approaches with trade-offs.
disable-model-invocation: true
---

# Rub

Develop the user's idea into a concrete design through collaborative dialogue.

## Procedure

1. **Understand the idea:** Ask clarifying questions one at a time. Prefer multiple-choice questions when possible. Focus on: purpose, constraints, success criteria.

2. **Explore approaches:** Once you understand the requirements, propose 2-3 different approaches with trade-offs. Lead with your recommended option and explain why.

3. **Converge on design:** Present the design section by section. Ask after each section whether it looks right. Cover: architecture, components, data flow, error handling.

4. **Save artifact:** Once the user approves the design, save it to `.goose-artifacts/{branch}/design.md`.

## Artifact Format

```
# Design: {Topic}

## Problem Statement
{What we're solving and why}

## Requirements
{Agreed-upon requirements from the brainstorming session}

## Chosen Approach
{Description of the selected approach}

### Why This Approach
{Reasoning and trade-offs considered}

### Alternatives Considered
{Other approaches and why they were not chosen}
```

## Rules

- One question per message. Do not overwhelm with multiple questions.
- Multiple choice preferred — easier to answer than open-ended.
- YAGNI ruthlessly — remove unnecessary features from all designs.
- Always propose 2-3 approaches before settling on one.
- Resolve the branch name via `git branch --show-current` for the artifact path.
