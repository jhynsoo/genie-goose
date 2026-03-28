@AGENTS.md

# Genie Goose

Pipeline-based development workflow plugin for Claude Code.

## Development

Test the plugin locally:

```bash
claude --plugin-dir ./plugin
```

Available skills (all require manual invocation):
- `/genie-goose:goose` — Full pipeline orchestrator
- `/genie-goose:brainstorm` — Step 1: Brainstorming
- `/genie-goose:architecture` — Step 2: Architecture design
- `/genie-goose:design-intent` — Step 3: Design intent (fork)
- `/genie-goose:evaluation-criteria` — Step 4: Evaluation criteria (subagent)
- `/genie-goose:implement` — Step 5: Implementation
- `/genie-goose:review` — Step 6: Code review (subagent)

## Plugin Structure

See `docs/superpowers/specs/2026-03-28-genie-goose-design.md` for the full design spec.
