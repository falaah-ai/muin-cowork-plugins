---
name: nonprofit-pulse
description: A weekly operating brief for a non-profit — cash and fund position, fundraising progress, donor stewardship signals, and the grant pipeline. Use when the user asks "how are we doing", "give me the weekly brief / month-end read", "what needs attention", or on the agreed weekly cadence.
---

# Non-Profit Pulse

The non-profit analogue of a business pulse: one read-only brief that tells a
busy executive director what's going well and what needs attention this week.

## What to pull (read-only)

1. `nonprofit_fundraising_dashboard` — total raised, donor counts, per-campaign
   goal-vs-raised.
2. `nonprofit_get_fund_balances` — restricted / designated / operating +
   endowment balances, with overspend and spend-policy flags.
3. `nonprofit_donor_summary` (no `donor_person_id`) — donor count and the
   at-risk/lapsing list.
4. `nonprofit_list_grants` — the grant-proposal pipeline. It returns `id`,
   `name`, `status` and `created_at`, and nothing else.

Optionally `nonprofit_upcoming_events` for verified public events/services.

## How to read it

- **Lead with what needs attention,** then what's going well. An ED scanning on
  a Monday wants the exceptions first.
- **Funds:** call out any `overspent: true` fund and any restricted fund whose
  balance is unusually low against its goal.
- **Donors:** surface the count of at-risk donors above the threshold and name
  the top few by risk score (route deep work to `donor-lapse-watch`).
- **Campaigns:** flag any campaign past 90% (celebrate / final push) or stalled
  well below pace.

## Gotchas

- **Never print a date beside a grant.** `nonprofit_list_grants` returns `id`,
  `name`, `status` and `created_at`, and nothing else: no deadline, no reporting
  date, no award date. Any date you put next to a grant would be invented, and an
  executive director would act on it. Flag a proposal by its STATUS and its AGE
  instead, and say plainly that Muin does not track grant dates here.
- **Empty sections are fine.** No grants or no at-risk donors → say "none this
  week", don't omit the section silently.
- **Don't double-count restricted money as spendable.** Restricted and
  endowment balances are not free operating cash; present them separately.
- **No writes.** The pulse never changes anything; it only reads.

## Tunable thresholds

- **At-risk donor cutoff:** `risk_threshold` default `0.5`. Raise to `0.7` for a
  tighter list, lower to `0.3` to catch early drift.
- **Campaign "final push" line:** 90% of goal (illustrative — adjust to taste).
- **Low-restricted-fund flag:** balance < 10% of the fund's goal (judgment call;
  state your reasoning).

## Output template

```
🔎 Needs attention
  • Funds: <overspent / low restricted funds, or "all within policy">
  • Donors: <n at-risk above threshold; top: …, or "none">
  • Grants: <proposals stalled in one status, or "none this week">

✅ Going well
  • Raised: <amount> across <n> donors
  • Campaigns: <name> at <pct>% of goal …

Suggested next step: <one skill, e.g. donor-lapse-watch>
```

## Worked example

> 🔎 Needs attention
>   • Funds: Building fund overspent by $1,240 — restricted, needs a transfer or
>     a board note.
>   • Donors: 7 donors above 0.5 lapse risk; highest are #A12, #A47, #A93.
>   • Grants: "Youth Program Expansion" has sat in progress since February.
>
> ✅ Going well
>   • Raised: $48,200 across 312 donors this period.
>   • Ramadan Campaign at 92% of its $50k goal — one good push from the line.
>
> Suggested next step: run `donor-lapse-watch` on those 7 donors.
