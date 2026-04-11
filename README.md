# Genie Goose

**English** | [한국어](README.ko.md)

Composable development workflow plugin for Claude Code and Codex.

Provides a structured set of skills — brainstorming, architecture design, intent documentation, implementation planning, evaluation criteria, code review, PR generation, and verified completion — that you can compose into workflows tailored to your task size.

## Installation

### Claude Code

```bash
# Add the marketplace
claude plugin marketplace add jhynsoo/genie-goose

# Install the plugin
claude plugin install genie-goose@genie-goose
```

Or install at project scope to share with your team:

```bash
claude plugin install genie-goose@genie-goose --scope project
```

### Codex

Codex plugins are installed from marketplaces. This repository includes a repo-local marketplace file at `.agents/plugins/marketplace.json` that points to `./plugin`, so you can test genie-goose in Codex without copying files elsewhere.

1. Restart Codex after pulling this repo so it reloads repo-local marketplaces.
2. Start Codex and open the plugin directory with `/plugins`.
3. Choose the `Genie Goose Local` marketplace.
4. Install `genie-goose`.
5. Start a new thread and invoke the plugin entrypoint with `@genie-goose`.
6. If you want to bypass the router and call a bundled skill directly, use `$goose`, `$lamp`, `$implement`, and so on.
7. Treat `@genie-goose` as the plugin/router entrypoint. Use `$goose` when you specifically want the full 9-step pipeline skill.

### Optional Codex Advanced Setup

The baseline Codex flow is single-agent and works without extra setup.

If you also want reusable custom agents for review-oriented delegation, copy the provided templates into your Codex agent directory (`.codex/agents/` for project scope or `~/.codex/agents/` for personal scope):

```bash
mkdir -p .codex/agents
cp ./plugin/assets/codex-agents/*.toml .codex/agents/
```

Those templates are opt-in. Codex only spawns subagents when you explicitly ask it to, so installing the files does not change the default `@genie-goose` behavior by itself.

## Setup

After installation, create a conventions file in your project. The plugin uses this to generate evaluation criteria for code review.

If you are working from this repository or using the repo-local Codex marketplace:

```bash
mkdir -p .goose
cp ./plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

If you installed via the Claude marketplace instead:

```bash
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

Edit `.goose/conventions.yaml` to match your team's coding standards. The file uses a flat YAML structure:

```yaml
general:
  - id: GEN-001
    rule: >
      A single function should not exceed 30 lines.
    stacks: [all]
```

Optionally, create a decisions file to track architecture decisions:

If you are working from this repository or using the repo-local Codex marketplace:

```bash
cp ./plugin/.goose/decisions.template.yaml .goose/decisions.yaml
```

If you installed via the Claude marketplace instead:

```bash
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/decisions.template.yaml .goose/decisions.yaml
```

Add `.goose-artifacts/` to your `.gitignore` — the plugin generates per-branch artifacts there:

```bash
echo ".goose-artifacts/" >> .gitignore
```

## Usage

There are three ways to use genie-goose:

### 1. Full Pipeline

Run the complete 9-step workflow with a single command:

```text
Claude Code: /genie-goose:goose build a user authentication system
Codex router entrypoint: @genie-goose build a user authentication system
Codex full pipeline skill: $goose build a user authentication system
```

This executes all steps in sequence — **rub → architecture → intent → write-plan → criteria → implement → honk → pr (optional) → finish** — pausing for your input between steps. Recommended for medium-to-large features.

### 2. Recommended Route

The `lamp` skill router classifies your task by scope and recommends a tailored route:

| Task Size | Recommended Route |
|-----------|-------------------|
| Full feature | `goose` (full 9-step pipeline), or `rub → architecture → intent → write-plan → criteria → implement → honk → finish` |
| Medium task | `rub → write-plan → implement → honk → finish` |
| Small task | `implement → finish` (or no skill needed) |
| Debug | `debug` |
| Documentation | `update-docs` |

Just describe what you want to do — lamp will suggest the appropriate route. You can accept, adjust, or skip the recommendation.

In Claude Code, `lamp` is auto-injected at session start via the bundled SessionStart hook.

In Codex, plugin-bundled hooks are not loaded automatically. The default path is to start with `@genie-goose` and let the plugin route the task, or call a specific bundled skill with `$lamp`, `$goose`, `$implement`, and so on.

If you want the same automatic session-start routing in Codex, wire the existing `plugin/hooks/session-start` script into `~/.codex/hooks.json` or `<repo>/.codex/hooks.json`. Codex's hooks documentation requires hooks to live next to Codex config layers rather than inside the plugin manifest.

### 3. Direct Invocation

Run any skill independently:

| Command | Description |
|---------|-------------|
| `Claude: /genie-goose:goose <topic>` `Codex: $goose <topic>` | Full 9-step pipeline preset. |
| `Codex: @genie-goose <topic>` | Plugin/router entrypoint. Let genie-goose classify the request and choose the next skill. |
| `Claude: /genie-goose:rub <topic>` `Codex: $rub <topic>` | Collaborative brainstorming — asks clarifying questions, proposes 2-3 approaches |
| `Claude: /genie-goose:architecture` `Codex: $architecture` | Design technical architecture based on the approved brief |
| `Claude: /genie-goose:intent` `Codex: $intent` | Document design decisions + detect conflicts with conventions/decisions |
| `Claude: /genie-goose:write-plan` `Codex: $write-plan` | Write detailed implementation plan with micro-tasks and test snippets |
| `Claude: /genie-goose:criteria` `Codex: $criteria` | Extract relevant conventions/decisions and build review criteria |
| `Claude: /genie-goose:implement` `Codex: $implement` | Execute the plan checklist step by step |
| `Claude: /genie-goose:honk` `Codex: $honk` | Evidence-based code review with ACCEPT/REJECT verdicts |
| `Claude: /genie-goose:receive-review` `Codex: $receive-review` | Process review feedback with discipline — verify before accepting, push back when warranted |
| `Claude: /genie-goose:pr` `Codex: $pr` | Generate PR body from pipeline artifacts |
| `Claude: /genie-goose:finish` `Codex: $finish` | Workflow completion — verify, merge/PR/keep/discard |
| `Claude: /genie-goose:debug` `Codex: $debug` | Systematic debugging — reproduce, isolate, prove, fix |
| `Claude: /genie-goose:update-docs` `Codex: $update-docs` | Manage conventions.yaml and decisions.yaml |
| `Claude: /genie-goose:polish` `Codex: $polish` | Verification gate — evidence before claims |

Missing prerequisite artifacts produce warnings, not blocks — you can always proceed with reduced context.

### Workflow Diagram

```
rub ───────────→ brief.md
                    │
architecture ───→ architecture.md
                    │
intent ─────────→ intent.md ──────────────────┐
 (conventions.yaml    │                       │
  decisions.yaml)     │                       │
                      │                       │
write-plan ─────→ plan.md                     │
                    │                         │
criteria ───────→ criteria.md ────────┐      │
 (conventions.yaml    │               │      │
  decisions.yaml)     │               │      │
                      │               ▼      ▼
implement       ←── plan.md + intent.md
                                      │      │
honk            ←── criteria.md + intent.md + git diff
                    │
                    → review-report.md
                                      │
receive-review  ←── review-report.md (optional, for stricter processing)
                                      │
pr (optional)   ←── intent.md + review-report.md + git diff
                    │
                    → pr-body.md
                                      │
finish          ←── review-report.md (optional) + pr-body.md (optional)
                    │
                    → merge / PR / keep / discard

debug           ←── (standalone, no prerequisites)
                    │
                    → debug-report.md
```

All artifacts are saved to `.goose-artifacts/{branch-name}/`.

## Guard Rails

Two skills apply to all workflows — full pipeline, recommended routes, and direct invocation alike:

- **polish** — Evidence-based verification before claiming completion. 5-step gate: IDENTIFY → RUN → READ → VERIFY → CLAIM.
- **finish** — Structured wrap-up with merge/PR/keep/discard options. Works with or without review-report.md.

## How Review Works

The review step (honk) checks every change against the evaluation criteria. Each comment must cite a specific criterion — no opinion-based reviews. In Codex, the baseline path keeps this in the current thread; optional custom-agent templates are provided only for explicit delegation workflows.

**Priority levels:**

| Level | Meaning | Action |
|-------|---------|--------|
| `[p1]` | Must fix (bugs, criteria violations) | User verdict → fix |
| `[p2]` | Strongly recommended | User verdict |
| `[p3]` | Recommended | User verdict |
| `[p4]` | Minor improvement | User verdict |

Each comment receives an **ACCEPT** (fix it) or **REJECT** (skip it) verdict from the user. Accepted items are fixed and verified automatically.

## Local Development

To test changes to the plugin locally in Claude Code:

```bash
claude --plugin-dir ./plugin
```

To test changes locally in Codex:

1. Restart Codex so it reloads `.agents/plugins/marketplace.json`.
2. Open `/plugins`.
3. Install or reinstall `genie-goose` from `Genie Goose Local`.
4. Start a new thread and use `@genie-goose` as the plugin/router entrypoint.
5. Use a specific `$skill` only when you want to bypass the router, for example `$goose` for the full pipeline.

## License

MIT
