---
name: goose
description: >
  Legacy alias for the rub-led full development workflow.
  Keep accepting goose for backward compatibility, but
  route users to rub as the canonical entrypoint.
disable-model-invocation: true
---

# Goose

`goose` is retained for backward compatibility.
The canonical full-flow entrypoint is now `rub`.

If the user invokes `goose`, do not maintain a separate execution model.
Instead, inform them briefly that `rub` is the preferred entrypoint for the full workflow, then continue using the same workflow behavior defined by `rub`.

## Procedure

1. Acknowledge that `goose` is a legacy alias.
2. State that `rub` is the canonical brainstorming-first full workflow entrypoint.
3. Execute the request using the exact same rules and flow defined in the `rub` skill.

## Notes

- Do not let `goose` drift from `rub`.
- Any future full-flow behavior changes should be made in `rub`, not duplicated here.
- If the user explicitly wants brainstorming only, honor `rub`'s brief-only mode.
- Any future autogoose behavior changes should also be made in `rub`, not separately here.
