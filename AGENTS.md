# Genie Goose

Composable development workflow plugin for Claude Code.

## Development

Test the plugin locally:

```bash
claude --plugin-dir ./plugin
```

Available skills (all require manual invocation):
- `/genie-goose:goose` — Full 9-step pipeline preset (recommended for large features)
- `/genie-goose:rub` — Brainstorming
- `/genie-goose:architecture` — Architecture design
- `/genie-goose:intent` — Design intent + conflict detection (fork)
- `/genie-goose:write-plan` — Implementation plan
- `/genie-goose:criteria` — Evaluation criteria (subagent)
- `/genie-goose:implement` — Implementation (with or without plan.md)
- `/genie-goose:honk` — Code review with verdicts (subagent)
- `/genie-goose:pr` — PR body generation (fork)
- `/genie-goose:finish` — Workflow completion — verify, merge/PR/keep/discard
- `/genie-goose:debug` — Systematic debugging with root-cause analysis (standalone)
- `/genie-goose:update-docs` — Convention/ADR management (standalone)
- `/genie-goose:polish` — Verification gate (standalone)

## Plugin Structure

See `docs/specs/2026-04-03-v2-pipeline-redesign.md` for the v2 design spec.
