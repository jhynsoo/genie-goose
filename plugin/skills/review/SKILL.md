---
name: review
description: >
  Evidence-based code review using evaluation criteria.
  Reviewer subagent takes criteria.md + intent.md + git diff
  as input to produce review-comments.md.
  p1 issues trigger auto-fix loop; p2+ issues are reported only.
disable-model-invocation: true
context: fork
agent: reviewer
---

# Review

Perform an evidence-based code review using the evaluation criteria established earlier.

## Prerequisites

- `.goose-artifacts/{branch}/criteria.md` must exist.
- `.goose-artifacts/{branch}/intent.md` must exist.
- Implementation must be complete (there should be a meaningful git diff).

## Procedure

1. **Read** criteria.md, intent.md, and the git diff against the base branch.

2. **Review** every changed file against the criteria. For each issue found:
   - Cite the specific criterion ID from criteria.md.
   - Classify by priority: [p1], [p2], [p3], or [p4].
   - Provide a concrete suggestion and explain side effects.

3. **Save** the review to `.goose-artifacts/{branch}/review-comments.md`.

4. **Return results** to the main agent for post-processing.

## Post-Processing (Main Agent)

After the reviewer subagent returns review-comments.md:

- **[p1] issues:** Automatically fix each issue. After fixing, re-invoke the reviewer subagent to verify the fix. Repeat up to 3 times. If still failing after 3 attempts, report to the user.
- **[p2]+ issues:** Report to the user. Do not auto-fix.

## Priority Levels

- `[p1]` Must fix — bugs, criteria violations, intent-implementation mismatch
- `[p2]` Strongly recommended — maintainability, readability with significant impact
- `[p3]` Recommended — nice to have, no functional impact
- `[p4]` Minor improvement — naming, formatting

## Rules

- All comments MUST cite a criterion from criteria.md. No opinion-based reviews.
- Respect intentional design decisions. Flag only intent-implementation mismatches.
- When suggesting changes, MUST explain side effects.
- Resolve the branch name via `git branch --show-current` for the artifact path.
