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

- `.goose-artifacts/{branch}/plan.md` — provides a verified task checklist with exact file paths and test code.
- `.goose-artifacts/{branch}/intent.md` — provides design rationale and intentional exclusions.

If prerequisite artifacts are missing:
1. **Warn:** "Without plan.md, implementation will lack a verified task checklist, increasing the risk of scope creep. Without intent.md, intentional exclusions cannot be checked."
2. **Ask:** proceed anyway, or run the prerequisite step first?
3. If the user confirms, proceed using their instructions directly.

**When operating without plan.md:** Create an ad-hoc task list from the user's request before starting implementation. Confirm the list with the user. Apply the same verification discipline (polish gate per task, frequent commits).

## Procedure

1. **Read** plan.md and intent.md (if available) to understand what to build, why, and in what order. If plan.md is missing and the user chose to proceed, work from the ad-hoc task list.

2. **Execute each task** in plan.md sequentially:
   - Follow the steps exactly as written in the plan
   - **Verification gate:** Before marking any task complete, apply the `/genie-goose:polish` gate:
     IDENTIFY the verification command → RUN it fresh → READ the full output → VERIFY success → only then commit.
   <!-- HARD-GATE: Do not commit without fresh polish gate evidence (IDENTIFY → RUN → READ → VERIFY → CLAIM) -->
   - Commit the changes with a descriptive message

3. **If a step fails or needs modification:**
   - Report the issue to the user
   - Propose an adjustment to the plan
   - Wait for user approval before proceeding with the change

<!-- HARD-GATE: Do not claim complete without all plan.md checkboxes verified with fresh evidence -->
4. **Report completion** with a summary of what was implemented and any deviations from the plan.

## Rules

- Follow the plan document. Do not deviate from agreed-upon tasks without user approval.
- Respect intentional exclusions from the design intent document.
- Write tests alongside implementation when the project has a test framework.
- Make frequent, small commits with descriptive messages.
- Do not skip steps or reorder tasks unless there is a blocking dependency issue.
- Before claiming any task is complete, apply the `/genie-goose:polish` verification gate. Never report "done" without fresh evidence from a verification command.

## Parallel Execution

When the plan contains independent tasks, consider dispatching them in parallel.

**When to parallelize:**
- 3+ plan tasks that modify **different files** with no shared interfaces
- Tasks are self-contained (each has its own test, no cross-task dependencies)
- plan.md does not specify a sequential dependency between them

**When NOT to parallelize:**
- Tasks share files or interfaces (e.g., Task 2 imports from Task 1's output)
- Tasks must be committed in a specific order for bisectability
- The plan explicitly marks tasks as sequential

**Pattern:**
1. Identify independent task clusters in plan.md.
2. For each cluster, dispatch an Agent with:
   - The specific task(s) from plan.md
   - intent.md for design context
   - Instruction to apply polish gate before reporting done
3. Review all agent results — verify no conflicting changes.
4. Run full test suite after integrating all results.
5. Commit per task (not per agent) for clean git history.

**Default is sequential.** Only parallelize when independence is clear. When in doubt, run sequentially.

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "This is too simple to test" | Simple code hides the most subtle bugs | Write the test first, then the code |
| "I already checked mentally" | Mental checks are not evidence | Run `/genie-goose:polish` gate — no exceptions |
| "The plan is slightly wrong, I'll just adjust" | Plan deviations require approval | Report issue → propose adjustment → wait for user approval |
| "I'll commit everything at the end" | Large commits hide mistakes | Commit after each task passes verification |
| "This small addition makes sense" | Intent exclusions exist for a reason | Check intent.md exclusions — do not add excluded items without user approval |
| "No test framework in this project" | Lack of test framework does not excuse skipping verification | Use build, lint, or manual verification — document what was checked |
