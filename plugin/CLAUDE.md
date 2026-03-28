# Genie Goose

Pipeline-based development workflow framework.

## Artifact Path

All artifacts are stored under `.goose-artifacts/{branch-name}/`.
Resolve the branch name via `git branch --show-current`.

Artifacts:
- `design.md` — Brainstorming result
- `architecture.md` — Architecture design
- `intent.md` — Design intent document
- `criteria.md` — Evaluation criteria document
- `review-comments.md` — Review comments

## Reference Documents

- `.goose/conventions.yaml` — Project coding conventions

## Pipeline Rules

1. Steps running in fork/subagent MUST only create artifact files and MUST NOT modify source code (except review auto-fix).
2. When generating evaluation criteria, do NOT include the entire conventions.yaml. Extract only items relevant to the current task.
3. All code review comments MUST be grounded in evaluation criteria. Do not leave opinion-based reviews without evidence.
4. When requesting user confirmation, always include a summary of ambiguous discussion points that need clarification.
