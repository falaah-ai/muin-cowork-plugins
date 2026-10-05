---
name: campaign-thermometer
description: Campaign progress brief and nudge plan for a non-profit. Use when the user asks about a specific fundraising campaign, "how's the campaign going", goal-vs-raised, the thermometer, or "what should we do to hit goal".
---

# Campaign Thermometer

Turn a campaign's goal-vs-raised numbers into a clear progress read and a
concrete nudge plan.

## What to pull (read-only)

`nonprofit_fundraising_dashboard` — returns each active campaign with
`goal_amount`, `raised_amount`, `progress_pct`, `donation_count`, and
`donor_count`, plus a tenant-wide summary.

For donor-level detail behind a campaign, follow up with
`nonprofit_list_contributions` (filter by date range) or `nonprofit_donor_summary`.

## How to read it

- **Anchor on `progress_pct`.** Group campaigns into: ahead (≥90%), on-track
  (~50–90%), behind (<50%).
- **Pace, not just position.** A campaign at 60% is great on day 3 and worrying
  on day 28 — factor in time remaining when you have it.
- **Average gift size** = `raised_amount / donation_count`. Use it to size the
  nudge ("we need ~18 more gifts at the current average to close the gap").

## The nudge plan

For a campaign that's behind, propose a short, concrete plan:
1. The exact remaining gap and how many average gifts close it.
2. Who to ask first (route to `donor-lapse-watch` for lapsing donors or major
   donors who haven't given to this campaign yet).
3. A simple message angle that fits the org's voice.

## Gotchas

- **Goal can be unset.** If `goal_amount` is 0/absent, `progress_pct` is null —
  report raised-to-date and ask the user for a goal instead of inventing one.
- **Only active campaigns** come back by default; if the user asks about a
  finished or draft campaign, say it isn't in the active set.
- **No writes.** This skill only reads and plans; it never records gifts.

## Tunable thresholds

- **Ahead / on-track / behind cutoffs:** 90% / 50% (illustrative — adjust).
- **Final-push line:** ≥90% of goal.

## Output template

```
🌡️ <Campaign name> — <pct>% of <goal> (<raised> raised, <n> gifts)
Status: <ahead | on-track | behind>
Gap: <amount>  ≈ <n> more gifts at your ~<avg> average

Nudge plan:
  1. <who to ask>
  2. <message angle>
  3. <route to donor-lapse-watch / next step>
```

## Worked example

> 🌡️ Ramadan Campaign — 92% of $50,000 ($46,000 raised, 268 gifts)
> Status: ahead — one good push from the line.
> Gap: $4,000 ≈ 23 more gifts at your ~$172 average.
>
> Nudge plan:
>   1. Ask the 14 prior-Ramadan donors who haven't given yet (I can pull them).
>   2. Angle: "We're $4k from fully funding this year's iftars — will you close
>      it with us in the last 10 nights?"
>   3. Want me to run `donor-lapse-watch` to find lapsing major donors too?
