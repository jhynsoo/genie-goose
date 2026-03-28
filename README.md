# Genie Goose

**English** | [한국어](README.ko.md)

Pipeline-based development workflow plugin for Claude Code.

Runs a structured 6-step pipeline — **brainstorm → architecture → design-intent → evaluation-criteria → implement → review** — so that every feature goes through proper design, documented intent, and evidence-based code review.

## Installation

```bash
claude plugin install genie-goose@https://github.com/jhynsoo/genie-goose
```

Or install at project scope to share with your team:

```bash
claude plugin install genie-goose@https://github.com/jhynsoo/genie-goose --scope project
```

## Setup

After installation, create a conventions file in your project. The plugin uses this to generate evaluation criteria for code review.

```bash
mkdir -p .goose
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/conventions.template.yaml .goose/conventions.yaml
```

Edit `.goose/conventions.yaml` to match your team's coding standards. The file uses a simple YAML structure:

```yaml
categories:
  - name: general
    rules:
      - id: GEN-001
        title: Function length limit
        stacks: [all]
        level: recommended
        description: >
          A single function should not exceed 30 lines.
```

Add `.goose-artifacts/` to your `.gitignore` — the plugin generates per-branch artifacts there:

```bash
echo ".goose-artifacts/" >> .gitignore
```

## Usage

### Full Pipeline

Run the complete workflow with a single command:

```
/genie-goose:goose build a user authentication system
```

This executes all 6 steps in sequence, pausing for your input between steps.

### Individual Steps

You can also run each step independently:

| Command | Description |
|---------|-------------|
| `/genie-goose:brainstorm <topic>` | Collaborative brainstorming — asks clarifying questions, proposes 2-3 approaches |
| `/genie-goose:architecture` | Design technical architecture based on brainstorm result |
| `/genie-goose:design-intent` | Document design decisions, trade-offs, and intentional exclusions |
| `/genie-goose:evaluation-criteria` | Extract relevant conventions and build review criteria |
| `/genie-goose:implement` | Implement the feature following architecture and intent docs |
| `/genie-goose:review` | Evidence-based code review — auto-fixes critical issues |

### Pipeline Flow

```
brainstorm ──→ design.md
                  │
architecture ──→ architecture.md
                  │
design-intent ─→ intent.md ──────────────┐
                  │                       │
evaluation    ─→ criteria.md ─────┐      │
 (conventions.yaml)               │      │
                                  ▼      ▼
implement     ←── architecture.md + intent.md
                                  │      │
review        ←── criteria.md + intent.md + git diff
```

All artifacts are saved to `.goose-artifacts/{branch-name}/`.

## How Review Works

The review step uses a dedicated subagent that checks every change against the evaluation criteria. Each comment must cite a specific criterion — no opinion-based reviews.

**Priority levels:**

| Level | Meaning | Action |
|-------|---------|--------|
| `[p1]` | Must fix (bugs, criteria violations) | Auto-fix loop (max 3 rounds) |
| `[p2]` | Strongly recommended | Reported to user |
| `[p3]` | Recommended | Reported to user |
| `[p4]` | Minor improvement | Reported to user |

## Local Development

To test changes to the plugin locally:

```bash
claude --plugin-dir ./plugin
```

## License

MIT
