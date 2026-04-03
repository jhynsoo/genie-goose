---
name: polish
description: >
  Verification before completion — evidence before claims, always.
  5-step gate: IDENTIFY, RUN, READ, VERIFY, CLAIM.
  Invoke before claiming any work is done, before committing,
  and before moving to the next pipeline step.
disable-model-invocation: true
---

# Polish

Verification before completion. Evidence before claims, always.

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

Before claiming ANY status or expressing satisfaction:

1. **IDENTIFY** — What command proves this claim? (test runner, build, type checker, linter)
2. **RUN** — Execute the FULL command fresh. Not a cached result. Not a partial run.
3. **READ** — Full output. Check exit code. Count failures.
4. **VERIFY** — Does the output actually confirm the claim?
   - If **NO**: State actual status with evidence.
   - If **YES**: State claim WITH evidence.
5. **CLAIM** — Only now may you report success.

Skip any step = lying, not verifying.

## Pipeline Verification Table

| Context | Claim | Required Evidence | Not Sufficient |
|---------|-------|-------------------|----------------|
| `implement` task | "Task N done" | Test output + build exit 0 | "Should pass", previous run |
| `implement` full | "Implementation complete" | Full test suite green + all plan.md checkboxes | Partial tests, "looks good" |
| `honk` ACCEPT fix | "Fixed the issue" | Tests pass after fix, no regressions | Code changed, assumed fixed |
| `honk` QA pass | "Review cycle complete" | Final build + full test suite 0 failures | "Already verified earlier" |
| Bug fix (non-pipeline) | "Bug fixed" | Reproduce symptom → fix → verify symptom gone | "The code looks correct now" |
| Feature (non-pipeline) | "Feature complete" | Relevant tests pass + build succeeds | "It should work" |
| Before commit | "Ready to commit" | Build + tests + lint all green | "Just a small change" |
| Before PR | "Ready for PR" | Full verification suite, clean diff | Partial check |

## Red Flags — STOP

These thoughts mean you are about to violate the iron law:

| Thought | What To Do |
|---------|------------|
| "This should work now" | RUN the verification |
| "I'm confident" | Confidence is not evidence |
| "Just this once" | No exceptions |
| "Tests passed earlier" | Earlier is not now. Run again |
| "The subagent reported success" | Verify independently |
| "It's a small change" | Small changes break things too |
| "I'm about to say Done/Fixed/Complete" | STOP. Run the gate function first |
| Using "should", "probably", "seems to" | Replace with evidence |

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "Confidence is high" | Confidence ≠ evidence |
| "Just this once" | No exceptions, ever |
| "Linter passed so build is fine" | Linter ≠ compiler ≠ tests |
| "Agent said it's done" | Agent claims need independent verification |
| "I'm tired of verifying" | Fatigue ≠ excuse |
| "Partial check is enough" | Partial proves nothing about the whole |

## Correct Patterns

**Tests:**
```
RUN:    npm test (or project's test command)
OUTPUT: 34/34 passing ✓
CLAIM:  "All 34 tests pass" ← evidence cited
```

**Build:**
```
RUN:    npm run build
OUTPUT: Build completed successfully (exit 0)
CLAIM:  "Build succeeds" ← exit code cited
```

**Bug fix:**
```
RUN:    Reproduce original symptom
OUTPUT: Error confirmed
FIX:    Apply the change
RUN:    Same reproduction step
OUTPUT: No error
CLAIM:  "Bug fixed — symptom no longer reproduces" ← before/after cited
```

**Plan checklist:**
```
RUN:    Each checkbox's verification step
OUTPUT: Green for each
CLAIM:  "Task N complete — all steps verified" ← per-step evidence
```

## When to Apply

- Before claiming any pipeline step is done
- Before committing code
- Before moving to the next pipeline step
- Before creating a PR
- Before reporting status to the user
- After fixing a `honk` ACCEPT verdict item
- After any subagent reports completion

## Integration

This skill is referenced by `/genie-goose:implement` and `/genie-goose:honk`.
It can also be invoked standalone via `/genie-goose:polish` for any task.
