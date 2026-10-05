---
name: donor-lapse-watch
description: Surface at-risk and lapsing donors for a non-profit and draft re-engagement outreach. Use when the user asks about donor retention, who's lapsing or about to lapse, at-risk donors, stewardship, or "who should we reach out to".
---

# Donor Lapse Watch

Find the donors a non-profit is at risk of losing and turn that into specific,
warm re-engagement outreach.

## What to pull (read-only)

- `nonprofit_donor_summary` with **no** `donor_person_id` and a `risk_threshold`
  → returns the at-risk list (`person_id`, `risk_score`) plus the total donor
  count.
- `nonprofit_donor_summary` **with** a `donor_person_id` → that donor's full 360
  (total given, frequency, segment, recency, largest gift, recurring flag). Use
  this to personalize outreach for the top names.

## How to work it

1. Pull the at-risk list at the chosen threshold.
2. For the top few by `risk_score`, pull each donor's 360.
3. Draft outreach that references their real history — last gift, what they've
   supported, how long they've given. Warm and specific beats generic.
4. Hand the user a prioritized list + drafts they can send from their own tools.

## Gotchas

- **You get IDs, not contact details.** The tool returns `person_id` and risk —
  not email/phone. Personalize from the 360, and let the user send through their
  own channel (this plugin does not email donors).
- **Risk is a signal, not a verdict.** Frame outreach as stewardship, not "we
  noticed you stopped giving." Lead with gratitude.
- **Recurring donors at risk are urgent** — a lapsing monthly donor is a bigger
  loss than a lapsed one-time donor of the same size.
- **No writes.** This skill reads and drafts; it doesn't change donor records.

## Tunable thresholds

- **`risk_threshold`:** default `0.5`. `0.7` = only the most at-risk; `0.3` =
  catch early drift. State which you used.
- **How many to deep-dive:** top 5 by risk score is a sensible default for a
  weekly pass.

## Output template

```
At-risk donors (threshold <t>): <n> of <total>
Prioritized:
  1. <id> (risk <score>) — <segment>, last gave <recency>, lifetime <total>
     Draft: "<warm, specific re-engagement message>"
  2. …
Send these from your own email/CRM — I can refine any of them.
```

## Worked example

> At-risk donors (threshold 0.5): 7 of 312
> Prioritized:
>   1. #A12 (risk 0.81) — recurring monthly donor, no gift in 75 days, lifetime
>      $3,400. This is your most urgent: a lapsing monthly supporter.
>      Draft: "Assalaamualaikum — your steady monthly support has helped us keep
>      the food pantry stocked all year, and we noticed your last gift didn't go
>      through. Is everything okay? If you'd like to continue, here's the link —
>      and either way, thank you."
>   2. #A47 (risk 0.68) — annual Ramadan donor, hasn't given this year yet…
>
> Send these from your own email/CRM — want me to tighten the tone for any?
