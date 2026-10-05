---
name: 990-prep
description: IRS Form 990 / 990-PF readiness snapshot and accountant hand-off for a US non-profit. Use when the user asks about the 990, IRS filing, tax-exempt annual return, readiness for the 990, or preparing for their accountant.
---

# 990 Prep

Give a US non-profit a clear, section-by-section read on how ready it is to file
its IRS Form 990 / 990-PF — and a clean hand-off for the accountant who actually
files it.

## What to pull (read-only)

`nonprofit_990_summary` with an optional `fiscal_year` (defaults to the most
recent complete year). It returns a deterministic readiness snapshot from Muin's
records — section completeness, missing items, and warnings.

> This is a **readiness snapshot**, not a filed return, and not tax advice. It
> uses no AI/LLM call — it's a straight read of what Muin has on file.

## How to read it

- **Lead with the gating items** — the missing pieces that would block or delay a
  filing — then the warnings, then what's already in good shape.
- **Translate to a to-do list** the ED can act on: "you still need X and Y before
  this goes to your accountant."
- **Frame the hand-off.** Summarize what's ready so the accountant starts from a
  known state rather than from zero.

## Gotchas

- **US-only.** For a non-US org the tool returns `available: false` with a
  reason. Don't force a 990 read on a non-US tenant — explain it doesn't apply
  and move on.
- **Readiness ≠ filed, and ≠ advice.** Be explicit: this helps them *prepare*; a
  qualified preparer still files. Never imply the 990 has been submitted.
- **Right fiscal year.** If the user cares about a specific year, pass
  `fiscal_year` — don't assume the default is the one they mean.
- **No writes.** This skill only reads.

## Tunable thresholds

- **None numeric** — readiness is driven by Muin's section completeness, not a
  cutoff you set.

## Output template

```
🧾 Form 990 readiness — FY<year>  (US)
Blocking / missing:
  • <item>
Warnings:
  • <item>
Ready:
  • <section/area in good shape>

Hand-off: <one-paragraph summary your accountant can start from>
```

## Worked example

> 🧾 Form 990 readiness — FY2025 (US)
> Blocking / missing:
>   • No board roster on file — Part VII (officers/directors) can't be completed.
>   • Program-service accomplishments not recorded for 2 of 3 programs.
> Warnings:
>   • Total contributions reconcile to the ledger, but one large in-kind gift
>     lacks a fair-market-value note.
> Ready:
>   • Revenue and functional-expense totals are complete and reconcile.
>
> Hand-off: "FY2025 revenue and expenses are reconciled and ready. Before filing,
> we still need the board roster and two program accomplishment write-ups, and a
> FMV note on one in-kind gift." This is a readiness check, not tax advice — your
> preparer files the return.
