---
name: catalyst
description: >-
  Runs the user's personal daily workflows by composing reusable task pieces
  into a flow, getting approval, then executing it. Use this whenever the user
  wants to run, start, or set up a daily routine — e.g. "start my day", "do my
  morning routine", "wrap up", "run my standup flow", or any mention of their
  daily workflow, checklist, or recurring routine. Also use it when the user
  describes a multi-step recurring task that resembles one of the documented
  workflows, even if they don't name it explicitly. Prefer this skill over
  improvising an ad-hoc sequence, so the user's established task pieces and
  ordering are reused.
---

# Catalyst

Catalyst turns the user's daily routines into something repeatable. The core
idea is small and composable:

- **Tasks** are the atomic pieces — a single, self-contained unit of work
  (e.g. "triage inbox", "review open PRs"). Each task's detail lives in its own
  file under `references/tasks/`.
- **Workflows** are ordered combinations of tasks that the user runs together
  (e.g. a morning kickoff). Each workflow's detail lives under
  `references/workflows/`.

The same task piece can appear in many workflows. That's the point: define a
task once, reuse it everywhere.

## Task contracts

Tasks compose, so each one declares a typed **Inputs** and **Output** contract —
the output of one piece becomes the input of the next inside a workflow. Honoring
these contracts is what lets pieces chain reliably instead of the agent
re-improvising the handoff every run.

- **Inputs** are a typed table. Each input's `source` is `user` (from the user),
  `aleph` (discovered at runtime), or `task:<id>.<field>` (an upstream task's
  output field — this is the chaining mechanism).
- **Output** is a single **JSON object** emitted when the task finishes. Every
  output carries two fields by convention:
  - `status` — `"success"`, `"partial"`, or `"failed"`, so a workflow can branch
    or stop on a failed upstream step.
  - `summary` — a one-line, human-facing recap (the line the user actually reads).
  - …plus any **named, typed artifacts** that downstream tasks reference by name
    (e.g. `requirements`).

A downstream task names exactly what it depends on, e.g.
`task:design-requirement.requirements`. See `references/tasks/_template.md` for the
shape and `references/tasks/design-requirement.md` for a worked example.

## Approval modes

Each workflow declares an `Approval` mode in its file, which controls whether
you pause for a go-ahead before executing:

- **`required` (default):** Propose the flow and wait for the user to confirm
  before running anything. Use this whenever the workflow changes state, sends
  messages, commits, or otherwise has side effects the user should steer.
- **`auto`:** Run without waiting for approval, then report back. This is for
  read-only, information-gathering routines — checking calendar, reading inbox,
  summarizing PR status — where the user just wants the result and a confirm
  step is pure friction.

`auto` is a privilege limited to workflows whose task pieces only gather
information. If an `auto` workflow would do anything with side effects (send,
write, commit, delete, post), stop and switch to the `required` loop instead —
treat the side-effecting step as needing approval even if the workflow is
marked `auto`. When in doubt, ask.

## How to run a workflow

Catalyst is flexible by design — you select the flow, then either propose it for
approval or run it directly depending on the workflow's approval mode.

1. **Identify intent.** Figure out which workflow the user wants. If they named
   one, use it. If they described a routine, match it to the closest documented
   workflow, or compose a new flow from existing task pieces.
2. **Load only what you need.** Read the relevant workflow file from
   `references/workflows/`, then read the task files it references from
   `references/tasks/`. Don't load everything — pull files on demand to keep
   context lean.
3. **Check the approval mode.** Read the workflow's `Approval` field.
   - If **`required`** (or you composed a new flow, or any piece has side
     effects): **propose the flow** — present the ordered list of task pieces,
     briefly noting what each does. Flag any adapting or recombining (skipping,
     reordering, swapping in a piece from another workflow) and why. Then **get
     approval** — wait for the user to confirm or adjust. A quick confirm is
     cheaper than redoing work.
   - If **`auto`** and every piece is read-only: skip straight to execution. A
     one-line "running your <workflow> flow" is enough; no need to wait.
4. **Execute in order.** Run the task pieces one at a time. Follow each task
   file's own guidelines and honor its Inputs/Output contract. Report progress as
   you go and surface anything that needs a decision. For `auto` runs, deliver the
   gathered information at the end.

Newly composed flows always use the `required` loop — only a workflow file that
explicitly declares `Approval: auto` may skip approval.

## Composing new flows

Workflows aren't rigid. If the user's request doesn't match a documented
workflow exactly, build a flow from the available task pieces in
`references/tasks/`. Pick the pieces that fit, order them sensibly, and run the
same propose → approve → execute loop. When a useful new combination emerges,
offer to save it as a new workflow file so it's reusable next time.

## Scripts

Deterministic, repetitive, or token-heavy task pieces live in `scripts/` rather
than being reimplemented by the agent each run. The main entry point is
`synoday.py` — a stdlib-only CLI that emits JSON on stdout, so you can parse its
output without reading the script into context.

Invoke it via the plugin root so the path resolves wherever the plugin is
installed:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/catalyst/scripts/synoday.py" <command>
```

Run `synoday.py --help` to list available commands. Each command maps to a task
piece; a task file in `references/tasks/` that's backed by a script should point
at the relevant command. If a command exits non-zero it returns
`{"ok": false, "error": ...}` — surface the error to the user rather than
guessing.

## Catalog

The catalog below is the index of what's available. Keep it in sync as tasks and
workflows are added.

### Tasks (`references/tasks/`)

| Task | File | What it does |
| --- | --- | --- |
| Design requirement | `references/tasks/design-requirement.md` | Read references and the user's ask, discuss to clarify, and produce agreed requirement items. |
| Task breakdown | `references/tasks/task-breakdown.md` | Break agreed requirements into clear actionable todos — sequenced by their dependencies and time-estimated — and merge them back in. |

### Workflows (`references/workflows/`)

| Workflow | File | Approval | Composed of |
| --- | --- | --- | --- |
| Idea to tasks | `references/workflows/idea-to-tasks.md` | required | design-requirement → task-breakdown → push to `aleph` |

## Adding tasks and workflows

- New task: copy `references/tasks/_template.md`, fill it in, add a row to the
  Tasks table above.
- New workflow: copy `references/workflows/_template.md`, list its task pieces,
  add a row to the Workflows table above.
