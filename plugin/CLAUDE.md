# Genie Goose

Composable development workflow framework with guard rails.

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
- `pr-body.md` — PR body (optional, from pr skill)
- `debug-report.md` — Debug report with root-cause analysis (from debug skill)

## Reference Documents

- `.goose/conventions.yaml` — Project coding conventions
- `.goose/decisions.yaml` — Architecture decision records

## Skill Router

The `lamp` skill is auto-injected at session start via the SessionStart hook.
It classifies tasks by scope, recommends routes, and routes user requests to skills.
When in doubt, consult the routing table in the lamp skill.

## Workflow Model

Genie-goose skills are independently invocable. There are three ways to use them:

1. **Full pipeline** — Run `/genie-goose:goose` for the complete 9-step workflow. Recommended for medium-to-large features.
2. **Recommended route** — Let `lamp` classify your task and suggest a tailored route (e.g., `rub → write-plan → implement → honk → finish`). You can adjust the route.
3. **Direct invocation** — Call any skill directly (e.g., `/genie-goose:implement`). Missing prerequisite artifacts trigger a warning, not a block.

Regardless of which approach, two guard rails always apply:
- **polish** — Evidence-based verification before claiming completion.
- **finish** — Structured wrap-up (works with or without review-report.md).

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
| `lamp` | Skill router + route recommender (auto-injected) | session start |
| `polish` | Verification gate | any |
| `goose` | Full 9-step pipeline preset | main |
| `rub` | Brainstorming | main |
| `architecture` | Architecture design | main |
| `intent` | Design intent + conflict detection | fork |
| `write-plan` | Implementation plan | main |
| `criteria` | Evaluation criteria | fork + criteria-builder |
| `implement` | Execute plan or direct tasks | main |
| `honk` | Code review + verdicts | fork + reviewer |
| `pr` | PR body generation | fork |
| `finish` | Workflow completion | main |
| `debug` | Systematic debugging | main |
| `update-docs` | Convention/ADR management | main |

## Workflow Rules

1. Steps running in fork/subagent MUST only create artifact files and MUST NOT modify source code (except review auto-fix).
2. When generating evaluation criteria, do NOT include the entire conventions.yaml or decisions.yaml. Extract only items relevant to the current task.
3. All code review comments MUST be grounded in evaluation criteria. Do not leave opinion-based reviews without evidence.
4. When requesting user confirmation, always include a summary of ambiguous discussion points that need clarification.
5. The `update-docs` skill is standalone and can be invoked at any time, independent of pipeline state. It always reads both documents for bidirectional consistency checking before making changes.
6. Skills can be invoked independently. Missing prerequisite artifacts produce warnings, not blocks. The user can always choose to proceed with reduced context.
7. The `polish` verification gate applies to ALL workflows — full pipeline, recommended routes, and direct invocation alike.
