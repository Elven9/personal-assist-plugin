# Workflow: <workflow name>

> One-line summary of the routine and when the user runs it.

**Approval:** required | auto

Set `required` (default) when any step changes state or has side effects — the
agent proposes the flow and waits for confirmation. Set `auto` only when every
task piece is read-only information gathering, so the agent can run it and just
report back without pausing.

## When to use
The phrases or situations that should trigger this workflow (e.g. "start my
day"). Note how it differs from adjacent workflows.

## Task pieces
The ordered default sequence. Each entry points at a file in
`references/tasks/`. Reorder, skip, or add pieces as the situation calls for —
this is a starting point, not a fixed script.

1. `references/tasks/<task>.md` — why it's here / what it contributes.
2. `references/tasks/<task>.md` — ...

## Flow notes
- Which steps are optional vs. essential.
- Common variations (e.g. "skip step 2 on Fridays").
- Decision points where you should pause and ask the user.

## Execution
For `required`: present the proposed sequence, confirm with the user, adjust as
needed, then run the pieces in order. For `auto`: run the pieces in order and
deliver the gathered information at the end — no confirmation step.
