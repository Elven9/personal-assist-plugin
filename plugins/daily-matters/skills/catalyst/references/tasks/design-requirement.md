# Task: Design requirement

> Read the provided references and the user's verbal ask, discuss to clarify, and
> produce a refined, agreed-upon requirement ready to be turned into work.

## When to use
At the start of any piece of work whose goal isn't yet crisp. The user has an idea
plus some raw material — a spec, a chunk of code, a Work+ issue, a chat thread, an
email — and wants to shape it into a clear, actionable requirement before anything
gets filed or built. This is the upstream piece that feeds task-creation: it
decides *what* needs doing, not *how* to track it.

## Inputs
| name | type | required | source | description |
| --- | --- | --- | --- | --- |
| verbal_requirement | string | yes | user | What the user wants, in their own words. |
| references | Reference[] | no | user | Solid source material to ground the requirement. Each is `{ kind, locator, content? }` where `kind` is one of `file` · `code` · `spec` · `issue` · `chat` · `email`. Strengthens the result but not strictly required. |

## Output
This task is conversational: it surfaces suggestions and refinements during the
discussion, and on agreement emits the settled requirement(s) as a list.

| field | type | description |
| --- | --- | --- |
| status | "success" \| "partial" \| "failed" | `success` when the user agrees the requirements are ready; `partial` when enough is agreed to proceed but open questions remain; `failed` when no requirement could be formed. |
| summary | string | One-line recap of what was agreed. |
| requirements | Requirement[] | The refined requirements (item shape below). |

Each `Requirement` is `{ title, brief, detail, references, dependencies }`:

- **title** — short name of the requirement.
- **brief** — one-line description of what it is.
- **detail** — the fuller, agreed description (HTML or prose): intent, context, scope, acceptance.
- **references** — which input references back this item (the relevant subset of the inputs).
- **dependencies** — the external dependencies the user explicitly confirmed (see
  Guidelines). Each is `{ kind, description }` where `kind` is one of
  `engineering` · `human` · `assumption` · `prior-task`. Record **only**
  user-confirmed entries; leave empty if there are none.

```json
{
  "status": "success",
  "summary": "Agreed 1 requirement: add JWT refresh to the auth service.",
  "requirements": [
    {
      "title": "JWT refresh for auth service",
      "brief": "Keep sessions alive without forcing re-login when access tokens expire.",
      "detail": "Access tokens expire at 24h (auth/config.go); add a refresh endpoint and client retry on 401. Out of scope: SSO migration. Done when an expired access token is transparently refreshed and the refresh token rotates on use.",
      "references": [{ "kind": "code", "locator": "auth/config.go" }],
      "dependencies": [
        { "kind": "engineering", "description": "auth service must expose a token-introspection endpoint" },
        { "kind": "human", "description": "security team signs off on refresh-token lifetime" }
      ]
    }
  ]
}
```

The next task chains off this by naming `task:design-requirement.requirements`
(it breaks each requirement into a todo list).

## Guidelines
- **Lead with architecture, not passive capture.** As you discuss, propose how the
  work could be structured — the shape of a solution, where it fits existing
  systems, sensible boundaries and trade-offs. A concrete suggestion the user can
  react to sharpens the requirement far faster than open-ended questioning. You're
  a thinking partner here, not a stenographer.
- **Draw out external dependencies for every requirement.** Most real work depends
  on things outside itself, and a missed dependency is what derails delivery
  later. Cover both kinds:
  - **`engineering`** — a service, API, library, dataset, or other code the work
    relies on.
  - **non-engineering relationships** — another **`human`** (an approval, an input,
    a handoff), an **`assumption`** that must hold for the work to make sense, or a
    **`prior-task`** result this one builds on.
- **Ask explicitly; infer only to prompt, never to record.** Walk requirements
  one at a time — dependencies differ per requirement, so a single blanket question
  won't surface them. You *may* propose likely candidates to jog the user's memory
  ("this probably needs the auth service and sign-off from security — right?"), but
  treat those as questions, not findings.
- **Never write down a dependency from your own imagination.** Only record what the
  user confirms. An unconfirmed guess captured as fact propagates into the
  breakdown and the filed tasks and quietly misleads everything downstream. If the
  user can't confirm a candidate, drop it or carry it as an open question instead.

## Notes
- The same discipline applies to scope and acceptance, not just dependencies:
  suggest freely, record only what's agreed.
- When a candidate dependency stays unresolved, prefer reflecting `status:
  partial` with the unknown noted over inventing an answer to reach `success`.
- If references contradict the user's verbal ask, surface the conflict and let the
  user reconcile it — don't silently pick one.
