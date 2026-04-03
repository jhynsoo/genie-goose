---
name: implement
description: >
  Execute the implementation plan step by step.
  Follow plan.md checklist with build/test verification
  and frequent commits.
disable-model-invocation: true
---

# Implement

Execute the implementation plan by following the plan.md checklist step by step.

## Prerequisites

- `.goose-artifacts/{branch}/plan.md` must exist. If not, inform the user to run `/genie-goose:write-plan` first.
- `.goose-artifacts/{branch}/intent.md` must exist.

## Procedure

1. **Read** plan.md and intent.md to understand what to build, why, and in what order.

2. **Execute each task** in plan.md sequentially:
   - Follow the steps exactly as written in the plan
   - **Verification gate:** Before marking any task complete, apply the `/genie-goose:polish` gate:
     IDENTIFY the verification command → RUN it fresh → READ the full output → VERIFY success → only then commit.
   - Commit the changes with a descriptive message

3. **If a step fails or needs modification:**
   - Report the issue to the user
   - Propose an adjustment to the plan
   - Wait for user approval before proceeding with the change

4. **Report completion** with a summary of what was implemented and any deviations from the plan.

## Rules

- Follow the plan document. Do not deviate from agreed-upon tasks without user approval.
- Respect intentional exclusions from the design intent document.
- Write tests alongside implementation when the project has a test framework.
- Make frequent, small commits with descriptive messages.
- Do not skip steps or reorder tasks unless there is a blocking dependency issue.
- Before claiming any task is complete, apply the `/genie-goose:polish` verification gate. Never report "done" without fresh evidence from a verification command.
