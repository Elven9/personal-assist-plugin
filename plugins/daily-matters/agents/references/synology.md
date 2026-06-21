# Area: Synology (the user's work)

> Synology is the user's company. Load this file whenever an intent concerns
> their work there — feature specs, bugs, QA/RD/PM coordination, internal
> projects and releases. It defines the **sources** for this area and the
> curation taste (**surface / downrank / drop**) that override scout's generic
> defaults for intents in this domain.

## When this area applies

Signals in the intent: an explicit mention of Synology or "work"; feature specs
or requirements; bugs, regressions, QA findings; the RD / QA / PM triad;
releases or the product roadmap; the names of internal projects, channels, or
teammates. When in doubt and the intent clearly concerns the user's job, treat
it as this area.

## Sources

### ChatPlus channels

Resolve each channel name to its `channel_id` at runtime with
`glaive chat channel list` (match on `name`) — never hard-code ids, they change.

- **`APM_QA_RD`** — the cross-functional channel where RD (engineering), QA, and
  PM coordinate on **feature specs and bugs**. The go-to for what a feature is
  supposed to do, spec changes, reported bugs and their status, and the
  back-and-forth between the three roles. For spec/feature or bug intents, check
  here first.

<!-- TODO(user): more Synology sources — additional ChatPlus channels, internal
wikis/docs, dashboards, repos. One line each on what it's good for. -->

## Curation taste

How to rank and filter findings in this area. Replace the placeholders below with
the user's real preferences — the more concrete, the sharper scout's filtering.

### Surface (high signal — lead with these)

<!-- TODO(user): what the user most wants flagged in a Synology context — e.g.
spec changes to features they own, bugs assigned to or blocking them, direct
@-mentions or direct questions, release-blocking issues, decisions that change
scope. -->

_None recorded yet — see the TODO above._

### Downrank (mention briefly, don't lead)

<!-- TODO(user): lower-priority-but-not-noise — FYI status updates, threads on
features they don't own, already-resolved bugs. -->

_None recorded yet — see the TODO above._

### Drop (skip entirely)

<!-- TODO(user): pure noise — bot/automation posts, off-topic chatter,
duplicates, items already handled. -->

_None recorded yet — see the TODO above._
