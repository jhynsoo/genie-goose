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

1. **Branch:** Resolve the current branch via `git branch --show-current`. The pipeline artifacts will be stored under `.goose-artifacts/{branch}/`.
2. **Conventions:** Check if `.goose/conventions.yaml` exists. If not, inform the user:
   > `.goose/conventions.yaml` not found. This file is needed for evaluation criteria (step 5). Copy the template and customize it for your project:
   > `mkdir -p .goose && cp {plugin-path}/.goose/conventions.template.yaml .goose/conventions.yaml`
3. **Decisions:** Check if `.goose/decisions.yaml` exists. If not, inform the user (optional):
   > `.goose/decisions.yaml` not found. This file is optional but recommended for tracking architecture decisions:
   > `mkdir -p .goose && cp {plugin-path}/.goose/decisions.template.yaml .goose/decisions.yaml`
4. **Artifact directory:** Create `.goose-artifacts/{branch}/` if it does not exist.
5. **Gitignore:** Check if `.goose-artifacts/` is in `.gitignore`. If not, suggest adding it.

## Pipeline Steps

Execute each step in order. Between steps, confirm with the user before proceeding to the next step.

### Step 1: Rub
Invoke `/genie-goose:rub $ARGUMENTS`
- Output: `.goose-artifacts/{branch}/design.md`
- Wait for user approval of the design before continuing.

### Step 2: Architecture
Invoke `/genie-goose:architecture`
- Input: design.md
- Output: `.goose-artifacts/{branch}/architecture.md`
- Wait for user approval of the architecture before continuing.

### Step 3: Intent
Invoke `/genie-goose:intent`
- Input: design.md + architecture.md + conventions.yaml + decisions.yaml
- Output: `.goose-artifacts/{branch}/intent.md` + convention/decision update proposals
- Runs in forked context. Presents draft + change proposals + discussion points for user feedback.

### Step 4: Write Plan
Invoke `/genie-goose:write-plan`
- Input: architecture.md + intent.md
- Output: `.goose-artifacts/{branch}/plan.md`
- Wait for user approval of the plan before continuing.

### Step 5: Criteria
Invoke `/genie-goose:criteria`
- Input: intent.md + conventions.yaml + decisions.yaml
- Output: `.goose-artifacts/{branch}/criteria.md`
- Runs via criteria-builder subagent. Presents draft + discussion points for user feedback.

### Step 6: Implement
Invoke `/genie-goose:implement`
- Input: plan.md + intent.md
- The main agent executes the plan checklist sequentially.

### Step 7: Honk
Invoke `/genie-goose:honk`
- Input: criteria.md + intent.md + git diff
- Output: `.goose-artifacts/{branch}/review-report.md`
- Runs via reviewer subagent. Presents ACCEPT/REJECT verdicts for user confirmation, then fixes and QA.
- For stricter review processing (e.g., pushback protocol, anti-sycophancy), the user can invoke `/genie-goose:receive-review` after this step.

### Step 8: PR (Optional)
Invoke `/genie-goose:pr`
- Input: all artifacts + git diff + review-report.md
- Output: `.goose-artifacts/{branch}/pr-body.md`
- Runs in forked context. Presents draft + discussion points for user feedback.
- Skip this step if the user does not intend to create a PR — ask before proceeding.

### Step 9: Finish
Invoke `/genie-goose:finish`
- Input: review-report.md + all pipeline artifacts
- Verifies all work, presents merge/PR/keep/discard options, executes the user's choice.
- This step always runs — it replaces the implicit pipeline ending.

<!-- HARD-GATE: Do not proceed to completion without explicit user approval of each prior step's output -->

## Completion

The `/genie-goose:finish` step handles final verification, user options, and cleanup summary.
If the user skips the finish step, fall back to reporting:
- Artifacts created in `.goose-artifacts/{branch}/`
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
