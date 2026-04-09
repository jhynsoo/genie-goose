---
name: pr
description: >
  Generate a structured PR body from pipeline artifacts and git diff.
  Use when creating a pull request after the review cycle is complete.
disable-model-invocation: true
context: fork
---

# PR Body Generation

Generate a structured PR body from accumulated pipeline artifacts and the git diff. The output is a ready-to-use PR description that traces design decisions back to their rationale.

## Prerequisites

- `.goose-artifacts/{branch}/intent.md` — provides design rationale for the "why" in the PR summary.
- There must be a meaningful git diff against the base branch.
- Optional but enriching: `architecture.md`, `review-report.md`, `design.md`, `plan.md`, `criteria.md`.

If intent.md is missing:
1. **Warn:** "Without intent.md, the PR summary will lack design rationale. The 'why' context will be limited to what can be inferred from the diff."
2. **Ask:** proceed anyway, or run the `intent` skill first?
3. If the user confirms, derive context from available artifacts and git diff.

## Procedure

1. **Resolve context:**
   - Branch name via `git branch --show-current`.
   - Base branch: detect via `git symbolic-ref refs/remotes/origin/HEAD | sed 's|refs/remotes/origin/||'`. If detection fails, ask the user.

2. **Read inputs:**
   - Required: `intent.md`, `git diff {base}...HEAD`
   - Optional: `architecture.md`, `review-report.md`, `design.md`, `plan.md`, `criteria.md`
   - Read whatever exists in `.goose-artifacts/{branch}/` — do not fail if optional artifacts are missing.

3. **Analyze changes:**
   - Correlate changed files from git diff with architecture components (if architecture.md exists).
   - Identify breaking changes: API signature changes, removed exports, changed defaults, database schema changes, configuration format changes.
   - Identify migration requirements from the diff.

4. **Draft PR body** following the artifact format below.

5. **Present the draft** along with discussion points:
   - Breaking changes found (ask user to confirm or add missing ones)
   - Related issue numbers (cannot be inferred — ask the user)
   - Migration steps needed (if any)
   - Anything unclear from the artifacts

6. **Incorporate feedback** and save to `.goose-artifacts/{branch}/pr-body.md`.

## Artifact Format

```
# {PR title — concise, under 70 characters}

## Summary
{1-3 sentences derived from intent.md: what this PR does and why}

## Key Decisions
{From intent.md key design decisions — decision title + brief rationale, bulleted}

## Changes

### {Module or area name from architecture.md, or group by directory}
- {File-level change descriptions derived from git diff}

### {Next module/area}
- ...

## Breaking Changes
{List each breaking change with migration guidance. "None" if no breaking changes.}

## Test Plan
- [ ] {Verification items from criteria.md and plan.md}
- [ ] {Build passes}
- [ ] {All tests green}

## Review Notes
{From review-report.md: total comments, accept/reject breakdown, notable REJECT items}
```

## Rules

- Complete this skill in the current thread by default. A delegated helper is optional only when the host supports it and the user explicitly asks for it.
- Only create artifact files — do NOT modify source code.
- All PR body content must be traceable to pipeline artifacts or git diff. Do not fabricate information.
- If an optional artifact is missing, omit or simplify the corresponding section — do not guess.
- The Summary section must explain "why" (from intent.md), not just "what" (from diff).
- Resolve the branch name via `git branch --show-current` for the artifact path.
