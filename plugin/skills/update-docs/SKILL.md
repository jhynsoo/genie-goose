---
name: update-docs
description: >
  Manage .goose/conventions.yaml and .goose/decisions.yaml.
  Bidirectional consistency checking between conventions and decisions.
  Use when adding, modifying, or reviewing project standards.
disable-model-invocation: true
---

# Update Docs

Manage `.goose/conventions.yaml` and `.goose/decisions.yaml` with bidirectional consistency checking. This is a standalone skill — invoke at any time, independent of the pipeline.

## Usage

- `update-docs` — Analyze and update both documents
- `update-docs convention` — Focus on conventions.yaml
- `update-docs adr` — Focus on decisions.yaml

## Procedure

1. **Parse arguments:** Determine scope — `convention`, `adr`, or both (default when no argument).

2. **Read both documents** (always, regardless of scope):
   - `.goose/conventions.yaml`
   - `.goose/decisions.yaml`
   - If either file does not exist, inform the user and offer to create it from the template.
   - If pipeline artifacts exist in `.goose-artifacts/{branch}/`, read them for additional context.

3. **Understand the user's intent:** What do they want to add, modify, or remove? Ask clarifying questions if the request is vague.

4. **Bidirectional consistency check** — analyze both documents together for:
   - **Conflict:** Does any existing or proposed convention contradict an existing or proposed decision (or vice versa)?
   - **Cascade:** Does a change in one document require updating the other? (e.g., new ADR adopting a technology requires new convention rules for that tech stack)
   - **Obsolete:** Are there entries in either document that are no longer valid given the proposed changes or current project state?

5. **Present change proposal list** with grades:
   - `[MUST]` — Required to maintain consistency
   - `[RECOMMENDED]` — Suggested improvement, not blocking

   Format:
   ```
   [MUST] Add TS-005 to conventions.yaml — {title}: {reason}
   [MUST] Update ADR-002 status to deprecated — superseded by new approach
   [CONFLICT] GEN-003 vs ADR-004 — {description of contradiction}
   [RECOMMENDED] Add ADR-006 to decisions.yaml — {title}: {reason}
   ```

6. **Wait for user approval.** The user selects which proposals to accept.

7. **Apply approved changes** to the relevant `.goose/` files.

8. **Report** what was changed, showing before/after for each modification.

## Schema Enforcement

### conventions.yaml

Entries must follow this schema:
```yaml
category:
  - id: {CATEGORY}-{NNN}    # GEN, TS, PY, NEXT, NEST, etc.
    rule: >
      Clear, actionable guidance.
    stacks: [...]            # [all] for universal rules
```

### decisions.yaml

Entries must follow this schema:
```yaml
- id: ADR-{NNN}
  title: {Short decision title}
  status: adopted            # adopted | deprecated | superseded
  date: YYYY-MM-DD
  stacks: [...]
  context: |
    {Concrete problem description — must answer "why was this decision necessary"}
  decision: {What was decided}
  consequence: {Outcomes and tradeoffs}
```

## Context Quality Gate

If an ADR `context` field is too vague to answer "why was this decision necessary" from context alone, prompt the user to provide more detail before saving. Examples of insufficient context:

- "We needed a better approach" — *Why? What was wrong with the previous one?*
- "For consistency" — *What inconsistency existed? What was the impact?*

## Rules

- Always read BOTH documents before modifying either one. The bidirectional check is not optional.
- Never auto-modify files without user confirmation.
- When adding new entries, auto-increment the ID based on existing entries in that category.
- When deprecating an ADR, require a reason and optionally a `superseded_by` reference.
- Present `[DISCUSS]` items for ambiguous points that need clarification before deciding.
- This skill does NOT run in fork context — it modifies `.goose/` files directly after user approval.
