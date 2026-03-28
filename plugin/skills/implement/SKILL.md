---
name: implement
description: >
  Implement based on architecture.md and intent.md.
  Main agent writes code directly.
disable-model-invocation: true
---

# Implement

Implement the feature based on the architecture and design intent documents.

## Prerequisites

- `.goose-artifacts/{branch}/architecture.md` must exist.
- `.goose-artifacts/{branch}/intent.md` must exist.

## Procedure

1. **Read** architecture.md and intent.md to understand what to build and why.

2. **Plan implementation** by breaking the work into discrete tasks. Consider:
   - Which files to create or modify
   - Order of implementation (dependencies first)
   - Build/test verification after each logical unit

3. **Implement** each task. After completing each logical unit:
   - Verify the build succeeds
   - Run relevant tests
   - Commit the changes

4. **Report completion** with a summary of what was implemented.

## Rules

- Follow the architecture document. Do not deviate from agreed-upon design decisions.
- Respect intentional exclusions from the design intent document.
- Write tests alongside implementation when the project has a test framework.
- Make frequent, small commits with descriptive messages.
