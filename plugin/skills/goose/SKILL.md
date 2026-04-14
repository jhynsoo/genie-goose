---
name: goose
description: >
  Run the full 9-step development workflow pipeline —
  the recommended preset for medium-to-large features.
  For lighter workflows, invoke skills individually
  or use lamp's route recommendations.
disable-model-invocation: true
---

# Goose Pipeline

The full 9-step development workflow — recommended for medium-to-large features where thorough design, review, and verification matter. This is one way to use genie-goose, not the only way. For smaller tasks, the lamp router can suggest lighter routes.

## Precondition Checks

Before starting the pipeline, verify:

1. **Branch:** Resolve the current branch via `git branch --show-current`. The pipeline artifacts will be stored under `.goose/artifacts/{branch}/`.
2. **Conventions:** Check if `.goose/conventions.yaml` exists. If not, inform the user:
   > `.goose/conventions.yaml` not found. This file is needed for evaluation criteria (step 5). Run the `update-docs` skill to create it from the template, or manually copy this plugin's `.goose/conventions.template.yaml` to `.goose/conventions.yaml` and customize it.
3. **Decisions:** Check if `.goose/decisions.yaml` exists. If not, inform the user (optional):
   > `.goose/decisions.yaml` not found. This file is optional but recommended for tracking architecture decisions. Run the `update-docs` skill to create it from the template, or manually copy this plugin's `.goose/decisions.template.yaml` to `.goose/decisions.yaml` and customize it.
4. **Artifact directory:** Create `.goose/artifacts/{branch}/` if it does not exist.
5. **Gitignore:** Check if `.goose/` or `.goose/artifacts/` is in `.gitignore`. If neither is ignored, suggest adding one of them.

## Pipeline Steps

Execute each step in order. Between steps, confirm with the user before proceeding to the next step.

### Step 1: Rub
Invoke the `rub` skill with the user's task as input.
- Output: `.goose/artifacts/{branch}/brief.md`
- Wait for user approval of the brief before continuing.

### Step 2: Architecture
Invoke the `architecture` skill.
- Input: brief.md
- Output: `.goose/artifacts/{branch}/architecture.md`
- Wait for user approval of the architecture before continuing.

### Step 3: Intent
Invoke the `intent` skill.
- Input: brief.md + architecture.md + conventions.yaml + decisions.yaml
- Output: `.goose/artifacts/{branch}/intent.md` + convention/decision update proposals
- By default, complete this in the current thread. If the host supports subagents and the user explicitly asks for delegation, a separate helper agent may draft supporting analysis, but the main agent still owns user-facing synthesis and final edits.

### Step 4: Write Plan
Invoke the `write-plan` skill.
- Input: architecture.md + intent.md
- Output: `.goose/artifacts/{branch}/plan.md`
- Wait for user approval of the plan before continuing.

### Step 5: Criteria
Invoke the `criteria` skill.
- Input: intent.md + conventions.yaml + decisions.yaml
- Output: `.goose/artifacts/{branch}/criteria.md`
- By default, complete this in the current thread. If the host supports custom agents and the user explicitly asks for delegation, a criteria-focused helper may assist with extraction.

### Step 6: Implement
Invoke the `implement` skill.
- Input: plan.md + intent.md
- The main agent executes the plan checklist sequentially.

### Step 7: Honk
Invoke the `honk` skill.
- Input: criteria.md + intent.md + git diff
- Output: `.goose/artifacts/{branch}/review-report.md`
- By default, complete this in the current thread. If the host supports custom agents and the user explicitly asks for delegation, a reviewer helper may assist with evidence gathering.
- For stricter review processing (e.g., pushback protocol, anti-sycophancy), the user can invoke `receive-review` after this step.

### Step 8: PR (Optional)
Invoke the `pr` skill.
- Input: all artifacts + git diff + review-report.md
- Output: `.goose/artifacts/{branch}/pr-body.md`
- By default, complete this in the current thread. A delegated helper is optional and only appropriate when the host supports it and the user explicitly asks for it.
- Skip this step if the user does not intend to create a PR — ask before proceeding.

### Step 9: Finish
Invoke the `finish` skill.
- Input: review-report.md + all pipeline artifacts
- Verifies all work, presents merge/PR/keep/discard options, executes the user's choice.
- This step always runs — it replaces the implicit pipeline ending.

<!-- HARD-GATE: Do not proceed to completion without explicit user approval of each prior step's output -->

## Completion

The `finish` step handles final verification, user options, and cleanup summary.
If the user skips the finish step, fall back to reporting:
- Artifacts created in `.goose/artifacts/{branch}/`
- Any remaining action items (REJECT verdicts to reconsider)

## When to Use the Full Pipeline

The full 9-step pipeline is recommended when:
- Building a new feature with multiple components
- Making architectural changes that affect multiple files
- Working in an unfamiliar codebase where design clarity matters
- The task involves API contracts, database changes, or public interfaces

For smaller tasks (single-file changes, straightforward bug fixes, simple refactors), individual skills or lamp-recommended routes are more appropriate.

## Pipeline Discipline

When running the full pipeline, maintain step discipline:

| Principle | Explanation |
|-----------|-------------|
| Each step has distinct outputs and approval gates | Do not merge architecture and planning into one step |
| Confirm with the user before advancing between steps | Never auto-advance — the user approves each artifact |
| If a step is genuinely trivial for the task, it will complete quickly | Fast completion is better than skipping |
| "Just do it" means run efficiently, not skip steps | Acknowledge urgency, run briskly, but complete all steps |
