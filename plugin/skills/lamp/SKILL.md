---
name: lamp
description: >
  Dynamic skill router — determines which genie-goose skill to invoke
  based on user intent. Auto-injected at session start via SessionStart hook.
disable-model-invocation: true
---

# Lamp

Dynamic skill router for genie-goose. This skill is auto-injected at session start.

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill entirely.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If the user's request matches ANY genie-goose skill, you MUST invoke it using the Skill tool.
This is not optional. Check the routing table below before every response.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.
</EXTREMELY-IMPORTANT>

## Instruction Priority

1. **User's explicit instructions** (project CLAUDE.md, AGENTS.md, direct requests) — highest priority
2. **Genie-goose skills** — override default system behavior where they conflict
3. **Default system prompt** — lowest priority

If the project's CLAUDE.md says "don't use TDD" and a skill says "always TDD," follow the user's instructions.

## How to Access Skills

Use the `Skill` tool to invoke skills by name:

```
Skill("genie-goose:rub")
Skill("genie-goose:architecture")
Skill("genie-goose:implement")
...
```

When you invoke a skill, its content is loaded and presented to you — follow it directly.

## The Routing Rule

**Check the routing table BEFORE every response.** If any skill matches the user's intent, invoke it before doing anything else — even before asking clarifying questions.

## Routing Table

| User Intent | Skill | Notes |
|-------------|-------|-------|
| Build a new feature end-to-end, full workflow | `/genie-goose:goose` | Full 9-step pipeline |
| "Let's brainstorm", "I have an idea", explore approaches | `/genie-goose:rub` | Step 1 entry point |
| Design architecture, structure, components | `/genie-goose:architecture` | Needs design.md |
| Document design intent, check convention conflicts | `/genie-goose:intent` | Needs design.md + architecture.md |
| Write implementation plan, break into tasks | `/genie-goose:write-plan` | Needs architecture.md + intent.md |
| Set up evaluation criteria, review standards | `/genie-goose:criteria` | Needs intent.md + conventions.yaml |
| Execute the plan, start implementing | `/genie-goose:implement` | Needs plan.md + intent.md |
| Code review, review changes | `/genie-goose:honk` | Needs criteria.md + intent.md + diff |
| Create PR, generate PR body, prepare for merge | `/genie-goose:pr` | Needs review-report.md + intent.md + diff |
| Finish up, done, merge after review, wrap up, clean up | `/genie-goose:finish` | Needs review-report.md |
| Update conventions, manage decisions, update ADR | `/genie-goose:update-docs` | Standalone — no pipeline prerequisites |
| Verify completion, prove it works | `/genie-goose:polish` | Any time |
| Fix a bug, debug, investigate error, something broken/failing | `/genie-goose:debug` | Standalone — no prerequisites |
| Quick one-line edit, ad-hoc non-bug task | No skill needed | Apply polish before claiming done |

## Routing Flowchart

```dot
digraph routing {
    rankdir=TB;
    node [shape=box];

    start [label="User message received" shape=doublecircle];
    is_subagent [label="Am I a subagent?" shape=diamond];
    skip [label="Skip this skill\nproceed normally" shape=box];
    full_pipeline [label="Full feature workflow?" shape=diamond];
    goose [label="Invoke /genie-goose:goose" shape=box style=filled fillcolor=lightgreen];
    specific_step [label="Matches a specific\npipeline step?" shape=diamond];
    check_prereqs [label="Prerequisites\npresent?" shape=diamond];
    invoke_skill [label="Invoke the matched skill" shape=box style=filled fillcolor=lightgreen];
    report_missing [label="Tell user which\nprior step to run" shape=box style=filled fillcolor=lightyellow];
    is_completing [label="About to claim\nwork is done?" shape=diamond];
    polish [label="Apply /genie-goose:polish" shape=box style=filled fillcolor=lightblue];
    normal [label="Respond normally" shape=doublecircle];

    start -> is_subagent;
    is_subagent -> skip [label="yes"];
    is_subagent -> full_pipeline [label="no"];
    full_pipeline -> goose [label="yes"];
    full_pipeline -> specific_step [label="no"];
    is_debugging [label="Debugging or\nfixing a bug?" shape=diamond];
    debug [label="Invoke /genie-goose:debug" shape=box style=filled fillcolor=lightsalmon];
    is_wrapping_up [label="Wrapping up after\ncode review?" shape=diamond];
    finish [label="Invoke /genie-goose:finish" shape=box style=filled fillcolor=lightgreen];

    specific_step -> check_prereqs [label="yes"];
    specific_step -> is_debugging [label="no"];
    check_prereqs -> invoke_skill [label="yes"];
    check_prereqs -> report_missing [label="no"];
    is_debugging -> debug [label="yes"];
    is_debugging -> is_wrapping_up [label="no"];
    is_wrapping_up -> finish [label="yes"];
    is_wrapping_up -> is_completing [label="no"];
    is_completing -> polish [label="yes"];
    is_completing -> normal [label="no"];
}
```

## Prerequisite Table

Before invoking a skill, check that its prerequisite artifacts exist in `.goose-artifacts/{branch}/`:

| Skill | Required Artifacts |
|-------|--------------------|
| `rub` | — (none) |
| `architecture` | `design.md` |
| `intent` | `design.md`, `architecture.md` |
| `write-plan` | `architecture.md`, `intent.md` |
| `criteria` | `intent.md` + `.goose/conventions.yaml` |
| `implement` | `plan.md`, `intent.md` |
| `honk` | `criteria.md`, `intent.md` + git diff |
| `pr` | `review-report.md`, `intent.md` + git diff |
| `update-docs` | `.goose/conventions.yaml` or `.goose/decisions.yaml` (at least one must exist) |
| `polish` | — (none) |
| `finish` | `review-report.md` |
| `debug` | — (none) |

If a prerequisite is missing, inform the user which prior step to run. Example:
> `architecture.md` not found. Run `/genie-goose:architecture` first (or `/genie-goose:goose` for the full pipeline).

## Pipeline Resumption

Users who already have some artifacts should be able to jump in at any step. Determine the pipeline position by checking which artifacts exist:

1. Check `.goose-artifacts/{branch}/` for existing artifacts
2. Identify the latest completed step based on artifacts present
3. Suggest the next logical step

Example: If design.md and architecture.md exist but intent.md does not, suggest `/genie-goose:intent` as the next step.

## Non-Pipeline Requests

Not every request needs a genie-goose skill. These can proceed normally:

- Quick bug fixes (one-file, obvious cause)
- Simple refactoring
- Questions about the codebase
- Exploratory tasks
- Configuration changes

**Note:** `/genie-goose:update-docs` is a standalone skill, not a pipeline step. Route to it when the user wants to manage conventions or decisions directly, regardless of pipeline state.

**However:** Even for non-pipeline work, apply `/genie-goose:polish` before claiming the work is done. Evidence before claims, always.

## Red Flags — STOP

These thoughts mean you are rationalizing skipping the router:

| Thought | Reality |
|---------|---------|
| "This is just a quick fix, no skill needed" | Check the routing table first |
| "I already know what to do" | Skills provide structure, not just knowledge |
| "The user didn't mention a specific step" | Infer the step from their intent |
| "I'll just edit the code directly" | Check if implement should be driving this |
| "This doesn't fit any pipeline step" | It might. Check the table. |
| "I'll use the skill next time" | Use it now. No exceptions. |
