---
name: goose
description: >
  Run the development workflow pipeline.
  Executes brainstorm, architecture, design-intent,
  evaluation-criteria, implement, review in sequence.
disable-model-invocation: true
---

# Goose Pipeline

Run the full development workflow pipeline.

## Precondition Checks

Before starting the pipeline, verify:

1. **Branch:** Resolve the current branch via `git branch --show-current`. The pipeline artifacts will be stored under `.goose-artifacts/{branch}/`.
2. **Conventions:** Check if `.goose/conventions.yaml` exists. If not, inform the user:
   > `.goose/conventions.yaml` not found. This file is needed for evaluation criteria (step 4). Copy the template and customize it for your project:
   > `mkdir -p .goose && cp {plugin-path}/.goose/conventions.template.yaml .goose/conventions.yaml`
3. **Artifact directory:** Create `.goose-artifacts/{branch}/` if it does not exist.
4. **Gitignore:** Check if `.goose-artifacts/` is in `.gitignore`. If not, suggest adding it.

## Pipeline Steps

Execute each step in order. Between steps, confirm with the user before proceeding to the next step.

### Step 1: Brainstorm
Invoke `/genie-goose:brainstorm $ARGUMENTS`
- Output: `.goose-artifacts/{branch}/design.md`
- Wait for user approval of the design before continuing.

### Step 2: Architecture
Invoke `/genie-goose:architecture`
- Input: design.md
- Output: `.goose-artifacts/{branch}/architecture.md`
- Wait for user approval of the architecture before continuing.

### Step 3: Design Intent
Invoke `/genie-goose:design-intent`
- Input: design.md + architecture.md
- Output: `.goose-artifacts/{branch}/intent.md`
- Runs in forked context. Presents draft + discussion points for user feedback.

### Step 4: Evaluation Criteria
Invoke `/genie-goose:evaluation-criteria`
- Input: intent.md + .goose/conventions.yaml
- Output: `.goose-artifacts/{branch}/criteria.md`
- Runs via criteria-builder subagent. Presents draft + discussion points for user feedback.

### Step 5: Implement
Invoke `/genie-goose:implement`
- Input: architecture.md + intent.md
- The main agent implements the feature directly.

### Step 6: Review
Invoke `/genie-goose:review`
- Input: criteria.md + intent.md + git diff
- Output: `.goose-artifacts/{branch}/review-comments.md`
- Runs via reviewer subagent. [p1] auto-fix loop (max 3 rounds). [p2]+ reported to user.

## Completion

After all steps complete, summarize:
- What was built (link to design.md)
- Architecture overview (link to architecture.md)
- Review results (link to review-comments.md)
- Any remaining [p2]+ items for the user to consider
