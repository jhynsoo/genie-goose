---
name: autogoose
description: >
  Enable autogoose / auto mode for the current workflow when the user
  says "autogoose", asks to stop asking for approvals, or wants the
  workflow to continue automatically. Persists branch-local workflow
  mode state so downstream genie-goose skills can skip in-scope
  approval prompts while keeping merge, PR creation, discard, and
  remote push under explicit user choice.
---

# Autogoose

Enable `autogoose` for the current workflow. This skill does not replace `rub`,
`implement`, or any other workflow step. It toggles a branch-local workflow mode
that downstream genie-goose skills must honor.

## Accepted Activation Surfaces

- free-text `autogoose` (host description-match routing)
- direct invocation of `autogoose` (lamp router or programmatic skill call)
- slash command `/genie-goose:autogoose` (Claude Code) / `$autogoose` (Codex)

## Procedure

1. **Resolve the current branch** via `git branch --show-current`.
2. **Ensure** `.goose/artifacts/{branch}/` exists.
3. **Infer the current workflow step** from the active skill or the latest
   workflow artifact already produced for this branch.
4. **Create or update** `.goose/artifacts/{branch}/autogoose.yaml` with the
   current workflow state.
5. **Treat the current artifact as approved** if activation happened in the
   middle of a workflow step.
6. **Resume the current workflow** without asking further questions that only
   gate normal workflow progress.

## State Artifact

Persist autogoose state to `.goose/artifacts/{branch}/autogoose.yaml`.

Recommended shape:

```yaml
enabled: true
mode: autogoose
source: skill
activated_at_step: architecture
workflow_id: 2026-04-20T02:10:00+09:00
scope: current-workflow
status: active
```

## Rules

- Scope autogoose to the **current workflow only**. It must not silently carry
  over into a new unrelated workflow.
- When the workflow reaches `finish`, is explicitly stopped, or is parked for
  later, downstream skills should clear this state or mark it inactive.
- Autogoose may skip workflow-internal approvals, reduced-context proceed
  confirmations, and similar in-scope questions.
- Autogoose must **not** justify merge, actual PR creation, discard, remote
  push, or other destructive/external actions without explicit user choice.
- If the user says to stop auto progression, disable autogoose immediately for
  the current workflow.
