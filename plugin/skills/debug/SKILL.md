---
name: debug
description: >
  Systematic debugging with root-cause analysis.
  4-phase gate: REPRODUCE, ISOLATE, IDENTIFY, FIX.
  Prove the cause before changing code.
disable-model-invocation: true
---

# Debug

Systematic debugging with root-cause analysis. Prove the cause before changing code.

## The Iron Law

```
NO FIX WITHOUT PROVEN ROOT CAUSE
```

If you cannot explain WHY the bug occurs, you have not found the root cause.
"I know what's wrong" is not proof. An experiment that confirms your hypothesis is.

## 4-Phase Gate

### Phase 1: REPRODUCE

1. Run the failing case and capture the **exact error output**.
2. Document: expected behavior vs actual behavior.
3. Establish reliable reproduction steps — the bug must be triggerable on demand.
4. If the bug cannot be reproduced, **STOP** — investigate environment differences before proceeding.

<!-- HARD-GATE: Do not proceed to ISOLATE without documented reproduction evidence (exact command + exact output) -->

### Phase 2: ISOLATE

1. Narrow the scope: which file, which function, which line?
2. Use binary search through code paths — comment out halves, check if bug persists.
3. Build a **minimal reproduction case** — strip away everything unrelated.
4. If applicable: identify which change introduced the bug (`git log`, `git bisect`).
5. Goal: reduce the search space to a single component or interaction.

<!-- HARD-GATE: Do not proceed to IDENTIFY without a narrowed scope (specific file and function identified) -->

### Phase 3: IDENTIFY

1. Form a hypothesis: "The bug occurs because [X]".
2. Design an experiment: "If my hypothesis is correct, then [test] should produce [result]".
3. Run the experiment.
4. Evaluate:
   - **Result matches prediction** → hypothesis supported. Proceed to Phase 4.
   - **Result does NOT match** → revise hypothesis, design new experiment, repeat.
5. The root cause is **PROVEN** when an experiment's prediction matches reality.

<!-- HARD-GATE: Do not proceed to FIX without a proven hypothesis (experiment + matching result documented) -->

### Phase 4: FIX

1. **Write a failing test** that captures the bug. The test MUST fail before the fix is applied.
2. **Apply the minimal fix** — change only what is necessary to address the proven root cause.
3. **Run the test** — verify it now passes.
4. **Run full verification** via `/genie-goose:polish` gate (IDENTIFY → RUN → READ → VERIFY → CLAIM).
5. **Check for related bugs** — search for the same pattern elsewhere in the codebase.
6. **Commit** with a message that references the root cause.

## Output Artifact

Save to `.goose-artifacts/{branch}/debug-report.md`:

```markdown
# Debug Report: {Brief description}

## Symptom
{What was observed — exact error message, failing test, unexpected behavior}

## Reproduction
{Steps to reproduce, with exact commands and output}

## Root Cause
{Proven explanation of why the bug occurred}

## Hypothesis Log

| # | Hypothesis | Experiment | Predicted Result | Actual Result | Verdict |
|---|-----------|------------|-----------------|---------------|---------|
| 1 | {hypothesis} | {what was tested} | {expected} | {actual} | Confirmed/Refuted |

## Fix Applied
{What was changed and why this fix addresses the root cause}

## Prevention
{How to prevent this class of bug — convention update, test pattern, etc.}
```

## Parallel Investigation

When multiple independent failures exist, use parallel agents to investigate simultaneously.

**When to parallelize:**
- 3+ test files failing with **different** error messages or stack traces
- Multiple subsystems broken with **no shared state** between them
- Independent reproduction cases that don't interfere with each other

**When NOT to parallelize:**
- Failures share a common root cause (cascade failure)
- Tests depend on shared state (database, file system, environment)
- You haven't completed Phase 1 (REPRODUCE) for each failure yet

**Pattern:**
1. Complete Phase 1 (REPRODUCE) for all failures first.
2. Group failures by independence — shared root cause = same group.
3. For each independent group, dispatch an Agent with:
   - The specific failure's reproduction steps and error output
   - Scoped file paths (only the relevant module)
   - Instruction to complete Phases 2-4 independently
4. Review all agent results before committing — check for conflicting fixes.

**Scope each agent narrowly.** "Fix all tests" is too broad. "Investigate the TypeError in `auth/login.ts:45` — reproduction: `npm test auth/login.test.ts`" is correct.

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "I know what's wrong" | Intuition is not evidence | Reproduce first, then prove with an experiment |
| "Let me just try this quick fix" | Untested fixes mask bugs or introduce new ones | Form hypothesis → design experiment → prove cause first |
| "It's probably X" | "Probably" is not evidence | State hypothesis explicitly, then test it |
| "The fix works, no need for a test" | If you can't test it, you can't prove it's fixed | Write the failing test before applying the fix |
| "This is an environment issue, not a code bug" | Prove it — reproduce in a clean environment | Document environment differences as evidence |
| "I've seen this bug before, same fix" | Past fixes may not apply to this instance | Reproduce and verify the root cause matches before applying |

## Integration

- Apply `/genie-goose:polish` for Phase 4 verification.
- Optionally invoke `/genie-goose:update-docs` to record bug patterns in `decisions.yaml`.
- **No pipeline prerequisites** — can be invoked standalone at any time.
- Resolve the branch name via `git branch --show-current` for the artifact path.
