---
name: grant-match
description: Grant pipeline status and age triage for a non-profit. Use when the user asks about grants, grant proposals, applications in progress, or where a proposal has reached.
---

# Grant Match

Give a non-profit a clear read on where each of its grant proposals stands.

## What to pull (read-only)

`nonprofit_list_grants` — returns the grant-proposal pipeline (`id`, `name`,
`status`, `created_at`) up to `limit` (default 20, max 50).

## How to read it

- **Group by status.** Cluster proposals into in-progress / submitted / awarded /
  declined (whatever statuses come back) so the user sees the funnel at a glance.
- **Triage by age.** Use `created_at` to spot proposals that have sat in one
  status too long (a draft that's been "in progress" for months needs a decision).
- **Surface what needs action now** — proposals to finish and submissions
  awaiting a response to follow up on.

## Gotchas

- **Pipeline ≠ matching engine.** This tool lists the org's own proposals; it
  does not search external open-grant databases. If the user wants *new* grant
  opportunities, say that's outside what Muin exposes here and offer to triage
  the existing pipeline instead — don't invent matches.
- **No dates except `created_at`.** The tool returns no deadline, no reporting
  date and no award date. Never print one: a date invented beside a grant is a
  date the user will act on. Say Muin does not track grant dates here and point
  them at the award letter or funder portal.
- **Limit is capped at 50.** For a large pipeline, summarize by status rather
  than listing every row.
- **No writes.** This skill reads and triages; it doesn't create or edit
  proposals.

## Tunable thresholds

- **"Stalled" age:** a proposal in the same non-terminal status for > 60 days is
  worth flagging (illustrative — adjust to the org's grant cadence).

## Output template

```
📋 Grant pipeline (<n> proposals)
  In progress: <n>  •  Submitted: <n>  •  Awarded: <n>  •  Declined: <n>

Needs action:
  • <name> — <status> since <created_at> (<stalled?>)
  • …
```

## Worked example

> 📋 Grant pipeline (6 proposals)
>   In progress: 2 • Submitted: 2 • Awarded: 1 • Declined: 1
>
> Needs action:
>   • "Youth Program Expansion" — in progress since Feb (≈4 months). This has
>     stalled; it needs a decision to finish or shelve it.
>   • "Community Reinvestment Fund" — awarded, and the only award this year.
>     Muin does not track its reporting dates, so check the award letter for what
>     the funder expects and when.
