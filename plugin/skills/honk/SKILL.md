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

1. **Present ALL comments at once** as a batch summary:

   ### Review: {N} comments ({p1_count} p1, {p2_count} p2, {p3_count} p3, {p4_count} p4)

   | # | Priority | File | Issue | Default |
   |---|----------|------|-------|---------|
   | 1 | [p1] | path:line | Brief issue | ACCEPT |
   | 2 | [p2] | path:line | Brief issue | (confirm) |
   | ... | ... | ... | ... | ... |

   Then expand details per priority group:

   **[p1] Auto-accept** — Default ACCEPT. User must provide justification to reject.
   **[p2] Judgment** — Present the agent's recommendation with reasoning. User confirms or overrides.
   **[p3] Recommendation** — Offer suggestion but defer entirely to the user.
   **[p4] Bulk** — User can "accept all", "reject all", or cherry-pick.

2. **Collect all verdicts** before making any code changes. Do not start fixing until the user has reviewed the entire batch.

<!-- HARD-GATE: Do not begin fixing until ALL verdicts are collected from the user -->

3. **Fix all ACCEPT items**, then apply the `/genie-goose:polish` gate (RUN tests → READ output → VERIFY pass) before committing.

4. **Record REJECT reasons** provided by the user.

5. **Append verdict** to each comment in review-report.md:
   ```
   - **Verdict: ACCEPT** — Fixed. {Brief description of the fix}.
   ```
   or
   ```
   - **Verdict: REJECT** — {User's reason for rejection}.
   ```

<!-- HARD-GATE: Do not claim review cycle complete without fresh full-suite polish gate evidence -->
6. **QA pass:** Apply the `/genie-goose:polish` gate for final verification — run full build + test suite, read output, verify 0 failures before claiming the review cycle is complete.

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

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "This criterion obviously applies, no citation needed" | Every comment must trace to criteria.md | Find and cite the specific criterion ID before writing the comment |
| "This is clearly an ACCEPT, I can auto-fix" | Verdicts come from the user, not the agent | Present all comments → collect all verdicts → then fix |
| "The user probably doesn't care about this p1" | p1 means must-fix — skipping requires explicit REJECT | Present the p1 with full detail; only skip on explicit user REJECT with justification |
| "All individual fixes passed, no need for final QA" | Individual passes do not prove integrated correctness | Run full build + test suite as final QA via `/genie-goose:polish` |
| "These are all minor, just ACCEPT all" | Batch-approving skips judgment on each item | Present each priority group separately; user must confirm each group |
