---
name: architecture
description: >
  Design technical architecture based on brainstorm results.
  Define component structure, data flow, and technology choices.
disable-model-invocation: true
---

# Architecture

Design the technical architecture based on the brainstorm design document.

## Prerequisites

- `.goose-artifacts/{branch}/design.md` — provides agreed requirements, constraints, and chosen approach from brainstorming.

If design.md is missing:
1. **Warn:** "design.md not found. Architecture decisions will lack grounding in agreed requirements."
2. **Ask:** proceed anyway, or run the `rub` skill first?
3. If the user confirms, proceed using their description and conversation context.

## Procedure

1. **Read design.md** (if available) to understand the agreed requirements and approach. If design.md is missing and the user chose to proceed, gather requirements from their description and codebase context.

2. **Design architecture** covering:
   - Component/module structure and responsibilities
   - Data flow between components
   - Technology choices with rationale
   - Key interfaces and contracts
   - File/directory structure

3. **Present section by section.** Ask the user to approve each section before moving on. If the user requests changes, revise and re-present.

4. **Save artifact** to `.goose-artifacts/{branch}/architecture.md` after full approval.

## Artifact Format

```
# Architecture: {Topic}

## Overview
{2-3 sentence summary of the architecture}

## Components

### {Component Name}
- **Responsibility:** {what it does}
- **Interface:** {how other components interact with it}
- **Dependencies:** {what it depends on}

## Data Flow
{Description or diagram of how data moves through the system}

## Technology Choices

| Choice | Technology | Rationale |
|--------|-----------|-----------|
| {area} | {tech}    | {why}     |

## File Structure
{Planned directory and file layout}
```

## Rules

- Read design.md before starting. Do not ask the user to repeat what was already decided.
- Present incrementally — do not dump the entire architecture at once.
- Resolve the branch name via `git branch --show-current` for the artifact path.
