# Genie Goose

Composable development workflow plugin for Claude Code and Codex.

## Development

Test the Claude Code plugin locally:

```bash
claude --plugin-dir ./plugins/genie-goose
```

Test the Codex plugin locally:

1. Restart Codex so it reloads `.agents/plugins/marketplace.json`.
2. Open `/plugins`.
3. Install or reinstall `genie-goose` from `Genie Goose Local`.
4. Start a new thread and use `@genie-goose` as the plugin/router entrypoint.
5. Use a specific `$skill` only when you want to bypass the router, for example `$rub`.

Available skills (all require manual invocation):
- `/genie-goose:rub` — Brainstorming-first full 9-step pipeline preset (recommended for large features)
- `/genie-goose:goose` — Legacy alias for `rub`
- `/genie-goose:architecture` — Architecture design
- `/genie-goose:intent` — Design intent + conflict detection (fork)
- `/genie-goose:write-plan` — Implementation plan
- `/genie-goose:criteria` — Evaluation criteria (subagent)
- `/genie-goose:implement` — Implementation (with or without plan.md)
- `/genie-goose:honk` — Code review with verdicts (subagent)
- `/genie-goose:receive-review` — Process review feedback with discipline (standalone)
- `/genie-goose:pr` — PR body generation (fork)
- `/genie-goose:finish` — Workflow completion — verify, merge/PR/keep/discard
- `/genie-goose:debug` — Systematic debugging with root-cause analysis (standalone)
- `/genie-goose:update-docs` — Convention/ADR management (standalone)
- `/genie-goose:polish` — Verification gate (standalone)
