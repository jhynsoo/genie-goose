# Codex Agent Templates

These files are optional templates for users who want manual Codex custom-agent setup.

Copy them into `.codex/agents/` for project-scoped setup or `~/.codex/agents/` for personal setup:

```bash
mkdir -p .codex/agents
cp ./plugin/assets/codex-agents/*.toml .codex/agents/
```

Installing the templates does not change the default plugin behavior. Codex only spawns subagents when the user explicitly asks for delegation.
