# Workflow: Idea to tasks

> Turn a raw idea plus references into clarified requirements, break them into a
> todo list with the user, and file the result into the task system via `aleph`.
> Run this when starting a new piece of work that should end up tracked.

**Approval:** required

The final step writes state (creates tasks in `aleph`), so the agent presents the
breakdown and waits for confirmation before pushing anything.

## When to use
"Help me plan this", "let's turn this spec/issue/thread into tasks", "I need to
break this down and track it" — any time the user has work to do that isn't yet
shaped or tracked. Differs from a bare capture: this clarifies and breaks the work
down first, rather than filing loose to-dos directly.

## Task pieces
1. `references/tasks/design-requirement.md` — read the references and discuss with
   the user to produce agreed requirement items.
2. `references/tasks/task-breakdown.md` — propose and refine a todo list against
   those requirements, merging the todos back into each requirement.

The final step — filing the agreed todos into the task system — is done by the
workflow itself (see Execution), not a separate task piece.

## Flow notes
- Pieces 1 and 2 are conversational and iterative — expect back-and-forth. Don't
  advance to filing until the user is happy with the breakdown.
- Skip ahead when the input allows: if the user already has an agreed requirement,
  start at piece 2; if they hand you a finished todo list, go straight to filing.
- The hand-off is explicit: piece 2 consumes
  `task:design-requirement.requirements` and produces
  `task:task-breakdown.requirements[].todos`, which the filing step pushes.

## Execution
This is a `required` flow.

1. Run pieces 1–2 conversationally until the user is happy with the breakdown.
2. Present the plan — for each requirement, the **main task** and the **todo
   subtasks** under it, plus the project and fields each will be filed with — and
   wait for confirmation.
3. On approval, push to `aleph`. Discover projects first
   (`aleph task project list`) and reuse the result; match each requirement to the
   best-fitting project, falling back to **Inbox**. Then, **per requirement**:
   1. **Create the main task** from the requirement — `title` as the name, with an
      HTML `--description` carrying its `detail` and the confirmed `dependencies`.
      Keep the returned id.
   2. **Spawn each todo as a subtask** — one `aleph task create` per todo (carry
      `detail` and `estimate` into the description; infer
      `due`/`priority`/`difficulty`/`label` conservatively), then link it with
      `aleph task relation create <todo_id> <main_id> subtask`. Record each todo's
      new aleph id against its local `id`.
   3. **Encode the sequence** — for every `depends_on`, link the prerequisite to
      the dependent: `aleph task relation create <prereq_id> <dependent_id> blocking`
      (the dependent gains the inverse `blocked` link automatically).
   - If `aleph` reports it isn't logged in, run `aleph task login`; if that fails,
     ask the user to fix credentials rather than guessing.
4. Report the created tasks at the end — each main task with its subtasks and their
   ids.
