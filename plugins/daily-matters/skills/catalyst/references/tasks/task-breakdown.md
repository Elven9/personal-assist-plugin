# Task: Task breakdown

> Propose and discuss a todo list against an agreed requirement, then merge the
> breakdown back into that requirement and hand it off.

## When to use
After a requirement is agreed (see `design-requirement`) and before anything is
filed. This piece decides *how* the work splits into trackable todos — the
actionable units a downstream piece will push into the task system. It's
conversational: it suggests a breakdown, the user refines it, and the result is
the requirement plus its todos.

## Inputs
| name | type | required | source | description |
| --- | --- | --- | --- | --- |
| requirements | Requirement[] | yes | task:design-requirement.requirements | The agreed requirements to break down. Each is `{ title, brief, detail, references, dependencies }` — the `dependencies` drive how todos are sequenced (see Guidelines). |

## Output
On agreement, emit the upstream requirements with a todo list attached to each, so
nothing about the requirement is lost on the way to filing.

| field | type | description |
| --- | --- | --- |
| status | "success" \| "partial" \| "failed" | `success` when the user agrees the breakdown is ready; `partial` when agreed enough to proceed but todos remain open; `failed` when no breakdown could be formed. |
| summary | string | One-line recap, e.g. "Broke 1 requirement into 4 todos." |
| requirements | Requirement[] | The upstream requirements, each augmented with `todos: Todo[]`. |

Each `Todo` is `{ id, title, detail, depends_on, estimate }`:

- **id** — a local handle (e.g. `t1`) so other todos and the sequence can reference it.
- **title** — a short, imperative action ("Add refresh endpoint").
- **detail** — optional; what the todo entails when the title isn't enough.
- **depends_on** — ids of the todos that must finish first. This encodes the
  sequence; empty means it can start immediately.
- **estimate** — rough required time, user-confirmed (keep the unit consistent
  within a breakdown, e.g. `2h`, `0.5d`).

```json
{
  "status": "success",
  "summary": "Broke 1 requirement into 3 todos (~1.5d total).",
  "requirements": [
    {
      "title": "JWT refresh for auth service",
      "brief": "Keep sessions alive without forcing re-login when access tokens expire.",
      "detail": "…",
      "references": [],
      "dependencies": [
        { "kind": "human", "description": "security signs off on refresh-token lifetime" }
      ],
      "todos": [
        { "id": "t1", "title": "Add refresh endpoint", "detail": "issue + validate refresh tokens", "depends_on": [], "estimate": "0.5d" },
        { "id": "t2", "title": "Rotate refresh token on use", "depends_on": ["t1"], "estimate": "0.5d" },
        { "id": "t3", "title": "Client retry on 401", "depends_on": ["t1"], "estimate": "0.5d" }
      ]
    }
  ]
}
```

A downstream piece chains off this by naming
`task:task-breakdown.requirements[].todos` to file each todo (using `depends_on`
to order them and `estimate` to gauge effort).

## Guidelines
The primary goal is three things: **clear actionable items, their sequence, and
the time each is likely to need.** Everything below serves that.

- **Make every todo actionable.** Each one should be a single concrete action
  someone can pick up and start — a verb on an object ("Add refresh endpoint"),
  not a vague theme ("auth work"). If a todo can't begin until something is
  decided, that decision is its own todo. Right-size them: small enough to
  estimate and order, big enough to be worth tracking on their own.
- **Sequence using the requirement's dependencies.** Revealing order is the whole
  point of breaking down. Use the upstream `dependencies` and the natural flow
  between todos to set `depends_on`: a `human` or `prior-task` dependency gates
  whatever needs it; `engineering` dependencies usually dictate which todo must
  land first; an `assumption` that's still unconfirmed blocks the work that rests
  on it. Make blocking explicit rather than leaving order implied.
- **Estimate the time each item needs.** Give every todo a rough required time so
  the user can see the shape of the effort and where the weight sits. Propose an
  estimate, but defer to the user's correction — they know their own pace and
  context. Keep the unit consistent across the breakdown.
- **Propose, then confirm — don't invent.** Same discipline as design-requirement:
  suggest a breakdown, an ordering, and estimates for the user to react to, but the
  recorded result is only what they agree to. Never fabricate todos, dependencies,
  sequence, or effort the user hasn't validated.
- **Preserve the requirement.** Carry each requirement's fields through untouched
  and attach its todos, so the filing step keeps the full "why" behind each task.

## Notes
- If the user can't yet commit to part of the breakdown (an unconfirmed estimate,
  an unresolved blocking dependency), reflect `status: partial` and leave the item
  sequenced-but-flagged rather than guessing to reach `success`.
- Keep `depends_on` pointing at real todo `id`s within the same requirement; a
  dangling reference will confuse the filing step.
- Estimates are rough planning aids, not commitments — don't over-precision them.
