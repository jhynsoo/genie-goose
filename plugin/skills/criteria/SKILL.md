---
name: criteria
description: >
  Establish evaluation criteria based on conventions.yaml,
  decisions.yaml, and design intent. Subagent extracts
  relevant items to produce criteria.md.
disable-model-invocation: true
context: fork
agent: criteria-builder
---

# Evaluation Criteria

Establish evaluation criteria for the upcoming code review by extracting relevant items from the project's conventions and architecture decisions.

## Prerequisites

- `.goose-artifacts/{branch}/intent.md` — provides task-specific context for filtering relevant conventions.
- `.goose/conventions.yaml` — **required**. The source of evaluation criteria. If not found, inform the user: "Run `/genie-goose:update-docs` to create it from the template, or manually copy this plugin's `.goose/conventions.template.yaml` to `.goose/conventions.yaml` and customize it."
- `.goose/decisions.yaml` — optional. If it exists, relevant ADR items will be included in the criteria.

If intent.md is missing:
1. **Warn:** "Without intent.md, criteria extraction will lack task-specific context and may include irrelevant conventions."
2. **Ask:** proceed anyway, or run `/genie-goose:intent` first?
3. If the user confirms, use the user's task description and git diff to infer scope for convention filtering.

## Procedure

1. **Read** `.goose/conventions.yaml`, `.goose/decisions.yaml` (if exists), and `.goose-artifacts/{branch}/intent.md`.

2. **Extract relevant conventions:**
   - Filter by `stacks` field matching the task's tech stack.
   - Secondary filter by content relevance to the changes.
   - Include all rules with `stacks: [all]`.

3. **Extract relevant decisions:**
   - Filter ADR items by `stacks` field and `status: adopted`.
   - Include items whose `decision` content is relevant to the changes.

4. **Draft criteria.md** merging common criteria, task-specific criteria, and ADR-based criteria. For each criterion, add a "How to apply" note specific to this task.

5. **Present the draft** to the user with discussion points about criteria whose scope is ambiguous.

6. **Incorporate feedback** and save to `.goose-artifacts/{branch}/criteria.md`.

## Rules

- Do NOT include the entire conventions.yaml or decisions.yaml. Extract only relevant items.
- This skill runs in a forked context with a subagent. Only create artifact files — do NOT modify source code.
- Resolve the branch name via `git branch --show-current` for the artifact path.
