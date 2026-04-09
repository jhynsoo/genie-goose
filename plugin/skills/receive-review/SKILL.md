---
name: receive-review
description: >
  Process incoming code review feedback — from honk's review-report.md,
  GitHub PR reviews, or human reviewers. Enforces disciplined response:
  verify before accepting, push back when warranted, no sycophancy.
disable-model-invocation: true
---

# Receive Review

Process incoming code review feedback with discipline. Verify before accepting, push back when warranted, never be sycophantic.

## The Iron Law

```
ACTIONS SPEAK. JUST FIX IT.
```

No performative agreement. No gratitude expressions. No hedging. If the feedback is correct, fix it silently. If it's wrong, push back with evidence.

## Sources

Review feedback can come from multiple sources. Trust level varies:

| Source | Trust Level | Default Stance |
|--------|------------|----------------|
| Human teammate / PR reviewer | High | Accept after understanding. Implement unless technically impossible |
| `honk` review-report.md | Medium | Verify against codebase independently. The earlier review draft may have missed context |
| External automated tool | Low | Verify every claim. Tools often flag false positives |

## 6-Step Response Pattern

For EACH review comment, follow this sequence:

### 1. READ — Absorb the full comment

Read the entire comment. Do not react to the first sentence.

### 2. UNDERSTAND — Identify the actual concern

What specific problem is the reviewer pointing out? Separate the concern from the suggested solution — they may have the right problem but wrong fix.

### 3. RESTATE — Confirm understanding

For human reviewers: restate the concern in your own words before acting. For automated reviews: skip this step.

### 4. VERIFY — Check against the codebase

Before accepting any suggestion:
- Read the actual code being discussed.
- Check if the suggested change would break existing functionality.
- Verify the reviewer's assumptions are correct.
- Grep for actual usage patterns before agreeing to "implement properly."

<!-- HARD-GATE: Do not accept or reject without verifying against current codebase state -->

### 5. EVALUATE — Accept, reject, or counter-propose

Based on verification:

**Accept** — The concern is valid AND the suggested fix is correct.
→ Fix it. No commentary needed. Apply the `polish` gate after fixing.

**Accept with modification** — The concern is valid but a better fix exists.
→ Explain the alternative briefly. Fix it the better way.

**Reject** — The concern is invalid or the cost outweighs the benefit.
→ Provide technical evidence (not opinion) for the rejection.

**Defer** — The concern is valid but out of scope for this change.
→ Acknowledge. Create a follow-up item. Do not fix now.

### 6. IMPLEMENT — Make the change

Fix all accepted items, then run the `polish` gate before committing.

## Forbidden Responses

These responses signal sycophancy or lazy processing. Never use them:

| Forbidden | Why | Instead |
|-----------|-----|---------|
| "You're absolutely right!" | Performative agreement skips verification | Verify first, then fix silently |
| "Great point!" / "Good catch!" | Flattery replaces analysis | Just fix it |
| "Let me implement that now" | Skips VERIFY step | Verify the suggestion against the codebase first |
| "I should have caught that" | Self-deprecation adds no value | Fix it and move on |
| "That makes sense" | Agreement without evidence | Show what you verified |

## Pushback Protocol

Push back when the review suggestion:

1. **Breaks existing functionality** — "This change would break {X} because {evidence}."
2. **Violates YAGNI** — Grep for actual usage. "Only 1 caller uses this interface; the abstraction adds complexity without current benefit."
3. **Conflicts with intent.md** — "This was intentionally excluded in the design intent: {quote from intent.md}."
4. **Is technically incorrect** — "The suggested approach would cause {problem} because {evidence}."

Pushback MUST include:
- The specific technical reason (not "I disagree")
- Evidence from the codebase (file paths, grep results, test output)
- An alternative if you have one

## Prerequisites

At least one of:
- `.goose-artifacts/{branch}/review-report.md` — from honk
- GitHub PR review comments — from human reviewers
- External review feedback — from any source

Optional enriching context:
- `.goose-artifacts/{branch}/intent.md` — for checking intentional exclusions
- `.goose-artifacts/{branch}/criteria.md` — for grounding counter-arguments

If no review feedback exists, there is nothing to process. Inform the user.

## Procedure

1. **Identify the source** of review feedback and set trust level.
2. **Read all feedback** before acting on any single item. Understand the full picture.
3. **Process each comment** through the 6-step pattern (READ → UNDERSTAND → RESTATE → VERIFY → EVALUATE → IMPLEMENT).
4. **Batch fixes** — collect all accepted items, fix them together, then run polish gate once.
5. **Report** — summarize what was accepted, rejected, deferred, and why.

## Rules

- Process ALL comments before starting fixes (same as honk verdict processing).
- Every rejection MUST have technical evidence, not opinion.
- Every acceptance MUST be verified against current codebase state.
- Apply the `polish` gate after all fixes.
- Do not add scope beyond what the reviewer requested (no "while I'm here" changes).
- Resolve the branch name via `git branch --show-current` for the artifact path.

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "The reviewer is senior, they must be right" | Seniority is not evidence — code is | Verify the suggestion against the codebase before accepting |
| "It's easier to just accept everything" | Blind acceptance introduces unnecessary changes | Evaluate each comment independently |
| "This is a minor style point, just accept" | Minor changes can have major side effects | Verify even minor suggestions |
| "I'll push back later" | Later never comes — deferred pushback becomes silent acceptance | Push back now with evidence, or accept now |
| "The reviewer will be upset if I reject" | Professional disagreement with evidence is respected | Provide technical reasoning; let the evidence speak |

## Integration

- Works standalone for external review feedback (PR reviews, human feedback).
- Works with `honk` — can re-process review-report.md with stricter discipline.
- Apply `polish` for post-fix verification.
- Optionally invoke `update-docs` if review reveals a convention gap.
- **No pipeline prerequisites** — can be invoked at any time.
