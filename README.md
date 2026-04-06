# Genie Goose

**English** | [한국어](README.ko.md)

Composable development workflow plugin for Claude Code.

Provides a structured set of skills — brainstorming, architecture design, intent documentation, implementation planning, evaluation criteria, code review, PR generation, and verified completion — that you can compose into workflows tailored to your task size.

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

There are three ways to use genie-goose:

### 1. Full Pipeline

Run the complete 9-step workflow with a single command:

```
/genie-goose:goose build a user authentication system
```

This executes all steps in sequence — **rub → architecture → intent → write-plan → criteria → implement → honk → pr (optional) → finish** — pausing for your input between steps. Recommended for medium-to-large features.

### 2. Recommended Route

The `lamp` skill router is auto-injected at session start. It classifies your task by scope and recommends a tailored route:

| Task Size | Recommended Route |
|-----------|-------------------|
| Full feature | `rub → architecture → write-plan → implement → honk → finish` |
| Medium task | `rub → write-plan → implement → honk → finish` |
| Small task | `implement → finish` (or no skill needed) |
| Debug | `debug` |
| Documentation | `update-docs` |

Just describe what you want to do — lamp will suggest the appropriate route. You can accept, adjust, or skip the recommendation.

### 3. Direct Invocation

Run any skill independently:

| Command | Description |
|---------|-------------|
| `/genie-goose:goose <topic>` | Full 9-step pipeline preset |
| `/genie-goose:rub <topic>` | Collaborative brainstorming — asks clarifying questions, proposes 2-3 approaches |
| `/genie-goose:architecture` | Design technical architecture based on brainstorm result |
| `/genie-goose:intent` | Document design decisions + detect conflicts with conventions/decisions |
| `/genie-goose:write-plan` | Write detailed implementation plan with micro-tasks and test snippets |
| `/genie-goose:criteria` | Extract relevant conventions/decisions and build review criteria |
| `/genie-goose:implement` | Execute the plan checklist step by step |
| `/genie-goose:honk` | Evidence-based code review with ACCEPT/REJECT verdicts |
| `/genie-goose:receive-review` | Process review feedback with discipline — verify before accepting, push back when warranted |
| `/genie-goose:pr` | Generate PR body from pipeline artifacts |
| `/genie-goose:finish` | Workflow completion — verify, merge/PR/keep/discard |
| `/genie-goose:debug` | Systematic debugging — reproduce, isolate, prove, fix |
| `/genie-goose:update-docs` | Manage conventions.yaml and decisions.yaml |
| `/genie-goose:polish` | Verification gate — evidence before claims |

Missing prerequisite artifacts produce warnings, not blocks — you can always proceed with reduced context.

### Workflow Diagram

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
