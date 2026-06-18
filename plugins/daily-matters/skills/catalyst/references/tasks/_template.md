# Task: <task name>

> One-line summary of what this atomic task accomplishes.

## When to use
What signals indicate this piece belongs in a flow, and which workflows commonly
include it.

## Inputs
A typed contract of what this task consumes. To use this task, the upstream task
or the user must supply these. `source` says where each value comes from:

- `user` — from the user's message or confirmation.
- `aleph` — discovered from the task backend at runtime (e.g. a project list).
- `task:<id>.<field>` — an upstream task's output field. This is how pieces chain
  inside a workflow: a downstream input names the exact output it depends on.

Use a single "none" row if the task is self-contained.

| name | type | required | source | description |
| --- | --- | --- | --- | --- |
| <field> | <type> | yes/no | user | <what it is> |

## Output
On completion, emit a single **JSON object** so the next piece can consume it
without re-parsing prose. Every output carries `status` (so a workflow can branch
or stop) and a human-facing `summary`; named artifacts hold the typed handoff that
downstream tasks reference. Define the shape here:

| field | type | description |
| --- | --- | --- |
| status | "success" \| "partial" \| "failed" | whether the task completed |
| summary | string | one-line recap for the user |
| <artifact> | <type> | the typed result a downstream task can name as its input |

Then give a concrete example so the runtime shape is unambiguous:

```json
{ "status": "success", "summary": "...", "<artifact>": [] }
```

## Guidelines
How to do this task *well* — principles, conventions, and the tools/commands to
reach for. Deliberately **not** an ordered procedure: sequencing (and any
approval gating) is the workflow's job, so the same task can slot into different
flows. Capture the judgment calls — how to infer fields, what to default, what to
never guess — so each run produces a consistent, high-quality result.

## Notes
Edge cases, common variations, failure modes, and things to ask the user about.
