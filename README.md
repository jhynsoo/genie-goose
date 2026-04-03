# Genie Goose

**English** | [한국어](README.ko.md)

Pipeline-based development workflow plugin for Claude Code.

Runs a structured 7-step pipeline — **rub → architecture → intent → write-plan → criteria → implement → honk** — so that every feature goes through proper design, documented intent, detailed planning, and evidence-based code review.

## Installation

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

## Setup

After installation, create a conventions file in your project. The plugin uses this to generate evaluation criteria for code review.

```bash
mkdir -p .goose
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

```bash
cp ~/.claude/plugins/cache/*/genie-goose/*/plugin/.goose/decisions.template.yaml .goose/decisions.yaml
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

This executes all 7 steps in sequence, pausing for your input between steps.

### Individual Steps

You can also run each step independently:

| Command | Description |
|---------|-------------|
| `/genie-goose:rub <topic>` | Collaborative brainstorming — asks clarifying questions, proposes 2-3 approaches |
| `/genie-goose:architecture` | Design technical architecture based on brainstorm result |
| `/genie-goose:intent` | Document design decisions + detect conflicts with conventions/decisions |
| `/genie-goose:write-plan` | Write detailed implementation plan with micro-tasks and test snippets |
| `/genie-goose:criteria` | Extract relevant conventions/decisions and build review criteria |
| `/genie-goose:implement` | Execute the plan checklist step by step |
| `/genie-goose:honk` | Evidence-based code review with ACCEPT/REJECT verdicts |

### Pipeline Flow

```
rub ───────────→ design.md
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
```

All artifacts are saved to `.goose-artifacts/{branch-name}/`.

## How Review Works

The review step (honk) uses a dedicated subagent that checks every change against the evaluation criteria. Each comment must cite a specific criterion — no opinion-based reviews.

**Priority levels:**

| Level | Meaning | Action |
|-------|---------|--------|
| `[p1]` | Must fix (bugs, criteria violations) | User verdict → fix |
| `[p2]` | Strongly recommended | User verdict |
| `[p3]` | Recommended | User verdict |
| `[p4]` | Minor improvement | User verdict |

Each comment receives an **ACCEPT** (fix it) or **REJECT** (skip it) verdict from the user. Accepted items are fixed and verified automatically.

## Local Development

To test changes to the plugin locally:

```bash
claude --plugin-dir ./plugin
```

## License

MIT
