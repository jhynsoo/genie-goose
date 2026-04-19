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

- `.goose/artifacts/{branch}/brief.md` — provides agreed requirements and chosen approach.
- `.goose/artifacts/{branch}/architecture.md` — provides component structure and data flow.
- `.goose/conventions.yaml` — optional but recommended for conflict detection.
- `.goose/decisions.yaml` — optional but recommended for conflict detection.

If brief.md or architecture.md is missing:
1. **Warn:** "Missing artifacts reduce context for intent documentation. Without brief.md/architecture.md, design decisions may lack grounding."
2. **Ask:** proceed anyway, or run the prerequisite step first?
3. If the user confirms, proceed using available context.
   - If autogoose is active, keep the warning but proceed immediately using the
     available context instead of asking for confirmation.

## Procedure

1. **Read** whatever artifacts exist (brief.md, architecture.md, conventions.yaml, decisions.yaml). If some are missing and the user chose to proceed, work from available context and conversation history.

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
   - If autogoose is active, make the in-scope judgments yourself, note any
     assumptions in the draft, and do not stop for confirmation.

6. **Incorporate feedback** and save:
   - `.goose/artifacts/{branch}/intent.md`
   - Update `.goose/conventions.yaml` if user approved convention changes
   - Update `.goose/decisions.yaml` if user approved decision changes
   - If autogoose is active, agent-owned `.goose/` document updates that are
     necessary to keep the current workflow consistent are allowed.

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

- Complete this skill in the current thread by default. A delegated helper is optional only when the host supports it and the user explicitly asks for it.
- Only create artifact files — do NOT modify source code. (`.goose/` configuration files may be updated after user approval.)
- `.goose/` document updates (conventions.yaml, decisions.yaml) are allowed
  after user approval, or after agent-owned judgment when autogoose is active.
- Always surface discussion points. Do not silently resolve ambiguity.
- decisions.yaml `context` field must be concrete enough to understand "why" from context alone.
- When autogoose is inactive, never auto-modify conventions.yaml or
  decisions.yaml without user confirmation.
- Read `.goose/artifacts/{branch}/autogoose.yaml` at step start when it exists.
- Resolve the branch name via `git branch --show-current` for the artifact path.
