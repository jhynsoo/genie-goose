# Genie Goose

Pipeline-based development workflow framework.

## Artifact Path

All artifacts are stored under `.goose-artifacts/{branch-name}/`.
Resolve the branch name via `git branch --show-current`.

Artifacts:
- `design.md` — Brainstorming result
- `architecture.md` — Architecture design
- `intent.md` — Design intent document
- `plan.md` — Implementation plan
- `criteria.md` — Evaluation criteria document
- `review-report.md` — Review report with verdicts

## Reference Documents

- `.goose/conventions.yaml` — Project coding conventions
- `.goose/decisions.yaml` — Architecture decision records

## Skill Router

The `lamp` skill is auto-injected at session start via the SessionStart hook.
It routes user requests to the appropriate pipeline skill based on intent.
When in doubt, consult the routing table in the lamp skill.

## Verification

The `polish` skill enforces evidence-based completion claims.
Apply its 5-step gate function (IDENTIFY → RUN → READ → VERIFY → CLAIM) before:
- Claiming any pipeline step is done
- Committing code
- Reporting build/test status
- Moving to the next pipeline step

## Available Skills

| Skill | Purpose | Context |
|-------|---------|---------|
| `lamp` | Skill router (auto-injected) | session start |
| `polish` | Verification gate | main |
| `goose` | Full pipeline orchestrator | main |
| `rub` | Brainstorming | main |
| `architecture` | Architecture design | main |
| `intent` | Design intent + conflict detection | fork |
| `write-plan` | Implementation plan | main |
| `criteria` | Evaluation criteria | fork + criteria-builder |
| `implement` | Execute plan | main |
| `honk` | Code review + verdicts | fork + reviewer |

## Pipeline Rules

1. Steps running in fork/subagent MUST only create artifact files and MUST NOT modify source code (except review auto-fix).
2. When generating evaluation criteria, do NOT include the entire conventions.yaml or decisions.yaml. Extract only items relevant to the current task.
3. All code review comments MUST be grounded in evaluation criteria. Do not leave opinion-based reviews without evidence.
4. When requesting user confirmation, always include a summary of ambiguous discussion points that need clarification.
