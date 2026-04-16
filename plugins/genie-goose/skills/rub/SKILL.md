---
name: rub
description: >
  Rub the lamp — brainstorming-first entrypoint for feature work.
  Refine requirements into a brief, then continue through
  the full development workflow when the user wants the
  end-to-end flow.
disable-model-invocation: true
---

# Rub

Develop the user's idea into a concrete brief through collaborative dialogue.
`rub` is the canonical entrypoint for feature work in genie-goose.

## Modes

`rub` supports two modes:

1. **Brief-only mode** — If the user explicitly wants brainstorming or ideation only, stop after saving `brief.md`.
2. **Full-flow mode** — If the user wants to build the feature end-to-end, use `rub` as the pipeline entrypoint and continue with:
   `architecture → intent → write-plan → criteria → implement → honk → pr → finish`

Unless the user explicitly limits the scope to brainstorming only, treat `rub` as the default full-flow entrypoint for feature work.

## Precondition Checks

Before starting:

1. **Branch:** Resolve the current branch via `git branch --show-current`. The workflow artifacts will be stored under `.goose/artifacts/{branch}/`.
2. **Conventions:** Check if `.goose/conventions.yaml` exists. If not, inform the user that it will be needed for the `criteria` step and recommend creating it via `update-docs` or the template.
3. **Decisions:** Check if `.goose/decisions.yaml` exists. If not, inform the user that it is optional but recommended for the `intent` and `criteria` steps.
4. **Artifact directory:** Create `.goose/artifacts/{branch}/` if it does not exist.
5. **Gitignore:** Check if `.goose/` or `.goose/artifacts/` is ignored. If neither is ignored, suggest adding one of them.

## Procedure

### Step 1: Brainstorm and Capture the Brief

1. **Understand the idea:** Ask clarifying questions one at a time. Prefer multiple-choice questions when possible. Focus on: purpose, constraints, success criteria.

2. **Explore approaches:** Once you understand the requirements, propose 2-3 different approaches with trade-offs. Lead with your recommended option and explain why.

3. **Converge on the brief:** Present the brief section by section. Ask after each section whether it looks right. Cover: architecture, components, data flow, error handling.

4. **Save artifact:** Once the user approves the brief, save it to `.goose/artifacts/{branch}/brief.md`.

### Step 2: Decide Whether to Continue

- If the user explicitly asked for brainstorming only, stop here.
- Otherwise, continue the rest of the workflow in order, confirming with the user between steps.

### Step 3: Continue the Full Workflow

If continuing, run these steps in sequence:

1. **Architecture**
   - Input: `brief.md`
   - Output: `.goose/artifacts/{branch}/architecture.md`
2. **Intent**
   - Input: `brief.md + architecture.md + conventions.yaml + decisions.yaml`
   - Output: `.goose/artifacts/{branch}/intent.md` plus any approved `.goose/` document updates
3. **Write Plan**
   - Input: `architecture.md + intent.md`
   - Output: `.goose/artifacts/{branch}/plan.md`
4. **Criteria**
   - Input: `intent.md + conventions.yaml + decisions.yaml`
   - Output: `.goose/artifacts/{branch}/criteria.md`
5. **Implement**
   - Input: `plan.md + intent.md`
   - Execute the plan checklist sequentially.
6. **Honk**
   - Input: `criteria.md + intent.md + git diff`
   - Output: `.goose/artifacts/{branch}/review-report.md`
7. **PR** (optional)
   - Input: all available artifacts + git diff
   - Output: `.goose/artifacts/{branch}/pr-body.md`
   - Ask before proceeding. Skip if the user does not want a PR.
8. **Finish**
   - Input: review-report.md + all relevant artifacts
   - Verify completion and present merge / PR / keep / discard options.

<!-- HARD-GATE: Do not advance automatically between major workflow steps. The user approves each artifact before the next step. -->

## Artifact Format

```
# Brief: {Topic}

## Problem Statement
{What we're solving and why}

## Requirements
{Agreed-upon requirements from the brainstorming session}

## Chosen Approach
{Description of the selected approach}

### Why This Approach
{Reasoning and trade-offs considered}

### Alternatives Considered
{Other approaches and why they were not chosen}
```

## Rules

- One question per message. Do not overwhelm with multiple questions.
- Multiple choice preferred — easier to answer than open-ended.
- YAGNI ruthlessly — remove unnecessary features from all designs.
- Always propose 2-3 approaches before settling on one.
- Resolve the branch name via `git branch --show-current` for the artifact path.
- When `rub` is being used as the feature workflow entrypoint, do not stop after `brief.md` unless the user explicitly wants ideation only.
