---
name: fund-compliance
description: Restricted-fund balances and spend-policy compliance for a non-profit. Use when the user asks about funds, restricted vs operating money, endowment balances, "can we spend this", overspend, or fund accounting / compliance.
---

# Fund Compliance

Show where a non-profit's money sits by restriction class and flag anything that
breaks spend policy — the core of fund accounting that generic finance tools
miss.

## What to pull (read-only)

`nonprofit_get_fund_balances` — returns each fund with its
`restriction_class` (`operating` / `restricted` / `endowment`), `balance`,
`has_spend_policy`, `overspent`, plus endowment detail (principal, accumulated
earnings, spending rate) and `balances_by_class` totals.

Pass `include_inactive: true` only if the user explicitly wants closed funds.

## How to read it

- **Separate the buckets.** Lead with `balances_by_class`: operating (spendable),
  restricted (donor-restricted — not free cash), endowment (principal protected;
  only earnings/spending-rate is spendable).
- **Flag overspend first.** Any fund with `overspent: true` (negative balance) is
  the headline — a restricted fund in the red is a compliance problem, not just
  a budgeting one.
- **Spend policy.** For funds with `has_spend_policy: true`, note that spending is
  constrained; recommend checking proposed spend against the policy before it
  happens.
- **Endowments.** Never present endowment principal as spendable. Surface the
  spending rate and accumulated earnings as the spendable portion.

## Gotchas

- **Restricted ≠ available.** The single biggest mistake is treating restricted
  or endowment balances as operating cash. Always keep them distinct.
- **Over-policy ≠ overspent.** A fund can be positive but still spending against
  a category it shouldn't — `has_spend_policy` is your cue to look closer; the
  tool gives the policy *existence*, the judgment is yours.
- **No writes.** This skill reads and advises; it doesn't move money.

## Tunable thresholds

- **Low-balance warning:** restricted fund balance < 10% of its `goal_amount`
  (illustrative — state your reasoning when you flag).

## Output template

```
💰 Fund position
  Operating (spendable): <amount>
  Restricted (donor-restricted): <amount>
  Endowment (principal protected): <amount>

⚠️ Flags
  • <fund>: overspent by <amount> (restricted) — needs attention
  • <fund>: has a spend policy — verify proposed spend before it posts
  • (or "no compliance flags")
```

## Worked example

> 💰 Fund position
>   Operating (spendable): $22,400
>   Restricted (donor-restricted): $61,900 across 4 funds
>   Endowment (principal protected): $250,000 (spendable this year ≈ $10,000 at
>   a 4% rate)
>
> ⚠️ Flags
>   • Building fund: overspent by $1,240 (restricted) — this is donor-restricted
>     money in the red; you'll want a transfer or a documented board decision.
>   • Scholarship fund: spend policy in place — check any disbursement against it
>     first.
