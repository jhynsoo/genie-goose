---
name: finish
description: >
  Pipeline completion — verify all work, present
  merge/PR/keep/discard options, execute choice, clean up.
disable-model-invocation: true
---

# Finish

Pipeline completion — verify, present options, execute, clean up.

## Prerequisites

**Full flow** (review-report.md exists):
- `.goose/artifacts/{branch}/review-report.md` — provides review verdicts.
- All ACCEPT items in review-report.md should have "Fixed" status.

**Lightweight flow** (no review-report.md):
- Warn: "No review-report.md found. Finish will verify tests and build but cannot check review verdicts."
- Ask the user to confirm before proceeding.
- The polish gate (tests + build) still applies regardless.
  - If autogoose is active, keep the warning but proceed to verification
    without asking for confirmation.

## Procedure

### Step 1: VERIFY — Final Health Check

1. Run full test suite + build via the `polish` gate (IDENTIFY → RUN → READ → VERIFY → CLAIM).
2. Check for uncommitted changes (`git status`). All changes must be committed.
3. If review-report.md exists: cross-reference and verify all ACCEPT items have "Fixed" status. If review-report.md does not exist (lightweight flow): skip this check.
4. If any check fails, report the issue and **stop** — do not present options until verification passes.

<!-- HARD-GATE: Do not present options until all verification checks pass with fresh evidence -->

### Step 2: PRESENT OPTIONS

Present the user with exactly 4 choices. Do not suggest a default — wait for the user to choose.

```
Pipeline complete. All verifications passed. Choose how to proceed:

1. **Merge locally** — Merge {branch} into {base-branch}, delete feature branch
2. **Create PR** — Generate PR body and create GitHub PR
3. **Keep branch** — Leave as-is for later
4. **Discard** — Delete branch and all changes (requires double confirmation)
```

### Step 3: EXECUTE

**Option 1 — Merge locally:**
1. Verify the base branch is up to date: `git fetch origin {base-branch}`
2. Switch to base branch: `git checkout {base-branch}`
3. Merge: `git merge {branch}`
4. Delete feature branch: `git branch -d {branch}`
5. Report: merge commit hash, branch deleted

**Option 2 — Create PR:**
1. Check if `.goose/artifacts/{branch}/pr-body.md` exists.
   - If not: invoke the `pr` skill to generate it.
2. Push branch to remote: `git push -u origin {branch}`
3. Create PR using `gh pr create` with the pr-body.md content.
4. Report: PR URL.

**Option 3 — Keep branch:**
1. Report current branch status: ahead/behind counts, uncommitted changes.
2. Summarize artifacts in `.goose/artifacts/{branch}/`.

**Option 4 — Discard:**
1. First confirmation: "This will delete branch {branch} and all uncommitted changes. Type 'discard' to confirm."
2. Second confirmation: "Are you sure? This cannot be undone."
3. Only after both confirmations: switch to base branch, delete feature branch.
4. Delete the artifact directory: `rm -rf .goose/artifacts/{branch}/`
5. Report: branch deleted, artifacts cleaned up.

### Step 4: CLEAN UP — Post-Completion Report

Regardless of the option chosen, summarize:
- What was done (merge / PR created / branch kept / discarded)
- Pipeline artifacts created during this session (list files in `.goose/artifacts/{branch}/`)
- Any remaining action items (e.g., REJECT verdicts to reconsider, PR review pending)
- Autogoose cleanup status (`removed` / `inactive` / `not present`)

## Rationalizations You Must Reject

| Excuse | Reality | Required Action |
|--------|---------|-----------------|
| "Tests already passed in implement" | If honk ran, code changed during review fixes. Even without honk, state may have changed since last test | Run the full suite again with fresh evidence |
| "PR doesn't need a body" | pr-body.md exists or can be generated — PRs deserve context | Generate pr-body.md via the `pr` skill if it doesn't exist |
| "I can skip verification, honk already reviewed" | honk reviewed the diff, not the integrated result after fixes | Run full verification — build, tests, lint — on the final state |
| "The user said merge, no need to verify first" | Merging broken code is worse than a delay | Always run Step 1 verification before presenting options |
| "Discard doesn't need double confirmation" | Deleting work is irreversible | Always require two explicit confirmations |

## Integration

- Invokes `polish` for Step 1 verification.
- Invokes `pr` if PR is chosen and pr-body.md is missing.
- Reads: `review-report.md`, `pr-body.md` (optional).
- Enriched by: `review-report.md` (from honk). Works without it in lightweight flows.
- Resolve the branch name via `git branch --show-current` for the artifact path.
- Read `.goose/artifacts/{branch}/autogoose.yaml` at step start when it exists.
- Always clear or inactivate autogoose when the current workflow ends or is
  explicitly parked. Do not carry it into a new workflow.
- Autogoose never changes the final user-owned choice among merge / create PR /
  keep branch / discard.
