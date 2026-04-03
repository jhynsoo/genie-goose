---
name: goose
description: >
  Run the development workflow pipeline.
  Executes rub, architecture, intent, write-plan,
  criteria, implement, honk in sequence.
disable-model-invocation: true
---

# Goose Pipeline

Run the full development workflow pipeline.

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

## Completion

After all steps complete, summarize:
- What was built (link to design.md)
- Architecture overview (link to architecture.md)
- Implementation plan (link to plan.md)
- Review results (link to review-report.md)
- Any REJECT verdict items for the user to reconsider
