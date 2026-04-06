---
name: intent
description: >
  Document design intent and detect conflicts with existing
  conventions and decisions. Proposes updates to conventions.yaml
  and decisions.yaml with MUST/RECOMMENDED grades.
disable-model-invocation: true
context: fork
---

# Design Intent

Document the design intent so that future reviewers understand WHY decisions were made. Detect conflicts with existing conventions and architecture decisions, and propose updates.

## Prerequisites

- `.goose-artifacts/{branch}/design.md` — provides agreed requirements and chosen approach.
- `.goose-artifacts/{branch}/architecture.md` — provides component structure and data flow.
- `.goose/conventions.yaml` — optional but recommended for conflict detection.
- `.goose/decisions.yaml` — optional but recommended for conflict detection.

If design.md or architecture.md is missing:
1. **Warn:** "Missing artifacts reduce context for intent documentation. Without design.md/architecture.md, design decisions may lack grounding."
2. **Ask:** proceed anyway, or run the prerequisite step first?
3. If the user confirms, proceed using available context.

## Procedure

1. **Read** whatever artifacts exist (design.md, architecture.md, conventions.yaml, decisions.yaml). If some are missing and the user chose to proceed, work from available context and conversation history.

2. **Draft intent.md** covering:
   - Task overview (what is being built and why)
   - Key design decisions with alternatives considered
   - Intentional exclusions (what was deliberately NOT built)
   - Caveats and known limitations

3. **Analyze current design decisions against existing documents:**
   - **Add:** New decisions from this design that should be recorded in decisions.yaml
   - **Conflict:** New decisions that contradict existing convention rules or decision items
   - **Cascade:** Changes in one document that affect the other
   - **Deprecate:** Existing items no longer valid after this change

4. **Present change proposal list** with grades:
   - `[MUST]` — Required change to maintain consistency
   - `[RECOMMENDED]` — Suggested improvement, not blocking

   Format each proposal as:
   ```
   [MUST] Add ADR-003 to decisions.yaml — {title}
   [CONFLICT] GEN-002 vs new naming approach — {description}
   [RECOMMENDED] Deprecate ADR-001 — {reason}
   ```

5. **Present the draft** to the user along with:
   - The intent.md draft
   - The change proposal list
   - `[DISCUSS]` items for ambiguous points that need clarification

6. **Incorporate feedback** and save:
   - `.goose-artifacts/{branch}/intent.md`
   - Update `.goose/conventions.yaml` if user approved convention changes
   - Update `.goose/decisions.yaml` if user approved decision changes

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

- This skill runs in a forked context. Only create artifact files — do NOT modify source code. (`.goose/` configuration files may be updated after user approval.)
- `.goose/` document updates (conventions.yaml, decisions.yaml) are allowed after user approval.
- Always surface discussion points. Do not silently resolve ambiguity.
- decisions.yaml `context` field must be concrete enough to understand "why" from context alone.
- Never auto-modify conventions.yaml or decisions.yaml without user confirmation.
- Resolve the branch name via `git branch --show-current` for the artifact path.
