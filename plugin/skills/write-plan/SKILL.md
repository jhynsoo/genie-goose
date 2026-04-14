---
name: write-plan
description: >
  Write a detailed implementation plan with micro-tasks,
  exact file paths, and test code snippets.
  Each task should be completable in 2-5 minutes.
disable-model-invocation: true
---

# Write Plan

Create a detailed implementation plan based on the architecture and design intent documents.

## Prerequisites

- `.goose/artifacts/{branch}/architecture.md` — provides component structure and file layout.
- `.goose/artifacts/{branch}/intent.md` — provides design rationale and intentional exclusions.

If prerequisite artifacts are missing:
1. **Warn:** "Without architecture.md, the plan may not align with the intended component structure. Without intent.md, intentional exclusions and design rationale will be unavailable."
2. **Ask:** proceed anyway, or run the prerequisite step first?
3. If the user confirms, proceed using codebase exploration and user direction.

## Procedure

1. **Read** architecture.md and intent.md (if available) to understand what to build and why. If artifacts are missing and the user chose to proceed, derive structure from codebase exploration and user direction.

2. **Explore the codebase** to understand:
   - Existing directory structure and file organization
   - Test framework and testing patterns in use
   - Dependencies and build tooling
   - Related existing code that will be modified

3. **Break work into micro-tasks.** Each task should:
   - Be completable in 2-5 minutes
   - Have a clear, verifiable outcome
   - List exact file paths (Create / Modify / Test)
   - Include step-by-step instructions with test code snippets
   - Follow TDD pattern where applicable (failing test → implement → verify)

4. **Present the plan section by section.** Ask the user to approve each task before moving on. If the user requests changes, revise and re-present.

<!-- HARD-GATE: Do not save plan.md until every task has exact file paths, test code, and 2-5 minute scope -->
5. **Save artifact** to `.goose/artifacts/{branch}/plan.md` after full approval.

## Artifact Format

```
# {Feature Name} Implementation Plan

**Goal:** {One sentence summary}
**Architecture:** {2-3 sentences on approach}
**Tech Stack:** {Key technologies/libraries}

---

### Task 1: {Component Name}

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**
  \```python
  def test_specific_behavior():
      result = function(input)
      assert result == expected
  \```
- [ ] **Step 2: Run and verify it fails**
- [ ] **Step 3: Implement minimal code to pass**
- [ ] **Step 4: Run tests and verify all green**
- [ ] **Step 5: Commit**
```

## Rules

- Each task MUST be completable in 2-5 minutes. If it takes longer, split it.
- File paths MUST be exact paths verified against the actual codebase. Do not guess.
- Include test code snippets for every task that involves code changes.
- Follow TDD pattern: write the failing test first, then implement.
- Read brief.md before starting. Do not ask the user to repeat what was already decided.
- Present incrementally — do not dump the entire plan at once.
- Resolve the branch name via `git branch --show-current` for the artifact path.

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "The implementer will figure out the details" | Vague tasks cause deviations and rework | Specify exact file paths, step-by-step instructions, and test code |
| "Tests are obvious, no need to specify" | Obvious tests get skipped; specified tests get written | Include test code snippets for every task with code changes |
| "This is really one logical unit" | If it takes more than 5 minutes, it is too big | Split into sub-tasks each completable in 2-5 minutes |
| "I'll add edge case handling later" | Later never comes — plan it now | Include edge case handling as explicit tasks in the plan |
| "The file paths are approximately right" | Approximate paths cause the implementer to guess | Verify every file path against the actual codebase before including it |
