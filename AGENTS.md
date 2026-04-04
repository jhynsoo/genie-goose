# Genie Goose

Pipeline-based development workflow plugin for Claude Code.

## Development

Test the plugin locally:

```bash
claude --plugin-dir ./plugin
```

Available skills (all require manual invocation):
- `/genie-goose:goose` — Full pipeline orchestrator
- `/genie-goose:rub` — Step 1: Brainstorming
- `/genie-goose:architecture` — Step 2: Architecture design
- `/genie-goose:intent` — Step 3: Design intent + conflict detection (fork)
- `/genie-goose:write-plan` — Step 4: Implementation plan
- `/genie-goose:criteria` — Step 5: Evaluation criteria (subagent)
- `/genie-goose:implement` — Step 6: Implementation
- `/genie-goose:honk` — Step 7: Code review with verdicts (subagent)
- `/genie-goose:pr` — Step 8: PR body generation (fork)
- `/genie-goose:finish` — Step 9: Pipeline completion — verify, merge/PR/keep/discard
- `/genie-goose:debug` — Systematic debugging with root-cause analysis (standalone)
- `/genie-goose:update-docs` — Convention/ADR management (standalone)

## Plugin Structure

See `docs/specs/2026-04-03-v2-pipeline-redesign.md` for the v2 design spec.
