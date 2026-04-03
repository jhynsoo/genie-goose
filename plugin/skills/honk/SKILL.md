---
name: honk
description: >
  Evidence-based code review using evaluation criteria.
  Reviewer subagent produces review-report.md with per-comment
  ACCEPT/REJECT verdicts confirmed by the user.
disable-model-invocation: true
context: fork
agent: reviewer
---

# Honk

Perform an evidence-based code review using the evaluation criteria established earlier. Each review comment receives a user-confirmed ACCEPT/REJECT verdict.

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

3. **Save** the review to `.goose-artifacts/{branch}/review-report.md`.

4. **Return results** to the main agent for verdict processing.

## Verdict Processing (Main Agent)

After the reviewer subagent returns review-report.md:

1. **Present each comment** to the user one by one (or grouped by priority), showing:
   - The criterion, issue, suggestion, and side effect
   - Ask: **ACCEPT** (fix this) or **REJECT** (skip this)?

2. **For ACCEPT verdicts:** Fix the issue, then apply the `/genie-goose:polish` gate (RUN tests → READ output → VERIFY pass) before committing.

3. **For REJECT verdicts:** Record the reason provided by the user.

4. **Append verdict** to each comment in review-report.md:
   ```
   - **Verdict: ACCEPT** — Fixed. {Brief description of the fix}.
   ```
   or
   ```
   - **Verdict: REJECT** — {User's reason for rejection}.
   ```

5. **QA pass:** Apply the `/genie-goose:polish` gate for final verification — run full build + test suite, read output, verify 0 failures before claiming the review cycle is complete.

## Priority Levels

- `[p1]` Must fix — bugs, criteria violations, intent-implementation mismatch
- `[p2]` Strongly recommended — maintainability, readability with significant impact
- `[p3]` Recommended — nice to have, no functional impact
- `[p4]` Minor improvement — naming, formatting

## Rules

- All comments MUST cite a criterion from criteria.md. No opinion-based reviews.
- Respect intentional design decisions. Flag only intent-implementation mismatches.
- When suggesting changes, MUST explain side effects.
- Never auto-fix without user verdict. Always present ACCEPT/REJECT choice first.
- Resolve the branch name via `git branch --show-current` for the artifact path.
